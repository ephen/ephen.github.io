---
title: 测试世纪互联CloudXNS公共DNS对edns-client-subnet的支持
permalink: CloudXNS-publicdns-edns
id: 202
categories:
  - 工作笔记
date: 2015-07-29 11:33:24
tags:
  - CloudXNS
  - 公共DNS
  - EDNS
---

## 什么是edns-client-subnet？

edns-client-subnet是Google公司提交的一份扩展协议，协议允许DNS resolver传递用户的ip地址给authoritative DNS server。[文档地址](https://tools.ietf.org/html/draft-vandergaast-edns-client-subnet-01)

在没有这个扩展协议的过去，网民用户在上网时，权威DNS是根据其本地DNS（即网民用户电脑上配置的那个）的出口ip地址判断来路后返回该条线路对应的解析结果。

那么就存在一个问题，用户可能选用的本地DNS并不是当地提供的。他们在A地使用B地的DNS来访问网站，得到的便会是对应B地的解析结果。这个结果一般就不会是最优的，极大的影响用户的上网体验。

<!--more-->

## 开始测试

### 一、初始配置

利用CloudXNS的私有线路配合完成网站的初始配置。

当前网络环境：武汉电信线路

公网IP地址：59.173.242.206

配置私有线路qa-test的ip段：59.173.242.206-59.173.242.206，如图：

![私有线路ip段](https://ww3.sinaimg.cn/large/5ebd877egw1f62h564n45j20fa062wel.jpg)

到CloudXNS域名管理首页开启私有线路：

![开启私有线路](https://ww1.sinaimg.cn/large/5ebd877egw1f62h562bgjj20lu02i0sq.jpg)

www.chinatesters.cn的预期初始配置如图所示（全网默认、湖北电信、qa-test）：

![域名解析配置](https://ww1.sinaimg.cn/large/5ebd877egw1f62h566o4pj20ti0b0q44.jpg)

### 二、使用支持edns-client-subnet的dig命令验证支持效果

如下图，执行命令`dig @124.251.124.251 www.chinatesters.cn +client=59.173.242.206`查看结果。

&nbsp;

从执行结果上来看：

> 1. 返回的CNAME记录为www.cloudxns.net而不是www.chinatesters.com
> 
> 2. 带有Client-Subnet标记，并且正确的对应ip为59.173.242.206
>

### 三、网站访问

如图所示，配置本地DNS为世纪互联公共DNS：124.251.124.251

![本地DNS配置](https://ww1.sinaimg.cn/large/5ebd877egw1f62h565kshj20bi0by3zq.jpg)

打开浏览器访问[http://www.chinatesters.cn](http://www.chinatesters.cn)

Bingo！访问内容是我配置的CloudXNS官方网站哟！

&nbsp;

## edns-client-subnet的实际作用

个人认为，虽说edns-client-subnet是否支持并不影响整个互联网的环境，但其还是有很大的实际作用的。

> 1. 如果CDN厂商使用的local dns支持此协议，那么站长们的网站加速效果会相对更明显；
> 
> 2. 如果路由器厂商内置的默认DNS支持此协议，那么无需在产品派发全国/全球各地时检查和更换为当地的DNS； 
> 
> 3. 如果您用的DNS支持此协议，既无劫持又能精准上网，何乐而不为？

当然，要能享受到此优势，还需要站长们使用的权威DNS解析也支持edns-client-subnet并且有丰富的线路来支撑整个全网络的配置。毫无疑问，CloudXNS完美的满足此需求。站长们使用CloudXNS权威解析，CDN厂商和用户们使用世纪互联公共DNS，还会有什么网络忧愁呢？

&nbsp;