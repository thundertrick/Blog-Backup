---
title: Android RemoteView 资源ID索引错误解决方案
date: 2016-10-21 14:42:39
tags: [Android,bugs]
---

应用升级时如果替换了通知等 RemoteView 中的资源文件，则可能会导致新的升级包资源 id 发生变动，在部分机型上体现为`android.app.RemoteServiceException`。崩溃日志大致如下：

~~~
Bad notification posted from package com.mypkg.name: Couldn't create icon: StatusBarIcon(pkg=com.mypkg.name user=0 id=0x7f0200dc level=0 visible=true num=0 )
android.app.ActivityThread$H.handleMessage(ActivityThread.java:1638) 
android.os.Handler.dispatchMessage(Handler.java:111) 
android.os.Looper.loop(Looper.java:194) 
android.app.ActivityThread.main(ActivityThread.java:5637) 
java.lang.reflect.Method.invoke(Native Method) 
java.lang.reflect.Method.invoke(Method.java:372) 
com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:960) 
com.android.internal.os.ZygoteInit.main(ZygoteInit.java:755)
~~~

前段时间升级直接导致 crash 收集平台被该 bug 刷屏，而测试过程又一次都没有复现。[StackOverFlow](http://stackoverflow.com/questions/31771204/bad-notification-posted-from-package)  上也没有太好的解决方案。这里探索了一种固化资源 id 的解决方案，线上验证也是可行的，bug率明显下降。但是另一种类似的`cannot expand view`还未能解决。

<!-- more -->
通过反编译可以发现，Android App 会在 `res/values/public.xml` 里存储所有的资源文件名到资源文件 id 的映射关系。如果在编译前手动添加部分资源的 id 到 public.xml 下，那么这些资源的 id 就得以固化，升级以及添加修改资源都不会对这些 id 产生影响。此外，`res/values/ids.xml` 也起到了类似作用，只不过仅覆盖 id 资源。

因此，解决思路应该是反编译 Apk 包，获得最新的`public.xml`，讲所有`RemoteView`涉及到的资源文件，包括`string, dimen, drawable, id, layout`等，提取出来并保存在项目源码中的`res/values/public.xml`和`res/values/ids.xml`文件中。

***注意：Gradle 自 1.3.0 以后版本编译时会默认忽略本地的 `public.xml` 文件，因此还需要添加 Gradle 脚本使 `public.xml` 生效。打开 app 的 `build.gradle` 文件，在最后添加如下脚本 ([参考连接](http://blog.csdn.net/ceabie/article/details/50867791)中也提到了利用 Gradle 插件的解决方案)：***

~~~java
	afterEvaluate {
	    for (variant in android.applicationVariants) {
	        def scope = variant.getVariantData().getScope()
	        String mergeTaskName = scope.getMergeResourcesTask().name
	        def mergeTask = tasks.getByName(mergeTaskName)
	
	        mergeTask.doLast {
	            copy {
	                int i=0
	                from(['res']) {
	                    include 'values/public.xml'
	                    rename 'public.xml', (i == 0? "public.xml": "public_${i}.xml")
	                    i++
	                }
	
	                into(mergeTask.outputDir)
	            }
	        }
	    }
	}
~~~

--- Update
如果需要对`submodule`中的资源也增加id固化，则可以在上面代码`['res']`中增加相应资源路径即可。

参考：
1. [android gradle plugin 1.3.0 以上使用 public.xml 固定 id](http://blog.csdn.net/ceabie/article/details/50867791)
