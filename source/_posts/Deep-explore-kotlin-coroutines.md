---
title: 探究高级的Kotlin Coroutines知识
date: 2019-02-12 09:14:52
tags: [Android, Kotlin, Kotlin Coroutines]
categories: Kotlin
comments: true
description: 深入解析Kotlin Coroutines 协程
thumbnailImage: https://res.cloudinary.com/dtn0pkdmg/image/upload/v1549934323/Coroutines_jnzotc.jpg
---

要说程序如何从简单走向复杂, 线程的引入必然功不可没, 当我们期望利用线程来提升程序效能的过程中, 处理线程的方式也发生了从原始时代向科技时代发生了一步一步的进化, 正如我们的Elisha大神所著文章[The Evolution of Android Network Access](https://medium.com/@elye.project/the-evolution-of-android-network-access-1e199fc6e9a2)中所讲到的, Future可能会是Kotlin Coroutines的时代.

<!--more-->

### 什么是Coroutines

Coroutines是Kotlin 1.1推出的实验性的一个扩展, 它被定义为一个轻量级的高效的线程框架, 并且在1.3版本正式发布, 去掉Experiment标签.

### 如何启动一个Coroutines

最基础的创建一个Coroutines的方法就是使用`launch`或者`async`, 二者的区别是前者返回的是一个`Job`, 不带结果 而后者可以将结果以`Deferred<T>`格式返回.

如:

```
val job = launch {
    delay(100)
}
```

而通常在Coroutines内执行的函数都会有一个`suspend`声明, 而有`suspend`声明的函数也只能在Coroutines Scope中调用.

`suspend`的意思是这个函数可以被suspend(挂起), 让Coroutines来调度它, 这也是为何Kotlin的`delay`函数可以不阻塞的进行延迟, 因为它就是一个suspend函数.


### Coroutines与线程的关系

Coroutines可以简单理解为一个有队列的任务链, 每一个Coroutines都有自己的Context, 而Context又可以决定其运行的线程.

所以可以看到, 并不是起一个Coroutines就是起了一个线程, 而只是启动了一个在某个Scope下运行的协程(Coroutines)罢了. 这里的Scope (CoroutineScope) 内部包含了一个 Context (CoroutineContext).

```
public interface CoroutineScope {
    public val coroutineContext: CoroutineContext
}
```

如果只是通过`launch`来启动一个协程, 那它将会运行在Parent Scope所定义的线程中, 但是如果使用`GlobalScope.launch`来启动一个协程, 它将会使用线程池中的线程来创建一个协程, 线程池的大小跟CPU的核数相关.

当然`launch`也支持自己传入一个CoroutinesContext来控制它运行的线程, 它叫做`CoroutineDispatcher`, 是Context的子类.

上面讲了默认的`launch`会启在父Scope(Context)的线程中, 而`launch(Dispatchers.Default)`则等于`GlobalScope.launch`, 还可以通过`launch(newSingleThreadContext("MyOwnThread"))`来启动自己的线程, 另外有一个不推荐在general code中出现的`launch(Dispatchers.Unconfined)`, 它将会运行在第一个进入Suspend状态的线程中.

可以举一个简单的例子:

```
val job = launch {
    log("hehe")
    delay(1000)
    log("haha")
}
```

这个协程是可以完全在main函数里执行完的, 即输出结果为:

```
hehe
haha
```

因为launch会跑在main的scope中. 如果替换成:

```
val job = GlobalScope.launch {
    log("hehe")
    delay(1000)
    log("haha")
}
```

则只会输出`hehe`, 因为主线程已经结束.

这里我们可以通过`job.join()`来等待子协程执行结束, 这一点跟大家熟知的线程的join是一样.

### 如何切换Context

如果把Context对应到我们平时认为的线程, 那么这个问题可以类比成 `如何切换线程`.

答案是使用`withContext`, 举一个简单的栗子.

```
launch(UI) {
    updateUI()
    val result = withContext(IO) {
        
    }
    setView(result)
}
```

它类似于`async(IO){ }.await()`.

### 如何共享资源

线程与线程之间会涉及到同步与资源竞争的关系, 协程亦是如此.

通常情况下在线程中我们解决问题的方式是`加锁`, 而不正确的使用可能会导致性能下降甚至死锁（dead lock. 或者在高级语言中使用已经实现线程安全的数据类型, 来进行夸线程操作。

而我们的Coroutines自然也考虑到了这一点, 它认为我们`不应该以共享资源来进行通信, 而是以通信来进行资源共享`.

```
Do not communicate by sharing memory; instead, share memory by communicating.
```

所以它提出了一个叫做`Channel`的东西来在不同的Coroutines之间进行通信.

譬如我们期望将一堆数据交给两个并行的协程进行处理, 那么我们可以把数据放进Channel, 其他的协程从这个Channel进行数据读取.

```
launch {
    for (o in data) { channel.send(o) }
    channel.close()
}

launch(One) {
    for (o in channel) {
        xxx
    }
}

launch(Two) {
    for (o in channel) {
        xxx
    }
}
```

一定要记得关闭channel, 否则从channel读取数据的协程都将会无限挂起等待数据传过来.

由于Channel本身实现了`iterator`, 所以直接通过`in`就可以挨个取出内部的数据.


### ReceiveChannel与SendChannel

上一个环节提到的协程之间是通过Channel来进行通信, 而Channel本身却是实现了接收管道与发送管道两个接口.

我们可以通过`producer`函数来进行生成数据, 提供给别的协程, 因为它的返回值是一个ReceiveChannel.

```
val channel = produce<XXX>() {
    for (o in data) send(o)
}
```

而且produce自己会做channel close的处理, 省去我们发送完毕还要掉close的烦恼.

如果我们多个协程需要发送请求并集中处理, 或者可以叫数据整合, 那么我们可能需要用到`actor`这个函数, 它的返回值是一个SendChannel.

```
val channel = actor<XXX>() {
                consumeEach {
                   xxx     
                }
            }

launch(One) {
    channel.send(xxx)
}
launch(Two) {
    channel.send(xxx)
}
```

由于`actor`返回的SendChannel有点像是一个邮箱, 它会不断的接收数据, 所以必须手动关闭才会停止.


### 多个Channel之间数据如何进行选择

Coroutines推出一个仍在Experiment阶段的关键字`select`来在多个suspend function中进行选择第一个到达available的, 其实有点像RxJava的concat+first.

比如我有两个接收Channel, 但是每一个Channel接收到数据的频率不得而知, 我想分别从中得到数据, 这里就需要使用select.

```
select<Unit> {
    channel1.onReceive {}
    channel2.onReceive {}
}
```

如果在配合外围的循环, 就可以做到不断的去接收两个Channel的数据.

再比如有两个发送Channel都可以处理我的需求, 我也不知道这个时候谁是空闲的, 那也可以通过select来解决.

```
select<Unit> {
    channel1.onSend(xxx) {}
    channel2.onSend(xxx) {}
}
```

有时候两个Channel是嵌套使用的.

比如一个咖啡店, 他们会不断的收到Oder, 只有两个打咖啡的服务员, 咖啡机也只有两个口,  如果我们对这个咖啡店进行抽象. 将Oder存在于一个Channel里, 服务员接收Order并不断的把咖啡递出来, 这也是一个Channel, 咖啡机会不断接收到服务员需要打咖啡的操作, 也这是一个Channel.

而在这个过程中, 两个服务员会有一个选择, 咖啡机的两个出口也会有一个选择的过程.

如果抽象成我们的Coroutines代码, 或许会是这个样子:

```
val orderChannel = producer {
    for (o in orders) send(o)
}

val waiter1 = producer {
    for (o in orderChannel) { 
        pullCoffee(o)
    }
}

// waiter2 is the same as 1

val coffeePort1 = actor {
    consumeEach { 
        //pass coffee through channel inside order
        it.channel.send(Coffee)
        it.channel.close()
    }
}

// coffeePort2 is the same as 2

pullCoffee {
    select<Coffee> {
        coffeePort1.onSend(Request(channel)) {
            //get coffee from coffeePort
            channel.recevie()
        }
        coffeePort2.onSend ....
    }
}

while(someCondition) {
    select<Coffee> {
        waiter1.onReceiveOrNull {
            //上菜了
        }
        waiter2.onReceiveOrNull {
            //上菜了
        }
    }
}
```

### 补充说明

协程作为未来non blocking编程的方向, 需要大家花时间去理解, 花时间去尝试, 在此特别推荐这个咖啡小程序帮助大家学习.

https://medium.com/@jagsaund/kotlin-coroutines-channels-csp-android-db441400965f

以及官方的Overview

https://kotlinlang.org/docs/reference/coroutines-overview.html

还有个CheatSheet可以参考

https://blog.kotlin-academy.com/kotlin-coroutines-cheat-sheet-8cf1e284dc35