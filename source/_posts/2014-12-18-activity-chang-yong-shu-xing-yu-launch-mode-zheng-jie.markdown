---
layout: post
title: "Activity LaunchMode 与 Intent Flags 揭秘"
date: 2014-12-18 19:24:32 +0800
comments: true
categories: android
keywords: Android,Activity,TaskState,Launchmode,taskAffinity,Intent,Flags
description: Activity Task State与Launch Mode 揭秘
tags: [android]
---
Android Activity所涉及的四种Launch Mode与其重要的几个属性，如taskAffinity，allowTaskReparenting等，包括Intent内的各种Flag的功效，一直是为广大开发者所苦恼，网上文章众说纷纭，开发文档又及其模糊且与实际情况有一定偏差，那么今天我们就来真正的揭秘，还原事实的真相。
<!--more-->


*Task概念*

----------

>Task相当于一个栈，用来管理Activity，默认情况下每一个程序都会拥有一个自己的Task，有自己独立的id，所以一个手机同时可能会拥有很多Task，标识就是id，默认情况下跟包名有关系。



*Mode*

----------
 1.  standard
>默认模式，启动的所有Activity都会入栈，无论是之前启动过的还是没有启动过的，最终会形成的特色栈况为：
>ABCDEF.......或者ABCDDCEFAB........

 2.  singleTop
>跟标准模式唯一的区别就是，当栈顶为要启动的Activity时，复用该Activity，只会调用onNewIntent，而不会onCreate。所以不会出现ABCDD,而只会有ABCD

 3.  singleTask
>官方文档解读，启动的Activity为栈底，并且会重新建立新的Task，在该Task启动的新Activity会依附于其。（但实际上并不是你想象中的这样）

 4.  singleInstance
>官方解读，启动的Activity会开启一个新的Task并独享，在其上启动的Activity会寻找其他合适的Task




*Mode揭秘*

----------

 1.  singleTask的Activity**不一定**是在栈低，在默认状况也**不会**创建新的Task，除非指定taskAffinity，或使用其他程序启动该Activity（或在另外的进程启动，如Receiver之类的回调中，需加NEW_TASK标识），如果没有对应的Task，则会创建，如果已有，则会附庸。其特性可利用之处在于，**其总能保证一个Task内只有一个实例，并且总会销毁位于它之上的所有Activity**
 2. singleInstance的Activity**永远会**创建新的Task并自己独享该Activity，不会受到taskAffinity的影响，并且在其上开启的其他Activity都会寻找适合自己的Task（如指定则会附庸或者创建，如未指定则会使用之前默认的栈），singleInstance能保证**整个操作系统**拥有**唯一**的实例对象。




*Activity对task state影响的其他属性介绍*

----------

 1. android:allowTaskReparenting
>默认false，程序A已经启动了数个Activity，其中包含Activity1，如果这个Activity1拥有该属性，当程序B也要启动它的时候，其可宿主到程序B内。
 
 2. android:alwaysRetainTaskState
>默认false，操作系统会在程序长时间不会动的时候，清除Task状态，如果开启这个，系统会继续保持直到再次打开，只对根Activity生效

 3. android:clearTaskOnLaunch
>默认false，如果开启，从桌面重新进入程序时，只会存在根Activity，如有其他程序的Activity，他们将会寻找宿主，这个属性也只对根生效
 
 4. android:finishOnTaskLaunch
>默认false，如果开启，程序重新启动，会销毁所有存在的Activity，也只对根Activity生效




*重要的几个Intent Flag介绍*

----------

Intent存在很多可以设置的Flag，但是其基本是为系统所使用，在不正确的状态下调用无效。下面说说几个比较有用的Flag。

FLAG_ACTIVITY_CLEAR_TOP
>如果在ABCD的堆栈状态下，以该标识启动B，则会销毁CD，且B也是重新创建的（与singleTask有区别），如果配合FLAG_ACTIVITY_SINGLE_TOP，则就会成为singleTask的模式

FLAG_ACTIVITY_BROUGHT_TO_FRONT
>如果ABCD中A以这个标识启动B，则D再启动B，B将会被调入前台，成为ACDB，且B不会重新创建

FLAG_ACTIVITY_REORDER_TO_FRONT
>这个Flag主要用来改变Task堆栈顺序，如果在ABCD的状态下，以该标识启动B，则会成为ACDB，且B不会重新创建

FLAG_ACTIVITY_RESET_TASK_IF_NEEDED
>这个标识主要用于创建一个还原点，再次启动该Task时会将还原点之上包括其本身都销毁掉，如果在一个程序中以该标识启动了另外一个程序的功能，如一个用于看图的软件，当退出桌面，再点击这个程序，看图软件会消失。

FLAG_ACTIVITY_NEW_TASK
>这个是最常用的，但是往往会被误解，在程序根Activity的Task栈里加此标识开启新Activity都不会创建新的Task，只有当另一程序（进程）启动带有改标识的Activity时，才会创建新的Task。如果配合FLAG_ACTIVITY_NEW_MULTI_TASK，则无论什么情况都会创建新的Task，就成了singleInstance的情况了。



复杂的mode, 复杂的属性，复杂的flag，含糊的doc，以及网络上很多只意象，为实践或者研究源码的文章，造就了对此相关问题的误解，在这里做以真实揭秘，所有都经过验证，如有披露，还请斧正。


----------

参考

http://www.cnblogs.com/mengdd/archive/2013/06/13/3134380.html


