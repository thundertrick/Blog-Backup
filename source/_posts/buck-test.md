---
title: Buck 安装配置与测试
date: 2016-08-12 19:40:32
tags: [Android,Buck,Facebook]
---
> Buck is a build system developed and used by Facebook. It encourages the creation of small, reusable modules consisting of code and resources, and supports a variety of languages on many platforms.

Facebook 开源的东东，据说微信也在用。接手的项目以前是 Ant，现在转 Gradle。但由于项目本身携带了大量历史遗留代码，并且集合了数个 submodule，编译起来非常缓慢，拖延了开发效率。这里测试一下 Buck 替代的可能性。

<!-- more -->
## 0x01 安装
OS X 下使用 homebrew [安装](https://buckbuild.com/setup/install.html)：

~~~shell
brew update
brew tap facebook/fb
brew install buck
brew install watchman
~~~

Watchman 是 FB 强力推荐安装的，可以有效提高 Buck 的编译效率。此外，对于 Android 开发，我们还必须把 Android SDK 和 NDK 的路径告诉 Buck。设置方法只需配置好ANDROID_HOME，ANDROID_NDK，ANDROID_NDK_REPOSITORY 这几个环境变量即可，本文暂不累述。

## 0x02 创建本地配置
... // TODO