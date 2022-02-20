---
title: 什么是 DNSSEC ？ DNSSEC 的概念及作用
permalink: dnssec
date: 2016-01-29
categories: 
  - 知识探索
tags: 
  - DNSSEC
  - DNS
  - 安全
  - DNS 污染
  - 域名攻击
  - 域名劫持
toc: true
comments: true
top: false
---

我们知道，过去由于环境良好，互联网先驱者们“ Too simple, Too Naive ”，他们设计了域名系统（ Domain Name System ，DNS ）。就像互联网的其他协议或系统一样，在过去可信的、纯净的环境里运行得很好。

而随着互联网的发展，如今充斥着各种欺诈、攻击，使得今天的互联网环境异常复杂，DNS 协议的脆弱性也就浮出水面。

DNS 最大的缺陷就是解析的请求者无法验证它所收到的应答信息的真实性，一些坏人对 DNS 的攻击可能导致互联网大面积的瘫痪，这种事件在国内外都屡见不鲜。

<!--more-->

## DNSSEC 是什么？

### 定义

DNSSEC 全称 Domain Name System Security Extensions ，即 DNS 安全扩展，是由 IETF 提供的一系列 DNS 安全认证的机制（可参考 [RFC2535](https://tools.ietf.org/html/rfc2535) ）。它提供一种可以验证应答信息真实性和完整性的机制，利用密码技术，使得域名解析服务器可以验证它所收到的应答(包括域名不存在的应答)是否来自于真实的服务器，或者是否在传输过程中被篡改过。

通过 DNSSEC 的部署，可以增强对 DNS 域名服务器的身份认证，进而帮助防止 DNS 缓存污染等攻击。

DNSSEC 给解析服务器提供了防止上当受骗的武器，是实现 DNS 安全的重要一步和必要组成部分。

### 原理

DNSSEC 通过公钥密码技术对 DNS 中的信息创建密码签名，为 DNS 信息同时提供认证和信息完整性检查，它的实施步骤如下：

1. DNS 服务器收到 DNS 查询请求后，用散列函数将要回复 DNS 报文的内容进行散列运算，得到“内容摘要”，使用私匙加密后再附加到 DNS 报文中；
2. DNS 查询请求者接收到报文后，利用公匙解密收到的“内容摘要”，再利用散列函数计算一次 DNS 查询请求报文中的“内容摘要”，两者对比；
3. 若相同，就可以确认接收到的 DNS 信息是正确的 DNS 响应；若验证失败，则表明这一报文可能是假冒的，或者在传输过程、缓存过程中被篡改了。

## DNSSEC 相关术语

### DNSKEY/RRSIG/DS/NSEC 资源记录

为了实现资源记录的签名和验证，DNSSEC增加了四种类型的资源记录：

```
  RRSIG 记录： Resource Record Signature ，存储资源记录集合（ RRSets ）的数字签名
  DNSKEY 记录： DNS Public Key ，存储公开密钥
  DS 记录： Delegation Signer ，存储 DNSKEY 的散列值，用于验证 DNSKEY 的真实性，从而建立一个信任链
  NSEC 记录： Next Secure ，用于应答那些不存在的资源记录
```

### 信任锚（ Trust anchor ）

DNSSEC 需要一个信任链，必须有一个或多个开始就信任的公钥（或公钥的散列值），称这些初始信任的公开密钥或散列值为“信任锚（ Trust anchors ）”。

### DLV(DNSSEC Lookaside Validation)

即 DNSSEC 旁路信任源。

### KSK/ZSK 与 Signed zone

权威域的管理员通常用两个密钥配合完成对区数据的签名，一个是 Zone-Signing Key(ZSK) ，另一个是 Key-Signing Key(KSK) 。

ZSK 用于签名区数据，而 KSK 用于对 ZSK 进行签名，被签名的区数据则被称为 Signed zone 。

## DNSSEC的应用场景

### 一、配置安全的域名解析服务器（ Resolver ）

即用户使用的 DNS ，常被称为本地 DNS （ Local DNS ）或公众 DNS。

通过该 Resolver 服务器可以保护使用它的用户，防止被 DNS 欺骗攻击， DNSSEC 在这里只涉及数字签名的验证工作。

### 二、配置安全的权威域名服务器（ Name Server ）

也被称为授权 DNS ，例如 CloudXNS 。

DNSSEC 对权威域的资源记录进行签名，保护服务器不被域名欺骗攻击。