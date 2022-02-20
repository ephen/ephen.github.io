---
title: 博客 Hexo 工作区同步至 Mac ，多个电脑可同时写博客啦～
permalink: hexo-syn
date: 2022-2-20
categories: 
  - 博客折腾
tags: 
  - hexo
  - git
  - mac
toc: true
toc_list_number: false
comments: true
---

一个月前，公司资产部在一年一度处理离职员工回收的资产时，我获得了一台 MacBook Air 。

由于过去从未用过 Mac ，在摸清了大致的用法后，突然想到再折腾下博客。

之前我没有个人电脑，每次更换工作电脑时，直接用 U 盘复制本地 Hexo 工作目录到新电脑上，安装相关依赖后使用。

这次准备把新到的 MacBook Air 作为我的个人电脑使用，然后让个人电脑和工作电脑均可更新博客，所以需要考虑 Hexo 源文件多端同步了。

<!--more-->

## 为什么要同步 Hexo 源文件

用 Hexo 搭建博客的同学应该比较清楚， Hexo 写博文后是通过 `hexo g` 命令生成 html 文件后，再通过 `hexo d` 将生成的 html 文件发布到 Github Page ，这样使得 Github Page 上直接就是 html 格式网址了。我们通过 markdown 写的源文件、hexo 主题文件和插件等信息都不会上传至 GitHub ， `hexo g` 的动作相当于网页编译行为。

因此，当我们需要在其他电脑上也可以同步写文章时，就需要把 hexo 工作区进行同步。

## 使用 GitHub 存储 hexo 源文件

Github 是个很好的代码协同平台，我的博客本身也是放 GitHub Page ，因此我选择在 GitHub Page 项目源代码上新建一个分支来存放 hexo 源文件作为我的线上工作区。

登录 GitHub 后进入到我博客所在项目查看源代码，基于 `master` 新建一个名为 `suorce` 的分支，并将该分支设置为默认分支。

工作电脑上，新建一个工作区目录，执行 `git clone 项目地址 .` 将 github 上的代码下载下来。由于默认分支现在是 `source` ，因此本地工作区也默认为 `source` 分支。

由于远程分支是基于原分支复制的，因此内容都是博客 html 文件。我删除掉这些文件后，将之前写博客的目录下所有文件复制过来。

然后 

## 