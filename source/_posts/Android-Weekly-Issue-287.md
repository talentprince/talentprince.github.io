---
title: 'Android Weekly Issue #287'
date: 2017-12-17 14:49:48
tags: [Android Weekly,Clean Architecture,Kotlin,Android Database,RSA,ASE,Android Security]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #287
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1513493321/287_h4ejd2.jpg
---

December 10th, 2017

## [Android Weekly Issue #287](http://androidweekly.net/issues/issue-287)

圣诞节快要来了,小编也偷懒了,本期内容包括如何通过AS添加网络字体库,以及如何使用Dagger.Android等实现Clean Architecture,还包含一篇Android安全系列的文章,介绍如何实现长数据的加密.

关于Kotlin的技巧以及系统数据库的替代品,也有各有两篇系列文章,值得一看.

<!--more-->

## ARTICLES & TUTORIALS


## [The Android Developer’s Guide to Better Typography ](https://medium.com/google-design/the-android-developers-guide-to-better-typography-97e11bb0e261)

文章介绍了如何利用Androis Studio为你的View添加新的Font.

在AS3.0中,可以通过从Layout的的Design Tab中的`Component Tree`里选择你想要添加font的View,然后在右侧的Attribute里面找到`font family`,然后就可以添加自己想要的font,还可以是通过Google Font Provider来的网络字体.

其会在你的`res/font`下生成字体配置文件,这里可以通过`app:fontProviderFetchStrategy`设置加载策略,如`block`,可以避免没下载完成字体被切.

在生成的配置文件中,还有一个比较重要的就是,`res/values`下面的`preloaded_fonts`,定义了会加载的字体名字.


## [Clean Architecture - Kotlin, Dagger 2, RxJava, MVVM and Unit Testing ](https://medium.com/@rahul.singh/clean-architecture-kotlin-dagger-2-rxjava-mvvm-and-unit-testing-dc05dcdf3ce6)

文章介绍了Clean Architecture的结构以及如何对ViewModel进行测试(mock Repository).

值得一提的是,这里大概介绍了Dagger2.Android,帮助我们轻松的进行DI,通过`ContributesAndroidInjector`将zhiding的Activity生成`AndroidInjector`,来帮助我们更轻松的对Activity以及Fragment进行注入.

关于Dagger2.Android详情也参考之前的[文章](http://talentprince.github.io/2017/09/30/Advanced-Dagger2-Skills/).


## [Secure data in Android — Encrypting Large Data ](https://proandroiddev.com/secure-data-in-android-encrypting-large-data-dda256a55b36)

文章继续介绍如何对大量数据的加密,在Android里非对称加密`RSA`对加密长度有限制,并且只能在Android18以上可以设置,最大`4096`bit,显然不符合我们的要求.

对于Android M以上,系统的KeyStore可以支持生成对称加密秘钥了,所以我们可以直接通过生成`ASE`秘钥对大量数据进行对称加密.

但是对于M以下的系统,我们只能通过Java的Key Provider生成一个秘钥文件,存放在某处,这样不是非常安全.解决方案就是,通过`RSA`对生成的`ASE`秘钥进行加密后在放在储存器中,解密时先通过`RSA`对秘钥进行解密,然后再进而解密我们的数据. (因为KeyStore是支持生成`RSA`非对称秘钥,并随着App的拆卸而销毁的.)


## [Kotlin Playground ](https://medium.com/@jcmsalves/kotlin-playground-aab8be8ac432)

作者通过一个Demo记录自己的Kotlin学习之旅,并附上了一些自己的学习资料,包含视频资料哦.


## [6 magic sugars that can make your Kotlin codebase happier - Part 1 ](https://medium.com/grand-parade/6-magic-sugars-that-can-make-your-kotlin-codebase-happier-part-1-ceee3c2bc9d3)

一个系列文章,介绍Kotlin的牛逼之处.

本篇介绍了通过`seal class`来将不同的类型关联起来实现更强功能的Enum,然后配合`when`来方便swich逻辑.

还可以将几种`seal class`合成Pair(`left to right`)或者Triple(`left to mid to right`)来穿给`when`做条件筛选.


## [6 magic sugars that can make your Kotlin codebase happier - Part 2 ](https://medium.com/grand-parade/6-magic-sugars-that-can-make-your-kotlin-codebase-happier-part-2-843bf096fa45)

系列文章的第二篇,涉及到如何用`with`来简化你的代码,并且还介绍了如何封装一个具备类型检测的`withCorrectType`,这里涉及到一个kotlin的知识点,我们在很早之前的Weekly有提到:

就是通过`inline`与`reified`让泛型参数可以在运行时被`is`检测,否则泛型类型会被摸去.


## [Lessons from my first multiplatform Kotlin project ](https://medium.com/@marcinmoskala/lessons-from-my-first-multiplatform-kotlin-project-d4e311f15874)

Kotlin 1.2推出了多平台功能的支持,其提出了将一个程序分为`common module`, `platform module`与`regular module`三层,并且还提供了`expect` 与 `actual`的新特性,可以在common层定义expect类与方法,并通过在platform层实现.

其相较于接口好的一点是,对于`expect`类用起来跟一个完整的类一样,可以有构造,可以实例化,可以调用所定义的expect方法.

文章介绍了自己的思路,并指出了MVP在这种跨平台代码中的妙用,让逻辑代码可以共用.


## [Tuning your SQLite with SQLDelight & SQLBrite: Part 1 ](https://medium.com/lalafo-engineering/tuning-your-sqlite-with-sqldelight-sqlbrite-part-1-9568543fe9af)

文章介绍了`SQLBrite`相较于系统ContentProvider好用之处,主要是其实对系统API的Rx封装,使得整个数据库的操作做到React.

## [Tuning Your SQLite With SQLDelight & SQLBrite: Part 2 ](https://medium.com/lalafo-engineering/tuning-your-sqlite-with-sqldelight-sqlbrite-part-2-11e59c5a3442)

文章介绍了`SQLDelight`,它是一个通过gradle plugin将`.sq`文件生成Java代码的db.

如:
```
getImagesById:
SELECT *
FROM images
WHERE ad_id = ?
ORDER BY _id ASC;
```
就可以生成对应`getImagesById(long ad_id)`的Java代码.


## LIBRARIES & CODE


## [Transitioner ](https://github.com/dev-labs-bg/transitioner)

定义两个使用`ConstraintLayout`封装的View,包装成一个`transitioner`,实现类似Shared Element的那种动画效果.

## [HighLite ](https://github.com/jeppeman/HighLite)

高度基于Annotation的一个Android DB,通过`@SQLiteDatabaseDescriptor`,`@SQLiteTable`,`@SQLiteColumn`,`@PrimaryKey`等为主要注解,并且通过`@JvmField`支持Kotlin.


## [android-disposebag ](https://github.com/kizitonwose/android-disposebag)

通过绑定Activity或者Fragment的生命周期,初始化一个`DisposeBag`,将Rx的析构交给它做.支持RxJava/RxKotlin.


## [EasyAdapter ](https://github.com/MaksTuev/EasyAdapter)

简化使用RecyclerView实现多类型复杂的List.


## [CounterView ](https://github.com/AnkitKiet/CounterView)

简单的安卓计数器View控件.