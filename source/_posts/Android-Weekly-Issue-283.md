---
title: 'Android Weekly Issue #283'
date: 2017-11-20 11:08:27
tags: [Android Weekly,Kotlin Test,RxJava2,Gradle,Kotlin DSL,Rx2Firebase,IoT,Android Security]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #283
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1511496201/283_j3gpwf.jpg
---

November 12th, 2017

## [Android Weekly Issue #283](http://androidweekly.net/issues/issue-283)

本期内容包括Gradle相关的几篇,如封装繁杂依赖的技巧,通过kotlin dsl让gradle支持kotlin,以及gradle入门指南等,还包括RxJava2的迁移介绍,以及关于IoT,图片压缩,Security相关的文章.
AS3.0也正式发布,feature满满,Firebase退出了Rx版本,Kata 测试教程也放出Kotlin版本供大家学习.


<!--more-->

## ARTICLES & TUTORIALS

## [Migration to RxJava 2.x ](https://medium.com/@d.khmelenko/rxjava-library-has-already-become-a-standard-in-android-development-f825fed8c6e4)

文章介绍了RxJava2的一些变化,如新的类型(Completable,Single,Maybe),Test Observable,特别值得注意的是`ErrorHandling`,之前RxJava1可以通过`registerErrorHandler`注册,但是只有监听功能,无法Hook(如OnErrorNotImplementedException),程序该崩还得崩.

新版本通过`RxJavaPlugins.setErrorHandler`可以router到所有Rx流中产生的任何异常,包括在onNext/onSuccess中产生的异常,以及没有实现onError发生错误产生的异常.

值得注意的是,RxJava1会在Rx流结束或者Cancel后吃掉所有后续产生的Throwable,但是RxJava2将会依旧发射错误,这点需要注意,之前不不崩溃的程序可能会出问题,最好自己设置一个handler来catch.


## [Android Things - IoT possibilities ](https://www.novoda.com/blog/android-things-iot-proof-of-concept-devices/)

文章介绍了Hackster.io联合Google搞的一个IoT的比赛,介绍了几个好的创意,如办公室监控,都是字母的表,监控狗狗睡眠的床等等,比较有趣.包含每个项目的Detail信息的链接.


## [Experimenting with Gradle dependencies ](http://alexfu.github.io/android/2017/11/07/experimenting-with-gradle-dependencies.html)

文章介绍了如何整理与简化我们gradle里面大量的dependencies,安装feature进行划分比较清晰,然后再利用`closure delegate`封装,使得`implementation`这个关键字可以在我们定义的函数中使用,将我们的dependencies改成大概这个样子.

```
dependencies {
  ui()
  network()
}
```

关于如何用`closure delegate`定义`ui`与`network`可以查看作者原文.


## [RadialGradient ](https://blog.stylingandroid.com/radialgradient-gradients/)

上期有篇文章讲了软硬加速的一些知识,这篇文章讨论Shader(RadialGradient)与Hardware/Software Layer相关的知识.

## [Kotlin DSL to write Gradle scripts on Android: Step by step walkthrough ](https://antonioleiva.com/kotlin-dsl-gradle/)

Gradle推出了对Kotlin的支持,`gradle-kotlin-dsl`,可以用kotlin来写你的gradle文件了,但是由于文档很少很少,作者写了一个demo来帮助大家.

- 你需要将你的gradle升级到最新.
- 你需要给build.gradle加上一个.kts的后缀
- 你可以在buildSrc文件夹里面定义一些kotlin类,将一些string的定义放进去,取代之前的`ext{...}`的写法.


## [Rx2Firebase : Firebase + RxJava ](https://medium.com/@franciscodurdingarcia/rx2firebase-firebase-rxjava-android-bde8158fb4cf)

作者介绍了Rx2Firebase的推出对于使用Firebase online database用户带来的福音,再也不用被嵌套回调烦扰.


## [Migrating Crashlytics to the Firebase Console ](https://proandroiddev.com/migrating-crashlytics-to-the-firebase-console-5e05b6ff8c12)

文章介绍了如何在FirebaseConsole看到Crashlytics(Fibric),其实就是把你的firic的依赖删掉,换成firebase.

需要注意的是,他没有数据的migrate,所以fibric上的不会出现在firebase里面.


## [Beginner’s Guide to Gradle for Android Developers ](https://journals.apptivitylab.com/beginners-guide-to-gradle-for-android-developers-7972bfdf0668)

文章介绍了Gradle的基本常识,包括每一个block的作用,很适合初学者学习,包括root的build.gradle,以及每一个子项目的build.gradle.


## [Secure data in Android — Encryption in Android (Part 2) ](https://proandroiddev.com/secure-data-in-android-encryption-in-android-part-2-991a89e55a23)

文章是一个安全系列文章的第三篇,介绍了如何通过`KeyguardManager`来要求手机必须设置LockScreen.
还有如何使用`Keystore`相关方法创建公钥私钥,通过`Cipher`进行加密解密.
Android 18之后,秘钥都是保存在系统服务里,并随着应用的卸载而删除,不像之间必须存在本地文件,存在被extract的风险了.


## [Alias free resize with RenderScript ](https://medium.com/@petrakeas/alias-free-resize-with-renderscript-5bf15a86ce3)

文章介绍了原生的`Bitmap.createScaledBitmap`在缩放图片时候会产生锯齿,通过`BitmapFactory.decode`配合`options.inSampleSize`可以达到比较好的效果,但是只支持偶数被缩小,并且Bitmap必须没有解码之前.

采用`RenderScript`进行缩放,可以达到非常好的效果.但是耗时比较多,是createScaledBitmap的30倍.

## [Welcome to Android Studio 3.0 ](https://proandroiddev.com/welcome-to-android-studio-3-0-466350f5b707)

AS3.0正式发布,配合gradle 3.0,支持Java8 feature build-in,支持kotlin,支持Instant App...等等新的功能.


## [Testing like a pro in Kotlin ](http://blog.karumi.com/testing-like-a-pro-in-kotlin/)

Karumi将Kata挪到了kotlin上,提供了好几个sample帮助学习kotlin测试.

包括Clean Architecture相关,Espresso相关,Stubbing Http相关等等.


## LIBRARIES & CODE

## [RIBs ](https://github.com/uber/RIBs)

优步的跨平台框架.

## [BottomSheetLayout ](https://github.com/qhutch/BottomSheetLayout)

View级别的一个BottomSheet.

## [Rx2Firebase ](https://github.com/FrangSierra/Rx2Firebase)

Firebase的Rx版本,使用online db更方便