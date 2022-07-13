---
title: VSCode 中使用 PlantUML 插件生成 UML
date: 2017-10-23
categories:
  - 工具介绍
tags:
  - VSCode
  - UML
  - 解析流程
toc_list_number: false
---

作为一个伪技术人员，灰姑娘用的 IDE 大概是不那么流行的 [VSCode](https://code.visualstudio.com/)。全称 Visual Studio Code ，微软家近两年才推出来的一个产品。

这个 IDE 的整个软件界面以及使用都和 [Atom](https://atom.io/) 非常相像，它本身比较轻量，但又可以安装很多扩展插件以满足使用者的需要。虽然对过去使用老古董 [Ultra Edit](https://www.ultraedit.com/) 的我来说，刚上手着实有些不习惯，但是用久了之后还是觉得不错的。（尽管我用 IDE 的频率不高。。

最近，同事希望我给 DNS 解析流程画个 UML 图，并推荐了一个通过脚本语言生成 UML 图的开源项目： [PlantUML](http://plantuml.com/) 。

上网搜索后发现，可以不需要下载这个项目的 APP ，很多编辑器都有它的插件。但是我翻了好久，并没有看到基于 VSCode 的实际应用示例。

我在 VSCode 上搜到这个扩展插件之后，环境部署还很折腾了一会儿。遂将一些经验在此分享给大家，希望能对其他路过的人有所帮助。

<!--more-->

## 环境配置

PlantUML 本身只是一个脚本语言，如果要生成图片，是基于 [GraphViz](http://www.graphviz.org/) 绘图的。而 GraphViz 又是基于 dot 脚本生成图片，它依赖于 Java 和 Dot 脚本环境的配置。

所以，使用 PlantUML 有如下基本环境要求：

### 1.安装 Java 环境

上官网下载后安装：https://www.java.com/zh_CN/

注意配置 `$JAVA_HOME` 环境变量。

配置完毕后启动 Windows Shell 执行 `java -showversion` 命令查看 Java 版本，以确定 Java 环境是否准备成功。

### 2.安装 GraphViz 程序

下载地址：http://www.graphviz.org/Download_windows.php

下载安装完后（灰姑娘将其安装到了 D 盘路径：`D:/program files/Graphviz/`下），也要配置环境变量。

将 GraphViz 可执行程序路径配置到 `$path`：

![GraphViz 环境变量](https://imephen.pek3b.qingstor.com/b_image/20190426152846.png)

**后面的所有配置（ dot 配置和 VSCode 插件配置）都是基于 GraphViz 的安装路径，需要参照的小伙伴请注意。**

### 3.配置 Dot 脚本环境

添加环境变量 `$GRAPHVIZ_DOT`。如下图所示：

![Dot 环境变量](https://imephen.pek3b.qingstor.com/b_image/20190426152918.png)

配置完毕后启动 Windows Shell 执行 `dot -v` 命令查看 Dot 版本，以确定 GraphViz 和 Dot 环境是否准备成功。

### 4.安装 VSCode 插件并配置

VSCode 需要安装两个插件： `PlantUML` 和  `Graphviz Preview`

打开 VSCode 切换到左侧扩展商店，搜索并安装他们，安装完毕后分别激活。

点击 VSCode 左下角齿轮按钮进入设置，在用户设置脚本中添加 `"graphviz-preview.dotPath": "D:/program files/Graphviz/bin/dot.exe"` 并保存设置：

![graphviz-preview.dotPath](https://imephen.pek3b.qingstor.com/b_image/20190426152953.png)

### 5.环境配置测试

**重启 VSCode 。**

新建文档输入以下测试代码：

```pl
@startuml

[*] --> State1
State1 --> [*]
State1 : this is a string
State1 : this is another string

State1 -> State2
State2 --> [*]

@enduml
```

同时按下 `Alt` 键和字母 `D` 键，预览生成的 UML 状态图。

![PlantUML 测试](https://imephen.pek3b.qingstor.com/b_image/20190426153029.png)

如果出现如图所示的错误，检查下是不是前面 GraphViz 安装和配置没做好。

![GraphViz 报错](https://imephen.pek3b.qingstor.com/b_image/20190426153134.png)

## 分享： DNS 解析流程图

为了画这个图，灰姑娘专门去学了下这个脚本语言。如果小伙伴们也想学习，官方中文指导手册在这里：[点击下载](http://translate.plantuml.com/zh/PlantUML_Language_Reference_Guide_ZH.pdf)。除此之外，[这里](http://www.jianshu.com/p/e92a52770832) 还有网友分享的快速入门教程。

回到正题，我的目的是要画个 DNS 解析流程图。

最后图是勉强画出来了，看看画得咋样？

![DNS 解析流程图](https://imephen.pek3b.qingstor.com/b_image/20190426153216.png)

上图源代码分享如下：

```pl
@startuml
title 域名解析流程图
skinparam state {
  StartColor MediumBlue
  EndColor Red
  BorderColor Gray
  ArrowColor Gray
  ArrowFontColor Indigo
  BackgroundColor<<Request>> AliceBlue
  BackgroundColor<<Answer>> HotPink
}

state UserClient {
    [*] --> 请求访问域名
    请求访问域名 --> 本机host
    本机host -right-> 成功访问域名 : 有映射
    本机host --> 本机DNS解析器 : 没有映射
    本机DNS解析器 -up-> 成功访问域名 : 有缓存
    成功访问域名 --> 本机DNS解析器 : 缓存更新
    本机DNS解析器 --> [*]
    note left : 本图展示正常解析流程\n未成功访问域名不能结束\n（最左支路径无效）
    
    state 请求访问域名 : ephen.me.
    state 请求访问域名<<Request>>
    state 本机host : Windows 目录：
    state 本机host : C:\Windows\System32\drivers\etc
    state 成功访问域名 : ephen.me.
    state 成功访问域名<<Answer>>
    state 本机DNS解析器 : Windows 查看命令：
    state 本机DNS解析器 : ipconfig /displaydns
}

state DNSServer {
    本机DNS解析器 -right-> 本地DNS : 没有缓存
    本地DNS -right-> 根域名服务器 : 没有解析
    本地DNS -left-> 成功访问域名 : 有解析
    根域名服务器 --> 顶级域名服务器
    顶级域名服务器 --> 域名服务器
    域名服务器 --> 本地DNS : 获取结果并缓存

    state 本地DNS : 用户配置（8.8.8.8）
    state 根域名服务器 : a.root-servers.net.
    state 根域名服务器 : b.root-servers.net.
    state 根域名服务器 : ...
    state 根域名服务器 : m.root-servers.net.
    state 顶级域名服务器 : a0.nic.me.
    state 顶级域名服务器 : b0.nic.me.
    state 顶级域名服务器 : ...
    state 域名服务器 : lv3ns1.ffdns.net.
    state 域名服务器 : lv3ns2.ffdns.net.
    state 域名服务器 : lv3ns3.ffdns.net.
    state 域名服务器 : lv3ns4.ffdns.net.
}
@enduml
```

希望我的分享能对路过这儿的你能有所帮助。