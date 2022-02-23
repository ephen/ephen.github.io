---
title: 桥接模式下如何设置虚拟机和本机在同一网段
date: 2015-03-31
id: 113
categories:
  - 工作笔记
tags:
  - Vmware
  - Network
  - 虚拟机
toc: false
---

1. 打开 VMware 软件，开启要设置的虚拟机，用 root 用户登录；
2. 点击 VMware 软件菜单栏上“虚拟机”—>“设置”—>“网络适配器”，选择“桥接模式”（默认是“ NAT 模式”）；
3. 进入 Linux 虚拟机修改网卡设置：

### 方法一，使用图形界面修改

<!--more-->

1 . 桌面右击 `open in terminal``，输入 setup 命令后回车；
2 . 通过键盘上下键选择 `Network Configuration`， Tab 到 `Run tool` 后回车；
3 . `Device Configuration` 高亮回车，选择要设置的网卡（一般虚拟机只有一个网卡）后回车，进入 ip 设置界面；
4 . 默认情况下 `Use DHCP` 是自动获取，图形界面上展示为 `*`，下面的静态 IP 不能进行设置。如下图：

![虚拟网卡图形界面配置](https://pek3b.qingstor.com/imephen/20190426161737.png)

5 . 将光标移动到 `Use DHCP` 的`[*]`上，点击空格键，即可开始设置下面的静态 IP；
6 . 查看本机（连到路由器上的网络而非虚拟网络）的 ip 设置，将虚拟客户机 Linux 设置成一样即可（除 IP 外， IP 设置到同一网段）；如下图：

![本机 IP 设置](https://pek3b.qingstor.com/imephen/20190426161803.png)![虚拟机IP设置](https://pek3b.qingstor.com/imephen/20190426161823.png)

7 . Tab 到 `Save` 、 `Save&Quit` 保存本次设置；

### 方法二，修改网卡配置文件

1 . 桌面右击 `open in terminal` ，输入 `setup` 命令后回车；
2 . 一般虚拟机网卡是 eth0 ，修改文件 `vi /etc/sysconfig/network-scripts/ifcfg-eth0`

```bash
  DEVICE=eth0   #描述网卡对应的设备别名，例如 ifcfg-eth0 的文件中它为 eth0
  BOOTPROTO=static    #设置网卡获得 ip 地址的方式，可能的选项为 static/dhcp/bootp （分别对应静态指定的 ip 地址 / 通过 dhcp 协议获得的 ip 地址 / bootp 协议获得的 ip 地址）
  BROADCAST=192.168.0.255   #对应的子网广播地址
  HWADDR=00:07:E9:05:E8:B4    #对应的网卡物理地址
  IPADDR=192.168.14.120   #如果设置网卡获得 ip 地址的方式为静态指定，此字段就指定了网卡对应的ip地址
  IPV6INIT=no
  IPV6_AUTOCONF=no
  NETMASK=255.255.255.0   #网卡对应的网络掩码
  NETWORK=192.168.0.0   #网卡对应的网络地址
  ONBOOT=yes    #系统启动时是否设置此网络接口，设置为yes时，系统启动时激活此设备
```

3 . 修改对应网卡的网关的配置文件 `vi /etc/sysconfig/network`

```bash
  NETWORKING=yes    #系统是否使用网络（一般设置为 yes 。如果设为 no ，则不能使用网络，而且很多系统服务程序将无法启）
  HOSTNAME=localhost    #本机的主机名（这里设置的主机名要和 /etc/hosts 中设置的主机名对应）
  GATEWAY=192.168.14.1    #本机连接的网关 IP 地址
```

4 . 重启网卡或重启机器以应用上述修改。

### 重启网卡方式

（以 eth0 为例）：先关闭（命令： `ifdown eth0` ），再开启（命令： `ifup eth0` ）