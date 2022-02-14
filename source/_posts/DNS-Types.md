---
title: DNS的各种记录类型的应用解析
date: 2022-02-09 16:55:30
tags: [ops]
comments: true
categories: Operation
description: DNS types analysis and application
thumbnailImage: https://res.cloudinary.com/dtn0pkdmg/image/upload/v1644834388/dns_bs6hrv.png
---

可能很多人平时工作中不会遇到DNS配置相关的问题， 但如果偶尔遇到不同类型DNS记录的配置， 在没有搞清楚它们都是干啥的情况下， 会眼花缭乱， 还记得很多年前实验室配置DNS不太对导致只能访问www.instlink.com, 而无法解析subdomain.instlink.com。

<!--more-->

现在回头想起来，可能是将A类记录配置成了www.instlink.com 导致的， 所以无法再CNAME到subdomain.instlink.com，而只能整成subdomain.www.instlink.com。由于网站需要备案，修改起来麻烦，所以只能将二级页面通过/subdomain的形式来展现。

刚好最近所做的工作中涉及到了一些邮箱方面的DNS的配置，使用Amazon Route53托管，这里简单的将不同种DNS记录类型做一些介绍。

## A/AAAA

这两种是最基本的DNS记录， 它负责绑定域名与对应的IP地址， 只不过`A`绑定的IPv4， 而`AAAA`绑定的IPv6。

例子：
```
foo.example.com.   A    192.0.2.23
```

其中86400是TTL

## CNAME

CNAME（canonical name）是A记录的alias或者叫做lieu， 一般用作绑定subdomain，或者其他domain与主domain的关系， 而且它只能指向一个domain。

例子：
```
bar.example.com.    CNAME  example.net.
```

CNAME的查找是单条的， 也就是如果xyz.bar.example.com来了就无法命中这条CNAME记录， 但是还有另外一个叫`DNAME`的可以映射整个subtree，且DNS服务器在处理DNAME时，其实也是产生对应的CNAME来解析。

所以如果存在一条DNAME，如：
```
bar.example.com.    DNAME  example.net.
```
那么解析的过程是通过如下的CNAME来完成的：
```
xyz.bar.example.com.  CNAME  xyz.example.net
```

## MX

Mail Exchange， 这个是专门给邮件用的，依据SMTP协议，通过这个DNS地址找到对应的邮件服务器host，所以这个类型的记录也只能指向一个域名，而不能是IP地址。

例子：

|Domain|TTL|Class|Type|Priority|Host|
|:---|:---|:---|:---|:---|:---|
|example.com.|1936|IN|MX|10|onemail.example.com.|

## TXT

TXT就跟他的名字一样， 会返回一段文本， 本来的设计是用来返回人可以阅读的信息， 现在主要用于邮件防Spam， 以及域名所有者声明。

例子：

|Domain|Type|Value|TTL|
|-|-|-|-|
|example.com.|TXT|This is an awesome domain! Definitely not spammy.|32600|

关于如何防止Spam， 有三个基于TXT的DNS记录（SPF，DKIM，DMARC）后面会提到。

基于IETF在1993年的规范，如果想在TXT里保存Key/Value值，需要双引号（“”）包裹K=V值，但是这个规范往往没有被采纳，所以如果我们去看防Spam的几种DNS记录，是没有加引号的。

## NS

NameServer用来指定DNS解析服务器的域名，它是用来告诉去哪里来找到对应域名的IP地址的。

例子：

```
example.com. 172800 IN NS ns1.example.com.
```

不同的subdomain可以通过NS来指向不同的DNS解析服务器，私有的DNS服务器也可以通过NS指向第三方云服务商提供的地址，这个地址可以是一个IP地址，但是这是一个不好的实践，因为IP地址可能会发生变化，导致DNS无法解析。

## SOA

SOA（start of authority）主要用来保存这个域名的一些重要的信息，包括管理员的email地址。

例子：

|Domain|Type|MNAME|RNAME|SERIAL|REFRESH|RETRY|EXPIRE|TTL|
|-|-|-|-|-|-|-|-|-|
|example.com.|SOA|ns.example.com|admin.example.com|1|7200|900|120960|86400|

