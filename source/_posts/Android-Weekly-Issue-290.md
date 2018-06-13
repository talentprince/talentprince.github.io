---
title: Android Weekly Issue #290
date: 2018-04-01 10:37:28
tags: [Android, IoT, Kotlin, Animation, LiveData,DataBinding,RxLog]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #290
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1522636507/290_udxyq8.jpg
---

December 31st, 2017

## [Android Weekly Issue #290](http://androidweekly.net/issues/issue-290)

本期内容包括介绍Kotlin逆变协变的一篇(虽然没说清楚,但我补充了),IoT相关制作MIDI Controler的Part two,以及比较炫酷的Shared Element Reveal动画,以及LiveData与DataBinding相关,Kotlin扩展Fragment/Activity方法做测试,Rx逐条打Log等等.

<!--more-->

## ARTICLES & TUTORIALS

## [Lessons learned implementing Redux on Android ](https://blog.pusher.com/lessons-learned-implementing-redux-on-android/)

文章介绍了模仿Web的Redux,实现其kotlin版本,实现`Reducer`,通过`State`与`Action`来驱动状态的转换.

```
State -> UI -> Action -> Reducer -> Store.
```


## [In and out type variant of Kotlin ](https://android.jlelse.eu/in-and-out-type-variant-of-kotlin-587e4fa2944c)

文章介绍了Kotlin中泛型添加`in`与`out`的意义.

实际上`in`作为参数表示的是consume方,可以将super type可以赋值给sub type.类似于Java里面的`<? super X>`,其作为泛型的Collection只能add数据,无法get访问内部成员.

而`out`作为返回值表示producer,与in相反,它可以将sub type赋值给super type.类似于Java中的<? extends X>,其作为反省的Collection<out T>只能get访问,不能add数据.


## [Building a distributed MIDI Controller with Android Things and Nearby API #2 ](https://proandroiddev.com/building-a-distributed-midi-controller-with-android-things-and-nearby-api-2-b4b0531d645e)

IoT的MIDI播放器第二篇,感兴趣的可以仔细看.


## [Meaningful Motion: Circular Reveal & Shared Elements ](https://medium.com/@jossiwolf/meaningful-motion-circular-reveal-shared-elements-ea495b99adf4)

在Shared Element Transaction Animation的基础上加上了`ViewAnimationUtils#createCircularReveal`实现Reveal效果. 即Activity/Fragment跳转过程中Shared Element先移动再充满Container.


## [RxAndroid: Handle Interrupt With “switchMap” ](https://tech.pic-collage.com/rxandroid-handle-interrupt-with-switchmap-3a650393299f)

通过switchMap将Happy与Unhappy的pass都加进来(Observer.merge)进行处理,switchMap与flatMap的区别是它内部只有一个active的observer,简单的来说,它不会对转换后的Observable进行merge,而是在新的来到的时候cancel之前的.


## [The curious case of haunting fragments ](https://jeroenmols.com/blog/2017/12/18/fragmentback/)

作者研究Fragment addToBackStack以及pop之间的事情,但是作者貌似没用对...

所以之后他居然推荐用Activity了,说Fragment太难用...


## [Unit testing protected lifecycle methods with Kotlin ](https://medium.com/@dpreussler/unit-testing-activity-lifecycle-4e740f71e68a)

作者写了个工具库,给Activity的生命周期方法都写了扩展,这样就可以直接通过对象调用了...可以用来写Activity的单元测试.


## [Kotlin Coding Conventions ](http://kotlinlang.org/docs/reference/coding-conventions.html)

Kotlin最新的code style,基本跟Java类似,但这里比较详细,包括什么时候换行,什么时候single line等等.


## [Lessons from my first multi-platform Kotlin project ](https://blog.kotlin-academy.com/lessons-from-my-first-multiplatform-kotlin-project-d4e311f15874)

作者对Kotlin Multiple Platform进行总结,首先platform层应该根据js/jvm/native进行划分,而不是操作系统,操作系统的划分应该属于之下的regular层,而最上层为common层.

MVP的应用非常重要,其次是下层可以访问上层的一切,上层需要访问下层应该通过expected与actual来实现.


## [Android Architecture Components LiveData with Data Binding ](https://android.jlelse.eu/android-architecture-components-livedata-with-data-binding-7bf85871bbd8)

Google最新的Databinding已经支持LiveData了,通过与LiveData进行绑定,可以保证UI在后台的时候不会因为数据变化而刷新,避免了没有必要的操作.


## [Briefly about RxJava Logging ](https://proandroiddev.com/briefly-about-rxjava-logging-20308b013e6d)

作者介绍了通过`doOnEach` (Flowable)以及`doOnEvent`(others)来了解Observable的状态,帮助你添加新的feature中debug遇到的问题,不至于整个Rx Chains出现问题而不知道问题处在哪里.


## LIBRARIES & CODE


## [TableView ](https://github.com/evrencoskun/TableView)

很项强大TableView,基于RecyclerView,用来显示复杂数据.有点类似数据库表格.


## [retrofit2-kotlin-coroutines-adapter ](https://github.com/JakeWharton/retrofit2-kotlin-coroutines-adapter)

Jake Warthon写的支持Kotlin Coroutine的Retrofit2, 返回`Deferred`类型.


## [RxTest ](https://github.com/RubyLichtenstein/RxTest)

像这个来测Rx的Observable,是不是很牛.
```
Observable.just("Hello RxTest!")
    .test {
        it shouldEmit "Hello RxTest!"
        it should complete()
        it shouldHave noErrors()
    }
```


## [MockK ](http://mockk.io/)

支持Koltin DSL的mock库, 叫mockk....


## [KotlinAndroidViewBindings ](https://github.com/MarcinMoskala/KotlinAndroidViewBindings)

其实感觉跟ViewBinding没多大关系, 主要是实现了Delegate,可以取代`findViewById`,`Butterknife`以及`Kotlin Android Extension`.

直接通过`by bindWithXX()`来找到View.


## [litho-kotlin ](https://github.com/vinc3m1/litho-kotlin)

Facebook litho的kotlin dsl support.


## [kotlin-math ](https://github.com/romainguy/kotlin-math)

支持很多vector计算的lib,帮助简化graphic math.