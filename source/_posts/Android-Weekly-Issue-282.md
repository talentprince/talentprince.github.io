---
title: 'Android Weekly Issue #282'
date: 2017-11-13 08:24:12
tags: [Android Weekly,Koin,Grox,Architecture Component,Room,Kotlin Coroutines,Android Prefetching]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #282
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1510532777/282_rtsvp0.jpg
---

November 5th, 2017

## [Android Weekly Issue #282](http://androidweekly.net/issues/issue-282)

本期内容相较上期丰富许多, 技术干货当然也有不少, 如`Koin`来替代Dagger应用在Kotlin项目,`Grox`做一个面向状态的程序,以及Room实现Relation的介绍,如何解决Architecture Component在Fragment上的漏洞,还有Kotlin Coroutines里面的一些新概念(Actor,Channel等).
当然也包含一些方法论,如将项目转Kotlin的好处及其方法,Java数据结构的基础介绍,伦敦Droidcon大会概况等等.

代码亮点主要在Koin的使用,可以特别关注一下.

<!--more-->

## ARTICLES & TUTORIALS

## [Moving from Dagger to Koin](https://medium.com/@giuliani.arnaud/moving-from-dagger-to-koin-simplify-your-android-development-e8c61d80cddb)

文章介绍了如何用[Koin](https://github.com/Ekito/koin)做到一个更Kotlin的依赖注入,与Dagger相比,使用`Koin`,无论是你的gradle配置会从很多行缩减到1行,而且你的初始化也将只需要调用`startAndroidContext`即刻.

最方便的是可以使用高阶Lambda定义你的`Module`,并且在注入的时候不需要写任何的Annotation,可以通过`by inject()`的lazy注入方法,或者直接通过primary constructor.

作者还提供了他将google的achitecture里面的Todo Sample用Koin改造后的效果,可以进行学习.


## [Exploring Firebase Predictions ](https://medium.com/@hitherejoe/exploring-firebase-predictions-fa22d093f98d)

文章介绍了Firbase的一个alpha阶段的新功能,`Prediction`,可以对RemotConfig,A/B Testing,以及Notification Composer进行一些基于机器学习的预测,帮助我们更快的改进产品,提升用户体验.


## [Architecture Components pitfalls — Part 1 ](https://medium.com/@BladeCoder/architecture-components-pitfalls-part-1-9300dd969808)

文章介绍了Google的Architecture Components的一个陷阱,虽然ViewModel会在Activity/Fragment销毁时候自动释放,但是在在Fragment使用的时候,很多时候这个Fragment不会销毁而是detach了,而重新attach后又会创建新的Observer,会有内存泄露的风险.

文章指出了好几个Approach来解决这个问题,包括最后一个用了还在试验的Data Binding直接bind我们定义的ViewModel.具体的方法可以查看详情.

## [Playing with elevation in Android ](https://blog.usejournal.com/playing-with-elevation-in-android-91af4f3be596)

文章介绍了一些有趣的事情,我们可以通过给View添加`OutlineProvider`(Layout#setOutlineProvider)来控制Shadow,但是实际上可以利用`elevation`来做到一些效果,Android Material Design系统有两个光源来呈现投影,一个在屏幕最上面向下照射,另外一个在屏幕正中间垂直照射,作者展示了了他的一个项目.

其使用了自定义的CardView,给其添加一个将Outline的大小设置为比Card本身小4dp的Provider,并有4dp的elevation,这样滚动的时候,越往屏幕的下放,影子就越深.

## [Grox: The Art of the State ](https://medium.com/groupon-eng/grox-the-art-of-the-state-b5223f48d696)

文章介绍了一个状态管理的库`Grox`,其提供了`Command`,`Action`,`Store`的概念,Command会通过`actions()`发射该命令下的不同状态下的Action,并交给store来进行`dispatch`,而store会被Grox的静态方法`states`封装后将dispatch的Action接收到UI线程来处理UI.

实际上的代码大致是这个样子

```
    states(store).observeOn(mainThread())
    .subscribe(this::updateUI);

    clicks(button)
        .map(click -> new XXXCommand())
        .flatMap(Command::actions)
        .subscribe(store::dispatch);
```


## [Canvas : The Real Play Ground! ](https://medium.com/@saurabhpant/canvas-the-real-play-ground-android-c0faa4b79943)

文章介绍了如何通过通Canvas画一个钟表,挺有意思,可以作为学习draw canvas的一个教程


## [RadialGradient – Layers ](https://blog.stylingandroid.com/radialgradient-layers/)

文章通过一个`RadialGradient`的view在不同layer上的不同表现来介绍图层以及软硬渲染的概念.

## [Using the ADS1015 analog to digital converter driver library ](http://blog.blundellapps.co.uk/tut-android-things-using-the-ads1015-analog-to-digital-converter-driver-library/)

开发`ADS1015` AD转换器的一个IoT相关的文章,对物联网感兴趣的可以看看.


## [Adopting Kotlin ](https://engineering.udacity.com/adopting-kotlin-c12f10fd85d1)

文章不是一篇技术偏向的文章,更多的是从各个方面介绍了kotlin的优势,以及作者在将他的项目转到kotlin过程中的一些经验.


## [Android Architecture Components: Room—-Relationships ](https://android.jlelse.eu/android-architecture-components-room-relationships-bf473510c14a)

文章介绍了Room如何实现外键关联,进行数据库JOIN查找. 除了我们经常用的`@ForeignKey`之外,还可以通过`@Embedded`与`@Relation`来定义外键关系.


## [Clean App with Kotlin & Architecture Components - Part 2 ](https://blog.elpassion.com/create-a-clean-code-app-with-kotlin-coroutines-and-android-architecture-components-part-2-4f585050d7d7)

文章接上次通过Kotlin的Coroutines来做的一个天气应用,这次引入`Actor`与`Channel`的概念,实际上Actor类似于一个信箱管家,而Channel则是信箱(包括信道),而且具有不同种类型的信道模式,有0缓存的,有无限缓存的,也有固定缓存的,Actor会将定义的数据类型通过Channel发送给接收方,通过Channel不同的类型,可以实现不同的消息循环效果.


## [Improving performance with background data prefetching ](https://engineering.instagram.com/improving-performance-with-background-data-prefetching-b191acb39898)

介绍了Instgram如何实现后台预取的,为了让用户的到更好的体验,特别是网络不好地区,他们实现了一套后台Wifi预取的逻辑.间接地,他们也实现了网络与应用使用的解耦,还节省了后台带宽,因为如果没有预取,每次开启程序都要请求主服务器抓取大量数据.


## [My take from Droidcon London 2017 ](https://medium.com/@michele.marchetti/my-take-from-droidcon-london-2017-d870c82e41c1)

介绍了作者有幸参加在伦敦举行的`Droidcon`大会,分享了在会上听到的诸多Presentation的概况,很多Topic,包括Kotlin Coroutines,Room,Test,Android Security等等.


## [Practical Data Structures Guide for Android developers ](https://blog.mindorks.com/practical-data-structures-guide-for-android-developers-73fdec190802)

作者在上大学之前就接触了Android开发,但当时没有进行数据结构与算法的系统学习,致使一些基础概念不清晰而有一些不好的习惯.
作者在这里分析了List,Map,Set,Stack等等的数据结构,包含ArrayList与LinkedList的区别,HashMap与LinkedHashMap等等的原理.

不过作者没有讲Android新的SparseArray与ArrayMap的特性.

## [How “Effective Java” may have influenced the design of Kotlin -  Part 3 ](http://www.lukle.at/blog/2017/08/how-effective-java-may-have-influenced-the-design-of-kotlin%E2%80%8A-%E2%80%8Apart-3/)

接着上期介绍的Kotlin的优势,本期介绍了, 总共有5点,包括用`by`来做代理实现类的组合,还有Kotlin没有装箱类型带来的好处,等等.


## LIBRARIES & CODE


## [Rings ](https://github.com/lalongooo/rings)

打不开了...

## [ads1015 ](https://github.com/blundell/ads1015)

IoT那个AD转换器的代码.

## [kotlinconf-app ](https://github.com/JetBrains/kotlinconf-app)

官方的Kolin会议应用,包含很多个版本,当然android版本使用kotlin编写

## [grox ](https://github.com/groupon/grox)

Android State保持的一个框架,可以将你的程序改成Command/Action驱动的哦.