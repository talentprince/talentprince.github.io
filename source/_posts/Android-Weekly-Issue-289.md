---
title: 'Android Weekly Issue #289'
date: 2017-12-31 11:36:24
tags: [Android Weekly,Clean Architecture,Android Notification,Room,Path Animation,Kotlin,ConstraintSet Animation]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #289
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1514691335/289_fyupzr.jpg
---

December 24th, 2017

## [Android Weekly Issue #289](http://androidweekly.net/issues/issue-289)

今年最后一篇, 包含了可以上传log记录的HyperLog,以及Android的面试技巧,还有Model的分层,以及如何迁移到Room.
还有比较炫酷的一个Path动画的实现方法值得去看.剩下的多是一些入门介绍,如kotlin,firebase messaging,contraintset animation等.

<!--more-->

## ARTICLES & TUTORIALS


## [HyperLog: Android Remote Logger Library for Debugging ](https://android.jlelse.eu/android-remote-logger-library-for-debugging-343443bd38b7)

作者讲了很多人苦恼有时候插线数据线Logcat给清掉了,也看不到日志信息,而Timber活着Logger也没法支持Production环境,推荐了`HyperLog`,可以上传日志文件到远端服务器,可以设定一些schedular定时上传.


## [Bring life to your custom view ](https://medium.com/@romandanylyk96/android-bring-life-to-your-custom-view-8604ab3967b3)

文章介绍了通过动画来绘制一个自定义View,原理是通用的,就是首先分析你的图形构成的元素,需要哪些变量的变化,然后通过`ValueAnimator`控制这些变量的变化,再通过`invalidate`触发onDraw依照变量的值进行绘制.


## [Interviewing Tips for Android Engineers ](https://eng.lyft.com/interviewing-tips-for-android-engineers-f01ce7fba163)

作者作为Lyfy的一员,作者很高兴的分享自己的一些新的帮助面试者,介绍了Android面试的一些tips,包含从开始的电话初面或者作业,到后来的Java面,Android UI相关技能面,Android Infrastructure面,以及Design与Background的面所应该注意的点以及准备的方法.


## [Using Architecture Components with Firebase Database - Part 3 ](https://firebase.googleblog.com/2017/12/using-android-architecture-components_22.html)

文章继续之前通过LiveData封装FirebaseDatabase,与ViewModel结合实现MVVM的工作.

这是第三部分,主要介绍如何优化没有必要的query,Activity有可能因为转屏导致configuration changed,从而引起LiveData瞬间切换到Inactive又变成Active,导致数据库重复的query.

解决方案是将销毁操作封装到延迟的Runnable里面,发送给Handler,并设置标记位.在onActive中检测标记为,如果发现仍然在pending状态,就`removeCallbacks`,清楚消息...


## [Data model mapping in Android Apps ](https://overflow.buffer.com/2017/12/21/even-map-though-data-model-mapping-android-apps/)

文章一步一步介绍了我们应该如何处理我们的Model,其实是希望我们应该对我们的Model进行分层.

API与Cache的原始数据,以及与我们核心业务相关的Domain层,再者就是包含UI state等信息的Presentation层了.

其实就是我们常说的Model->Domain->Presentation.

当然在我们所谓BFF(backends for frontends)理论下(简单来说就是Server为Mobile加一层转换,直接输出显示内容相关数据),可以省去Model与Domain层,但一些UI的state我们仍需想办法维护.


## [Incrementally migrate from SQLite to Room ](https://medium.com/google-developers/incrementally-migrate-from-sqlite-to-room-66c2f655b377)

介绍了如何逐步将你的db迁移到room.

- 首先根据你的table定义`Entity`.
- 其次实现`RoomDatabase`,增加版本号并加上空的Migration逻辑.
- 使用`SupportSQLiteOpenHelper`替换原有的SQLiteOpenHelper,如果你之前没有使用raw的sql语句,需要用使用`SupportSQLiteQuery`拼出query条件.
- 写自己的`DAO`,替换掉Cusor的操作.


## [Boost your app reviews with Firebase Predictions! ](https://medium.com/@Tajchert/boost-your-app-reviews-with-firebase-predictions-19aff4001f27)

介绍了利用Firebase正在测试阶段的`Predictions`帮助我们决定何时弹提醒用户评分的框框.


## [Playing with Paths ](https://medium.com/@crafty/playing-with-paths-3fbc679a6f77)

介绍了如何绘制轨迹动画,挺炫酷的,主要使用到了`PathDashPathEffect`.


## [How to add Push Notification capability to your Android app ](https://medium.com/@nileshsingh/how-to-add-push-notification-capability-to-your-android-app-a3cac745e56e)

文章介绍了如何通过Firebase Messaging来做推送,包含了详细的Client端的配置,以及Server端的Initialize.


## [Animations with ConstraintLayout and ConstraintSet ](https://hellsoft.se/animations-with-constraintlayout-and-constraintset-b4634d38981f)

文章介绍了通过`ConstraintSet`对`ConstraintLayout`添加一些如`ChangeBounds`的动画.


## [Getting started with Kotlin on Android ](https://blog.sourcerer.io/getting-started-with-kotlin-on-android-6242a5f6fd57)

Kotlin简单的入门教程,包含IDE的配置,以及几个Kotlin的特点,如when等等.非常初级,想入门的可以多看看...呵呵...


## LIBRARIES & CODE


## [hyperlog-android ](https://github.com/hypertrack/hyperlog-android)

可以将log记录发到后台的一个库.

## [android-clean-architecture-mvi-boilerplate ](https://github.com/bufferapp/android-clean-architecture-mvi-boilerplate)

所谓基于MVI的Clean Architecture.


## [ReactiveNetwork ](https://github.com/pwittchen/ReactiveNetwork)

基于RxJava监听网络状态的库.