---
title: 'Android Weekly #275'
date: 2017-09-27 09:00:14
tags: [Android, Google Map, Kotlin, Architecture Component]
categories: Android Weekly
comments: true
description: Android Weekly 中文概要 #275
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1506482907/275_zyahvm.jpg
---

September 17, 2017

## [Android Weekly Issue #275](http://androidweekly.net/issues/issue-275)


本期内容包括给Google Map实现一个Marker Adapter, 如何更好的让Kotlin类可测试, MVI的优势 Google的Room与Paging Library相关文章, 以及Realm如何实现React, 还有比较冷门的AsyncListUtil如何使用的介绍哦.

<!--more-->


# ARTICLES & TUTORIALS

## [MapMe — the Android maps adapter ](https://medium.com/default-to-open/mapme-the-android-maps-adapter-bfca21713772)

作者基于Google Map写了一个类似`RecyclerAdapter`的东西, 叫做`GoogleMapMeAdapter`, 可以通过继承实现其`onCreateAnnotation`与`onBindAnnotation`接口, 来控制Map Marker的显示, 在地图初始化成功后通过`attach(mapView: MapView, googleMap: GoogleMap)`来绑定即可.

## [CloudRail - Connect to APIs 10x Faster ](https://cloudrail.com/)

请注意, 这是一个广告, 一个Android课程的广告.

## [Kotlin Testability – Part 1 ](https://blog.stylingandroid.com/kotlin-testability-part-1/)

作者论述了如何创建一个可测的类, 相较我们平时常用的将内部其他模块最为参数注入的方法, 作者指出会暴露我们的具体实现, 转而抽象出一个接口, 并给与内部实现, 在测试的时候只需另行实现接口进行mock即可.

大概是这个样子

```
class DateStringProvider internal constructor(private val factory: Factory) {
 
    constructor() : this(DefaultFactory)
 
    fun buildDateString() =
            factory.getLocalisedDateTimeFormatter().format(factory.getLocalDateTime()) as String
 
    internal interface Factory {
        fun getLocalDateTime(): LocalDateTime
        fun getLocalisedDateTimeFormatter(): DateTimeFormatter
    }
 
    private object DefaultFactory : Factory {
        override fun getLocalisedDateTimeFormatter(): DateTimeFormatter = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL)
 
        override fun getLocalDateTime(): LocalDateTime = LocalDateTime.now()
    }
}
```

## [ViewModels and LiveData: Patterns + AntiPatterns](https://medium.com/google-developers/viewmodels-and-livedata-patterns-antipatterns-21efaef74a54)

