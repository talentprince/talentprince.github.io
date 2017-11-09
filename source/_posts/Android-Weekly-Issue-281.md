---
title: 'Android Weekly Note #282'
date: 2017-11-03 09:36:41
tags: [Kotlin Delegate,Google Actions,Dialog Flow,Android 8.1]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #281
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1510191605/281_ce3k6m.jpg
---

October 29th, 2017

## [Android Weekly Issue #281](http://androidweekly.net/issues/issue-281)

本期内容不多,包含了小众DI库`牙签`帮助测试的文章,Kotlin中Delegate的强大之介绍,以及基于Google Actions与Dialog Flow的AI应用,Android 8.1 API的改动与增强.
第三方库有一个kotlin写的时间计算库比较有趣,代码部分Kolin的Delegate可以看两眼,没准可以简化你的项目哦.

<!--more-->

## ARTICLES & TUTORIALS

## [How I Learned to Love Unit Testing with Toothpick ](https://medium.com/groupon-eng/how-i-learned-to-love-unit-testing-with-toothpick-13ad305b35d)

文章推荐了一个新的DI库叫做[ToothPick(牙签)](https://github.com/stephanenicolas/toothpick),它对UnitTest有很好的支持,用它提供的TestRule并配合Mocktio的TestRule可轻松完成测试类mock依赖类的注入.

其实我觉得`Mokito.initMocks`已经很好很强大了...

关于TP与Dagger2的一些优势可以见其github的[doc](https://github.com/stephanenicolas/toothpick/wiki/FAQ#why-creating-toothpick)

## [Making your app Switch Access Compatible ](https://riggaroo.co.za/android-accessibility-switch-access/)

文章比较有爱,只为了帮助有障碍的人,将你的App做到支持Switch Accessible(一种有一到两个按键的遥控器,来控制手机).

## [Drag and Drop and Swipe to Dismiss ](https://therubberduckdev.wordpress.com/2017/10/24/android-recyclerview-drag-and-drop-and-swipe-to-dismiss/)

介绍了如何通过RecyclerView的`ItemTouchHelper`实现侧滑删除,拖拽移动位置.

## [Your first Google Assistant skill ](https://medium.com/@froger_mcs/your-first-google-assistant-skill-883cf6c21545)

通过Google Action API与DialogFlow呈现了一个记录你每天喝水量的App.

## [Kotlin is Dope And So Are Its Custom Property Delegates ](https://robots.thoughtbot.com/kotlin-is-dope-and-so-are-its-custom-property-delegates)

文章介绍了Kotlin里面`Delegate`的强大之处,比如通过`ReadWriteProperty`代理get与set,`Delegates.observable`监听新老值的变化,`ObservableProperty#changeBefore`可以hook数据变化等等...

## [Migrate to Android Plugin for Gradle 3.0.0 ](https://developer.android.com/studio/build/gradle-plugin-3-0-0-migration.html)

介绍了如何迁移到新的Android Gradle Plugin 3.0.
需要升级Gradle plugin到4.1,新加的falvorDimensions必须制定,还有一些api也发生了变化,如`implementation/api`取代了以前的compile,等等.
Google的官方文档,主要以你遇到什么错就该怎么改来帮你做迁移.

## [How Oreo's color profiles work on the Pixel 2 and 2 XL ](http://www.androidpolice.com/2017/10/25/oreos-color-profiles-work-pixel-2-2-xl/)

介绍了Pixel2 升级到Oreo对`DCI-P3`颜色的支持,O可以通过设置`android:colorMode="wideColorGamut"`开启广域颜色,但是其内置的应用没有一个支持的,包括`GooglePhoto`,只有直接用OpenGl渲染的一些游戏支持,Camera也支持可惜拍照完看图片的是有依旧不支持...

## [Random Musings on the Android 8.1 Developer Preview 1 ](https://commonsware.com/blog/2017/10/25/random-musings-android-8p1-developer-preview-1.html)

介绍了一部分8.1 API上的新变化.
如添加了`NotificationListenerService`可以查看是否拥有通知权限.
还有可以在Manifest里面指定程序需要的RAM情况,帮助安装的时候进行过滤.`SQLiteDatabase`可以创建数据库信息到内存里了.等等.

## LIBRARIES & CODE


## [RxSSE ](https://github.com/EnricSala/RxSSE)

一个基于RxJava用kotlin些的SSE(H5里面基于Http的移送协议/服务)客户端.

## [Mezzanine ](https://github.com/anthonycr/Mezzanine)

基于Annotation的可以在编译阶段读文件内容的一个第三方库,支持Java,Kotlin

## [Time ](https://github.com/kizitonwose/Time)

基于kotlin写的一个安全的时间计算库
可以写出这个的计算
```
50.seconds < 2.hours
```