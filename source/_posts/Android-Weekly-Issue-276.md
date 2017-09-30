---
title: 'Android Weekly Issue #276'
date: 2017-09-30 14:00:48
tags: [Android Weekly, Android, Life Circle, Room Persistence, ObjectBox, Realm, Anko, Android Studio Debug]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #276
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1506751408/download_rarkl4.jpg
---

September 24th, 2017

## [Android Weekly Issue #276](http://androidweekly.net/issues/issue-276)

本期内容包括LifeCycle与Architecture的相关文章,以及新的JSON解析库Moshi的介绍,还有
ConstraintLayout的一些特性,还包括一个加速你Debug的小技巧,喜欢数据库的也不容错过,有介绍Realm,Room,ObjectBox,Anko SQLite等等的对比与分析的文章哦.


<!--more-->

## ARTICLES & TUTORIALS

## [Is your Android Library, Lifecycle-Aware? ](https://android.jlelse.eu/is-your-android-library-lifecycle-aware-127629d32dcc)

还在为必须在固定的生命周期里处理你的init与destroy吗, 文章介绍了通过google最新的LifeCircle库来实现自主监听.

你只需要引入相关依赖.

```
implementation "android.arch.lifecycle:runtime:1.0.0"
annotationProcessor "android.arch.lifecycle:compiler:1.0.0-alpha9-1"
```

然后让你需要关注生命周期的类继承于`LifecycleObserver`.

这样你就可以通过注解来监听生命周期了.

```
@OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
or 
@OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
..and so on
```

当然别忘了需要注册.

```
getLifecycle().addObserver(LifecycleObserver);
与
getLifecycle().removeObserver(LifecycleObserver);

```

## [Creating a Custom Type Adapter for Moshi ](https://medium.com/@emmaguy/creating-a-custom-type-adapter-for-moshi-ae7e1cf469a9)

