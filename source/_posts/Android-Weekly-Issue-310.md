---
title: 'Android Weekly #310'
date: 2018-06-13 09:00:01
tags: [Android Weekly,Google IO,Kotlin Espresso,Navigation Architecture Component]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #310
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1528851719/310_zapsd1.png
---

May 20th, 2018

## [Android Weekly Issue #290](http://androidweekly.net/issues/issue-310)

本期既有本次Google IO对于Play Console的更新简介, 又有数篇对于简化UI Test的工具与方法的介绍,还有JetPack Worker Manager的推介,以及如何仅仅通过Firebase快速搭建一个自己的Instgram小App,当然还有一篇关于Google最新的Navigation Architecture Component的介绍,精彩内容不容错过.

<!--more-->

## ARTICLES & TUTORIALS


## [Rxify : The startWith { MVI } pitfall ](https://medium.com/@ragdroid/rxify-the-startwith-mvi-pitfall-68764ae8946d)

在实现MVI返回State Intent的时候, 可以通过startWith可以添加一些初始化状态,它就类似于concat,将一个新的Observable与之结合,再通过onErrorReturn返回一个错误的状态,保证我们的chain的完备性.


## [Discover everything new in the Google Play Console ](https://android-developers.googleblog.com/2018/05/io-2018-everything-new-in-google-play.html)

最新的GoogleIO更新了Play Console的新功能,有一些还是挺厉害的.

- App Bundle, 就是你的res可以根据设备加载,比如图片资源,对于xxxdpi的设备就下载一份,语言包也类似.
- 提升了质量检测过程, 通过100个内部测试机运行你upload的程序,还有各种report与analysis帮助你分析潜在问题.
- dashboard以及一些数据呈现更加优化,还可以通过acquisition report获得更多信息,如用户从哪里得到的你的app等等.
- 提升用户订阅服务,通过Billing Library可以轻松实现订阅服务,在Subscriptions Center可以轻松管理用户的订阅订单,还有详细的report帮你分析多种订阅之间的时间,地点等等因素的关系.
- 八月份之前必须得Target到26了.


## [Kakao - how to make UI testing great again ](https://medium.com/@ilyalim/kakao-how-to-make-ui-testing-great-again-19972cf13740)

Kakao是一个kotlin的UI测试框架,基于Espresso,可以大幅度简化使用Espresso的过程,通过Lambda以及Kakao内置的一些操作符,完成测试.

如:

```
 screen {
      content { isVisible() }

      textViewSmall {
          isVisible()
          hasAnyText()
      }
}
```


## [Best Practices for Unit Testing in Kotlin ](https://blog.philipphauer.de/best-practices-unit-testing-kotlin/)

作者提了好几条建议来提升Kotlin测试的体验

- 使用JUnit5的`@TestInstance(Lifecycle.PER_CLASS)`来避免出现一些静态field,这些静态field主要是为了保证在整个测试类中只初始化一次,而JUnit4每一个Test都会重新创建新的Class.取而代之的是可以使用`init{}`加`@BeforeEach`.
- 使用JUnit5的`@Nested`包装测试类中一些特殊的模块,使之更加清晰.
- 使用``(backticks)去自定义方法名.
- [AssertJ](http://joel-costigliola.github.io/assertj/)依旧好使.
- 使用[Mokito-Kotlin](https://github.com/nhaarman/mockito-kotlin)跟[MockK](http://mockk.io/)来做mock


## [Exploring Moshi’s Kotlin Code Gen ](https://medium.com/@sweers/exploring-moshis-kotlin-code-gen-dec09d72de5e)

Moshi是一个JSON解析库,文章介绍了1.6版本使用了新的Kotlin代码生成器,更好的处理了类似`Mutablility`,`Nullability`,`in/out`等等的问题,感兴趣可以自己看一下.


## [Life with/without services and WorkManager ](https://medium.com/google-developer-experts/services-the-life-with-without-and-worker-6933111d62a6)

随着Android对于内存管理的越来越严格,首先是26以上Service无法在后台`startService`被限制,而系统提供的`JobScheduler`在23以下有问题,而`JobDispatcher`又需要Google Service...让人甚是苦恼.

值得高兴的是,Google最近推出的强大的JetPack里面的`WorkerManager`将解决这个问题.

它内部融合了`JobScheduer`,`JobDispather`,`AlarmManager`等等,并提供了两种Woker,`OneTime`跟`Periodic`.让后台操作变得非常轻松.


## [A year as Android Engineer ](https://proandroiddev.com/a-year-as-android-engineer-55e2a428dfc8)

文章介绍了作者从一个QA转型为一个Android Dev,并找到新工作的经历,可以当故事看看...


## [The missing migration guide to the Gradle Kotlin DSL ](https://github.com/jnizet/gradle-kotlin-dsl-migration-guide)

一个REAMME,介绍了如何从Gradle迁移到Kotlin DSL,喜欢Kotlin的朋友们可以对照着迁移了,一步两步.


## [Build an Instagram-Like Android App Using Google Firebase ](https://dragosholban.com/2018/05/13/build-an-instagram-like-android-app-using-google-firebase/)

通过Firebase搭建一个类似Instgram的App.

通过Firebase Authenicate来实现登录认证,通过Firebase Storage来实现图片上传,通过Firebase Database来实现数据存储.

自己只需要实现简单的UI,便可以完成简易化的Ins,是不是很开森.


## [A problem like Navigation ](https://medium.com/a-problem-like-maria/a-problem-like-navigation-e9821625a70e)

介绍了Google最新推出的Navigation Archtecture Component,来处理Fragment之间的跳转,通过定义xml中的`action`,就可以完成跳转,通过定义`argument`就可以完成值传递,以及定义`deepLink`,可以轻松支持Deeplink.


## [Pleasant fun painless delightful Espresso testing with Kotlin ](https://medium.com/@Zhuinden/pleasant-fun-painless-delightful-espresso-testing-with-kotlin-3ffeda58d45c)

一个基于Kakao(Kotlin DSL for Espresso)的Kotlin Espresso库,用于简化UI test的书写.

如可以直接通过Id来索引View直接perform操作,而不必在通过`onView`...`withId`...`perform`来搞的很长.


## LIBRARIES & CODE


## [MockK ](http://mockk.io/ANDROID)

kotlin的Mock lib

## [espresso-helper ](https://github.com/Zhuinden/espresso-helper)

Espresso的Kotlin封装

## [Kakao ](https://github.com/agoda-com/Kakao)

上面那个库基于的,也是对于Espresso的封装.

