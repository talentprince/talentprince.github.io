---
title: 'Android Weekly Issue #285'
date: 2017-12-05 14:30:47
tags: [Android Weekly,Kotlin,RxJava2,MVI,Clean Code,Kotlin Coroutines]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #285
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1512456415/285_q5mnle.jpg
---

November 26th, 2017

## [Android Weekly Issue #285](http://androidweekly.net/issues/issue-285)

本周包含好几篇Kotlin的文章,如通过Property Delegate实现SharedPreference的自动读写,Coroutines编写Weather应用的终结篇,还有关于如何写好代码的方法论的Tips,以及MVI的全面介绍,如果不喜欢Mosby的approach,可能这个作者从头到尾实现的更有看头.

当然如果想了解Android最流行的pipeline如何开发,可以去看一篇关于Devops的文章.对代码感兴趣的,看看MVI也是不错的.


<!--more-->

## ARTICLES & TUTORIALS

## [Shrinking APKs, growing installs ](https://medium.com/googleplaydev/shrinking-apks-growing-installs-5d3fcba23ce2)

文章通过数据分析每提升6M的apk大小,就会损失1%的用户,原因大多因为流量.
文章还拿出了google内部的数据图,发现北美使用的应用大,东亚非洲使用的小,殊不知天朝不用google play...


## [Multi-currency support in Java | Drivy Engineering ](https://drivy.engineering/multi-currency-java/)

一个简单的支持数字转货币格式的库,包括同种货币在不同国家不同的显示方式.


## [The Contract of the Model-View-Intent Architecture ](https://proandroiddev.com/the-contract-of-the-model-view-intent-architecture-777f95706c1e)

作者对MVI进行了详细的分解,实现了MVI不同模块的Contract,并分析了各个模块的基本职责.

整体的思路大致如下,形成一个回路:

View -> Intent --(intent to action)--> Action --(processor)--> Result --(reducer)--> State --(render)--> View


## [Function references in Kotlin: use functions as lambdas everywhere ](https://antonioleiva.com/function-references-kotlin/)

文章讲解了Function Reference (`::function`)的使用,可以作为lmbda当参数传递,并且生成的bytecode里面会少一个形参的临时变量的创建,相当于把lambda表达式的回调参数inline了.


## [Onboard your users with Lottie of Spritz ](https://www.novoda.com/blog/onboard-your-users-with-lottie-of-spritz/)

文章介绍了作者自己弄的一个库Spritz,来做引导画面,将ViewPager于AirBnB的LottieAnimationView结合起来,可以通过swipe来触发动画,或者自动触发.


## [Room Migration ](https://android.jlelse.eu/android-architecture-components-room-migration-1a269e1aeef7)

文章介绍了Room如何做DB Migration,与一般的数据库相同,其提供了直接Drop&ReConstruct以及提供相应的Migeration语句两种办法.
`fallbackToDestructiveMigration`与`addMigrations`


## [Simple but painful steps for writing a better code ](https://medium.com/car2godevs/simple-but-painful-steps-for-writing-a-better-code-afb2651cef86)

作者介绍了几条写更好的代码的原则.

- 不要在Class内部再new新的object,应该通过构造注入.
- 不要依赖于单例,如果有可直接通过构造传入.
- 不要再取Manager/Processor/Hanlder之类的类名,这样会使这个类加入越来越多不太相关的东西.
- 不要无脑的继承,有时候组合也挺好.


## [The Art of Android DevOps – Undabot ](https://blog.undabot.com/the-art-of-android-devops-fa29396bc9ee)

文章介绍了什么是Android DevOps,其实这些Work我们都在做,只是没有独立出来这个Role罢了,其工作内容包括:
- CI/CD
- Automated Test (UT, InstrumentTest, PIT)
- Code check (Findbugs, checkstyle, PMD)
- Deploy (fastlane)


## [9 RxJava 2 Migration Learnings At Runtastic ](https://www.runtastic.com/blog/en/tech/rxjava-2-migration-learnings/)

介绍了九条从RxJava1升到RxJava2应该注意的东西,比较值得注意的是在测试的时候可以通过Observable.test转换后进行assert,还有应该推广使用`Completion`,`Maybe`等在恰当的场合,还有不能滥用`Flowable`除非你有背压需求.


## [Kotlin: Contexts & SharedPreferences ](https://blog.stylingandroid.com/kotlin-contexts-sharedpreferences/)

作者通过Property Delegation写了个SharedPreferenceDelegate,这样需要储存的变量只需要通过`by`交给这个delegate就可以完成数据的自动读写.

如
```
var value: Int by bindSharedPreference(context, KEY, DEFAULT_VALUE)
```

## [Clean-Code App with Kotlin and Architecture Components — Part 3 ](https://blog.elpassion.com/create-a-clean-code-app-with-kotlin-coroutines-and-android-architecture-components-part-3-f3f3850acbe6)

使用Kotlin Coroutines优化程序的最后一个Part,主要是UI部分,结合了Architecture Components的ViewModel与LiveData.

对于Coroutines,作者的看法是它是用来与RxJava竞争的,帮助大家接触Callback Hell,而且它也可以跟RxJava结合使用.

其核心思想就是对于suspendable function的理解,帮助实现看似同步的异步世界.


## [Kotlin From The Trenches ](https://blog.devexperts.com/kotlin-from-the-trenches/)

一片宏观介绍Kotlin如何牛逼的文章,Android的Java被锁定在了1.6,即便有了sugar,还是不能完全支持1.8,Kotlin应运而生.

至于Kotlin的优势,这里不做阐述了,类似的文章太多了,建议大家都去用一下,因为只有真正使用了,才能有比较大的进步.


## LIBRARIES & CODE


## [spritz ](https://github.com/novoda/android-demos/tree/master/spritz)

做一个基于ViewPager的引导界面.


## [Droid-Snippet ](https://github.com/KingsMentor/Droid-Snippet)

一个AS的查件,有点像代码宝典,内置了各种Util类的方法,可以一键呼出,复制粘贴分分钟.包含Network,Image,File,Permission,Service等等等等....


## [koin ](https://github.com/Ekito/koin)

之前有介绍过一个Kotlin的Inject框架,相较于Dagger还是有很大的优势,至少配置起来容易.