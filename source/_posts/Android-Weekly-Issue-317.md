---
title: 'Android Weekly Issue #317'
date: 2018-07-08 19:00:01
tags: [Android Weekly,Jetpack,Page,React Native,Kotlin Extension,Android P,Universal Music Player]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #317
thumbnailImage: https://res.cloudinary.com/dtn0pkdmg/image/upload/v1531988405/317_vuwbrb.jpg
---

July 8th, 2018

## [Android Weekly Issue #317](http://androidweekly.net/issues/issue-317)

本期主要内容包括"重磅"的Udacity放弃RN(其实是因为他们RN写的那个Feature不要了),还包括如七部使用Google Page Lib,如何用Room设计与创建可维护的数据库等Code Guide的文章,以及Android P字体渲染,放大镜,Google新版Universal Music Play Sample,还有Retrofit如何工作,如何通过Kotlin Extension Generation来改善Dagger Butterknife使用体验,等等.

<!--more-->

## ARTICLES & TUTORIALS


## [State of Kotlin ](https://pusher.com/state-of-kotlin)

Kotlin状态的一些列数据,包括它的使用率,渗透率,在项目中的应用率等多种多样的数据,并且可以订阅.

一切数据表明,Kotlin在过去的一年中发展迅速,使用量Double,并且好评如潮.


## [How does Retrofit work ](https://medium.com/@theneckmaster/how-does-retrofit-work-6ecad1bb683b)

文章讲述了Retrofit如何工作,实际上是通过Proxy而非Processor去生成代码来实现的.文章表示在运行速度与编译速度上的Compromise是一个值得思考的问题.


## [Exploring Android P: Magnifier ](https://medium.com/@hitherejoe/exploring-android-p-magnifier-ddfd06bdecbe)

Android P提供了放大镜功能,并且TextView默认就实现了.
国内用户应该不是很陌生,几年前国内的一些大Android浏览器厂商在WebView里面也有类似的功能,不过主要是Android copy了 iOS的默认属性...


## [Tracking Android app metrics ](https://medium.com/@emmaguy/tracking-android-app-metrics-431cbea2113d)

