---
title: 'Android Weekly Issue #280'
date: 2017-10-25 08:47:11
tags: [Clean Architecture,Kotlin Migration,RxJava,Android Testing,MVVM,Kotlin]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #280
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1509324637/280_kutqb9.jpg
---

October 22nd, 2017

## [Android Weekly Issue #280](http://androidweekly.net/issues/issue-280)

本期主要内容包含几篇关于迁移kotlin以及kotlin帮助提升test与其他实践的文章,还包含迁移过程遇到的一些坑的总结,以及为什么要迁移的方法论.还包含Clean Architecture的HighLevel分析,以及为何要坚持Native App开发的讨论分析.

比较有意思的是开启文本大小测试模式,文字都变成了火星文,帮助你测试多语言环境truncate的问题.

Lib部分有一个比较炫酷的开源App, Ribble, 可以作为学习Modern Android开发的playground.

<!--more-->

## ARTICLES & TUTORIALS

## [Smooth your migration to Kotlin ](https://fernandocejas.com/2017/10/20/smooth-your-migration-to-kotlin/)

介绍了为何以及如何一步一步让整个团队迁移到kotlin上来,主要是方法论,喜欢理论的可以看看.


## [Thoughts on Clean Architecture ](https://android.jlelse.eu/thoughts-on-clean-architecture-b8449d9d02df)

从HighLevel的角度介绍了CleanArchitecture,包括一些作者的Concept,比较抽象.


## [Why we are not cross-platform developers ](https://medium.com/@pixplicity/why-we-are-not-cross-platform-developers-fd7ef70e976d)

从各个方面介绍了想象中的只写一次多个平台公用的思想存在的问题,文章覆盖了Hybrid,React,还有像Xamarin这种针对UI封装的特殊的.


## [Engineering NullAway ](http://eng.uber.com/nullaway/)

介绍了Uber的一个NPE check工具,检测代码NullPointerException的位置,属于静态检测,需要使用@Nullable与@NotNull进行声明.


## [Testing Concurrency in RxJava ](https://proandroiddev.com/testing-concurrency-in-rxjava-831804a9e526)

文章介绍了对于通过delay模拟网络延迟该如何进行测试,通过引入TestScheduler可以轻松的模拟时间流逝,不用真实的去等.


## [How Kotlin’s “@Deprecated” Relieves Pain of Colossal Refactoring? ](https://hackernoon.com/how-kotlins-deprecated-relieves-pain-of-colossal-refactoring-8577545aaed)

文章介绍了kotlin的`@Deprecated`注解的强大之处,相较于Java,它还可以指定`ReplaceWith`, 如果使用了已经被废弃的方法,可以一键切换.

```
@Deprecated("It's deperacated",
        replaceWith = ReplaceWith("xxx","yyy"))
```

## [Removing Logic from Your Views with BindingAdapters ](http://www.donnfelker.com/android-mvvm-with-databinding-removing-logic-from-your-views-with-bindingadapters/)

文章简单的介绍了MVVM应该讲xml里面的逻辑挪到BindAdapter里面去,这样可以直接调用来做简单的Espresso测试,并且对于VM相关的逻辑可以做UT,一定要将VM与Android Framework隔离.


## [Testing your Room DAO classes ](https://medium.com/exploring-android/android-architecture-components-testing-your-room-dao-classes-e06e1c9a1535)

文章介绍了如何测试`Room DAO`,可以使用`inMemoryDatabaseBuilder`来初始化一个Memeroy的数据库,好处是每一个testcase完就会删掉.
比较有特点的是一个测试思想.

测试`DAO.insert`的时候利用`DAO.get.isNotEmpty`,而不是通过raw的query来做.

测试`DAO.get`的时候,则是通过查看order是否正确来判定是否正确.


## [Odd things to look out for when converting code to Kotlin ](https://medium.com/@Zhuinden/odd-things-to-look-out-for-when-converting-code-to-kotlin-a00b6239828c)

文章介绍了从Java一键转换到Kotlin遇到的各种奇怪的事情,有不少是因为占用了kotlin的关键字导致的,还有一些是不符合Kotlin的语法,如接口转换为object,但是不支持将lambda直接传入,必须显示创建,或者可手动的讲接口声明为typealias的lambda也可以解决.
```
typealias KolinInterface<T> = (T) -> Unit
```
总共有近20条,还是值得看一下的.


## [Testing Android Apps with Pseudolocalization ](https://www.thedroidsonroids.com/blog/testing-android-apps-pseudolocalization)

在gradle里面开启`pseudoLocalesEnabled true`,再把语言改成` English (XA)`, 你就可以看到你所有text都变成了很奇怪的样子,如`[NŃEĘ]`,这样你就可以看看上下左右是否超出了边界.


## [Getting rid of boilerplate with Kotlin Android Extensions ](https://www.kotlindevelopment.com/kotlin_android_extensions_eliminate_findviewbyid/)

文章介绍了用kotlin extension替换吊`findViewById`或者`Butterknife`,而且kotlin有lazy加载与cache,可以让整个过程更快,但是只对Activity,Fragment还有自定义View有效,但是ViewHolder不支持,文章提供了几种办法,不过都挺坑的.


## [Open-sourcing RacerD: Fast static race detection at scale ](https://code.facebook.com/posts/293371094514305)

文章介绍了一个线程安全分析工具RaceD,可以分析代码中标记为@ThreadSafe的类是否真的safe.


## [Kotlin, you’ve broken my heart! ](https://medium.com/@krzychukosobudzki/kotlin-youve-broken-my-heart-ea0e8f991b70)

文章介绍了作者的一个教训,kotlin对于map有一个内置方法叫做getOrDefault,但是忽略了上面写的依赖系统,需要在Jdk1.8上才能用.结果就UT可以过,android就崩了...


## LIBRARIES & CODE

## [Ribble ](https://github.com/armcha/Ribble)

一个使用了各种流行框架,Material Design,等等等的APP,非常fancy,值得学习.

## [ok-gradle ](https://github.com/scana/ok-gradle)

一个类似gradle的spotlight,可以cmd+shift+a呼出直接查库

## [actions-on-google-kotlin ](https://github.com/TicketmasterMobileStudio/actions-on-google-kotlin)

Actions on Google的一个kotlin port
