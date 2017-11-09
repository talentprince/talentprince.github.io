---
title: 'Android Weekly Issue #278'
date: 2017-10-13 13:26:02
tags: [Advanced Kotlin,Room,Paging Library,RxJava,Kotlin Coroutines,Genymotion,Emulator]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #278
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1507874161/278_plbzt0.jpg
---
October 8th, 2017

## [Android Weekly Issue #278](http://androidweekly.net/issues/issue-278)

本周内容主要包括两篇涉及Kotlin高级用法的文章,RXJava解决本地与远端存储的冲突,应该如何选择模拟器来测试,Paging Library的使用等等.

代码部分值得看的是Kolin相关的文章.

<!--more-->

## ARTICLES & TUTORIALS

## [MidiPad – Tricks With Kotlin And Architecture Components](https://blog.stylingandroid.com/midipad-tricks-with-kotlin-and-architecture-components/)

文章介绍了如何利用Kotlin的特性来结合`Google Architecture Component`进行开发.

其中有三个`Trick`

1. 利用lazy {} 来做延迟加载Field变量, 并重写去除了其本身的线程安全,提升速度
2. 利用by实现delagate,把SpinnerAdapter的一部分功能交给ArrayAdapter来实现,自己只需要关注其特有回调即可.
3. 给Activity添加Extension,将`ViewModelProvider.Factory`与其`create`封装其中,使得创建`ViewModel`简单又Lazy.

## [Database with Room using RxJava ](https://medium.com/@alahammad/database-with-room-using-rxjava-764ee6124974)

文章比较简单,主要就是在`Room`的`Dao`定义的时候,`query`的返回值可以直接声明为`Flowable`,这样查询的时候就`react`了,可以轻松的讲查询放在`io`线程,写删改等操作数据库的方法直接用`RxJava`直接封装即可,文章用的`Completable`

##[Create a Clean-Code App ](https://blog.elpassion.com/create-a-clean-code-app-with-kotlin-coroutines-and-android-architecture-components-f533b04b5431)

文章比较前沿,使用`Architecture Component`开发了一个天气应用,亮点在于用Kotlin实验室的`Kotlin Coroutines`取代了`RxJava`,核心逻辑在用`suspendCoroutine`把Retrofit的异步操作转换为`suspend function`.相关`Coroutine`的Doc可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md)找到.

## [12 Practices every Android Development Beginner should know ](https://android.jlelse.eu/12-practices-every-android-beginner-should-know-cd43c3710027)

看名字就知道了,一些小常识,但是这个是Part1,只有10条,感兴趣的可以看看.

## [Check out local storage and RxJava backpressure ](https://medium.freecodecamp.org/want-to-optimize-network-usage-check-out-local-storage-and-rxjava-backpressure-8b91b1db298a)

文章通过使用`RxJava`的`groupBy`与`backpresure`来帮助我们更好的做一个同步的feature,保证远端与Local保持一致.

简单的说就是对同一个item的操作标注一个ID,本地把`Add`与`Remove`操作都记住,然后通过`groupBy`配合`backpresure`每次拿一个最新的Event,完成之后再继续拿,这样中间就会省掉很多重复的操作,大致以最后一次为准.

不过我觉得如果还有`Update`操作,只拿后压后最后一个Event来处理了就不对了...

## [Demystifying Advanced Kotlin Concepts ](https://dev.to/praveenkajla/demystifying-advance-kotlin-concepts-a97)

介绍了不少Kotlin高级理念,如`Lambda Exstension`,`inline`与`infix`function,等等.

## [Genymotion vs Android Emulator ](https://www.plightofbyte.com/android/2017/09/03/genymotion-vs-android-emulator/)

通过数据对比,得出X86 with google api的模拟器性价比最高.

## [Kotlin Function Literal with Receiver](https://tech.io/playgrounds/6973/kotlin-function-literal-with-receiver)

介绍了Kotlin的`Lambda Exstension`特性, 函数本体作为参数被访问,并函数本体内可以访问函数的Receiver.

## [Architecture Components: Paging Library ](http://androidkt.com/paging-library/)

文章介绍了使用`Paging`对读取大量数据显示List的帮助.

注意`PagedList#setEnablePlaceholders`设置为`true`当滚动速度大于读取速度onBind中获取的item可能会是null,当数据再次加载时会重新再被调用.如果设置为`false`,则会跳过null的item,但是滚动条会抖动,建议隐藏滚动条.

## [The Care and Feeding of Elephants ](http://blog.evernote.com/tech/2017/10/06/announcing-android-job-library-1-2-0/)

Evernote推出了最新的`android-job-lib-1.2.0`,这个库主要是封装了AlarmManager(pre 5.0),JobScheduler(after 5.0),GcmNetworkManager(device with Google Service),让你通过统一的API来做run tasks.

## [Fast and lazy .apk distribution with Crashlytics ](https://medium.com/@urmanschi.mihail/fast-and-lazy-apk-distribution-with-crashlytics-9a6d33804fde)

文章介绍通过git获取最新Tag到上一个Tag之间的Log生成Release Note,然后通过Crashlytics(Fibric)来分发.

## LIBRARIES & CODE

## [diagonal-imageview ](https://github.com/santalu/diagonal-imageview)

一个帮助你斜着切图片的ImageView

## [purrge](https://github.com/cesarferreira/purrge)

node的一个库,可以通过命令轻松删除手机里面的程序

## [ScalingLayout ](https://github.com/iammert/ScalingLayout)

一个可以做出来Scaling效果的Layout,需要将你的Layout包在它里面.
