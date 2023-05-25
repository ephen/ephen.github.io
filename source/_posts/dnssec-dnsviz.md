---
title: 使用网页工具 DNSViz 检查域名 DNSSEC 配置
date: 2016-10-13
categories:
  - 工具介绍
tags:
  - DNSViz
  - DNSSEC
  - 域名劫持
  - 安全
---

## 关于 DNSSEC

DNSSEC 全称 Domain Name System Security Extensions，即 DNS安全扩展，是由 Internet 工程任务组 （ IETF ）提供的一系列 DNS 安全认证的机制（可参考 [RFC2535](https://tools.ietf.org/html/rfc2535) ）。

它是对 DNS 提供给 DNS 客户端（解析器）的 DNS 数据来源进行认证，并验证不存在性和校验数据完整性验证，但不提供或机密性和可用性。[来源：域名系统安全扩展 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%BB%9F%E5%AE%89%E5%85%A8%E6%89%A9%E5%B1%95)

一些基础的相关介绍可参考我在 CloudXNS 官网发的文章（现均已迁移至本博）：

1. [什么是 DNSSEC ？ DNSSEC 的概念及作用](/2016/dnssec/)
2. [如何验证 DNS 服务器是否支持 DNSSEC ？](/2016/dnssec-verify/)

通俗点讲， DNSSEC 的主要作用就是防止 DNS劫持，从而保证 DNS安全。

但在这个保证安全的过程中，DNSSEC 的 RR 和 RRSIG 的传输过程都是未加密的（即：不提供机密性），我们通过一些嗅探工具可以读取 RRSIG 记录以及由 DS 记录建立的信任链。

今天我们就来聊聊 DNSSEC 信任链的那些事。

<!--more-->

## DNSSEC 相关的资源记录

以国际知名的支付平台 Paypal 为例。

Paypal.com 域名配置了 DNSSEC 签名，以下是使用支持 DNSSEC 的 DNS服务器 4.2.2.4 查询 A记录 的解析结果：

```
~ dig paypal.com +dnssec @4.2.2.4

;; QUESTION SECTION:
;paypal.com.			IN	A

;; ANSWER SECTION:
paypal.com.		204	IN	A	64.4.250.23
paypal.com.		204	IN	A	64.4.250.24
paypal.com.		204	IN	RRSIG	A 5 2 300 20161023213752 20160923205517 11811 paypal.com. lHYZ/5xp9uCfUKGVbvLWr3dA3Ri8X3G1O9fdi6ludo1qYDbYFFyyAdWp szGoszxSRE0jT1ggUOXufBv7nEF++ODkzXB4U4QmKt6ooqQRG8TMDsGw WbJtAgMq0ul8PRPZz8MvkZGxucB2pC4+HwtQaJ/g43sxP9ouHsCePnU7 +H4=

```

### RRSIG：

RRSIG 资源记录值很长，乍一看是一堆乱码，咱们还是对照格式来看：

> `A` - 记录类型
> 
> `5` - 算法类型 (参考附录「算法类型列表」)
> 
> `2` - 标签 (泛解析中原先 RRSIG 记录的名称)
> 
> `300` - 原 TTL 大小
> 
> `20161023213752` - 签名失效时间
> 
> `20160923205517` - 签名签署时间
> 
> `11811` - Key 标签 (一个简短的数值，用来迅速判断应该用那个 DNSKEY 记录来验证)
> 
> `paypal.com.` - 签名名称 (用于验证该签名的 DNSKEY 名称)
> 
> "...太长了就不写了..." - 加密签名
> 

----

DNSKEY 和 DS （Delegation Signer）记录也都看下。

```
~ dig paypal.com +dnssec @4.2.2.4 DNSkey

;; QUESTION SECTION:
;paypal.com.			IN	DNSKEY

;; ANSWER SECTION:
paypal.com.		25	IN	DNSKEY	256 3 5 AwEAAc/7r7w6qEg59vzyfcKgIm7K3h43tglKOJjoFyK5lxhl2e7vh8Cw vj3cwKQHccsvWuvAK0ummTSysxZT0JJgw8gkhIGGiZ63MJTcbocDezgZ iA/q4ejpjdrj27gs5mCHAsyC9BAVZiysfIwqtRFsP0GjivNYzIc6qGWA XcAJKLLX
paypal.com.		25	IN	DNSKEY	257 3 5 AwEAAdVtmC6yOQb0+5MT5ezC9GJsCb10WkVE1qMAbilAN5KZ0/wJD+4P 1/WB7ctXC0RnEjHzVrKmLsFzRb3mpI9wm0cf5pN8BHMSVfdDpycjNyGb My7EKJ77POD7lSimJouMx5Tp+HQaJZeU0MnXJkR4qaAvShr5iNtVaopO uIXQfRDnLdDoxETw9XplIL9wkpe1gW5uQNk+Fhy4PRyf+e5yhgiZjemm RiEDJH+FxCUAf8ZV+xHSuucWKY3V1tEheptdUaIAbnCGDWGTbi9ziai6 SrLzRrrDeImwm42PkxD5cURMWqHQIBfEJlDB+koqWPm73sDPMlPgoBUy xb/w5JFLV+c=
paypal.com.		25	IN	RRSIG	DNSKEY 5 2 600 20161030075650 20160930070223 11811 paypal.com. botQ3ANf+tWS4AFxKF+xITOkAOfMaMlwSz7lucnzSKl2XUHKMCOgk3yn ux0VwzGITuwFyg61f6qlwAbKEgVZHjVq+ERwEIiUzyKMi7NUDKbTQbB/ w2+dsTwKX5A7oCOQR3gJytfw4eFbrKKXfCvkTk/WVFjdaCnh+MNDn/x+ 37Y=
paypal.com.		25	IN	RRSIG	DNSKEY 5 2 600 20161030075650 20160930070223 21037 paypal.com. H3Ms6s//q4actymLzj5xLh+ficbqet6iyzwc/eA1/KUxpQISydXTVmuT jLcn+nbILH6QXHiNZiMmkIZm2YsxYBAEa+V9botN0HqFz5iBgcaBmIAk L5xp66SZdQQyJTfWbi1ILYCiDeJmS5mkvqwdFbjIW4Cxouj8WRBPYHL2 r0k+icmq0fSI/m0ujdK0tstodc2jd2nb9Vvj1q19DooE6jcWN6KwEa3k p2pe8/nNSeCfsqqxPb0wzvj14YTpOHq3R6y1Y84T2/dOdoAKsMhKJddK l30nQOuodh7FV7AvO4uf/ewDMlMB65hEH8zwBaxhpPyGHzeWj1t/YzOW PzMWOQ==

```

### DNSKEY：

> `256` - 标识符 (Zone Key (DNSSEC密钥集) 以及 Secure Entry Point (KSK和简单密钥集))
> 
> `3` - 协议 (固定值3 向下兼容)
> 
> `5` - 算法类型 (参考附录「算法类型列表」)
> 
> "...最后的..." - 公钥内容
> 

----

```
~ dig paypal.com +dnssec @4.2.2.4 DS

;; QUESTION SECTION:
;paypal.com.			IN	DS

;; ANSWER SECTION:
paypal.com.		3553	IN	DS	21037 5 2 0DF17B28554954D819E0CEEAB98FCFCD56572A4CF4F551F0A9BE6D04 DB2F65C3
paypal.com.		3553	IN	RRSIG	DS 8 2 86400 20161017041535 20161010030535 27452 com. cDatvEwBK3yOTI2QTyX/xZBxGQMKg006mT2kzNy+olS751dbWlyETgpz gv2RyaSY9eoHVXAE+2DBD/BzKV1lV3sp1Pwgqb3PCYuJx6TAvAs7YkK6 B5uZ+prCUEKf8pHF325FLzXs7S23cZe7k1PmTfkepMTH4r0lcRYJFlP1 sog=

```

### DS：

> `21037` - Key 标签 (一个简短的数值，用来迅速判断应该用那个 DNSKEY 记录来验证)
> 
> `5` - 算法类型 (参考附录「算法类型列表」)
> 
> `2` - 摘要类型 (创建摘要值的加密散列算法)(参考附录「摘要类型列表」)
> 
> "...最后的..." - 摘要：引用的 DNSKEY记录 的加密哈希值。
> 

## DS记录 建立的 DNSSEC 信任链

DS，是 Delegation Signer 的缩写，即“委派签名者”，用于构建到子 zone 的身份验证链。

对于 Paypal.com 来说，其 DS记录值 是上一级 .com 的私钥对其 DNSKEY 进行加密后得到的。

而如果 Paypal.com 存在独立的子域时，又将利用其私钥对其子 zone 的 DNSKEY 执行加密从而生成子域的 DS记录值 。

下图是用 DNSViz 查询到的 Paypal.com 的信任链：

![Paypal.com 的 DNSSEC 信任链](https://iephen.pek3b.qingstor.com/b_image/20190426155405.png)

图中展示的三个框，从上至下依次代表域名`.`、`com.`和`paypal.com.`，每个框中间的内容分别对应这三个 zone 的 DNSKey 和 DS 的传递关系。

可以通过点击<http://dnsviz.net/d/paypal.com/dnssec/>查看更多细节。

## DNSSEC 配置检查神器：DNSViz

点开上面的网站，可以看到 DNSViz 是一个很强大的工具。

它是由 美国Sandia国家实验室 和 Verisign,Inc. 联合推出的专门用于分析域名的 DNSSEC 配置检查的一款分析工具。

在上图中，鼠标移到上述生成的信任链中的每个椭圆标记的元素上，都能看到很多详细内容。除了包括上面我通过 dig 命令查询到的信息之外，还有服务器信息以及状态是否安全。

> 点击上方的 `Response` 可以查看到各种 DNSSEC 资源记录类型的查询记录以及和根域、 NS 服务器之间的关系；
>
> 点击切换到 `Servers` 可以查看到 Paypal.com 的 4 个权威域名服务器（ NameServer ）的信息；
>
> 最后的 `Analyze` 用于简洁的展示查询分析过程：

> ```
Analyzing paypal.com
  Querying paypal.com/NS (referral)...
  Querying paypal.com/NS (auth)...
  Querying paypal.com/A...
  Preparing query jld3x6hmi0.paypal.com/A (NXDOMAIN)...
  Preparing query paypal.com/CNAME (No data)...
  Preparing query paypal.com/MX...
  Preparing query paypal.com/TXT...
  Preparing query paypal.com/SOA...
  Preparing query paypal.com/DNSKEY...
  Preparing query paypal.com/DS...
  Preparing query paypal.com/AAAA...
  Executing queries...
Success!
```

> 左侧的树形标签可展开，查看各个点的状态汇总及图例。
>
> 更多的图例解释，可点击左侧下方的 [Full legend](http://dnsviz.net/doc/dnssec/) 。英文太渣，我就不做半调子翻译官了，感兴趣的话自己去看。

既然是配置检查，当存在不正确的配置时，也会根据影响做 Error 或 Warning 的标记。例如我的博客域名 ephen.me，没有配置 DNSSEC ，查询出来是这样的：

![ ephen.me 的 dnssec 检查](https://iephen.pek3b.qingstor.com/b_image/20190426155455.png)

除了上图中最下一排的各种感叹号外，左边还有一大排的 Error 和 Warning ，描述了我的域名具体的配置出错的地方。

> 如果以后要给域名部署配置 DNSSEC ，我会记得再来这个网站查查看。 

## 附录

### 算法类型列表

- 1: RSA/MD5
- 2: Diffie-Hellman
- 3: DSA/SHA-1
- 4: Elliptic Curve
- 5: RSA/SHA-1
- 6: DSA-NSEC3-SHA1
- 7: RSASHA1-NSEC3-SHA1
- 8: RSA/SHA-256
- 10: RSA/SHA-512
- 12: RSA/SHA-512
- 13: ECDSA Curve P-256 with SHA-256
- 14: ECDSA Curve P-384 with SHA-384
- 252: Indirect
- 253: Private DNS
- 254: Private OID

### 摘要类型列表

- 1: SHA-1
- 2: SHA-256
- 3: GOST R 34.11-94
- 4: SHA-384 

## 参考资料及其他

1. [Check DNSSEC Signatures tool](http://support.simpledns.com/kb/a173/check-dnssec-signatures-tool.aspx)（这个也是一 DNSSEC 配置检查工具，由于看介绍没有 DNSViz 看起来直观，就没有进一步了解。附：[下载链接](http://www.simpledns.com/outbox/chksig.zip)）
2. [Microsoft MSDN - DNSSEC 概述 ](https://msdn.microsoft.com/zh-cn/library/jj200221%28v=ws.11%29.aspx)
3. [台湾大学电子报 - DNSSEC 安全技術簡介](https://www.cc.ntu.edu.tw/chinese/epaper/0022/20120920_2206.html)
4. [imlonghao - 我所理解的 DNSSEC ](https://imlonghao.com/41.html)
5. [DNSSEC 原理、配置与布署简介 | 段海新( Haixin Duan ) @ Tsinghua University](http://netsec.ccert.edu.cn/duanhx/?p=1479)
6. [Verisign - DNSSEC](https://www.verisign.com/zh_CN/domain-names/dnssec/index.xhtml)
