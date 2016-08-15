---
title: Facebook Android SDK Tips Part 1 LikeView Bugs and Fixes
date: 2016-08-15 19:59:17
tags: [Android, Facebook]
---

Solutions for some bugs I met when using Facebook LikeView. The docs of Facebook SDK is totally #@!$#@!$!

Official Reference: [LikeView Reference](https://developers.facebook.com/docs/reference/android/current/class/LikeView)

<!-- more -->
##BUG 0x01: The developers or testers cannot access the Like Page

1. Check the internet connection.
2. Check if the page is allowed to be liked.
3. Check the account of developers or testers have the permission of developer or tester under the APP ID.
4. Check Facebook App support before initialize LikeView, and DO NOT just check the installation of Facebook App. For some lower version of Facebook App, things may not work properly. Using SDK for checking as following:

~~~java  
    if (Build.VERSION.SDK_INT >= 15) {
        if (LikeDialog.canShowNativeDialog() || LikeDialog.canShowWebFallback()) {
        }
    }
~~~

## BUG 0x02: Crash when initializing
1. DO NOT initialize LikeView in your layout.xml as the Official guide recommended, which is unsafe for API level < 15. ( min version for FB SDK 4.10 ) All of the FB SDK method should be used after API level judgement.
2. If you only want to open LikeDialog in Facebook App and only use `LikeDialog.canShowNativeDialog()` for judgement, LikeDialog may fail to open after swipe all the data of Facebook App. I recommend to use web dialog just in case.

## BUG 0x03: Crash in onActivityResult() 
Stack overflow give a way to get the like result in onActivityResult() method. But it should add a `data != null` judgment, otherwise app will crash when not data returns.

## BUG 0x04: Native Like Button permission

App Review should be done to get Native Like Button permission. Otherwise, only developers and test users can access like button.

(Moved from [another blog site of my](http://huxyz.blogspot.com/2016/03/facebook-android-sdk-tips-part-1.html))