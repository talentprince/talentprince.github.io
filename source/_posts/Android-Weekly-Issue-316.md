---
title: 'Android Weekly Issue #316'
date: 2018-07-01 19:00:01
tags: [Android Weekly,Google IO,Kotlin Espresso,Navigation Architecture Component]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #316
thumbnailImage: https://res.cloudinary.com/dtn0pkdmg/image/upload/v1531988402/316_u2dqxh.jpg
---

July 1st, 2018

## [Android Weekly Issue #316](http://androidweekly.net/issues/issue-316)

本期内容包含教你使用Kotlin通过Annotation Processor生成代码文件, JetPack中的Android KTX, 以及升级到Target26所需要注意的东西,还包含如何使用KTX简化AndroidX里面的Slice的Build,以及如何通过MotionLayout方便实现动画的系列,还有MLKit中扫条形码的Lib介绍,以及一些譬如Kotlin MVVM, Koin依赖注入做Test, UI Test去除动画,入行一年感受,DialogFLow来做面试机器人等等的有趣内容.


<!--more-->

## ARTICLES & TUTORIALS

## [Generating Code via Annotations in Kotlin ](https://willowtreeapps.com/ideas/generating-code-via-annotations-in-kotlin)

文章介绍了如果用Kotlin来写Annotation Processor生成代码.

两个关键的Lib

```
	\\ Code generation library for kotlin, highly recommended
	implementation 'com.squareup:kotlinpoet:0.7.0'

	\\ configuration generator for service providers
	implementation "com.google.auto.service:auto-service:1.0-rc4"
	kapt "com.google.auto.service:auto-service:1.0-rc4"
```

- 首先需要通过`annotation class`定义自己的Annotation.
- 其实继承`AbstractProcessor`实现自己的Processor, 通过`@AutoService`注册到系统的`Processor.class`上去.复写`process`方法生成代码.


## [Android KTX - Android development with Kotlin ](https://kotlinexpertise.com/android-ktx-kotlin/)

之前已经有不少Kotlin的语法糖了,这次JetPack推出的是Android官方的KTX,其主要实现原理就是通过给已有的Framework添加很多方法或者变量的Extension.

类似于Anko, 支持的列表在[这里](https://developer.android.com/kotlin/ktx),当然对于类似的Project, 学习是一个过程.


## [Remember, remember… to target API 26 on November! ](https://medium.com/@fabionegri/remember-remember-to-target-api-26-on-november-7ce4fdde2c08)

文章列举了要逐步将Target升级到26所需要注意的东西.

Marshmallow 6.0 Level 23

- Runtime Permission
- 删除了Apache Http Client
- 从OpenSSL到BoringSSL
- 删除`Notification.setLatestEventInfo()`

Nougat 7.0 Level 24

- 限制后台网络,删除了`CONNECTIVITY_ACTION`广播通知,以及Intent中的 `ACTION_NEW_PICTURE`与`ACTION_NEW_VIDEO`.
- App外无法通过`file://`访问私有文件.
- 链接非NDK的Lib不行了.


Nougat 7.1 Level 25

- App Shortcuts
- 支持Image到Keyboard里,通过[Commit Content API](https://developer.android.com/guide/topics/text/image-keyboard).
- 添加了`android:roundIcon`


Oreo 8.0 Leve26

- 必须通过`startForeground`和`startForegroundService`来启动前台的服务,无法启动后台服务.
- 移除了几乎所有的隐式广播,除了[部分](https://developer.android.com/guide/components/broadcast-exceptions).
- 后台位置信息收到限制,推荐使用GCM的`FusedLocationProviderClient`或者`Geofencing`.
- 必须制定Notification Channel,否则收不到通知.


## [My first weeks as an Android Dev ](https://engineering.udacity.com/my-first-weeks-as-an-android-dev-e956ee49418d)

介绍了自己第一周做Android开发的经历,面对自己的任务,接触到了譬如`Design`,`ConstraintLayout`,`Data Binding & ViewModel`, `DI`,以及`Kotlin`的东西,有点应接不暇.


## [Espresso animations disabled flag ](https://github.com/ghostbuster91/espresso-animations-disabled-test)

通过设置flag来禁止测试中动画, 方便测试.

```
testOptions {
    animationsDisabled = true
}
```

## [Exploring Firebase MLKit on Android: Barcode Scanning (Part Three) ](https://medium.com/@hitherejoe/exploring-firebase-mlkit-on-android-barcode-scanning-part-three-cc6f5921a108)


文章介绍了用Firebase的VisionBarcode来扫条形码.


## [Introducing Slice Builders KTX ](https://medium.com/google-developers/introducing-slice-builders-ktx-2218ebde356)

Android Slice是最新推出的AndroidX里面的控件,主要是给Google Search App来提供一些类似Google Assistant的那种UI,支持静态与动态的布局.

KTX也推出了对该组件的支持,以便通过Kotlin来简化Slice Build使用过程.

如:

```
list(context, sliceUri, ListBuilder.INFINITY) {
        row {
           setTitle("Hello world")
        }
    }
```


## [Building an Action to Solve a Real World Problem: Part 1, Plan & Design ](https://medium.com/google-developers/building-an-action-to-solve-a-real-world-problem-part-1-plan-design-2a701fa004c8)

通过Dialogflow等一系列技术搭建了一个给面试者出题的训练系统,根据对话提供不同的题目.


## [Introduction to MotionLayout (part I) ](https://medium.com/google-developers/introduction-to-motionlayout-part-i-29208674b10d)

介绍了Google最新的ConstraintLayout 2.0的子类`MotionLayout`的使用,它可以通过在xml文件里定义的`MotionScene`来指定一些动画.

这些都是通过`TransitionManager`完成的,它会自己计算开始与结束之间的位置,完成动画,有点像Activity之间的SharedElement Transition Animation.


## [Testing with Koin ](https://proandroiddev.com/testing-with-koin-ade8a46eb4d)

通过Koin依赖注入框架来简化Kotlin Test里面的mock注入.


## [MVVM with Kotlin ](https://proandroiddev.com/mvvm-with-kotlin-android-architecture-components-dagger-2-retrofit-and-rxandroid-1a4ebb38c699)

介绍了Kotlin下Android的MVVM的全套实践,包含了DataBinding, Retrofit, Dagger等等.


## [Introducing adb-enhanced: A swiss army knife for Android development ](https://ashishb.net/tech/introducing-adb-enhanced-a-swiss-army-knife-for-android-development/)

一个Python的应用, enhance的adb,可以方便的进行clean安装,节能模式,doze模式的调试等等.

```
pip3 install adb-enhanced
```

## LIBRARIES & CODE


## [ColorPickerView ](https://github.com/skydoves/ColorPickerView)

一个选择颜色的picker view


## [adb-enhanced ](https://github.com/ashishb/adb-enhanced)

上面介绍的加强型Adb

