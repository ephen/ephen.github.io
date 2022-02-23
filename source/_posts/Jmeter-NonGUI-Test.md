---
title: Jmeter 以 non-gui 模式进行分布式测试
original: false
id: 254
date: 2016-01-06
categories:
    - 工作笔记
tags:
    - 测试
    - Jmeter
---

由于 Jmeter 是一个纯 JAVA 的应用，用 GUI 模式运行压力测试时，对客户端的资源消耗是相当惊人的，所以在进行正式的压测时一定要使用 non-gui 模式运行，如果并发数很高或者客户端的硬件资源比较一般的话，还可以以 server 模式用多个 client 进行分布式测试。

下面以武汉快网 CMCC 项目为例，结合官方文档和自己的实际经验来谈一谈， Jmeter 如何以 non-gui 模式进行分布式测试。

<!--more-->

## 前置工作

### linux 下安装 jdk1.7

负载机使用的是 CentOS 6.7 版本的 linux 系统， 自带有 OpenJDK runtime environment (openjdk) ，它是一个在 linux 上实现开源的 java 平台。

1 . **查看 CentOs 自带的 JDK 是否已安装**，输入： `yum list installed |grep java`

![查看 JDK 是否安装](https://pek3b.qingstor.com/imephen/20190426152046.png)

2 . 若有自带安装的 JDK ，则表示 JAVA 环境已经安装成功，可以**查看安装的 java 版本信息**

输入： `java -version` 可查看 Java 版本；

![查看 Java 版本](https://imephen.pek3b.qingstor.com/20190426152205.png)

3 . 如果想**卸载 CentOS 系统自带 Java 环境**，可输入： `yum -y remove java1.7.0-openjdk*` （ `*` 表示卸载掉 java1.7.0 的所有 openjdk 相关文件）

4 . **卸载 tzdata-java** 输入： `yum -y remove tzdatajava.noarch` ，当结果显示为 Complete ，即卸载完毕。

5 . **检查 CentOS 系统网络连接是否正常**。

使用 yum 方式安装需要连接网络下载相应安装文件，故需要使用 ping 命令测试网络，如 `ping www.baidu.com`

![测试网络](https://pek3b.qingstor.com/imephen/20190426152436.png)

6 . **查看 yum 库中的 java 安装包**，输入： `yum -y list java*`

![查看 yum 中 java 安装包](https://pek3b.qingstor.com/imephen/20190426152504.png)

7 . **使用 yum 安装 java 环境**，输入： `yum -y install java1.7.0-openjdk*` ，当结果显示为 Complete ，即安装完毕。

### 安装 JMeter

安装好 java 环境，就可以在此基础之上安装 jmeter 。

1 . **将 jmeter 安装包上传到服务器指定目录**，比如： `/root/mobile/`

2 . **解压 zip 安装包**，输入： `unzip apache-jmeter-2.13.zip`

![解压 jmeter 安装包](https://pek3b.qingstor.com/imephen/20190426152540.png)

## 非 GUI 模式下运行 Jmeter

JMeter 远程测试原理图如下：

![Jmeter 远程测试原理图](https://pek3b.qingstor.com/imephen/20190426152614.png)

如果运行 JMeter 客户端的机器性能不能满足测试需要，那么测试人员可以通过单个 JMeter 客户端来控制多个远程 JMeter 服务器，以便对服务器进行压力测试，模拟足够多的并发用户。通过远程运行 JMeter ，测试人员可以跨越多台低端计算机复制测试，这样就可以模拟一个比较大的服务器压力。

用一台机器（称为 JMeter 客户端）上的 jmeter 同时启动另外几台机器（称为 JMeter 远程服务器）上的 jmeter 。

1 . 保证 jmeter 客户端和 jmeter 远程服务器采用相同版本的 jmeter 和 JVM 。
2 . jmeter 客户端和 jmeter 远程服务器最好在同一个网段内。
3 . 在 jmeter 远程服务器上运行 `root/mobile/apache/jmeter-2.13/bin/jmeter-servere` 脚本。指定 server 的 ip ，如： `./jmeter-server -Djava.rmi.server.hostname=118.192.6.15` ， jmeter-server 正常启动会提示"创建远程服务"
4 . 在 jmeter 客户端上修改 `/bin/jmeter.properties` 文件，找到属性 `"remote_hosts"` ，使用 JMeter 远程服务器的 IP 地址作为其属性值。可以添加多个服务器的 IP 地址，以逗号作为分隔。

![找到 Jmeter 属性文件](https://pek3b.qingstor.com/imephen/20190426152641.png)

例如：

```
remote_hosts=118.192.6.15:1099,118.192.6.16:1099   #RMI port to be used by the server (must start rmiregistry with same port)
server_port=1099
```

![修改 Jmeter 属性](https://pek3b.qingstor.com/imephen/20190426152709.png)

5 . 在 jmeter 客户端上启动 jmeter 。

```
./jmeter -n -t plan.jmx -l result.jtl -r
```

`plan.jmx` 测试计划可以在 windows 环境下先创建，最好不要添加监听器，应为命令行启动的话监听器可能会占用资源而且有没有任何视图效果。

> 参数说明 :
> 
> -n  告诉 jmeter 使用 nogui 模式运行测试
> 
> -t  执行的测试脚本名
> 
> -l  后面跟着测试结果记录的路径与文件名 
> 
> -r  表示远程启动 (remote) 
> 
> jmeter 客户端会自动向 jmeter 远程服务器上分发测试计划。

最后，可以将 result.jtl 结果文件下载到 windows 机上在不同的监听器上分析测试结果。