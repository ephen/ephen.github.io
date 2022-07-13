---
title: 需要上线多个域名（网站）却只有一个公网 IP 怎么办？
date: 2018-08-31 20:01:32
categories:
  - 工作笔记
tags:
  - 域名
  - 网站
  - 负载均衡器
  - VPC
  - VirtualHost
  - LB
toc: true
---

以前做 DNS ，推出过一个系列的《如何让多 ip 域名配置游刃有余》的几篇文章，毕竟作为智能 DNS ，站点多 ip 的配置是其职责所在。

然而最近，有的小伙伴问题又来了：

  > 没那么多钱申请那么多不同机房的公网 ip ，但又有必要部署多个不咋占资源的站点，又该怎么办呢？

![ 穷到变形 ](https://imephen.pek3b.qingstor.com/b_image/20190426163201.png)

别急，穷人不止你一个，自然有又穷又聪明的 PM & 技术给出解决方案，啥都能给变出来。

<!--more-->

## 穷到啥都不剩：别说 ip 了，连主机都买不起

在云计算行业，虚拟化是一个不陌生的词。只要资源够，一台机器可以生出无数台虚拟机来。

 Web 服务上同样也有类似的概念，人们可以通过创建不同的 VirtualHost 指向不同的域名（或端口）和目录，即可实现同一台机器上生出多个不同的站点。

以 Apache 为例，Apache 安装好后在默认配置中一般有两个重要的配置文件： `httpd.conf` 和 `welcome.conf`

其中 `httpd.conf` 是程序的配置文件，决定了程序将要使用什么组件、监听什么端口、加载什么配置、各类程序文件及日志文件都放到哪里等等；而 `welcome.conf` 是程序提供的 Web 配置示例，告诉使用者要搭建网站该怎么配置。

 - 注：Apache 中 Web 站点配置文件名不一定是 `welcome.conf` ，只要 `httpd.conf` 文件中有指向，可以是任意的文件名。本文仅以 `welcome.conf` 做示例。

我们知道，http 服务的默认端口是 80 ，使用 80 端口则用户在访问时不用刻意在浏览器输入端口号，非常方便。 `httpd.conf` 文件中默认 `Listen 80` ，要使用 80 端口，我们只需要配置好 Web 站点即可。

默认的 `welcome.conf` 文件内容中的 VirtualHost 配置是这样的（截取关键内容）：

```
<VirtualHost *:80>
    DocumentRoot "/var/www/html/"
    ServerName ephen.me
</VirtualHost>
```

`DocumentRoot` 后面的目录表示站点搭建目录， `ServerName` 表示站点域名。

如果我们需要配置多个不同的站点，将站点搭建到不同的目录，在该文件中多配置几条 VirtualHost 就好：

```
<VirtualHost ephen.me:80>
    DocumentRoot "/var/www/html/ephen.me/"
    ServerName ephen.me
</VirtualHost>

<VirtualHost demo.com:80>
    DocumentRoot "/var/www/html/demo.com/"
    ServerName demo.com
</VirtualHost>
```

使用其他的 Web 服务程序，道理是一样的。 Nginx VirtualHost 配置可参考：<http://tabalt.net/blog/nginx-virtual-host/>

## 只是有点小气：不差钱，就是只想用一个 ip

假设多个不同的网站已经在不同的云主机上已搭建好，而目前能提供服务的可用 ip 只有一个。通常咱们会想到要是前面再有一个分发装置用于绑定这个 ip ，然后通过访问端口或域名区分真正到达的主机，那真真是极好的~

在云计算行业，这个分发装置肯定是有的，就看你怎么用了。

下文将以 QingCloud 云平台为例，来解决这个问题。

### 负载均衡器（ Load Balancer ）

创建负载均衡监听器，监听 80 端口，添加多个后端，然后为这些后端分别绑定不同的转发策略，就能成功满足需求。

操作步骤示例：

1. 创建负载均衡器，绑定仅有的一个公网 ip

![Load Balancer](https://imephen.pek3b.qingstor.com/b_image/20190426163253.png)

2. 为该负载均衡器创建基于 80 端口的 HTTP 监听器，并将已搭建好的几个网站添加为该监听器的后端

  - 注：我只有一个站点，但不影响后续的示例

![HTTP 监听器](https://imephen.pek3b.qingstor.com/b_image/20190426163314.png)

3. 为对应的后端添加转发策略，让来自 `ephen.me` 站点的请求指向对应的后端地址

  - 注意看上面第二步附图中后端列表倒数第 3 个字段——“转发策略”，点击下面对应的“绑定”二字；
  - 若没有转发策略可绑，会指引创建：

  ![create forward](https://imephen.pek3b.qingstor.com/b_image/20190426163335.png)

  - 为转发策略添加规则，选择“按域名转发”：

  ![add rules](https://imephen.pek3b.qingstor.com/b_image/20190426163354.png)

  - 回到负载均衡监听器，为对应后端添加转发策略

  ![choose forward](https://imephen.pek3b.qingstor.com/b_image/20190426163416.png)

通过应用上述配置后，即可完成：**用户访问对应 ip 的 80 端口 HTTP 服务，若域名为 `ephen.me` 则转发至该后端主机。**

如果有存在多台已配置好不同 Web 服务的云主机要共用这个 ip ，只需要继续往这个负载均衡器添加后端主机并绑定不同规则的转发策略就可以了。

当然，负载均衡器的作用远不仅仅如此，本文的介绍可以看作是为解决问题而抖个机灵罢了。更多的介绍可参照 QingCloud 官网：[负载均衡_LB Cluster | 青云QingCloud](https://www.qingcloud.com/products/loadbalancer/)

### VPC 网络

“负载均衡器 + 转发策略”的解决方案确实很好用，但如果多个不同业务的站点已经搭建到不同的云主机，除了 ip 没有多的可用外，连域名都想用同一个，只靠端口号来区分，又该怎么办呢？

当然负载均衡器也是可以解决的，看到这里的小伙伴可以一起思考下。不过这时候容许灰姑娘再抖个机灵，通过创建“ VPC 网络 + 端口转发”来解决。

关于 VPC 网络的介绍，可以先看看：[专属私有网络_VPC | 青云QingCloud](https://www.qingcloud.com/products/vpc/)

懂的童鞋可跳过，且看操作步骤：

1. 创建 VPC 网络，绑定仅有的一个公网 ip

![VPC](https://imephen.pek3b.qingstor.com/b_image/20190426163448.png)

2. 在 VPC 网络中创建一个 vxnet 私有网络，并将业务承载主机加入到这个网络，在开启 DHCP 服务的情况下，它将自动分配到一个私有网络地址

![VxNet](https://imephen.pek3b.qingstor.com/b_image/20190426163534.png)

3. 管理 VPC 网络配置，添加端口转发，将来自 80 端口的业务转发到私有网络中的这台云主机上

![Port Forward](https://imephen.pek3b.qingstor.com/b_image/20190426163552.png)

通过应用上述配置后，即可完成：**用户访问对应 ip 的 80 端口 HTTP 服务，则转发至私有网络中 ip 地址为`192.168.128.2`的主机。**

同理，其他业务的云主机，可以都加入到前面已创建好的私有网络并获得各自的私有网络 ip 地址，再设置不同的端口转发到不同的地址就好了。

## 总结

以上只是灰姑娘的个人想法，其实标题所描述的问题解法还有很多，欢迎相互交流。

也许有些小伙伴的业务过于庞大后会有更多的难题需要解决，不过办法都是人想的，如果有童鞋有别的难题，也欢迎相互交流。

最后，感谢耐心看完此文。