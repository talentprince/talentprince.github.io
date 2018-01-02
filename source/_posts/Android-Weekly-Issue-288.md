---
title: 'Android Weekly Issue #288'
date: 2017-12-31 11:35:55
tags: [Android Weekly,Kotlin DSL,Kotlin Lambda,Lifecycle,Multi-platform Architecture]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #288
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1514691333/288_fuowvr.jpg
---

December 17th, 2017

## [Android Weekly Issue #288](http://androidweekly.net/issues/issue-288)

本期内容主要包括介绍Kotlin DSL使用kotlin来写gradle,如何组织Session,以及MVP如何通过LifeCycler来简化,如何让多个presenter之间相互交互.
特别推荐的有如何使用kotlin优化多方法的接口,有多达四种方法,是学习kotlin lambda优化的好机会,以及如何使用kotlin架构跨平台应用.

代码部分有趣的是一个可以画dagger依赖关系的库,但还支持的不完善,以及西班牙人封装Espresso的一个库,叫`Barista`.


<!--more-->

## ARTICLES & TUTORIALS


## [How to develop image gallery app in Android using Kotlin ](http://developine.com/develop-android-image-gallery-app-kotlin-with-source-code/)

作者分析了自己用kotlin写的一个相册应用,里面用了很多kotlin相关的知识.


## [The Power of Gradle Kotlin DSL ](https://blog.simon-wirtz.de/the-power-of-gradle-kotlin-dsl/)

文章介绍了作者将本来`groovy`的gradle.build改成了使用`kotlin.dsl`写的gradle.build.kts,做到用kotlin来写gradle,是不是很嗨森.


## [How to host a conference within your team ](https://medium.com/car2godevs/how-to-host-a-conference-within-your-team-2540f8df10d7)

教你如何组织公司内部的session,首先通过表单收集话题,然后对话题进行投票,然后根据话题的多少指定break,呵呵,感觉外国人真的是很天真烂漫~~~


## [Using Architecture Components with Firebase (part 1) ](https://firebase.googleblog.com/2017/12/using-android-architecture-components.html)

文章用`Firebase Realtime`来讲解如何通过`Architecture Component`将程序重构成成MVVM架构,将Firebase数据库封装成`LiveData`并放在`ViewModel`里,这样生命周期与activity绑定,也利于测试.


## [Task Stack ](https://blog.stylingandroid.com/task-stack/)

文章介绍了通过解决通过Notification打开某activity,但后退直接退出而不是回到上一级的用户体验问题.
即通过`TaskStackBuilder`来创建具有parent activity的pending intent.
具体可以看代码,作者没有提到,如果在manifest里面声明了,还可以使用`NavUtils`来简化一些流程.


## [MVP & Lifecycles & Dispatchers Oh My! ](https://medium.com/s23nyc-tech/mvp-lifecycles-dispatchers-oh-my-19eda37a1a52)

文章介绍了MVP的实际应用,我觉得除了大家所熟知的一些基本思路,这里有两点值得说道说道.

一是通过Presenter订阅`Lifecycle`,可以自动实现onAttach与onDetach.

二是实现一个`Dispatcher`,为所有presenter持有.内部是一个`PublishSubject`,不同presenter的方法通过`ofType`监听自己关注的的`state`,不同模块的presenter之间通过`publish state`相互通信.


## [Listeners with several functions in Kotlin ](https://antonioleiva.com/listeners-several-functions-kotlin/)

文章介绍了如何使用好的方法在kotlin里处理多方法的接口.并以大家熟知的Animator.Listener举例子.

这个四个方法的接口,如果我们只希望实现一个,最直接的方法就是使用`AnimatorListenerAdapter`,但是它是一个抽象类,继承该类就不能继承别的类了.

第一种解决方法就是kotlin的interface支持写code,所以自定义的没必要是一个抽象类而还是一个接口,所有的方法赋值`= Unit`即可.

第二种方法是使用Exstension,给`ViewPropertyAnimator`写扩展,参数是`(Animator)->Unit`,而在内部实现`AnimatorListnerAdapter`,然后将其回调中的参数通过我们扩展方法的Lambda返回,由于内部套用回调,为了实现整体扩展方法的inline,参数`(Animator)->Unit`需要加`crossinline`标识.

```
inline fun ViewPropertyAnimator.onAnimationEnd(crossinline continuation: (Animator) -> Unit) {...}
```

第三种方法是基于第二种,传入四个lambda参数(`(Animator)->Unit`),使用了`named argument`特性,并都赋默认值`{}`,这样想实现哪个,就指定名字即可.

第四种使用了`Lambda with Receiver`,或者交`Extension function lambda`,最kotlin的一种封装,扩展方法setListener的参数是`AnimListenerHelper.() -> Unit`,而AnimLisnerHelper本身实现了所有Animator接口的四个方法,并提供四个方法将`(Animator)->Unit`作为参数保存在Helper类内,代理给之前override Animator的四个回调.


```
view.animate()
        .setListener {
            onAnimationStart {...}
            onAnimationEnd {...}
        }
```


## [Architecture for Multiplatform native development in Kotlin ](https://blog.kotlin-academy.com/architecture-for-multiplatform-development-in-kotlin-cc770f4abdfd)

文章介绍了使用kotlin实现跨平台应用的架构思想,通过Kotlin/Jvm,Kotlin/JS,以及正在beta研发阶段的Kotlin/Native,可以实现横跨backend到所有前端设备的庞大系统,而整体又基于MVP的思想.

由上之下大概的分层是

common (DataModel)

common-platform (平台相关DomainModel, 使用Kotlin Multiplatform的`require/actual`)

common-client (MVP抽象)

common-client-repo-platform (平台相关的Repo)

views (不同平台的view实现)

对于iOS与Android,又会在view之上抽象出来一层为`watch`,`tv`,`car`,`phone`等公用.叫做`Common Android/iOS Elements`.

是不是很六百~.


## [Testing RxJava code made easy ](https://medium.com/@vanniktech/testing-rxjava-code-made-easy-4cc32450fc9a)

RxJava2提供了test observable,可以通过`Observable.test`进行各种assert.

## LIBRARIES & CODE

## [daggraph ](https://github.com/dvdciri/daggraph)

画dagger依赖图的,但是还有不少不支持的,如`Construtor Inject`等等.在开发中.


## [Cipher.so ](https://github.com/MEiDIK/Cipher.so)

很好用的native加密库.


## [Barista ](https://github.com/SchibstedSpain/Barista)

封装了Espresso,简化了很多API的使用.


## [artist ](https://github.com/uber/artist)

可以给View里面添加方法的一个插件,如添加一些判断Visibility的方法.


## [TimeLineView ](https://github.com/po10cio/TimeLineView)

一个基于`ConstraintLayout`与`RecyclerView`显示时间线的View.


## [Kotshi ](https://github.com/ansman/kotshi)

给JSON解析库Moshi写的支持Kotlin的库,不过我查了一下,Moshi已经有自己的了,叫`moshi-kotlin`.
