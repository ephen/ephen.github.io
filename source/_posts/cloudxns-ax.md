---
title: CloudXNS 中的 AX 记录类型测试
id: 123
date: 2015-04-10
categories:
  - 工作笔记
tags:
  - 测试
  - AX
  - DNS
toc: false
---

![CloudXNS Logo](https://pek3b.qingstor.com/imephen/20190426151836.png)

## 首先，简单介绍下 CloudXNS 中的 AX 记录类型：

X 意为 eXtend ， AX 即表示扩展的 A 记录解析，具体表现为将多个 A 记录进行任意指定比例的负载均衡。我们知道， Local DNS 服务器集群会对用户的请求进行缓存，当您有多个 A 记录时，它会随机将这些缓存的 A 记录其中一条反馈给用户。这样我们就无法预估每个 ip 被用户请求到的数量，或许久而久之这些 ip 的访问量会被平均。而使用 CloudXNS 的 AX 记录，就能当有 Local DNS 对 CloudXNS 进行请求时，按照预设比例返回给它其中一个值传递给用户，而这个比例是由站长您自己来控制的。

<!--more-->

## 然后，开始测试：

首先将 `www.chinatesters.cn` 域名在 CloudXNS 系统中做如下图所示配置：

![CloudXNS 解析配置](https://pek3b.qingstor.com/imephen/20190426151910.png)

优先级即代表预设比例，上图配置中表示记录值 `2.2.2.1` 、 `1.1.1.2` 和 `1.1.1.1` 出现的比例为 `30:100:50` 。灰姑娘写了个脚本通过多次执行同一个 `dig` 命令获取每个结果的出现次数。测试代码示例如下：

```bash
#!/bin/sh
i=0;
rm -f .xtest_tmp.log
while [ $i -lt $1 ]
do
  dig www.chinatesters.cn @lv3ns4.ffdns.net +short >> .xtest_tmp.log
  i=expr $i + 1
done
awk ‘{name[$1]++};END{for(count in name)print count,name[count]}’ .xtest_tmp.log
```

将脚本带参数 200 ，表示执行 200 次命令。等待执行完毕后，得到结果如下：

```bash
[root@localhost 02_mytest]# ./xtest.sh 200
2.2.2.1 38
1.1.1.1 60
1.1.1.2 102
```

我们可以看到，与配置的 `30:50:100` 非常接近。经过多次测试之后就会发现，当 Local DNS 向 CloudXNS 请求次数越来越多时，这个值就越来越接近于配置值。因此，当您的网站有多个服务器但他们的硬件资源悬殊较大时，就可以采用 AX 记录将域名解析设置为不同的负载比例，以充分利用您现有的资源进行负载均衡。

同样的， CloudXNS 中的 CNAMEX 、 301 跳转、 302 跳转和隐式跳转记录中的优先级设置和上述 AX 记录设置是一样的效果，有需要的用户可以一试。