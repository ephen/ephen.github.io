---
title: 网站 API 测试工具： chrome 插件 - Postman
id: 154
date: 2015-04-24
categories:
  - [工具介绍]
  - [工作笔记]
toc_list_number: false
tags:
  - postman
  - api
  - web
  - 测试
---

## 一、插件介绍

当你在调试一个网页是否运行正常时，除了简单地调试网页的 HTML 、 CSS 、脚本等信息之外，还需要检查网页能否正确处理各种 HTTP 请求。

网页的 HTTP 请求是网站与用户之间进行交互的非常重要的一种方式，在动态网站中，用户的大部分数据都需要通过 HTTP 请求来与服务器进行交互。

Postman 插件就充当着这种交互方式的“桥梁”，它可以利用 Chrome 插件的形式把各种模拟用户 HTTP 请求的数据发送到服务器，以便开发人员能够及时地作出正确的响应，或者是对产品发布之前的错误信息提前处理，进而保证产品上线之后的稳定性和安全性。

<!--more-->

## 二、插件安装

用 chrome 浏览器并且能轻松绕过 qiang 的童鞋，可以直接通过应用商店搜索 Postman-REST-Client 来安装。

然而用 360 极速浏览器等 chrome 核心浏览器的、或者不想绕 qiang 的童鞋，可下载插件后使用开发者模式加载（有些插件可以直接将 crx 文件拖到浏览器便自动安装了，但是 postman 不行，拖进去会提示要前往应用商店下载，所以只能采用开发者模式安装）。以 chrome 为例，安装步骤如下：

1. 下载 Postman-REST-Client_v0.8.1.crx 扩展程序，~~版本已更新，下载链接已失效，请自行上网搜索~~；
2. 修改 crx 后缀为 rar ，然后解压到指定文件夹，比如 `D:\crx\Postman-REST-Client_v0.8.1` ；
3. 进入 `chrome://extensions/` 页，勾选右上角的“开发者模式”，点击“加载正在开发的扩展程序”按钮，载入上一步保存的文件夹，然后启用插件即可；

## 三、 API 测试

以 CloudXNS 的 API 版本 1.0 为例。
<span style="font-size: 8pt; color: #800080;">『现在 CloudXNS 的 API 已经升级到2.0了，不日1.0会被废弃，请小伙伴们不要再尝试使用，同时注意更新自己已经在用的 API 版本代码。』</span>

一张图描述各个区域的作用和测试方法：

![ postman 各区域功能图解](https://iephen.pek3b.qingstor.com/b_image/20190426162846.png)

## 四、保存配置

经常测试的内容可以保存下来以后即可在左侧边栏“ collections ”中的列表一键调用。

保存很简单，点击测试时“ send ”按钮那排最右边的“ collection ”按钮即可。

如果是第一次保存，需要顺便创建一个 collection 文件夹。

![postman_addcollection 测试数据保存](https://iephen.pek3b.qingstor.com/b_image/20190426162940.png)

如果担心插件重装内容会丢失，或是想将自己的测试内容分享给同事使用，可以将 collection 导出成 json 文件或者上传到 postman 服务器提供分享链接。导入和导出操作入口如下：

![postman 测试数据导出备份](https://iephen.pek3b.qingstor.com/b_image/20190426162959.png)