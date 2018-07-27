---
title: 'Android Weekly Issue #319'
date: 2018-07-27 09:00:01
tags: [Android Weekly,MotionLayout,Kotlin,Shortcut,Gradle,Kotlin Test,Kotlin Delegate,LiveData,Kotlin Eventbus]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #319
thumbnailImage: https://res.cloudinary.com/dtn0pkdmg/image/upload/v1532653705/319_lb48lj.jpg
---
July 22nd, 2018

## [Android Weekly Issue #319](http://androidweekly.net/issues/issue-319)

本期内容包括MotionLayout如何做动画的介绍,Kotlin when完备性的实现,以及如何Move一些Gradle的东西到Kotlin,还包括一个比较好的Kotlin Test库,以及如何用LiveData整合不同数据源,还有一个轻量级的Kotlin Eventbus的推荐.

Lib部分有Jake大神的Android与Chrome Extension的android sdk查看器,还有一个android黄瓜测试生成工具等...

<!--more-->

# NOTES

## [Creating Animations With MotionLayout for Android ](https://code.tutsplus.com/tutorials/creating-animations-with-motionlayout-for-android--cms-31497)

通过定义MotionLayout布局xml,指定定义好的MotionScene就可以完成动画.

MotionScene内可以定义ConstraintSet指定位置,然后定义Transition指定起始Constraint,也可以指定KeyFrameSet包含多个KeyPostion,或者KeyCircle.

还可以绑定事件,OnClick,OnSwipe来启动动画.


## [When is “when” exhaustive? ](https://medium.com/@ataulm/til-when-is-when-exhaustive-31d69f630a8b)

为了防止大家写when的时候忘记写else导致问题,作者想出来了一些好办法.

如when {}.let{}或者扩展方法返回自己.

```
val <T> T.exhaustive: T
    get() = this
```

## [Android Studio - Taming the interface ](https://jeroenmols.com/blog/2018/07/16/androidstudioshortcuts3/)

AS的一些快捷键,有一些很多人没注意到的,如小窗口的大小调整,等等.配有动图.


## [Becoming Google Certified Associate Android Developer ](https://medium.com/@sodiqOladeni/becoming-google-certified-associate-android-developer-907bdb61d79f)

Google推出了149美元考助理安卓工程师的认证,24小时完成一个project并提交,如果通过在线10分钟回答5个问题,就可以轻松通过...

如果不通过可以免费补考,如果还不过那就得交钱了...

只考安卓四大组件,不考语言知识,因为安卓更重要的是框架组件..


## [Maintainable Architecture – Daily Forecast ](https://blog.stylingandroid.com/maintainable-architecture-daily-forecast/)

系列最后一节,主要讲了如何注入你的ViewModel,还给出了demo地址.

## [Cloud Continuous Integration on Android with Kotlin Project ](https://proandroiddev.com/cloud-continuous-integration-on-android-with-kotlin-project-8d6f12cbf0c4)

文章介绍了如何大家CI,本篇作者使用的是Travis,并且添加了Jacoco生成覆盖率报告,并且push到Codecov.io上.

最后当然不要忘记加高大上的badge到你的README...

## [Moving Your Gradle Build Scripts to Kotlin ](https://pspdfkit.com/blog/2018/moving-your-gradle-build-scripts-to-kotlin/)

本篇文章实际上是个标题党,与之前介绍的用Kotlin DSL写gradle的[文章](https://antonioleiva.com/kotlin-dsl-gradle/)不太一样.

这篇文章只是自己定义了一个plugin,并且可以在自己的buildSrc里面用kotlin声明一些变量, 然后在gradle里面去调用,比如说dependencies的版本.

之前的那篇文章使用了gradle 4.5.1以上的新特性,然后将gradle后面加上后缀kts即可,当然在buildSrc里面声明变量的同时还需要引用`kotlin-dsl`插件.这样你的kts文件里就可以写成kotlin dsl风格了.这是题外话...可以查看#283的文章.


## [Data Driven Testing with KotlinTest ](https://proandroiddev.com/data-driven-testing-with-kotlintest-a07ac60e70fc)

作者推荐一个叫[KotlinTest](https://github.com/kotlintest/kotlintest)的库可以提供很多有趣的assert方法.

来支持他自己的Data driven test的理念,来做数据的对比.

如forAll可以添加很多组数据,然后通过shouldBe进行判断.

```
forAll(
    row(...)
) { ... ->
   ... shouldBe ...
}
```

## [Reactive patterns using Transformations and MediatorLiveData ](https://medium.com/google-developers/livedata-beyond-the-viewmodel-reactive-patterns-using-transformations-and-mediatorlivedata-fda520ba00b7)

作者介绍了如何用MediatorLiveData+Transformations来实现RxJava类似于zip的功能来进行combine不同的DataSource.

在这之前作者介绍了使用LiveData过程中的一些注意事项以及解决办法.

如是否去Share一个LiveData需要值得考虑它会导致的问题,使用MediatorLiveData添加外部source会导致泄露,使用switchMap去解决问题的时候也一定注意需要在构造的时候用,因为map/switchMap都会创建新的LiveData,等等.

最后一句话写到,如果使用了AutoDisposal的Rxjava第三方控件,就没必须这么用LiveData了.


## [Delegate your Lifecycle to Kotlin ](https://blog.blueapron.io/delegate-your-lifecycle-to-kotlin-17c1d0d876c9)

为了防止一些Activity或者Fragment的变量有跟生命周期同等的初始化与销毁,使用Kotlin的`by`来交给代理来改变它的Value,代理通过很多方式都可以监听生命周期,如LifecycleObserver,或者RxLifeCycle,来实现setValue.


## [KDispatcher simple and light-weight event bus for Kotlin ](https://medium.com/@sphc/kdispatcher-simple-and-light-weight-event-bus-for-kotlin-e0fa4aaea1c7)

一个Kotlin的轻量EventBus,而且是与平台无关的,backend也可以用.


# LIBRARIES & CODE


## [wearfaceutils ](https://github.com/purposebakery/wearfaceutils)

Android Wear 表盘


## [pickle ](https://github.com/fourlastor/pickle)

Android 黄瓜测试代码生成器.


## [kotlintest ](https://github.com/kotlintest/kotlintest)

一个kotlin test framework, 可以写出 shouldBe should之类的assert.

## [SdkSearch ](https://github.com/JakeWharton/SDKSearch)

Jake Wharton大哥的android app跟chrome扩展,查询android sdk的.

## [ketro ](https://smilecs.github.io/ketro/)

Retrofit + LiveData的组合.


## [KDispatcher ](https://github.com/Rasalexman/KDispatcher)

Kotlin轻量级Event Bus