这里面的`RNAME`是管理员的Email，只是没有带`@`，看起来不像而已。

## SRV

这个代表Service记录，一般是为一些特殊的服务提供host与端口的，如VoIP，即时消息，并且只能指向A或者AAAA类地址，不能是一个CNAME。

例子：

```
_xmpp._tcp.example.com. 86400 IN SRV 10 5 5223 server.example.com.
```

如果使用表格来描述，即：

|Service|Protocal|Name|Type|Class|Priority|Weight|Port|Target|
|-|-|-|-|-|-|-|-|-|
|XMPP|TCP|example.com|SRV|IN|10|5|5223|service.example.com|

## PTR

PTR是Pointer Record的缩写， 它可以被理解成A/AAAA记录的反向，一般被用于做防Spam，解决邮件发送问题，系统Log IP <-> 域名转换等。

例子：

|Host|Type|Points to|TTL|
|-|-|-|-|
|1.0.168.192.in-addr.arpa|PTR|host1.example.com|86400|
|

这里面之所以要给IP地址加上`in-addr.arpa`，是因为IP地址存在.apra顶级域名下，如果是IPv6的地址，需要使用.ip6.arpa。

## DNSKEY

这条记录内保存了一个公钥，主要是为了用来验证DNSSEC（Domain Name System Security Extensions）的数字签名，保证DNS信息来源于一个真实可靠的DNS服务器。

例子：

|Host|TTL|Class|Type|Flags|Protocal|Algorithm|Public Key|
|-|-|-|-|-|-|-|-|
|example.com|3600|IN|DNSKEY|257|3|13|ZhCa3rGLofZcndFN2aVd==|

这里Protocal只能填3，代表`DNSSEC`， 而算法可以选多种：
- 1 = RSA/MD5
- 2 = Diffie-Hellman (This is not supported by BIND and Infoblox appliances.)
- 3 = DSA
- 4 = Reserved
- 5 = RSA/SHA1
- 6 = DSA/SHA1/NSEC3
- 7 = RSA/SHA1/NSEC3
- 8 = RSA/SHA-256
- 10 = RSA/SHA-512

Flags相对复杂一些，它是一个2位的长度，并且0-6，8-14都是反向的，bit7表明是否包含DNS Zone Key (ZSK)，bit 15表明是否包含KSK（Key signing key），所以只会有两个值，即256说明里面是ZSK，257是KSK。

## SPF

这个是邮件防Spam里重要的一员，全称Sender Policy Framework，用来认证发送者。

例子：
```
mail.example.com   TXT   v=spf1 ip4=192.0.2.0 ip4=192.0.2.1 include:examplesender.net -all
```
其中`v=spf1`表明这是一条SPF记录，`ip4`说明了可以使用这个host发送邮件的主机IP地址，`include:examplesender.net`表明可以允许第三方可以以你的域名来发送邮件，`-all` 表明不在list上的都没有被授权，应当拒收，如果是`+all`则表示任何人都可以使用该域名来发送。

## DKIM

全称为DomainKeys Identified Mail，实际上是也是保存了一个公钥，用来验证邮件的数字签名，保证邮件来路正确。

发送者在发送邮件的时候带上签名与DKIM的selector以及其他的一些字段在邮件头，接收者通过selector来请求DKIM公钥对签名进行验证。

例子：

|Name|Type|Content|TTL|
|-|-|-|-|
|selector._domainkey.example.com|TXT|v=DKIM1 p=76E629F05F70...|6000|

其中v=DKIM1表明这是一套DKIM记录，同时也可以看到，通过selector就可以组成DKIM host来请求公钥。

## DMARC

全称为Domain-based Message Authentication Reporting and Conformance， 这个记录会告诉邮件服务器在通过SPF，DKIM检测后，还需要做些什么。

例子：

```
v=DMARC1; p=quarantine; adkim=s; aspf=s;
```

v表明这条TXT记录是一条DMARC记录。
p表明在SPF与DKIM失败后，应该怎么做，选项有none，reject，以及quarantine。
adkim表明DKIM检测是strict（s），还是relax（r）。
aspf跟adkim一样，只不过是给spf设定。
其中adkim，aspf都是可选的。

## Reference:

https://www.cloudflare.com/learning/dns/dns-records/