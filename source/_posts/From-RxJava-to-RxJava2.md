---
title: 如何从RxJava升级到RxJava2
date: 2018-04-02 10:29:31
tags: [Android, RxJava]
categories: Android
comments: true
description: RxJava 升级 RxJava2
thumbnailImage: http://res.cloudinary.com/dtn0pkdmg/image/upload/v1522636507/rxjava2_dgnvie.jpg
---

RxJava2已经推出有一年半的时间,由于之前RxJava已经在现有项目中广泛使用,而RxJava2在除了很多命名外并没有太多革新,所以相信有很多人跟我一样都还没有升级.

随着老版本渐渐的失去维护,更重要的是有一定时间允许我来做这个迁移,其实弃老从新一直都是程序员的喜好.

虽然官方提供了[文档](https://github.com/ReactiveX/RxJava/wiki/What's-different-in-2.0)详尽的介绍了区别,但是文章之长,可能很多人读不下去,却有想快速的迁移过来,我将除了命名改变之外有用的地方总结成了几点,供大家参考.

<!--more-->


## 不能再发射Null了

RxJava2的最大改变就是不能再流里发射Null了,有人会问发射了就怎么了,答案是你的流会因为`NPE`断开.

比如以前我们会写出这样的代码(详见RxPermission):

```
Observable.just(null).compose....
```

在RxJava二中我们需要将它改为(详见RxPermission2):

```
TRIGGER = new Object()
Observer.just(TRIGGER).compose(xxx)
```

还有我们常常完成某个工作而不需要返回值,或者根本不关心返回值,将返回的Observable定义为Observable<Void>, 如:

```
xxx.flatMap {
   ....
   return null;
};
```

现在不能这么写了,对于不需要返回值的,我们应该使用Completable,当然这个在RxJava的时候也已经存在了.

```
xxx.flatMapCompletable { Completable.fromAction{ } }
```

还有我们在实现Local Cache与Remote Cache的时候常用的办法:

```
localObservable = just(localReference);

concat(localObservable, remoteObservable).filter{ i != null }.first()...

```

会因为在没有Local Cache的时候出错,所以应该改成:

```
localObservable = just(Optional.fromNullable(localReference));

concat(localObservable, remoteObservable).filter{ i.isPresent() }.firstElement()/.first(defaultValue)...
```

## flatMap方法多了

在上面的介绍中可能已经发现了,老版本只有同类型的flatMap,即Observable <-> Observable, Single <-> Single, 而RxJava2除了同类型的flatMap,还增添了flatMapCompletable,flatMapSingle,flatMapObservable帮助你任意切换.


## 订阅与反订阅

我们有时候需要在必要的时刻手动的将订阅取消,而防止产生我们不想要的问题,如在跳出定位页面时取消订阅,防止位置信息后面回来造成程序崩溃.

而在RxJava中,我们一般是这么做的:

```
Subscription subscription = xxxx.subscribe(xxxSubscriber);

subscription.unsubscribe();
```

在RxJava2中,这个发生了变化,因为你会发现`subscribe`方法基本上都返回void的,如果你需要手动取消的话,需要使用`T subscribeWith(T extends Disposal)`方法.

其实我们可以看到,新版的`Subscriber`或者`Observer`都多了一个方法`void onSubscribe(Subscription s)`或者`void onSubscribe(Disposable d)`, 也就是说以前的`Subscription`是通过订阅后通过回调返回了.

这里RxJava2统一接口到`Disposable`,提供`dispose`方法进行反订阅,并且还提供了`DisposableObservable`,`DisposableSingle`,`DisposableCompletable`已经帮我们处理了回调返回的Disposable对象.

所以需要做的改动不大:

```
Disposable disposable = xxx.subscribeWith(xxxDisposableObserver);

disposable.dispose();
```

## 错误处理

错误处理最棒的一点是之前必须实现onError来handle错误,如果不实现,就会抛出`OnErrorNotImplement`,导致程序崩溃,根据最新的Doc,在RxJava2中,可以轻松Handle未处理的错误.

```
RxJavaPlugins.setErrorHandler(xxx);
```

还有一点变化需要注意是的是,当你有并行任务的时候,如果一个线程出错,将会导致整个流中断,其他线程可能会抛出`IOInterupedException`并且onError无法Handle,这时候必须有上面讲到的ErrorHandler来处理这一类`UnDeliveriedException`,否则程序会Crash.


## Flowable

RxJava2将处理背压(BackPressure)的部分抽出来弄了一个新的对象,叫做`Flowable`.

以前我们处理背压可能直接通过

```
xxx.onBackpressureXXXStrategy()...

```

就可以了.

现在我们得通过Flowable来处理.

```
xxx.toFlowable(XXXStrategy)...

```

当然Flowable还提供比较强大的新方法,来处理并发.

比如之前我们需要实现并发,得通过flatMap来实现.

```
Observable.from(urls).flatMap { 
       v -> Observable.just(v).subscribeOn(io()).....
}.subscribe(...)

```

使用Flowable,可以简化为:

```
Flowable.fromIterable(listingIds)
    .parallel().runOn(io())
    .map { v -> xxx }
    .sequential()
```

看起来是不是有点炫酷...


## 测试

对于RxJava2,任何一个Observable都可以转化为一个`TestObservable`, 通过`...test()`来进行转换.

而TestObservable提供很多与测试相关的方法,就不用我们亲自去判断.

如`assertResult`,`assertError`,`assertSubscribed`.


## 其他改动

关于名字的变化,这里都不一一论述,包含`Func1` -> `Function`, `Action` -> `Consumer`, `Observable.Transformer` -> `ObservableTransformer`等等. 



