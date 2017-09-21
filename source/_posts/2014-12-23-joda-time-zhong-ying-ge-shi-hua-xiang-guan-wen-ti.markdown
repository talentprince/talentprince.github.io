---
layout: post
title: "Joda Time 中英格式化相关问题 for Java"
date: 2014-12-23 14:19:17 +0800
comments: true
categories: Java
tags: [Java Joda]
description: Joda Time 中英格式化相关问题 for Java
keywords: Java,Joda,Android,时间格式
---
# Joda Time

Joda-Time提供了一组Java类包用于处理包括ISO8601标准在内的date和time。可以利用它把JDK Date和Calendar类完全替换掉，而且仍然能够提供很好的集成。  

## Install

Joda已经更新到2.6版本,jar包的下载可以到[Joda Jar](https://github.com/JodaOrg/joda-time/releases/tag/v2.6)进行下载.
如果使用gradle管理,可添加 

```
dependencies {
    compile 'joda-time:joda-time:2.6'
}
```
<!--more-->
## 使用
大致有两种方式可以对ISO8601时间进行格式化

```Java
DateTime dateTime  = new DateTime();//也可以传入ISO8601格式时间作为参数
textView.setText(dateTime.toString("EEEE MMMM dd yyyy HH:mm:ss"));
```

```Java
LocalDate date = LocalDate.now();
DateTimeFormatter fmt = DateTimeFormat.forPattern("EEEE MMMM dd yyyy HH:mm:ss");
String str = date.toString(fmt);
```

## 中英文格式区别
通过官方文档可知, 常用的`Symbol`有`E`(星期) `y`(年) `d`(日) `M`(月) `H`(小时0~23) `k`(时钟小时1~24) `m`(分钟) `s`(秒) `K`(小时0~11) `a`(am/pm)

 - **E** 中文永远都是`星期X` 英文`E`代表简写,如`Mon`, 而`EEEE`代表`Monday`, 调皮的话可以发现`EE`跟`EEE`都是简写,而再多的`E`都是全写

 - **y** 可能由于目前的时间都是公元四位数年(2014), 所以超过四个`y`都会在当前年份前加`0`,如`yyyyy`->`02014`, 英文其他个数都是全写, 中文如两个`y`,则为简写,如2014就成了14

 - **d** 所有超过两个的使用都会在应有的数字前面加0, 而且中文也不会加上`日`字

 - **M** 英文`M`跟`MM`为数字月份,如`03`, `MMM`则为简写`Mar`, 而`MMMM`或者更多(调皮)为`March`. 中文三个或以上为`X月`,其他都为纯数字

 - `H` `K` `k` `m` `s`等小时分钟,一个为0,超过加0

 - `Z`表示时区, 个数多少也有不同, 可参见官方文档


------------------
参考:http://www.joda.org/joda-time/key_format.html



