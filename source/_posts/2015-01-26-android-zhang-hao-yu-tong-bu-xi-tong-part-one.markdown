---
layout: post
title: "Android 账号与同步服务 Part One"
date: 2015-01-26 08:33:37 +0800
comments: true
categories: Android
Tags: [android]
keywords: Android SyncAdapter Account Authenticator 账号 同步
description: Android 账号与同步服务 SyncAdapter  Account Authenticator
---

####账号与同步

Android从`API Level5`就有了自己的同步服务, 但很少有程序使用到, 一来大多数程序不需要所谓的同步,二来很多程序自己实现了后台的同步更新. 随着Android程序开发的逐渐程序, 越来越的的程序使用到了系统提供的服务来完成`账号认证`与`同步更新`, 我们可以打开系统设置-->账号进行查看, 就能看到很多应用都这么做了. 这样做有两个好处, 一来系统服务做更新同步(`SyncAdapter`)唤醒更加绿色环保, 二来实现了账号认证(`Authenticator`)还可以为其他应用提供第三方认证服务, 如大家常见的使用QQ或者微博账号登录, 由于你手机上安装的QQ与微博实现了该接口, 便可以通过开发者账号获得授权Token来做第三方认证.

本期博客分三部分来讲, 通过一个小应用来概述所有相关内容, 大体章节如下

* 数据模型建立与加载 (ContentProvider LoaderManager)

* 更新系统建立 (SyncAdapter)

* 账号系统建立 (Account Authenticator)

下面先来讲讲如何轻松本地数据库并完成数据到界面的加载

<!--more-->




