---
title: 闲聊 DNS 系统中域名的格式标准：下划线“_”是被允许的吗？
date: 2019-3-1
categories:
    - [知识探索]
    - [工作笔记]
tags:
    - DNS
    - RFC
    - 域名格式
toc: true
---

朋友最近在做解析配置的时候遇到问题，因 gitlab 域名所有权验证时提供的 host 是 `_gitlab-pages-verification-code` 被解析商认为域名不符合标准而拒绝填写，便来与我讨论：**域名的格式标准是什么？到底是否可以有下划线呢？**

虽说没事去研究`“茴”字有几种写法`着实有些多此一举，不过偶尔在做些研究时带点儿孔乙己精神还是挺好玩的一件事儿。

其实关于下划线的争议，我估计也不是一天两天了。国内外一些解析商的标准都是各有不同，而一些需要配置域名解析的商家提供的要求也是五花八门。比如最常见的邮件服务提供商，它们的 MX、SPF、DomainKey 等要求非专业人员要弄懂得费可大的力气。

今天灰姑娘就来从标准规范中剖析闲聊域名格式标准的那些事。

<!--more-->

**想直接看结论的童鞋可直接点击左侧文章目录跳转。**

## RFC-1034 & RFC-1035

[RFC 1034](https://tools.ietf.org/html/rfc1034) 和 [RFC 1035](https://tools.ietf.org/html/rfc1035) 是专门讲 DNS 以及域名基础的标准，业内也常常将这两篇拿出来分析和培训。

关于域名的格式，这两篇中有内容相同的段落来具体描述，RFC-1034 的 3.5 章节和 RFC-1035 的 2.3.1 章节。以下摘取自 RFC-1034 ：

> 3.5. Preferred name syntax
> 
> The DNS specifications attempt to be as general as possible in the rules for constructing domain names.  The idea is that the name of any existing object can be expressed as a domain name with minimal changes. However, when assigning a domain name for an object, the prudent user will select a name which satisfies both the rules of the domain system and any existing rules for the object, whether these rules are published or implied by existing programs.
> 
> For example, when naming a mail domain, the user should satisfy both the rules of this memo and those in RFC-822.  When creating a new host name, the old rules for HOSTS.TXT should be followed.  This avoids problems when old software is converted to use domain names.
> 
> The following syntax will result in fewer problems with many applications that use domain names (e.g., mail, TELNET).
> 
> `<domain> ::= <subdomain> | " "`
> `<subdomain> ::= <label> | <subdomain> "." <label>`
> `<label> ::= <letter> [ [ <ldh-str> ] <let-dig> ]`
> `<ldh-str> ::= <let-dig-hyp> | <let-dig-hyp> <ldh-str>`
> `<let-dig-hyp> ::= <let-dig> | "-"`
> `<let-dig> ::= <letter> | <digit>`
> `<letter> ::= any one of the 52 alphabetic characters A through Z in upper case and a through z in lower case`
> `<digit> ::= any one of the ten digits 0 through 9`
> 
> Note that while upper and lower case letters are allowed in domain names, no significance is attached to the case.  That is, two names with the same spelling but different case are to be treated as if identical.
> 
> **The labels must follow the rules for ARPANET host names.  They must start with a letter, end with a letter or digit, and have as interior characters only letters, digits, and hyphen.  There are also some restrictions on the length.  Labels must be 63 characters or less.**
> 
> For example, the following strings identify hosts in the Internet:
> 
> `A.ISI.EDU  XX.LCS.MIT.EDU  SRI-NIC.ARPA`

上面加粗的这一段，便明确指出 **“域名要遵循 ARPANET 的主机名格式：必须以字母开头、以字母或者数字结尾，中间部分为字母、数字或连字符，长度必须是 63 个字符或者更短。”**

从这里，我们可以看出，下划线“_”是不被允许的，无论它在什么位置。但有意思的是，从这描述上来看，数字竟然也是不允许放在开头的！

我们知道，在做反向域名解析的时候，需要指定的 IP 除了最后的`in-addr.arpa`和`ip6.arpa`前面就都是纯数字。所以，会不会是这个标准有什么问题呢？

### 学会查阅 RFC

这里顺便提一下，我们通常在查阅 RFC 的时候，多是关注该文档的内容，但其实也要关注下文档的发布信息。

以下是 RFC-1034 的发布信息：

![ rfc1034 comment ](https://pek3b.qingstor.com/imephen/20190426150600.png)

它明确的表明了这份标准：

1. 发布于 1987 年 11 月，是因特网标准。
2. 存在勘误，RFC 1101, 1183, 1348, 1876, 1982, 2065, 2181, 2308, 2535, 4033, 4034, 4035, 4343, 4035, 4592, 5936, 8020, 8482 对本标准有所更新。
3. 淘汰了 882, 883, 973 这三个 RFC 标准。

顶上的链接除了提供各种格式的查看方式外，还有文档跟踪及勘误表。我们可以去勘误表里面查查看都有哪些问题，而后去更新的文档里面找找还有没有我们需要的内容。

## 数字能否放到开头？

通过查阅勘误表，大致也就能发现几个英文拼写或语法错误。通过追踪接下来的更新标准，我找到了前面关于反向域名的格式疑惑。

以下节选自[ RFC 1101 ](https://tools.ietf.org/html/rfc1101)：

> 3.1. Network name syntax
> 
> The current syntax for network names, as defined by [RFC 952] is an alphanumeric string of up to 24 characters, which begins with an alpha, and may include "." and "-" except as first and last characters.  This is the format which was also used for host names before the DNS.  Upward compatibility with existing names might be a goal of any new scheme.
> 
> However, the present syntax has been used to define a flat name space, and hence would prohibit the same distributed name allocation method used for host names.  There is some sentiment for allowing the NIC to continue to allocate and regulate network names, much as it allocates numbers, but the majority opinion favors local control of network names.  Although it would be possible to provide a flat space or a name space in which, for example, the last label of a domain name captured the old-style network name, any such approach would add complexity to the method and create different rules for network names and host names.
> 
> For these reasons, we assume that the syntax of network names will be the same as the expanded syntax for host names permitted in [HR]. **The new syntax expands the set of names to allow leading digits, so long as the resulting representations do not conflict with IP addresses in decimal octet form.**  For example, `3Com.COM` and `3M.COM` are now legal, although `26.0.0.73.COM` is not.  See [HR] for details.

还是注意加粗的这段话，它表明：**新的语法标准允许以数字开头，但结果不能与十进制八位字节形式的 IP 地址冲突。**

更多信息大家可以自行去查看，最终 RFC-1101 定义了基于网络名称的 DNS 编码规范。

那么，网易家的`163.com`也合法了。

## 下划线“_”到底能不能允许存在呢？

事实上，域名系统可以看做一种用来存储具备层级关系的数据的服务。随着互联网的发展，DNS 能做的事儿越来越多，对主机名格式的定义也开始越来越宽容。

于是，于 1997 年发布的 [RFC-2181](https://tools.ietf.org/html/rfc2181) 再次对域名语法格式做了澄清：

> 11. Name syntax
> 
> Occasionally it is assumed that the Domain Name System serves only the purpose of mapping Internet host names to data, and mapping Internet addresses to host names.  This is not correct, the DNS is a general (if somewhat limited) hierarchical database, and can store almost any kind of data, for almost any purpose.
> 
> **The DNS itself places only one restriction on the particular labels that can be used to identify resource records.  That one restriction relates to the length of the label and the full name.  The length of any one label is limited to between 1 and 63 octets.  A full domain name is limited to 255 octets (including the separators).  The zero length full name is defined as representing the root of the DNS tree, and is typically written and displayed as ".".  Those restrictions aside, any binary string whatever can be used as the label of any resource record.**  Similarly, any binary string can serve as the value of any record that includes a domain name as some or all of its value (SOA, NS, MX, PTR, CNAME, and any others that may be added). Implementations of the DNS protocols must not place any restrictions on the labels that can be used.  In particular, DNS servers must not refuse to serve a zone because it contains labels that might not be acceptable to some DNS client programs.  A DNS server may be configurable to issue warnings when loading, or even to refuse to load, a primary zone containing labels that might be considered questionable, however this should not happen by default.
> 
> Note however, that the various applications that make use of DNS data can have restrictions imposed on what particular values are acceptable in their environment.  For example, that any binary label can have an MX record does not imply that any binary name can be used as the host part of an e-mail address.  Clients of the DNS can impose whatever restrictions are appropriate to their circumstances on the values they use as keys for DNS lookup requests, and on the values returned by the DNS.  If the client has such restrictions, it is solely responsible for validating the data from the DNS to ensure that it conforms before it makes any use of that data.
> 
> See also [RFC1123] section 6.1.3.5.

在这份标准文档中，明确了“**除了限制长度（包括每个标签之间的长度以及完整域名总长）外，任何二进制字符串都可以作为任何资源记录的标签**”。

至此，关于下划线的谜底彻底揭开：**下划线不应被 DNS 服务所拒绝。**不过，上文中也有提到**特殊的应用和场景可以有所限制，但并不是默认的。**通常这种情况下，对应的应用服务往往会有所判断。

此外，感兴趣的童鞋还可以继续看看上述节选中提到的 [ RFC-1123 的 6.1.3.5 章节](https://tools.ietf.org/html/rfc1123#page-79)。

## 其他提及

细心的童鞋可能有看到前面 RFC-1034 中有提到大小写的内容：**仅大小写不一致视为同一域名。** 除此之外，没有提及更多。

对于这种类似说法可能存在一些不确定性，2006 年 1 月发布的[ RFC-4343 ](https://tools.ietf.org/html/rfc4343) 再次针对大小写、ASCII 码转义等格式标准做了澄清。

感兴趣的童鞋可以去看看，我这里就不再引用和释义了。

## 总结

对于域名格式到底是怎样的，可以参考 AWS Route53 ，他们给了非常完整的定义：[ DNS 域名格式 ](https://docs.aws.amazon.com/zh_cn/Route53/latest/DeveloperGuide/DomainNameFormat.html)

总的来说有如下几个关键要点：

1. 长度限制：每个 label 之间 63 个字符以内，域名全长（含所有的“.”，包括根）不超过 255 个字符；
2. 不区分大小写，常规默认小写；
3. 特殊字符：需要用`\三位八进制 ASCII 码`格式转义；
4. 中、日、韩等非拉丁文字符域名：需要进行 punycode 转码 （附：[在线 punycode 转码工具：Punycoder](https://www.punycoder.com/)）；

值得一提的是，**DNS 业内针对 Zone 一般是“字母、数字和连字符（-），且中划线不能在 label 开头和结束”**。它来自早期已废弃的 RFC-882 & RFC-973 ，业内已约定俗成并非是如今的硬性要求。