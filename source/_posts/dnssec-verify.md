---
title: 如何验证 DNS 服务器是否支持 DNSSEC ？
date: 2016-01-29
categories: 
  - 知识探索
tags: 
  - DNSSEC
  - DNS
  - 安全
  - 域名劫持
toc: true
comments: true
top: false
original: false
---

众所周知，DNSSEC 对于 DNS 劫持虽然有极强的防御性，但由于被劫持的数据都会在验证失败后被丢弃，因而并不能让我们在 DNS 劫持的情况下获得正确的解析结果。（请先参考：[什么是 DNSSEC ？ DNSSEC 的概念及作用](/2016/dnssec/)）

所以，我们需要 DNS 安全传输的前提是加固客户端到 DNS 服务器之间的网络连接的安全性。

也就是说， DNSSEC 除了客户端支持之外，更重要的是 DNS 服务器本身必须有部署 DNSSEC 的支持。

否则：

1. 客户端没有配置 dnssec-check-unsigned 而 DNS 服务器支持 DNSSEC ，此时 DNSSEC 默认信赖所有没签名的数据，等于没有提供任何防护；
2. 客户端配置启用了 dnssec-check-unsigned 而 DNS 服务器不支持 DNSSEC ， dnsmasq 因为不能获取任何 DNSSEC 签名信息导致不信任全部解析结果，进而使得整个 DNS 解析服务完全无法工作。

那么...

<!--more-->

## 如何判定一台 DNS 服务器是否支持 DNSSEC 呢？

很简单，还是用 dig 命令。 [dig 命令参考手册](http://linux.51yip.com/search/dig)

### 方法 A ：检查一个有 DNSSEC 签名的域名的 RRSIG(Resource Record Signature)

为了让结果看得更清楚，我们找一个配置了 DNSSEC 签名的域名（ paypal.com ），一个支持 DNSSEC 的 DNS 服务器（ 4.2.2.4 ），和一个不支持  DNSSEC 的 DNS 服务器（ 114.114.114.114 ）。

_使用支持 DNSSEC 的 4.2.2.4 查询结果如下：_

```
root@OpenWrt:~# dig paypal.com +dnssec @4.2.2.4

; <<>> DiG 9.9.4 <<>> paypal.com +dnssec @4.2.2.4
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 48979
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 4096
;; QUESTION SECTION:
;paypal.com.                    IN      A

;; ANSWER SECTION:
paypal.com.             293     IN      A       66.211.169.3
paypal.com.             293     IN      A       66.211.169.66
paypal.com.             293     IN      RRSIG   A 5 2 300 20140728175119 20140628172604 11811 paypal.com. ka3J7csLBUiZIrh7YTKJ7eUBzpACe7jmr6M2wURsNCQ/dFjB9Jl018OZ 6i3BzzSYqSS2jw9TmVZMKxRLH3cmt5jc1BNI6Q9uB46DLpJJoAmXQ1rQ ss37Mb4BlK8dD4rxLJmEJh19+Kg8xXxE8iGYwLM7tkyayIjVdxbt80TE vgg=

;; Query time: 224 msec
;; SERVER: 4.2.2.4#53(4.2.2.4)
;; WHEN: Tue Jul 15 21:49:25 CST 2014
;; MSG SIZE  rcvd: 241
```

_使用不支持 DNSSEC 的 114.114.114.114 查询结果如下：_

```
root@OpenWrt:~# dig paypal.com +dnssec @114.114.114.114

; <<>> DiG 9.9.4 <<>> paypal.com +dnssec @114.114.114.114
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12717
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;paypal.com.                    IN      A

;; ANSWER SECTION:
paypal.com.             300     IN      A       66.211.169.3
paypal.com.             300     IN      A       66.211.169.66

;; Query time: 577 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: Tue Jul 15 21:49:57 CST 2014
;; MSG SIZE  rcvd: 60
```

我们可以看到，支持 DNSSEC 的服务器多返回了一串非常长的 RRSIG(Resource Record Signature) ，这就是 DNSSEC 中非常重要的数据签名。

而不支持 DNSSEC 的服务器就完全没有返回这些信息了，由此我们可以很容易区分一台 DNS 服务器是否支持 DNSSEC 。

不过，目前配置了 DNSSEC 签名的域名非常少，据我所知一般有些国外政府域名有，当然 paypal 也有，而国内几乎没有（国内的支付宝并没有）。

如果一个域名本身就没有配置 DNSSEC 签名，那么无论你是通过什么样的 DNS 服务器查询 dnssec 数据，结果都是一样的。

那如果我们不知道哪些域名配置了 DNSSEC 签名，难道就无法验证 DNS 服务器是否支持 DNSSEC 了？

当然不是！我们有...

### 方法 B ：自残测试法！

由于当我们启用了 Dnsmasq 的 dnssec–check–unsigned 选项， Dnsmasq 就会检查 DNS 返回的数据是否经过签名。（注意，这个签名是 DNS 服务器本身对发出的数据包的签名，用于证明这个数据包确实发送自我这个服务器且没有被修改，和域名本身是否有 DNSSEC 签名毫无关系。）

如果 DNS 服务器返回的数据包没有经过 DNS 服务器签名，那么 Dnsmasq 就会丢弃这个解析结果，因为启用这个选项后它只接受签名过的数据。

所以，做法就是：

1. 把 Dnsmasq 的 DNS 设置为你要测试的那个 DNS 服务器（注意！只写这一个 DNS ，别的都不写）；
2. 启用 Dnsmasq 的 dnssec-check-unsigned 选项；
3. 域名解析验证。

如果你还可以解析域，说明这个 DNS 支持 DNSSEC ；如果你发现什么域名都解析不了了，那么就是因为这个 DNS 服务器不支持 DNSSEC ，导致 Dnsmasq 不信任返回的所有数据了。

国内目前支持 DNSSEC 的服务器，不能说稀少吧，只能说我一个都没发现。而国外的公共 DNS ，我们熟知的那些除了 OpenDNS 有它自己的一套 DNScrypt 加密体系而没有采用 DNSSEC 其他倒基本都是支持的。

_我测试过支持 DNSSEC 的国外公共 DNS 如下：_

```
Google Public DNS： 8.8.8.8 | 8.8.4.4
Norton DNS： 199.85.126.20 | 199.85.127.20
Comodo DNS： 8.26.56.26 | 8.20.247.20
DNS Advantage： 156.154.70.1 | 156.154.71.1
NTT DNS： 129.250.35.250 | 129.250.35.251
Verizon DNS： 4.2.2.1 | 4.2.2.2 | 4.2.2.3 | 4.2.2.5 | 4.2.2.6
```

目前，由于国内并没有支持 DNSSEC 的公共 DNS ，也几乎没有配置了 DNSSEC 签名的域名，因此 CloudXNS 暂无支持 DNSSEC 的打算。

不过，在未来产品优化提升足够美好的时候，不排除会率先在国内支持 DNSSEC 的可能。

原文系转载，出处：LifeTyper

灰姑娘在此基础上略有改动。