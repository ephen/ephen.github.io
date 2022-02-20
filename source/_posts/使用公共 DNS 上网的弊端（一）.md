---
title: 使用公共 DNS 上网的弊端（一）
permalink: PublicDns_1
date: 2017-08-04
categories: 
  - 知识探索
tags: 
  - DNS
  - Public DNS
  - 公共 DNS
  - 本地 DNS
  - 递归 DNS
toc: true
toc_list_number: true
comments: true
top: false
---

作为资深网民的我们都知道，本地 DNS 是非常重要的，它能决定你到底能不能上网。

由于本地运营商 DNS 常常被大家诟病容易遭到劫持等风险，使得很多人选用公共 DNS 来作为本地上网用的 DNS ，例如 114DNS 、阿里 DNS 等。

但是，喜欢用公共 DNS 的你，知道公共 DNS 在上网时会有什么弊端吗？

<!--more-->

## 智能 DNS 和 CDN 简介

中国国内的互联网环境复杂，跨地区跨运营商之间的网络延迟较大，所以出现有智能 DNS 和 CDN 来解决这个问题。我所在的公司，北京快网就是咱们国内一家老牌的 CDN 服务提供商；[CloudXNS](https://www.cloudxns.net) 是一款智能 DNS 域名解析产品。

如果互联网站主财大气粗技术牛，在全国各地各运营商乃至全球都专门独立部署了站点，那么他可以通过智能 DNS 分区分运营商分别将域名解析到对应的站点；如果站主不具备这样多点部署的条件，那么他可以通过我们公司完成 CDN 的需求。

这就是智能 DNS 和 CDN 的最初的应用。

没错，就是【加速】。让北京联通的网民能访问到北京联通（或附近同运营商）的站点，广东电信访问到广东电信（或附近同运营商）的站点，从而达到加速的目的。

而智能 DNS 或 CDN 在调度判断网民归属时，是通过网民的本地 DNS 所属 IP 段来的。

所以，今天我要讨论到的公共 DNS 上网的弊端就是：**它一定程度上违背了智能 DNS 和 CDN 加速的规则**，变得不快了。

## 本地 DNS 和公共 DNS

网民在访问网站时，其本地 DNS 会去请求网站地址，如果本地 DNS 没有，它会再向上级 DNS 请求查询。

公共 DNS 是由一些企业提供的本地 DNS 服务，通常会给用户提供一个或多个 Anycast IP ，但实际背后有多个集群服务。

当用户上网时，客户端会向这个集群中的 A 地址请求解析，这个 A 地址被称为 DNS 入口；智能 DNS 在判断网民用户来源时，会使用 DNS 集群中的 B 地址与 IP 库进行比对，这个 B 地址被称为 DNS 出口。

所以上网过程中，用户将从 DNS 入口获取解析，NS 服务器会向 DNS 出口分配智能解析。如果用户的 DNS 入口没有请求的解析缓存时，将向上级 DNS 请求查询，最终会请求到 NS 服务器，然后用户才获取到解析结果。

那么当用户的 DNS 入口和 DNS 出口和用户的实际网络不一致时，就可能导致智能 DNS 提供的解析结果并不是最优的。

## 测试示例

北京快网官方门户网站 [www.fastweb.com.cn](http://www.fastweb.com.cn) 使用了[牛盾云安全](https://www.newdefend.com)作了页面加速和安全防护，正常情况下使用运营商 DNS 在全国各地的解析情况如下图所示：

![FastWeb 分区解析](https://pek3b.qingstor.com/imephen/20190426154730.png)

这是通过智能 DNS 和 CDN 双剑合璧的最优解析结果。

例如灰姑娘所在地是湖北武汉，当使用本地运营商 DNS 并存在缓存时，`ping www.fastweb.com.cn`时延如下：

![本地 DNS ](https://pek3b.qingstor.com/imephen/20190426154755.png)

而我在本地模拟`ping`浙江台州的牛盾节点`122.226.182.43`，会看到时延稍微长一些：

![牛盾台州](https://pek3b.qingstor.com/imephen/20190426154839.png)

那么公共 DNS 的出入口会不会跟实际网络不一致呢？

### 公共 DNS 的入口

以阿里 DNS 223.5.5.5 为例。

我在本地对阿里 DNS 的做了一个路由跟踪：

![DNS入口](https://pek3b.qingstor.com/imephen/20190426154905.png)

可以发现，阿里 DNS 的入口在浙江杭州，而我本地网络是湖北电信。

### 公共 DNS 的出口

刚刚和北京同事进行远程会议时提到这个议题，正好他的 PC 配置的是阿里公共 DNS ，我们让他访问[ CloudXNS 运维工具 - 本地 DNS 优化诊断](http://tools.cloudxns.net/index/diag)获取其出口 DNS ，得到的反馈如下：

![出口DNS](https://pek3b.qingstor.com/imephen/20190426154956.png)

本地网络北京联通，出口 DNS 竟是广东电信，这远远比我前述举例的湖北电信和浙江电信的差距大得多。

### 解析影响验证

该同事分别用阿里和运营商 DNS 对网易域名做了测试对比。

_使用阿里 DNS 测试结果：_

![阿里 DNS](https://pek3b.qingstor.com/imephen/20190426155020.png)

_使用运营商 DNS测试结果：_

![运营商 DNS](https://pek3b.qingstor.com/imephen/20190426155043.png)

使用运营商 DNS 时，`ping` 值快得多。