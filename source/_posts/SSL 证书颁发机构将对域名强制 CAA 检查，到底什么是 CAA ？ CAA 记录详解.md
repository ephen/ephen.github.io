---
title:  SSL 证书颁发机构将对域名强制 CAA 检查，到底什么是 CAA ？ CAA 记录详解
permalink: dnsrr-caa
date: 2017-06-13
categories:
    - 知识探索
tags:
    - DNS
    - https
    - SSL
    - CAA
toc: true
---

## 强制 CAA 检查的提议

2017 年 3 月 7 日， CA|B Forum （一个全球证书颁发机构与浏览器的技术论坛）发起了一项关于对域名强制检查 CAA 的一项提议的投票，获得 187 票支持，投票有效，提议通过。

提议通过后，将于 **2017 年 9 月 8 日**根据 Mozilla 的 Gervase Markham 提出的检查 CAA 记录作为基准要求来实施。

详情参见： [Ballot 187 – Make CAA Checking Mandatory](https://cabforum.org/2017/03/08/ballot-187-make-caa-checking-mandatory/)

## 什么是 CAA 记录？

CAA ，全称 Certificate Authority Authorization ，即证书颁发机构授权。它为了改善 PKI （ Public Key Infrastructure ：公钥基础设施）生态系统强度、减少证书意外错误发布的风险，通过 DNS 机制创建 CAA 资源记录，从而限定了特定域名颁发的证书和 CA （证书颁发机构）之间的联系。从此，再也不能是任意 CA 都可以为任意域名颁发证书了。

<!--more-->

关于 CAA 记录，其实早在 4 年前便在 [RFC 6844](https://datatracker.ietf.org/doc/rfc6844/) 中有定义，但由于种种原因配置该 DNS 资源记录的网站寥寥无几。如今， SSL 证书在颁发之前对域名强制 CAA 检查，就对想要 https 访问的网站域名提出了解析配置的要求。

## CAA 资源记录详解

CAA 记录可以控制单域名 SSL 证书的发行，也可以控制通配符证书。当域名存在 CAA 记录时，则只允许在记录中列出的 CA 颁发针对该域名（或子域名）的证书。

在域名解析配置中，咱们可以为整个域（如 example.com ）或者特定的子域（如 subzone.example.com ）设置 CAA 策略。当为整域设置 CAA 资源记录时，该 CAA 策略将同时应用于该域名下的任一子域，除非被已设置的子域策略覆盖。

### CAA 记录格式

根据规范（ [RFC 6844](https://www.rfc-editor.org/rfc/rfc6844.txt) ）， CAA 记录格式由以下元素组成：

```
CAA <flags> <tag> <value>
```

释义：

|元素|说明|
|----|----|
| CAA | DNS 资源记录类型|
| `<flags>` |认证机构限制标志|
| `<tag>` |证书属性标签|
| `<value>` |证书颁发机构、策略违规报告邮件地址等|

`<flags>` 定义为 0~255 无符号整型，取值：

> Issuer Critical Flag：0
> 1~7 为保留标记

`<tag>` 定义为 US-ASCII 和 0~9 ，取值：

> CA 授权任何类型的域名证书（ Authorization Entry by Domain ） : issue
> CA 授权通配符域名证书（ Authorization Entry by Wildcard Domain ） : issuewild
> 指定 CA 可报告策略违规（ Report incident by IODEF report ） : iodef
> auth 、 path 和 policy 为保留标签

`<value>` 定义为八位字节序列的二进制编码字符串，一般填写格式为：

> [domain] [";" * 参数]

### CAA 资源记录示例

当需要限制域名 `example.com` 及其子域名可由机构 `letsencrypt` 颁发不限类型的证书，同时可由 `Comodo` 颁发通配符证书，其他一律禁止，并且当违反配置规则时，发送通知邮件到 `example@example.com` 。

配置如下（为便于理解，二进制 Value 值已经过转码，下同）：

```
example.com.  CAA 0 issue "letsencrypt.org"
example.com.  CAA 0 issuewild "comodoca.com"
example.com.  CAA 0 iodef "mailto:example@example.com"
```

如果子域名有单独列出证书颁发要求，例如：

```
example.com.  CAA 0 issue "letsencrypt.org"
alpha.example.com.  CAA 0 issue "comodoca.com"
```

那么，因子域策略优先，所以只有 Comodo 可以为域名 `alpha.example.com.` 颁发证书。

### CAA 记录查询

CAA 记录是一个相对较新的资源记录，目前很多工具并不支持。以 `dig` 为例，不能适用其标准语法。若需要查询 CAA 记录， `dig` 时需要直接指定 RR 类型（ type257 ）。

例如：

```
$ dig google.com type257

; <<>> DiG 9.8.3-P1 <<>> google.com type257
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64266
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;google.com.            IN  TYPE257

;; ANSWER SECTION:
google.com.     86399   IN  TYPE257 \# 19 0005697373756573796D616E7465632E636F6D

;; Query time: 51 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Thu Dec 29 21:07:18 2016
;; MSG SIZE  rcvd: 59
```

该查询的输出是二进制编码记录，需要转码才能知道具体 CAA 策略。

## 互联网证书环境前景剖析

> 以下所有言论仅代表个人观点。

CAA 记录规范的提出已 4 年之久，但从未被普及；如今 CA|B Forum 关于强制 CAA 检查的提议亦结束 3 月有余，支持检查 CAA 的证书颁发机构依旧寥寥无几，支持配置 CAA 记录的 DNS 厂商更是少之又少。因而个人认为，这项规范的实施进程预计还是会挺艰难的。

不过从互联网安全发展来看，互联网受众越来越大，概念、架构越来越复杂，社会的重视程度一直在稳步提高。全面 https 也推行有一段时间了，强制 CAA 的普及大概只是时间问题。