文章介绍了CI上使用[StatHat](http://www.stathat.com/manual/start)上传APK的一些数据,比如size等,然后发现问题后通过[Danger](http://danger.systems/ruby/)给Github的PR上面发出报警.

这两个都是Ruby的应用.


## [Kotlin extension function generation ](https://medium.com/the-fabulous/kotlin-extension-methods-generation-15b5e6499dc8)

文章介绍了自己给Dagger与Butterknife写的ktx.

由于Dagger使用了生成文件进行注入,所以必须先编译一边才能通过Compiler生成文件,使用起来不便.

而ButterKnife使用了反射通过类名将自己Generate的辅助类在运行时创建,我们想消除这个反射.

这些原因都因为Processor只能生成文件,而无法改变已有文件.

所以作者使用了Kotlin Extension,首先在Lib里定义了一些Mock的接口,所以编译之前可以调用这些空实现.而在编译过程中,Compiler会生成对应的扩展方法,这样Dagger与Butterknife的问题都得以解决.


## [What's new for text in Android P ](https://android-developers.googleblog.com/2018/07/whats-new-for-text-in-android-p.html?linkId=53827942)

Android P在TextView上下了狠功夫,增加了很多功能.

- PrecomputedText

复杂字体Font在显示的时候实际上90%的时间都耗费在Measure计算上, 可以在后台线程通过`PrecomputedText`计算,然后在UI线程set

- Magnifier

放大镜,上篇有提到.TextView自己默认实现了.

- Smart Linkify

TextView方按选中可以通过Google的ML处理解析,提供可能相关的应用显示在Copy Cut Paste旁边,这一切都是通过`TextClassifier`实现的.

- Line Height & Baseline Alignment

提供了几个Attribute控制文字行高,以及baseline的top与bottom margin.

```
app:lineHeight
app:firstBaselineToTopHeight
app:lastBaselineToBottomHeight
```


## [React Native: A retrospective from the mobile-engineering team at Udacity](https://engineering.udacity.com/react-native-a-retrospective-from-the-mobile-engineering-team-at-udacity-89975d6a8102)

Udacity也放弃了RN,看了整片博客,感觉是他们可能只是为了凑个热度.

首先他们放弃的最主要原因是他们就一个Feature用了RN,而且当时也是因为它很独立,然后尝试了RN,现在这个Feature不用了,就删掉了.

然后他们的Android Dev细数了RN的十宗罪,但是都是我们大概能想到的.

比如实际上两个平台尤其android还是需要通过改code来修bug或者特殊需求,RN与native互通麻烦,RN导致CI编译长,包增大,启动变慢,RN文档欠缺,代码变更太快,自身bug多,不同设备不同表现,遇到问题常常需要改源码等等等等.

而之所以当初要使用,主要原因是为了省事,可惜后来发现还是费事了,所以以后再也不会考虑RN了.


## [Automated testing will set your engineering team free ](https://medium.com/azimolabs/automated-testing-will-set-your-engineering-team-free-a89467c40731)

作者介绍了他们的应用是如何保证质量的,答案就是写测试,一般feature手工只测一遍,剩下就得自动化测试.

测试他们分为三类, UT是开发来写, 基本覆盖每一个函数,集成测试用Robolectric是QA负责来写,End to End test使用Espresso,Instrument Test等,整个跑下来不能超过三小时.


## [Compiler-based security mitigations in Android P ](https://android-developers.googleblog.com/2018/06/compiler-based-security-mitigations-in.html?linkId=53786461)

从Android P开始基于Clang编译器做了很多优化,提升了稳定性,降低了被攻击的风险.

如CFI (Control Flow Integrity
) 技术,主要是增加了虚函数指针偏移指向地址的检查,如果发现指向非法地址就会终止编译.

IOS (Integer Overflow Sanitization
) 技术会检测有符号或者无符号的Integer在算法中溢出的问题,并且优化后运用到了一些库的编译中,如libui, libnl,libexif等...


## [7 steps to implement Paging library in Android ](https://proandroiddev.com/8-steps-to-implement-paging-library-in-android-d02500f7fffe)

文章介绍了如何使用Google Architecture Component里面的Page来实现分页加载.

总共有七步,关键是实现PageKeyedDataSource接口封装MutableLiveData. 实现DataSource.Factory将DataSource封装成Factory.

然后就可以通过LivePagedListBuilder来生成数据了.


## [A New Universal Music Player ](https://android-developers.googleblog.com/2018/06/a-new-universal-music-player.html?linkId=53783436)

鉴于大家对Google的Universal Music Player的喜爱,推出V2版,采用Kotlin与MVVM的架构,但是还有一些小功能没有加进来.


## [Publishing your library to jCenter from Android Studio ](https://android.jlelse.eu/publishing-your-android-kotlin-or-java-library-to-jcenter-from-android-studio-1b24977fe450)

作者在网上找如何上传jcenter,找了很多说法不一的,所以一怒之下自己写了一篇.


## [Maintainable Architecture – Five Day Forecast Data Layer ](https://blog.stylingandroid.com/maintainable-architecture-five-day-forecast-data-layer/)

作者通过实现一个天气预报软件的数据库,来介绍如何设计以及使用正确的方案来确保软件架构正确与维护的成本.

PS:使用的是Room.


## LIBRARIES & CODE


## [RecyclerView-FastScroller ](https://github.com/quiph/RecyclerView-FastScroller)

Kotlin写的快速滑动的滚动条,按照首字母.


## [vector-analog-clock ](https://github.com/TurkiTAK/vector-analog-clock)

Vector实现的石英表,适配各种屏幕.


## [androme ](https://github.com/anpham6/androme)

可以将带JS的HTML5页面转换为多个Android Layout.


## [android-UniversalMusicPlayer ](https://github.com/googlesamples/android-UniversalMusicPlayer)

Google重写的UAMP

## [LazyDatePicker. ](https://github.com/lopspower/LazyDatePicker)

替代系统DatePicker的一个第三方组件.
