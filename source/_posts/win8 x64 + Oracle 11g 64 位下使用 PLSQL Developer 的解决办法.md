---
title: win8 x64 + Oracle 11g 64 位下使用 PL/SQL Developer 的解决办法
id: 120
permalink: win8x64-oracle-plsql
date: 2015-03-31
categories:
  - 工作笔记
tags:
  - Oracle
  - Win8
  - PL/SQL
---

win8 64 位操作系统在安装 Oracle 11g 和 PL/SQl 时有两个非常重要的坑！

一个是 Oracle 安装环境检测没有支持到 win8 会提示不符合系统要求的最低配置，另一个就是 PL/SQL 没有 x64 位的软件版本导致不能读取 Oracle x64 位客户端配置。

本文将通过巧妙的方式解决这两个问题。同时，市面上大多 SQL 客户端软件都没有 x64 位的版本（比如 Navicat ），也不能方便的和 Oracle x64 客户端一起使用，经测试也可以通过本文同样的方式解决。

<!--more-->

## 安装 Oracle 11g 64 位

注意：**不要安装在中文目录！！！**

1. 解压安装包，以管理员身份运行 cmd ；
2. `cd` 到你解压安装文件的目录，运行 `setup.exe   -ignoreSysPrereqs` ；
3. 提示不符合系统最低配置，选择仍然继续；
4. 选“管理员模式”，安装成功；

## 32 位 Oracle 客户端文件合并到安装目录

1. 到官网下载一个 32 位的 instantclient 客户端到任意非中文目录安装好（如果以前装过 32 位客户端并且目录保留在机器上则跳过此步，也可以直接从别人电脑上拷贝这个已安装好的目录）；
2. 将 32 位的 Oracle 客户端文件目录 client_1 移动到和 64 位目录一起，例如：

> 32 位客户端文件夹为 `E:\app\Cinderella\product\11.2.0\client_x86`
>
> 64 位客户端文件夹位 `E:\app\Cinderella\product\11.2.0\client_1`

## 安装PL/SQL Developer

## 配置PL/SQL Developer的OCI Library和Oracle_Home

以上面的安装路径为例：

英文版： perference->Connection

```
Oracle Home ： E:\app\Cinderella\product\11.2.0\client_x86
OCI Library ： E:\app\Cinderella\product\11.2.0\client_x86\bin\oci.dll
```

汉化版：工具 -> 首选项 -> 连接

```
Oracle 主目录名（自动检测为空）： E:\app\Cinderella\product\11.2.0\client_x86
OCI 库（自动检测为空） ： E:\app\Cinderella\product\11.2.0\client_x86\bin\oci.dll
```

## 设置操作系统环境变量

1. 右击"我的电脑" - "属性" - "高级系统设置" - "环境变量" ，可以看到上面是“用户环境变量”，下面是“系统环境变量”；
2. 在“系统环境变量”中找到 Path ，将 `E:\app\Cinderella\product\11.2.0\client_x86;` 添加到最前面（**有分号**）；
3. “系统环境变量”中点击"新建", 变量名设置为 `TNS_ADMIN` , 变量值设置为 `E:\app\Cinderella\product\11.2.0\client_x86\NETWORK\ADMIN` , 点击"确定"（**无分号**）；
4. “系统环境变量”中点击"新建", 变量名设置为 `NLS_LANG` , 变量值设置为 `SIMPLIFIED CHINESE_CHINA.ZHS16GBK` , 点击“确定”；
5. 设置完成，启动 PL/SQL Developer ，运行无问题。