文章介绍了如何给JSON解析库[Moshi](https://github.com/square/moshi)添加一个自定义的Adapter来完成非1:1的Json2Object的解析, 主要逻辑在于实现了@FromJson的方法.

```
class StageAdapter {
  @FromJson fun fromJson(jsonReader: JsonReader, delegate: JsonAdapter<Stage>): Stage? {
    val value = jsonReader.nextString()
    return if (value.startsWith("in-progress")) Stage.IN_PROGRESS else delegate.fromJsonValue(value)
  }
}
```

把所有的以`in-progress`开头的都转换成`Stage.IN_PROGRESS`.

## [Register: Better In App Billing Testing on Android ](https://open.nytimes.com/register-better-in-app-billing-testing-on-android-73af5fcc36dc)

介绍了开发的一个框架`Register`用来帮助使用`Google In-App Billing`开发应用的人来进行测试, mock掉Server.

原因是Google API必须是内嵌在Play Store上线的应用里, 去订阅一个已经上线的应用, 而且必须使用开发上号来进行购买测试, 并且重复测试也受到限制, 因为取消之前的购买是唯一的方式, 但是需要等很久状态才会被Reset.

## [ConstraintLayout - Guidelines, Barriers, Chains and Groups ](https://riggaroo.co.za/constraintlayout-guidelines-barriers-chains-groups/)

介绍了ContraintLayout的一些新的特性, 包括`Guidelines`, `Barriers`, `Chains` 跟 `Groups`.

## [Repository layer using Room and Dagger 2 ](https://medium.com/@saurabh30pant/repository-layer-using-room-and-dagger-2-android-12d311830fd9)

文章介绍了如何搭建MVP中的M层, 源自Google的MVP套路, `Presenter`持有一个`Repository`,而`Repository`里面持有的对应数据的`DataSource`,用来真正操作数据的`IDUQ`, 而`DataSource`内部是通过`Room Persistence`来实现的.

## [Stop testing on emulators - Access Real Devices ](http://www.kobiton.com/freetrial?utm_source=Android%20Weekly%20Newsletter&utm_medium=Newsletter&utm_campaign=Android%20Weekly%208.6%20Newsletter&utm_term=Emulators&utm_content=Stop%20using%20emulators)

一个云端真机测试的广告.

## [Realm, ObjectBox or Room. Which one is for you? ](https://notes.devlabs.bg/realm-objectbox-or-room-which-one-is-for-you-3a552234fd6e)

文章介绍了根正苗红新秀`Room Persistence`, 成熟稳重`Realm`, 与没什么人听过的`ObjectBox`三个数据库的优劣.

- 如果你关心安装包体积与方法数, 用`Room`
- 如果你关心速度与效率,那么用`ObjectBox`
- 如果你嫌`ObjectBox`不够成熟稳定,那就使用七年老大哥`Realm`

## [Making the most out of Android Studio Debugger ](https://medium.com/@droidchef/making-the-most-out-of-the-android-studio-debugger-61713131d065)

常常调试的时候,你只想看看程序里某个地方运行时候的值,或者想测试新加的一段逻辑是否正常(可以Debug时跑在Evluate的环境里),你完全可以不编译程序,但是默认的情况下你`Ctrl+R`运行却需要等Gradle跑好久,非常恼火.

文章介绍了个小技巧, 自己创建一个Build选项, 让IDE不再去做Gradle编译, 只是单纯运行程序,想知道怎么弄么,进去看看就知道了.

## [Kotlin Testability – Part 2 ](https://blog.stylingandroid.com/kotlin-testability-part-2/)

文章继续上次的套路,将同样的方法运用在一些Android Framework组件上也是可以的.

开启
```
 testOptions {
        unitTests.returnDefaultValues = true
    }
```
后相关Framework的方法也会调用Stub的.

作者最后强调使用`Factory`封装接口是因为有多个需要Mock的对象,如果只有一个第三方组件需要Mock,我们可以直接通过internal primary用于测试与secondary用于production的办法来玩.

## [Building a Guitar Chord Tutor for Actions on Google: Part Two ](https://medium.com/@hitherejoe/building-a-guitar-chord-tutor-for-actions-on-google-part-two-9e0786dfa108)

继续了上次的项目,通过`Actions on Google`实现吉他和弦教程APP

## [Stress-free SQLite with Anko ](https://www.kotlindevelopment.com/anko-sqlite-database/)

推荐了一个SQLite的包装库，纯Kotlin的，叫做[Anko SQLite](https://www.kotlinresources.com/library/anko/), 看了下官网居然还有一个叫Anko Layout的库,使用办法都是Lambda.

```
database.use {
    // 增删改查
}
```

## [From design to android, part 2 ](https://medium.com/@saulmm2/from-design-to-android-part-2-2a6c141547d9)

介绍了由设计入手来做Android动画的思想,介绍了通过Skech+[Shape Shifter](https://shapeshifter.design/)制作Vector图片,然后通过Android的[AnimatedStateListDrawable](https://developer.android.com/reference/android/graphics/drawable/AnimatedStateListDrawable.html)设置给ImageView当资源,最后通过[ImageView#setImageState](https://developer.android.com/reference/android/widget/ImageView.html#setImageState%28int[],%20boolean%29)来控制显示哪个State下的Vector动画

## LIBRARIES & CODE

## [RxLifecycle ](https://github.com/florent37/RxLifecycle)

关于流式控制逻辑在相应的生命周期里完成,防止内存泄露的一个库.

## [Anko ](https://github.com/Kotlin/anko)

一系列Kotlin的框架,让Android开发更简单更快捷,挺厉害的样子.

## [Register ](https://github.com/NYTimes/Register)

帮助测试PlayStore Billing的一个框架

## [Moshi ](https://github.com/square/moshi)

解析JSON的一个库,通过实现一个Adapter可以轻松做原始数据到Model之间的转换.






