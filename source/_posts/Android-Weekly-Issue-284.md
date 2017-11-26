---
title: 'Android Weekly Issue #284'
date: 2017-11-26 13:02:46
tags: [Android Weekly,Android Things,Proguard,OpenSTF]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #284
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1511672886/284_bs7nou.jpg
---

November 19th, 2017

## [Android Weekly Issue #284](http://androidweekly.net/issues/issue-284)

本期内容丰富.有趣的有如何搭建真机测试平台,Proguard里面各类keep的区别,如何运行时获得泛型类型,Android的Internal Storage到底是什么,以及Android Things的一篇文章.

代码部分有介绍了一个twiiter的序列化库,还有个比较炫酷的圆形Menu可以应用到自己项目中去.


<!--more-->

## ARTICLES & TUTORIALS

## [The art of staging a rollout ](https://medium.com/bleeding-edge/the-art-of-staging-a-rollout-8e203b337b75)

文章介绍了Google Play Store分阶段发布的特性，可以帮助你去降低发布风险。
简单来说就是可以控制用户升级到新版本的比例，遇到问题可以发新版覆盖，更好的是，HotFix版本的发布范围是取自之前已经收到更新的用户的，帮助你去观察问题是否已经修复，而尽量的不会去影响其他未收到更新的用户。


## [Getting your Android app ready for Autofill ](https://android-developers.googleblog.com/2017/11/getting-your-android-app-ready-for.html)

文章介绍了如何更好的支持Android O最新的AutoFill功能，如添加hint为其分类，设置一些attribute标注其需要或者不需要autofill,因为默认都是开启的...

```
setImportantForAutofill(IMPORTANT_FOR_AUTOFILL_NO)
```


## [Android Device Farm at Mercari ](https://medium.com/mercari-engineering/android-device-farm-at-mercari-3197237df0e1)

文章介绍了如何通过STF(Smartphone Test Farm)与GCP(Google Cloud Platform)来搭建公网测试平台.

由于没有公网静态地址,所以这里采用微服务平台来搭建.

手机与Local的机器adb连接, Local与GCP通过SSH Tunnel连接,使用autossh保活,GCP上跑STF.如果有多个区域的Local实体机,可以通过nginx均衡.


## [The Storage Situation: Internal Storage ](https://commonsware.com/blog/2017/11/13/storage-situation-internal-storage.html)

介绍了`Internal Storage`的历史渊源以及他的消失.

最早(1.x-2.x)实际上是分Internal跟外挂的SdCard,内部储存很小只有几十M,我记得以前得通过脚本开启app2sdcard,来将程序数据挪到sd卡,当然2.3开放了这个功能.

3.x-4.4之前,内部储存越来越大,并且也可以挂载非sd卡的储存空间,我们称之为固有ROM.

4.4以后把所有的内部储存分成了Internal Storage与External Storage,其实主要是读写权限的区别,并限制了挂载储存(sdcard)的读写权限.

8.0以后取消了Internal Storage的叫法,内部储存统一叫Storage


## [Android Color Management: What Developers and Designers Need to Know ](https://medium.com/google-design/android-color-management-what-developers-and-designers-need-to-know-4fdd8054557e)

介绍了新版本手机支持`sRGB`也就是Wide color的情况, 可以通过Manifect设置Activity是否开启`wideColorGamut`模式,还可以通过`values-widecg`文件夹指定色值,在支持Wider色的机器上使用.


## [Distinguishing between the different ProGuard “-keep” directives ](https://jebware.com/blog/?p=418)

文章以图表的方式介绍了Proguard里面不同的keep之间的区别.

总结一下就是加了`names`就会shrink,加了`member`就是只针对于member,加了`with`就是必须`{}`里面的member都有的时候才会执行.


## [Building a distributed MIDI Controller with Android Things and Nearby API ](https://medium.com/@tomaszrykala/building-a-distributed-midi-controller-with-android-things-and-nearby-api-7011f1f3189c)

Android Things相关的分享,作者使用MIDI协议,结合Google的Nearby API,相当于手机与Things之间通过Nearby通信,然后Things将MIDI数据传给电脑上的Software.


## [Runtime generics in an erasure world ](http://helw.net/2017/11/09/runtime-generics-in-an-erasure-world/)

文章讲了运行时获得泛型信息,并通过这个办法,Gson的TokenType将类型信息交给反序列化使用.

大致原理:

对于`InnerType#Internal<String>`,通过其`GenericSuperclass`可以获得很多信息.

- `OwnerType`是`InnerType`
- `RawType`是`InnerType$Internal`
- `ActualTypeArguments`是`java.lang.String`

## [Introducing Serial: Improved Data Serialization on Android ](https://blog.twitter.com/engineering/en_us/topics/open-source/2017/introducing-serial.html)

文章介绍了一个更牛的序列化工具`Serial`.

它的特点很多,如更快的`Perfermance`,更容易`Debug`,更好的`backward`兼容性,以及加入已有系统的`flexibility`.


## LIBRARIES & CODE


## [CircleMenu ](https://github.com/Ramotion/circle-menu-android)

一个UI空间,可以点开一个圆圈类型的Menu.


## [tivi ](https://github.com/chrisbanes/tivi)

连接`Trakt.tv`的一个App,非官方,WIP,使用了很多新框架.


## [AutoAdapter ](https://github.com/Zuluft/AutoAdapter)

通过写一写注解来绑定layout与Java,生成RecyclerView的adapter.


## [AndroidSDKPoster ](https://github.com/PGSSoft/AndroidSDKPoster)

包含了android14-27的changelog的poster


## [idea-gradle-dependencies-formatter ](https://github.com/platan/idea-gradle-dependencies-formatter)

一个gradle的format插件,支持包括compile不同格式的切换,还有maven到gradle的转换等等.


## [Serial ](https://github.com/twitter/serial)

Twitter的序列化框架.