作者介绍了Android新推出的Archietecture Component相关的架构实现, 其实大部分与Android官方的[Document](https://developer.android.com/topic/libraries/architecture/livedata.html), 讲述了如何正确使用ViewModel去实现数据加载, 防止内存泄露, 并且阐述了一些关键的要素.

- ViewModel不要与Android Framework耦合
- Activity与Fragment的逻辑应该最简化
- ViewModel不能持有View的对象
- UI应该通过Observe的方式从ViewModel得到数据更新, 而不应该是Push进去
- 通过DataRepository与数据进行交互
- LiveData最好与Repository也是订阅关系,防止ViewModel泄露

## [Taming state and side effects on Android ](https://medium.com/@charlag/taming-state-and-side-effects-on-android-d4749a8a50bc)

讲述了一个很复杂的React方式进行状态管理的东西.

## [Reactive Apps With MVI - PART 7 ](http://hannesdorfmann.com/android/mosby3-mvi-7)

文章继续MVI的推介, 间接抨击了Google的Architecture Component, `LiveData`通过`SingleLiveEvent`来规避如何保存View#State的问题, 通过SnackBar Show/Hide的例子介绍了MVI很好的解决了这种问题.

## [Make your Android Project pop with Remixer ](https://android.jlelse.eu/make-your-android-project-pop-with-remixer-by-google-26366a02f146)

介绍了Google的一个NB lib [Remixer](https://github.com/material-foundation/material-remixer-android), 不需要重新编译就可以适配你的UI, 支持输出`Number`,`String`,`Boolean`,`Color`.

效果如下

![Remixer](/source/images/remixer.gif)

## [Creating a Reactive Data Layer with Realm and RxJava2 ](https://academy.realm.io/posts/creating-a-reactive-data-layer-with-realm-and-rxjava2/)

文章介绍了Realm React的演进, 从Rx1升级到Rx2, 让创建一个react的`DataLayer`变得很简单.

```
// not singleton!
public class TaskRepository  {
    private final Realm realm;

    public TaskRepository(Realm realm) {
        this.realm = realm;
    }

    // this implementation works on any thread.
    public Flowable<Task> getTasks() {
        if(realm.isAutoRefresh()) { // for looper threads. Use `switchMap()`!
            return realm.where(Task.class)
                    .findAllSortedAsync(TaskFields.ID)
                    .asFlowable()
                    .filter(RealmResults::isLoaded);
        } else { // for background threads
            return Flowable.just(realm.where(Task.class).findAllSorted(TaskFields.ID));
        }                    
    }
}
```

## [How to use AsyncListUtil ](https://android.jlelse.eu/how-to-use-asynclistutil-16b5175bb468)

介绍了如何使用隐藏很深缺缺乏文档的`AsyncListUtil`去优化`RecyclerView`的加载, 减少内存占用, 当你有很多数据的却不想全部存在内存, 核心是去实现`AsyncListUtil.DataCallback`与`AsyncListUtil.ViewCallback`两个接口, 第一个提供`fillData`与`getCount`两个方法, 第二提供`onDataRefresh`,`getItemRangeInto`,`onItemLoaded` 三个方法, 然后需要添加`ScrollListener`在滚动的时候触发`AsyncListUtil#onRangeChanged`.

但其核心思想还是通过Cursor来读取数据, 在`fillData`里面来填充, 所以根本在于数据库提供的优势, 其实还不如用[CursorRecyclerAdapter](https://gist.github.com/sin3hz/109db92c03e8f0506f52)来的直接.

## [Android Development Syllabus ](https://novicedock.com/learn/computer-science/android-development)

又是一个课程.

## [Large Database Queries on Android ](https://medium.com/google-developers/large-database-queries-on-android-cb043ae626e8)

介绍了`SQLiteCursor`实现大数据查阅的弊端.

- `SQLiteCursor`不持有任何打开的数据库会话, 每次都是新开窗口
- `SQLiteCursor.getCount`会扫描所有query的数据然后进行count
- `SQLiteCursor.getCount`每次都会load第一个Window, 如果你使用到第三个Window的数据,它会强制load然后丢掉不用的
- `SQLiteCursor`会加载你不需要的数据,因为当你读到超过窗口1/3的时候,它会触发并填充整个窗口
- `SQLiteCursor`加载位置不确定, 因为它需要提前加载1/3个窗口,需要猜测一个窗口需要多少列
- 需要Close
- `SQLiteCursor`不知道数据发生了变化

规避办法,就是尽量的分页读取,推荐使用[Room](https://developer.android.com/topic/libraries/architecture/room.html)&[Paging Library](https://developer.android.com/topic/libraries/architecture/paging.html)

## [Building a Guitar Chord Tutor for Actions on Google: Part One ](https://medium.com/@hitherejoe/building-a-guitar-chord-tutor-for-google-actions-part-one-975e4f855e58)

介绍了自己使用[Google Actions](https://api.ai/)做一个教吉他和弦的Demo.

# DESIGN

## [On the Bottom Navigation Bar ](https://medium.com/android-ui-patterns/on-the-bottom-navigation-bar-d07d9b4b5e18)

文章介绍了使用`BottomNavigationView`后由`BackButton`带来的交互问题, Back了该跳到哪里去.

# Library

## [material-remixer-android ](https://github.com/material-foundation/material-remixer-android)

上面介绍过, 不需要重新编译就可以通过代码来控制UI.

## [AdaptiveIconPlayground ](https://github.com/nickbutcher/AdaptiveIconPlayground)

一个测试Adaptive Icon的东西.

## [MapMe ](https://github.com/TradeMe/MapMe)

给GoogleMap的Marker做了一个Adapter.

## [Paging Library ](https://developer.android.com/topic/libraries/architecture/paging.html)

让LiveData支持Paging.
