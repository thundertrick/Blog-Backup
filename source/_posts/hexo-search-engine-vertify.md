---
title: HEXO next 主题增加搜索引擎验证
date: 2016-08-16 12:20:19
tags: [Hexo,Baidu,Google]
---

Github 上利用 hexo 建立的博客是无法被搜索引擎搜索到的。Github 本身也不会将信息提交给引擎。所以，为了让博客信息被检索到，我们需要手动将博客网站提交给搜索引擎并验证，实际上就是验证网站是我们自己的。对 hexo 比较友好的验证方法包括：

1. 文件验证：将一个 html 文件配置到 source 目录下，使之可以访问并获取其中的字符串。
2. HTML meta 标签验证：在主页的 html 头里增加一个 meta 标签，记录搜索引擎提供的字符串。

如果使用了 Next 主题，source 目录下所有的 html 都会按主题模板化，这使得文件验证生成的页面中字符串无法被搜索引擎识别。虽然理论上可以对单独的地址进行模板屏蔽，但一个网站冷不丁冒出来一个风格迥异的页面还是不能让人接受。所以，我最后选择了 HTML meta 标签验证方法。

<!-- more -->

[Google 引擎验证申请地址](https://www.google.com/webmasters/tools/home?hl=zh-CN)
[Baidu 引擎验证申请地址](http://www.baidu.com/search/url_submit.htm)

Next 主题下，页面的 header 设置在`themes/next/layout/_partials/head.swig`内。默认也给我们提供了模板，感谢[开发者 iissnan](https://github.com/iissnan)。我们只需要把下面标签补全即可：

~~~html
{% if theme.google_site_verification %}
  <meta name="google-site-verification" content="IQs30MFrhmrm-Nfmv-_neEbXELyUuxvZtdin1ALJ4aA" />
{% endif %}

{% if theme.baidu_site_verification %}
  <meta name="baidu-site-verification" content="H1J5y75sN7" />
{% endif %}
~~~

同时，在`themes/next/_config.yml`中将`google-site-verification`和`baidu_site_verification`的值设置为`true`即可。

P.S.
百度现在不支持 https 网址的验证，ORZ。

Referrence:

1. [hexo干货系列：（六）hexo提交搜索引擎（百度+谷歌）](http://www.jianshu.com/p/619dab2d3c08)

