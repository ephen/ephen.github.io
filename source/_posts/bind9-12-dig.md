---
title: bind 9.12 stable 版 dig 工具有问题，请更新或回退！（附 Windows dig 安装方法）
date: 2019-04-26
categories: 
  - 工具介绍
tags: 
  - bind
  - dig
  - DNS
  - Windows
toc: false
comments: true
top: false
---

> 查看安装方法请直接跳转至：[Windows 上如何安装 dig 工具？](/2019/bind9-12-dig/#Windows-上如何安装-dig-工具？)

前段时间又开始捣鼓网络 & DNS 了，于是去 [ Bind9 官方 ](https://www.isc.org/downloads/) 去下载了一个最新稳定版 Windows 环境的 Bind 。

当时 Bind 9.14.1 还未正式发布为稳定版，所以我下载的版本是 9.12.3-P1 。下载完成后，安装到我的本地 PC 上。

随后我在**使用这个版本的 dig 时，只要不指定 DNS 服务器就没有返回，每次使用该工具时必须指定 DNS 服务器。**

![ dig 12 版本有问题 ](https://imephen.pek3b.qingstor.com/digproblem.jpg)

<!--more-->

起初我以为是公司办公网络环境自动获取设置的本地 DNS 地址有问题，看也能使用就没怎么放在心上。

刚刚又在使用时，我突然想好好查看一番。Google 了一圈没有搜到结果。于是只好自己查问题了啵。

**网络抓包并没有抓到任何 DNS 请求！**

![ 抓包没有 dns 请求 ](https://pek3b.qingstor.com/imephen/capture.jpg)

换 DNS 配置、换手机热点做网络环境的一番折腾，依然还是这样！开始决定要不换个旧点的版本试试。

重新下载版本 bind 9.11.6-P1 安装后，没问题了鸭！

![ dig 9.11.6-P1 版没问题 ](https://pek3b.qingstor.com/imephen/versionchange.jpg)

真是喜极而泣。

下载了目前最新稳定版 bind 9.14.1 又试了下，也没有问题。

![ dig 9.14.1 版没问题 ](https://pek3b.qingstor.com/imephen/dig914.jpg)

**如果你和我一样正在使用 dig 9.12.3-p1 版，为不影响使用，记得将软件升级或回退哦。**

## Windows 上如何安装 dig 工具？

最简单的方法，到官网下载相应版本 Bind 后，解压。

![ Bind 源程序解压 ](https://pek3b.qingstor.com/imephen/binddir.jpg)

使用管理员运行 `BINDInstall.exe` ，指定本地安装目录。

如果只想安装 dig ，安装选项选择 `Tools Only` 即可。如下图所示：

![ 安装 dig ](https://pek3b.qingstor.com/imephen/bindinstall.jpg)

是不是很简单鸭~

不太记得是不是需要自己加下环境变量，要是小伙伴**安装后不能像我一样直接在 CMD 里不指定目录使用 dig ，那就需要添加下 PATH 。**

![ 添加环境变量 ](https://pek3b.qingstor.com/imephen/paths.jpg)

我个人的环境变量有些多，涂画得乱起八糟，请勿见怪。

虽说这种安装方法不够绿色，不过这是最快且最好上手的方式了。

想要绿色一点，就需要自己把 dig 程序和其应用的 DLL 库都收集起来放到一个文件夹里，然后再将此文件夹添加到 PATH 即可。