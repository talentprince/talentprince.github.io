---
title: 'Android Weekly Issue #279'
date: 2017-10-20 12:42:58
tags: [Clean Architecture,IoT,DialogFlow,Kotlin Coroutines,RxJava,Firebase,Dagger,Modular Architecture]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #279
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1508475006/279_eqp0pr.jpg
---

October 15th, 2017

## [Android Weekly Issue #279](http://androidweekly.net/issues/issue-279)

本期主要内容包含与`Clean Architecture`相关的两篇(包结构,离线app),IoT,Google AI (`DialogFlow`),以及`Kotlin Coroutines`的相关知识,还包含如何使用Kotlin以及RxJava2提升编码效能的文章,以及Dagger与Firbase配置的文章,以及一篇有意思的制作世界上最小APK的文章.

代码主要看点在于Kotlin Coroutines.

<!--more-->

## ARTICLES & TUTORIALS

## [Package-by-Feature in Modularised Android Projects ](https://overflow.buffer.com/2017/10/13/package-feature-modularised-android-projects/)

文章介绍了对于`Android Clean Architecture`如何按照`feature`来分包,app会被分为`Data`,`Remote`,`Cache`,`Presentation`,`View`五个部分,每个部分都会有一个`Conversations`的子包包含`Model`,`Mapper`等相关模块,并且还包含一些特有的子包,如`View`会有`Inject`模块,具体可看原文.

## [Exploring the .class side of Kotlin — Part 2 ](https://proandroiddev.com/exploring-the-class-side-of-kotlin-part-2-60d71a780279)

文章继续介绍作者的习惯,开一个[CS Bytecode Viewer](https://github.com/borisf/classyshark-bytecode-viewer)关注每次Kotlin代码改变后, .class的变化.

## [Android Things – Temperature Sensor, I2C on the Rainbow Hat ](http://blog.blundellapps.co.uk/tut-android-things-temperature-sensor-i2c-on-the-rainbow-hat/)

文章介绍了如何通过I2C协议开发IoT,这里用到的是[Rainbow Hat](https://shop.pimoroni.com/products/rainbow-hat-for-android-things),一个温度传感器.

## [Building Offline-First App using MVVM, RxJava, Room and Job Queue ](https://proandroiddev.com/offline-apps-its-easier-than-you-think-9ff97701a73f)

文章介绍了作者的一个Demo,实现了离线功能,架构上使用了`Clean Architecture`,UI与Database唯一绑定,后台运行SchedulerJob对数据库的数据进行Sync,如果成功将`isSynced`更新为True,如果失败则将其删掉.

## [Exploring Dialogflow: Understanding Agent Interaction ](https://medium.com/@hitherejoe/exploring-dialogflow-understanding-agent-interaction-8f3323e3b738)

Google的`API.IA`十月份换名字啦,现在叫`DialogFlow`,听起来是不是更生动了一些.
文章介绍了`DialogFlow`的一些概念,如`Invocation`定义如何启动会话,`Intent`定义一个话题,`User Saying`定义某个话题的关键词来触发话题,`Entities`是Request关键字的映射,`Fulfillment Request`是通过Entity Value来查询结果填充问题答案的过程,`Response`是最终给User的答案.

## [Setup Firebase on Android with multiple environments ](https://medium.com/bam-tech/setup-firebase-on-ios-android-with-multiple-environments-ad4e7ef35607)

如何在Android与iOS上配置Firebase...

## [Improve your tests with Kotlin in Android — Pt.2 ](https://proandroiddev.com/improve-your-tests-with-kotlin-in-android-pt-2-f3594e5e7bfd)

文章介绍了通过kotlin的特点以及`mockito-kotlin`来对传统的代码进行改造.

如使用了``(backticks)把你的test方法名包起来可以加空格标点成为一句话,更有意义.

使用`apply`或者`with`来省略receiver

使用
```
mock { on {} then {} }
```
来封装一些无参的`when().then()`等.


## [Playing APK Golf ](https://fractalwrench.co.uk/posts/playing-apk-golf-how-low-can-an-android-app-go/)

文章讲述了如何制作一个世界上最小的APK,只有`1757Bytes`,并称如果谁能再缩小可以提pr,这个最小的apk连dex文件都删掉了,整个apk只有包含一个touch出来的dex与简化后的manifest,而且在Android O上是合法的apk.

## [Keeping the Daggers Sharp ](https://medium.com/square-corner-blog/keeping-the-daggers-sharp-%EF%B8%8F-230b3191c3f)

介绍了Dagger2的一些基本知识,包括@Provides与@Inject,@Binds等等,内容不多.

## [The missing RxJava 2 guide: Supercharge your Android development ](https://techbeacon.com/missing-rxjava-2-guide-supercharge-your-android-development)

文章是一个帮助你从异步回调世界来到React世界的说明书,先讲述了RxJava的优势.
- 轻松掌管多线程
- 再也不会为回调无底洞烦恼
- 很好的错误处理机制
- 强大的operator
- 代码更少错误更少bug更少
- 跨平台(主要每个平台都有对应的Rx框架)

而后介绍了`3O`,`Observable`,`Observer`,`Operator`,以及线程相关的操作符.

## [Modular Architecture for faster Build Time ](https://proandroiddev.com/modular-architecture-for-faster-build-time-d58397cb7bfe)

介绍了如何提升Build速度,当然除了常设置的gralde属性,如开启daemon,parallel,单元测试开启多线程,多VM支持.
更重要的是,程序最好模块化,这样不改动的module将不会编译,提升了整体速度.

## [Diving deep into Kotlin Coroutines ](https://www.kotlindevelopment.com/deep-dive-coroutines/)

介绍了Kotlin实验室的`Coroutines`的一些知识,最简单的`launch{}`与可以返回值的`async{}`,可以阻塞当前`coroutine`的`suspend function`,以及可以等待线程完成的`await()`与`join()`

## [Reactive Mythology: Interrupt Patterns ](https://medium.com/@tomarShashank/reactive-mythology-interrupt-patterns-e244979b865a)

文章介绍了通过一些操作符来实现Intercept,如`takeUtil`,或者通过`compose`来截断(`switchOnNext`到`Observable.nerver`),或者与会抛异常的Observable来`merge`,具体可以查看他的code.

## LIBRARIES & CODE

## [literallytoast ](https://github.com/dvoiss/literallytoast)

说是Toast,但实际上是个弹框,这个库给你真正弹出一个烤面包来.

## [chips-input-layout ](https://github.com/tylersuehr7/chips-input-layout)

可以把输入框输入字串包在一个Chips里面,每一个Chips可以单独关闭或者放大..有的库可能叫Pills.