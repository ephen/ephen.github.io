---
title: DNS 资源记录（ Resource Record ，简称 RR ）介绍
date: 2016-10-08
categories:
    - 知识探索
tags:
    - DNS
    - 记录类型
    - SOA
toc: true
---

## DNS 资源记录简介

DNS server 内的每一个域名都有自己的域文件（ zone file ）， zone file 是由多个记录组成的，每一个记录就被称为资源记录（ Resource Record ，简称 RR ）。

当在设定 DNS 域名解析、反向解析及其他的管理目的时，往往需要使用不同类型的 RR ，也就是我们常说的记录类型。

上述 CloudXNS 的记录类型列表中，除 AX 、 CNAMEX 、 LINK 、 DR301X （ 301 跳转）、 DR302X （ 302 跳转）以及 DRHIDX （隐式跳转）为 CloudXNS 自研的扩展解析记录类型之外，其余都是 DNS 体系中常见的标准 RR 类型。

但除了我们系统中支持的这些类型外，事实上还有一些不太常用的目前 CloudXNS 暂不支持的其他 RR 类型，感兴趣的朋友可以和灰姑娘一起学习下。

<!--more-->

## DNS 体系中的标准 RR 类型及表示法集合

| 类型 | RFC 来源 | 描述 | 功能 |
|:----:|:------:|:-----|:-----|
| A | [RFC 1035](https://tools.ietf.org/html/rfc1035) | IP 地址记录 | 传回一个 32 比特的 IPv4 地址，最常用于映射主机名称到IP地址，但也用于 DNSBL（[RFC 1101](https://tools.ietf.org/html/rfc1101)）等。|
| AAAA | [RFC 3596](https://tools.ietf.org/html/rfc3596) | IPv6 IP 地址记录 | 传回一个 128 比特的 IPv6 地址，最常用于映射主机名称到 IP 地址。 |
| AFSDB | [RFC 1183](https://tools.ietf.org/html/rfc1183)| AFS 文件系统 | （ Andrew File System ）数据库核心的位置，于域名以外的 AFS 客户端常用来联系 AFS 核心。这个记录的子类型是被过时的 [DCE/DFS](https://zh.wikipedia.org/wiki/DCE/DFS)（ DCE Distributed File System ）所使用。 |
| APL | [RFC 3123](https://tools.ietf.org/html/rfc3123) | 地址前缀列表 | 指定地址列表的范围，例如： CIDR 格式为各个类型的地址（试验性）。 |
| CERT | [RFC 4398](https://tools.ietf.org/html/rfc4398) | 证书记录 | 存储 PKIX 、 SPKI 、PGP 等。 |
| CNAME | [RFC 1035](https://tools.ietf.org/html/rfc1035) | 规范名称记录 | 一个主机名字的别名：域名系统将会继续尝试查找新的名字。 |
| DHCID | [RFC 4701](https://tools.ietf.org/html/rfc4701) | DHCP （动态主机设置协议）识别码 | 用于将 FQDN 选项结合至 DHCP 。 |
| DLV | [RFC 4431](https://tools.ietf.org/html/rfc4431) | DNSSEC （域名系统安全扩展）来源验证记录 | 为不在 DNS 委托者内发布 DNSSEC 的信任锚点，与 DS 记录使用相同的格式，[RFC 5074](https://tools.ietf.org/html/rfc5074) 介绍了如何使用这些记录。 |
| DNAME |[RFC 2672](https://tools.ietf.org/html/rfc2672) | 代表名称 | DNAME 会为名称和其子名称产生别名，与 CNAME 不同，在其标签别名不会重复。但与 CNAME 记录相同的是， DNS 将会继续尝试查找新的名字。 |
| DNSKEY | [RFC 4034](https://tools.ietf.org/html/rfc4034) | DNS 关键记录 | 于 DNSSEC 内使用的关键记录，与 KEY 使用相同格式。 |
| DS | [RFC 4034](https://tools.ietf.org/html/rfc4034) | 委托签发者 | 此记录用于鉴定 DNSSEC 已授权区域的签名密钥。 |
| HIP | [RFC 5205](https://tools.ietf.org/html/rfc5205) | 主机鉴定协议 | 将端点标识符及 IP 地址定位的分开的方法。 |
| IPSECKEY | [RFC 4025](https://tools.ietf.org/html/rfc4025) | IPSEC 密钥 | 与 IPSEC 同时使用的密钥记录。 |
| KEY | [RFC 2535](https://tools.ietf.org/html/rfc2535) 和 [RFC 2930](https://tools.ietf.org/html/rfc2930)| 关键记录 | 只用于 SIG(0) （ [RFC 2931](https://tools.ietf.org/html/rfc2931) ）及 TKEY （ [RFC 2930](https://tools.ietf.org/html/rfc2930) 。 [RFC 3455](https://tools.ietf.org/html/rfc3455) 否定其作为应用程序键及限制 DNSSEC 的使用。 [RFC 3755](https://tools.ietf.org/html/rfc3755) 指定了 DNSKEY 作为 DNSSEC 的代替。 |
| LOC | [RFC 1876](https://tools.ietf.org/html/rfc1876) | 位置记录 | 将一个域名指定地理位置。 |
| MX | [RFC 1035](https://tools.ietf.org/html/rfc1035) | 电邮交互记录 | 引导域名到该域名的邮件传输代理（ MTA, Message Transfer Agents ）列表。 |
| NAPTR | [RFC 3403](https://tools.ietf.org/html/rfc3403) | 命名管理指针 | 允许基于正则表达式的域名重写使其能够作为 URI 、进一步域名查找等。 |
| NS | [RFC 1035](https://tools.ietf.org/html/rfc1035) | 名称服务器记录 | 委托 DNS 域（ DNS zone ）使用已提供的权威域名服务器。 |
| NSEC | [RFC 4034](https://tools.ietf.org/html/rfc4034) | 下一代安全记录 | DNSSEC 的一部分; 用来验证一个未存在的服务器，使用与 NXT（已过时）记录的格式。 |
| NSEC3 | [RFC 5155](https://tools.ietf.org/html/rfc5155) | NSEC 记录第三版 | 用作允许未经允许的区域行走以证明名称不存在性的 DNSSEC 扩展。 |
| NSEC3PARAM | [RFC 5155](https://tools.ietf.org/html/rfc5155) | NSEC3 参数 | 与 NSEC3 同时使用的参数记录。 |
| PTR | [RFC 1035](https://tools.ietf.org/html/rfc1035) | 指针记录 | 引导至一个规范名称（ Canonical Name ）。与 CNAME 记录不同， DNS “不会”进行进程，只会传回名称。最常用来运行反向 DNS 查找，其他用途包括引作 DNS-SD。 |
| RRSIG | [RFC 4034](https://tools.ietf.org/html/rfc4034) | DNSSEC 证书 | DNSSEC 安全记录集证书，与 SIG 记录使用相同的格式。 |
| RP | [RFC 1183](https://tools.ietf.org/html/rfc1183) | 负责人 | 有关域名负责人的信息，电邮地址的 **`@`** 通常写为 **`a`**。 |
| SIG | [RFC 2535](https://tools.ietf.org/html/rfc2535) | 证书 | SIG(0) （ [RFC 2931](https://tools.ietf.org/html/rfc2931) ）及 TKEY （ [RFC 2930](https://tools.ietf.org/html/rfc2930) ）使用的证书。 [RFC 3755](https://tools.ietf.org/html/rfc3755) designated RRSIG as the replacement for SIG for use within DNSSEC. |
| SOA | [RFC 1035](https://tools.ietf.org/html/rfc1035) | 权威记录的起始 | 指定有关 DNS 区域的权威性信息，包含主要名称服务器、域名管理员的电邮地址、域名的流水式编号、和几个有关刷新区域的定时器。 |
| SPF | [RFC 4408](https://tools.ietf.org/html/rfc4408) | SPF 记录 | 作为 SPF 协议的一部分，优先作为先前在 TXT 存储 SPF 数据的临时做法，使用与先前在 TXT 存储的格式。 |
| SRV | [RFC 2782](https://tools.ietf.org/html/rfc2782) | 服务定位器 | 广义为服务定位记录，被新式协议使用而避免产生特定协议的记录，例如： MX 记录。 |
| SSHFP | [RFC 4255](https://tools.ietf.org/html/rfc4255) | SSH 公共密钥指纹 | DNS 系统用来发布 SSH 公共密钥指纹的资源记录，以用作辅助验证服务器的真实性。 |
| TA | 无 | DNSSEC 信任当局 | DNSSEC 一部分无签订 DNS 根目录的部署提案，，使用与 DS 记录相同的格式。 |
| TKEY | [RFC 2930](https://tools.ietf.org/html/rfc2930) | 秘密密钥记录 | 为 TSIG 提供密钥材料的其中一类方法，that is 在公共密钥下加密的 accompanying KEY RR。 |
| TSIG | [RFC 2845](https://tools.ietf.org/html/rfc2845) | 交易证书 | 用以认证动态更新（ Dynamic DNS ）是来自合法的客户端，或与 DNSSEC 一样是验证回应是否来自合法的递归名称服务器。 |
| TXT | [RFC 1035](https://tools.ietf.org/html/rfc1035) | 文本记录 | 最初是为任意可读的文本 DNS 记录。自 1990 年起，些记录更经常地带有机读数据，以 [RFC 1464](https://tools.ietf.org/html/rfc1464) 指定： opportunistic encryption 、 Sender Policy Framework （虽然这个临时使用的 TXT 记录在 SPF 记录推出后不被推荐）、 DomainKeys 、 DNS-SD 等。 |

## 其他伪资源记录类型

| 类型 | RFC来源 | 描述 | 功能 |
|:----:|:------:|:-----|:-----|
| * | [RFC 1035](https://tools.ietf.org/html/rfc1035) | 所有缓存的记录 | 传回所有服务器已知类型的记录。如果服务器未有任何关于名称的记录，该请求将被转发。而传回的记录未必完全完成，例如：当一个名称有 A 及 MX 类型的记录时，但服务器已缓存了 A 记录，就只有 A 记录会被传回。 |
| AXFR | [RFC 1035](https://tools.ietf.org/html/rfc1035) | 全域转移 | 由主域名服务器转移整个区域文件至二级域名服务器。 |
| IXFR | [RFC 1995](https://tools.ietf.org/html/rfc1995) | 增量区域转移 | 请求只有与先前流水式编号不同的特定区域的区域转移。此请求有机会被拒绝，如果权威服务器由于配置或缺乏必要的数据而无法履行请求，一个完整的（ AXFR ）会被发送以作回应。 |
| OPT | [RFC 2671](https://tools.ietf.org/html/rfc2671) | 选项 | 这是一个“伪 DNS 记录类型”以支持 EDNS 。 |

## 什么是 SOA ？

SOA ，即 Start Of Authority ，放在 zone file 中，用于描述这个 zone 负责的 name server ， version number… 等资料，以及当 slave server 要备份这个 zone 时的一些参数。

每个 zone file 中必须有且仅有一条 SOARR ，并在 zone file 中作为第一条资源记录保存。

举个栗子：

```
@ IN SOA lv3ns1.ffdns.net. webmaster.ffdns.net. (
    2009092868 ; Serial
    604800 ; Refresh
    3600 ; Retry
    2419200 ; Expire
    3600 ) ; Minimum 
```

> **第一行：** `@` 指代该 zone ；  `lv3ns1.ffdns.net.` 是该 zone 的授权主机； `webmaster.ffdns.net.` 代表 `webmaster@ffdns.net` ，即该 zone 的管理者信箱。
>
> **Serial：**代表 zone file 的版本，每当 zone file 内容有变动，name server 管理者就应该增加这个号码，因为 slave 会将这个号码与其 copy 的那份比对以便决定是否要再 copy 一次（即进行 zone transfer ）。
>
> **Refresh：** slave server 每隔这段时间(秒)，就去检查 master server 上的 serial number 。
>
> **Retry：**当 slave server 无法和 master 进行 serial check 时，要每隔这段时间（秒） retry 一次。
>
> **Expire：**当时间超过 Expire 所定的秒数而 slave server 都无法和 master 取得连络，那么 slave 会删除自己的这份 copy 。
>
> **Minimum：**代表这个 zone file 中所有 record 的内定的 TTL 值，也就是其它的 DNS server cache 这笔 record 时，最长不应该超过这个时间。

## 参考资料

1. [ DNS 線上教學研究計畫 ](http://dns-learning.twnic.net.tw/index.html)
2. [ SimpleDNS.com - Help ](http://www.simpledns.com/help)
3. [ DNS 资源记录( Resource Record )介绍 ](https://www.cnblogs.com/sddai/p/5703394.html)