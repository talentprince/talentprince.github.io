---
title: 'Android Weekly Issue #318'
date: 2018-07-15 09:00:01
tags: [Android Weekly,Jetpack,Navigation,AAR,Kotlin DSL,Scope Functions,MVI,DataBinding,Realm]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #318
thumbnailImage: https://res.cloudinary.com/dtn0pkdmg/image/upload/v1532311724/318_jqq6fx.jpg
---
July 15th, 2018

## [Android Weekly Issue #318](http://androidweekly.net/issues/issue-318)

本期内容包括Jetpack NavigationUI的介绍, FAT AAR的讨论, Realm迁移到Kotlin的方案,以及如何通过MVI+DataBinding来写程序.还包含DSL改造Android Dialog以及Kotlin scope function的详细解读.

<!--more-->

## ARTICLES & TUTORIALS


## [Android Jetpack - NavigationUI ](https://proandroiddev.com/android-jetpack-navigationui-a7c9f17c510e)

文章简单介绍了如何将NavigationView或者BottomNavigation与JetPack提供的Navigation控件连起来,通过定义<navigation> xml来实现自动跳转.


## [Why We Need “fat” AARs for Android Libraries ](https://handstandsam.com/2018/07/13/why-we-need-fat-aars-for-android-libraries/)

Gradle aar不支持打包所有依赖,一般情况下可以使用一个叫做FAT aar的插件,但是已经不更新了.

还可以通过transitive来让gradle下载依赖,但是作者的疑问在于如果依赖也是不公开的就没办法了.

Google已经表示3.3会考虑加这个功能.


## [Maintainable Architecture – UI Layer ](https://blog.stylingandroid.com/maintainable-architecture-five-day-forecast-ui-layer/)

继上次介绍数据层,这篇文章介绍了一个Weather App如何使用MVVM实现UI Layer.


## [Migrating your Realm to Kotlin – Blue Apron Engineering ](https://blog.blueapron.io/migrating-your-realm-to-kotlin-ee0fa5fc29b)

文章介绍了如何将Realm迁移到Kotlin.

- 为了不想让主键为空导致Kotlin nullable变量使用起来需要进行let之类的null check,所以给与初始化值,但是一定要加Migration逻辑保证数据库迁移正确.
- 可以通过一些Helper function加!!来避免Null check
- Extension function包装关键字`in`,因为它是kotlin的关键字.



## [Model-View-Intent & Data Binding ](https://proandroiddev.com/model-view-intent-data-binding-39c7a6a6512f)

作者通过一个登陆页面介绍了如何使用了MVI+DataBinding,适合初步了解MVI的机制以及Reducer的思想.

## [Social Network Integration on Android ](https://www.raywenderlich.com/191933/social-network-integration-on-android)

作者介绍了如何使用诸如FB Twitter这样的social media sdk来实现自己app内的登录与分享.


## [Kotlin Demystified: What are 'scope functions' and why are they special? ](https://medium.com/google-developers/kotlin-demystified-scope-functions-57ca522895b1)

作者介绍了Kotlin里面的scope functions,其实就是let/run/with/apply/also.

作者的总结比较复杂,其实有一个比较简单的图可以清楚了解之间的关系.

如果你需要返回本身,就使用apply或者also,其区别就是使用this或者it.

如果你不需要返回本身,又想要做null判断,那就是用T.run或者let,当然区别也是this或者it.

如果你即不需要返回本身,又不用判断null,那就用with或者run,区别也是this或者it.


## [Seedbank — discover machine learning examples ](https://medium.com/tensorflow/seedbank-discover-machine-learning-examples-2ff894542b57)

可以在线的试一试Machine Learning


## [From Java Builders to Kotlin DSLs ](https://kotlinexpertise.com/java-builders-kotlin-dsls/)

教大家如何把Dialog Builder改造成Kotlin DSL,使用起来非常炫酷.

其实大家也都清楚了,就是通过Extension Function.

具体可以去看看作者的对比,DSL化后使用起来跟CSS一样,花括号套起来就实现了Dialog.

```
draw {
    drawerLayout = xxxx
    onItemClick {
        xxxxx
    }
    onOpen {
        Toast.xxxx
    }
}
```

## LIBRARIES & CODE


## [android-face-detector ](https://github.com/husaynhakeem/android-face-detector)

实时面部识别的lib,使用的Firebase ML kit.


## [UnderlinePageIndicator ](https://github.com/dcampogiani/UnderlinePageIndicator)

类似于系统design的TabLayout那种效果.

## [Seedbank ](https://tools.google.com/seedbank/)

在线测试ML的.
