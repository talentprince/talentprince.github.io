---
title: 'Android Weekly Issue #286'
date: 2017-12-14 08:46:53
tags: [Android Weekly,Kotlin,RxJava2,Instant App,Android Oreo]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #286
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1513212624/286_uy2c3v.jpg
---

December 3rd, 2017

## [Android Weekly Issue #286](http://androidweekly.net/issues/issue-286)

本期文章包含如何通过踩坑来学习Kotlin,以及利用Kotlin的`data class`做MVVM状态保存,还包含一些基础知识的介绍,如RxJava2线程切换,Kotlin与Java容器分析.

另外,还包括Intant App的软文一篇,以及 Android O对Notification进行Channel管理的文章,帮助大家适配O以上的通知.


<!--more-->

## ARTICLES & TUTORIALS


## [Some useful insights on Instant apps ](https://medium.com/nos-digital/some-useful-insights-on-instant-apps-67cc7d177695)

文章介绍了荷兰的新闻应用NOS支持IA的实例,技术成分不多,更像是新闻报道,需要了解IA具体实现的可能得不到想要的.


## [Using Espresso to Test Opening Links ](https://collectiveidea.com/blog/archives/2017/11/29/using-espresso-to-test-opening-links)

一个女博主的小发现,如何通过Espresso测试通过TextView的autolink打开其他程序.

其实是通过[openLinkWithText](https://developer.android.com/reference/android/support/test/espresso/action/ViewActions.html#openLinkWithText(java.lang.String))来发出这个事件.


## [Learning Kotlin by Mistake ](https://engineering.udacity.com/learning-kotlin-by-mistake-b99304b7d724)

文章介绍了在错误中不断前行,学习Kotlin相别于Java的特性.

如尽量的通过`applay` `run` `let` `with`等操作符将你的逻辑连起来.

`CompanionObjects`与`@JvmStatic` ` @JvmField`的取舍

`lateinit`与`by lazy`的故事,以及自定义Delegate等等.

最笨的办法也可以通过自动转换来学习,但是自动转换出来的并不是完全纯粹的Kotlin哦.


## [Paper Signals: A Voice Experiment ](https://papersignals.withgoogle.com/)

一个IoT的教学,制作一个声音盒子,通过你的语音可以变形. 比较有趣的是盒子的模型零件可以打印出来自己剪裁.
需要的Code他们已经提供了.

当然最重要的是,需要买材料,$24.95.

## [Kotlin Collections Inside. Part 1 ](https://www.runtastic.com/blog/en/tech/kotlin-collections-inside-part-1/)

一个分析Kotlin容器的系列文章,这是第一篇,关于List.

主要讲了Java与Kotlin容器的关系,对于Kotlin来说,所有Java的容器都是Mutable的,而对于Java来说Kotlin的Immutable容器可以调用改动操作,但是会抛异常.

并且介绍了Kotlin如何初始化Immutable与Mutable的List,通过ByteCode分析,虽然MutableList没有继承与Java的ArrayList,但是通过`arrayListOf`与`mutableListOf`生成的List可以互转,原因是MutableList在生成ByteCode后,也同样继承了ArrayList....


## [Multi-Threading Like a Boss in Android With RxJava 2 ](https://blog.gojekengineering.com/multi-threading-like-a-boss-in-android-with-rxjava-2-b8b7cf6eb5e2)

文章主要讲了RxJava2如何在线程之间随意切换的,虽然没有涉及实现原理,但是通俗的讲解了`subscribeOn`与`observerOn`的使用.一个是改变`source`,一个是改变`downstream`.


## [Oreo Notifications: Channels – Part 1 ](https://blog.stylingandroid.com/oreo-notifications-channels-part-1/)

文章介绍Android O对于Notification的新概念,`Channel`,对于没有使用新的Notification Compat API设置Channel的,将不会再Android O上弹出通知.

Channel是为了让用户对程序的不同Notification进行分组管理,可以对不同Channel分别设置开关,以及通知方式(震动,亮灯,静音等).

与Channel配合的还有Group,可以将某几个Channel归类于一个Group,在设置页面可以看到不同的Group下的有不同Channel.


## [Representing View State with Kotlin Data Classes ](https://medium.com/@trionkidnapper/viewmodel-and-kotlin-data-class-7d3a3b854805)

文章介绍了把所有状态封装在一个ViewState的`data class`里,并通过其`copy`的方法,对发生变化的状态进行改变,这样可以保持其他状态不变.

该状态可以作为ViewModel里面的一个Observable被订阅,获取不同状态下的ViewState,对UI进行操作.


## [Kotlin on the Backend ](https://medium.com/rocket-travel-engineering/kotlin-on-the-backend-at-rocket-travel-31da239888db)

Rocket Travel已经使用Kotlin做Spring Boot开发一年有余,评价很好,可以在后端开发中使用到Kolin的feature,一定很High.


## LIBRARIES & CODE


## [RoboPOJOGenerator ](https://github.com/robohorse/RoboPOJOGenerator)

一个插件可以直接将JSON转成Java或者Kotlin的POJO文件...


## [avdo ](https://github.com/alexjlockwood/avdo)

Python的包,可以优化Vector动画或者Drawable文件.
