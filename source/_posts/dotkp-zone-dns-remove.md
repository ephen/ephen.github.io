---
title: 隔壁粗事了！朝鲜顶级域 .kp 域名 DNS 数据配置可被转移
date: 2016-09-22
categories:
  - [当时就被震惊]
  - [知识探索]
tags:
  - 朝鲜
  - 顶级域
  - DNS
  - 域转移
  - bind
toc: true
---

9 月 20 日上午 10 点左右，朝鲜的一台顶级域名服务器 `ns2.kptc.kp` 不小心配置[允许全球 DNS 区域转移](https://github.com/mandatoryprogrammer/NorthKoreaDNSLeak)，任何人都可以向这台域名服务器发出区域转移请求，获取到一份朝鲜的顶级 DNS 数据拷贝。朝鲜泄漏的 DNS 数据显示，它的互联网/局域网规模确实非常小，域名寥寥无几。

<!--more-->

DNS 数据披露了 28 个可访问的 `.KP` 域名：

```
airkoryo.com.kp.
cooks.org.kp.
friend.com.kp.
gnu.rep.kp.
kass.org.kp.
kcna.kp.
kiyctc.com.kp.
knic.com.kp.
koredufund.org.kp.
korelcfund.org.kp.
korfilm.com.kp.
ma.gov.kp.
masikryong.com.kp.
naenara.com.kp.
nta.gov.kp.
portal.net.kp.
rcc.net.kp.
rep.kp.
rodong.rep.kp.
ryongnamsan.edu.kp.
sdprk.org.kp.
silibank.net.kp.
star-co.net.kp.
star-di.net.kp.
star.co.kp.
star.edu.kp.
star.net.kp.
vok.rep.kp.
```

嗯，三胖家的域名果然有够少。（至于他们国家的那些“机密信息”，本宝宝着实不感兴趣

各位一定要警惕 DNS 数据安全，配置的严谨性不能忽视。

----

下面分享一点相关知识。

## 关于 DNS 区域传送（ DNS zone transfer ）

它指的是一台备用服务器使用来自主服务器的数据刷新自己的域（ zone ）数据库。

这为运行中的 DNS 服务提供了一定的冗余度，其目的是为了防止主的域名服务器因意外故障变得不可用时影响到整个域名的解析。

一般来说， DNS 区域传送操作只在网络里真的有备用域名 DNS 服务器时才有必要用到，但许多 DNS 服务器却被错误地配置成只要有 client 发出请求，就会向对方提供一个 zone 数据库的详细信息，所以说**允许不受信任的因特网用户执行 DNS 区域传送（ zone transfer ）操作是后果最为严重的错误配置之一**。

区域传送漏洞的危害：

> 黑客可以快速的判定出某个特定 zone 的所有主机，收集域信息，选择攻击目标，找出未使用的 IP 地址，黑客可以绕过基于网络的访问控制。

## DNS zone transfer 的正确配置

要知道，区域传送是 DNS 常用的功能，只有正确的配置才将发挥有利的作用。

为避免被利用该漏洞，一般常用两种方法：

### 方法一：严格限制允许区域传送（zone transfer）的主机IP

> 针对 bind ，可以在 Global 选项或 zone 选项中增加 `allow-transfer` 参数来控制

> 例如：

```
allow-transfer ｛192.168.1.1; 202.103.24.68;｝;
```

### 方法二：使用 TSIG key 来严格定义区域传送的关系

> 使用基于 IP 地址的访问控制列表可能会被某些“意志坚定”黑客绕过，最好加上 TSIG key 的配置

> 例如：

```
allow-transfer ｛key "dns1-slave1"; key "dns1-slave2";｝;
```

## 结语

> 每次事件都是一次学习的机会，我有所学习，相信其他人也是。