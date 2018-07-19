---
title: 'Android Weekly Issue #315'
date: 2018-06-24 15:00:01
tags: [Android Weekly,Priority Bucket,Cloud IoT Core,ML Kit,WorkManager]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #315
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1531988402/315_bdstoc.jpg
---

June 24th, 2018

## [Android Weekly Issue #315](http://androidweekly.net/issues/issue-315)

本篇内容包括,Android P的优先级队列管理,Yelp性能测试系列最后一篇帧率监控,以及近期比较火的Airbnb下一步移动战略,还包含连接Clould IoT Core的Library介绍,ML Kit文字识别,与如何从AndroidJob迁移到WorkManager.还有一篇如何搭建面试机器人的介绍,以及尚在早期的Kotlin Native跨平台数据库的概况等.


<!--more-->

## ARTICLES & TUTORIALS


## [Exploring Android P: Priority Buckets ](https://medium.com/google-developer-experts/exploring-android-p-priority-buckets-d34d12059d36)

这次Google IO Android P推出了Priority Buckets来提供更好的电池管理.

通过使用频率,将App分为`Active`,`Working Set`,`Frequent`,`Rare`,`Never`几个级别, 不同级别会对Jobs还有Alarms有不同的最大延时.

分组是动态改变的.


## [My checklist for fixing build issues ](https://medium.com/@mikewolfson/android-studio-is-borked-my-checklist-for-fixing-build-issues-e41a9dd8cba8)

作者的AndroidTest里面Import都无法识别了,提示`Cannot find symbol`,解决办法是清掉一些缓存.

包括本地Gradle,Project idea与gradle,以及AS的Cache...而后解决了.


## [Understanding the emotions of users through NLP ](https://medium.com/azimolabs/understanding-the-emotions-of-users-through-natural-language-processing-4535ff90f50b)

作者讲解了通过Firebase functions platform + Google Cloud Natural搭建一个App的客服系统.


## [Introducing Billingx ](https://ryanharter.com/blog/introducing-billingx/)

Google唯一渠道实现App内购买的API就是Billing Library,但是对于测试者来说使用起来很麻烦,而且直到2018 Google I/O也没有新的动作.

之前也分享过一个叫做[Register](https://github.com/NYTimes/Register),通过Mock,实现了本地的测试,但是会在Production Apk中引入没必要的代码.

作者自己封装了一个叫[BillingX](https://github.com/pixiteapps/billingx)的库,提供了空实现给Production版,通过`releaseImplementation`引入.


## [Text Recognition with ML Kit ](https://www.raywenderlich.com/197292/text-recognition-with-ml-kit)

介绍了这次ML Kit中的云端文字识别API,通过Firebase服务Enable,可以实现图片中文字的` in-cloud text recognition`,不想掏钱可以试用哦,选择Blaze Plan按需缴费,头1000次请求免费的.


## [Publishing your Android, Kotlin or Java library to mavenCentral ](https://medium.com/@vanniktech/publishing-your-android-kotlin-or-java-library-to-mavencentral-e22f343b9659)

大家平时常用的[Chris Banes](https://github.com/chrisbanes/gradle-mvn-push)的脚本要迁移到Kotlin上还要做一些改动,作者自己弄了一份,并且把它做成了plugin,只需要加到自己的plugin dependencies里面就行了.


## [Performance Improvements for Search on The Yelp Android App - Part 3 ](https://engineeringblog.yelp.com/2018/06/android-search-perf-improvements-part-3.html)

本篇为系列文章的最后一部分,主要介绍了CI上面的Performance如何检测系统帧率变化.

Yelp通过[FrameMetrics API](https://developer.android.com/reference/android/view/FrameMetrics.html)来获取帧率信息,低于16ms就是快帧,高于就是慢帧,他们会对装有信息的JSON进行分析,其中dopped frame会列出各个部分所消耗的时间.

最后又总结了整个性能提升过程中所采取的措施,除去这节的Performance Monitoring来防止Regression导致的问题,还包括之前的减少主线程工作,异步inflating layout,对搜索结果view model的caching等.


## [Android Things client library for Google Cloud IoT Core ](https://android-developers.googleblog.com/2018/06/android-things-client-library-for.html?linkId=53347194)

为广大IoT爱好者带来福音,client library提供了硬件设备轻松连接google Cloud IoT Core,通过几行代码,便可以轻松的上传传感器信息到云端进行控制.

```
implementation 'com.google.android.things:cloud-iot-core:1.0.0'
```

由于硬件设备所在的环境多变,library还提供了很多错误处理机制,数据信息缓存等.


## [How to Migrate from Android-Job to WorkManager ](https://articles.caster.io/android/how-to-migrate-from-android-job-to-workmanager/)

作者以前用的是Evernote搞的AndroidJob,本篇介绍了他如何迁移到google最新的WorkManager上面.

基本使用方法跟AndroidJob类似,Woker通过`OneTimeWorkRequestBuilder`去build单发事件,通过`PeriodicWorkRequestBuilder`去build周期性事件,通过`setInputData`可以添加一些数据,通过`setConstraints`添加约束,如网络要求等,通过`WorkManager.getInstance().enqueue`讲Work加入队列.

Work触发时会执行`doWork`方法, 返回值`Worker.Result.SUCCESS`表示成功,类对象`inputData`可以获取传进来的数据.


## [What’s Next for Mobile at Airbnb ](https://medium.com/airbnb-engineering/whats-next-for-mobile-at-airbnb-5e71618576ab)

作者介绍了在Airbnb放弃RN之后下一步要走的路线.

- 通过[DSL](https://airbnb.design/building-a-visual-language/)定义跨平台统一的设计语言,实现`Server-Driven Rendering`,通过自己开发的基于Sketch的设计软件[Lona](https://github.com/airbnb/Lona/)做到一套设计生成不同平台的代码.当然这一切都是在Build的时候做的.
- 基于之前的[Epoxy](https://github.com/airbnb/epoxy),推出新的`MvRx`,并且支持Android与iOS,Android上是对RecyclerView的封装,可以更方便的实现复杂List的渲染.
- 通过[gradle product flavors](https://developer.android.com/studio/build/build-variants#product-flavors)来实现编译速度的提升,只去下载自己关心的Module.


## [SQLite on Kotlin/Native ](https://medium.com/@kpgalligan/sqlite-on-kotlin-native-9bcf47854cae)

[Knarch](https://github.com/touchlab/knarch.db/tree/96bb6a2e370bd90d2b42d46b23ba1b2e74b0d4ff), Kotlin跨平台数据库的一个非常早起的版本.很多地方还不成熟,尤其是对Android的支持,如何架构还在讨论总,在使用过程中,多线程也是个问题.


## LIBRARIES & CODE


## [SaveState ](https://github.com/PrototypeZ/SaveState)

保存状态的一个库,相较于Icepick支持Kotlin.


## [morph-bottom-navigation ](https://github.com/tommybuonomo/morph-bottom-navigation)

基于Bottom Navigation的一个库,有比较好看的动画效果.


## [gradle-maven-publish-plugin ](https://github.com/vanniktech/gradle-maven-publish-plugin)

支持kotlin的maven发布插件.
