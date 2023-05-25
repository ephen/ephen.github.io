---
title: 记录下多年未更新博客的后遗症
date: 2022-2-16
categories: 
  - 博客折腾
tags: 
  - hexo
  - node
  - git
  - jquery
toc: true
comments: true
---

朋友网站换链接了，到 [Telegram](https://telegram.me/imephen) 上提醒我更新下友链。本以为不就是改个链接嘛，没想到问题接踵而至。果然太长时间没动过的东西，没有想象中那么简单。

本次更新发现的主要后遗症列表：

1. 多个朋友发生域名更换及闭站或撤链；
2. hexo & git 命令都忘了怎么用；
3. Coding Pages 服务已关闭；
4. hexo generate 生成空文件，造成网页白屏；
5. 博客头像、标签、菜单和分类均加载失败；

以下详述。

<!--more-->

## 友链更新

一个个的检查现有友链后，将能找到私下联系方式的朋友确认换域名、闭站及撤链情况，将友链都更新好。有些找不到联系方式的朋友，我就直接撤链了。

因此事实证明，友链这东西，一定要**先友后链**，否则自己是怎么被撤链的都不知道。

## 部署问题

如前文《[全新出发，正式启用 Hexo ！](/2016/hello-hexo/)》所述，我原本是启用了个人云主机、Github Page 和 Coding Page 多点部署的。

这次更新完友链后，直接执行 `hexo g` 和 `hexo d` 准备发布，发现 Coding 出现报错。去搜索了下，才发现我曾经收到过如下邮件：

![Coding Page 服务关闭](https://iephen.pek3b.qingstor.com/b_image/20220216150100.png)

于是屏蔽掉 Coding deploy 配置，重新执行 `hexo d` ，发现 Github 提示 `nothing commit` 。度娘了下，**删除掉 `.deploy_git` 文件夹后解决**。想来可能是因为我换电脑本地数据迁移时未验证迁移配置的缘故吧~

接着，上云主机运行同步脚本，发现 git 报认证错误。于是**重新执行 `ssh-keygen -t rsa -C "github账号邮箱"` 生成ssh密钥后添加到 Github Settings --> SSH and GPG Keys 页面**。再次运行脚本后成功。

最后，上域名服务商删掉使用 Coding Page 的分区解析。

## 网页白屏修复

本以为做完以上操作一切就绪，当我洋洋得意打开我的博客网站时，居然是一片白。本地执行 `hexo s` 发现又可以正常显示，这让我一个头两个大。。

去检查云主机发现，上面的文件都是 0KB ，内容全部为空；接着去 GitHub 上看，文件也是空的；最后才发现本地 `/public` 目录下的文件居然也是空的！

那么应该是 `hexo generate` 命令的问题了，又执行了一遍 `hexo clean` 和 `hexo generate` 后确认。于是再次求助度娘，原来是本地 hexo 版本和 node 版本不匹配（hexo 版本低了，node 版本高了）。

于是，**升级 hexo** ：

```
npm install -g hexo-cli
```

**降级 node** ：登录 [node.js 官网历史版本下载页](https://nodejs.org/en/download/releases/)找到一个 12.x 的版本（我下载的版本是 12.22.10）下载下来，卸载掉本地已安装的 node 后重新安装该版本，然后重新启动。

再次 `hexo clean` 后 `hexo generate` ，`/public` 目录下的文件已有内容了。

重新执行 `hexo d` 成功发布到 Github Page 。到云主机执行脚本时，却一直提示 `Already up-to-date` ，真是见鬼了。淦！

直接 **`rm -rf .` 删除掉同步目录，执行 `git clone git@github.com:ephen/ephen.github.io.git .` 将所有文件重新生成到当前目录**后解决。

## 网页 JS 修复

长吁一口气后再去“欣赏”网站，却怎么也找不到自己的友链入口。仔细一看，头像、标签、分类全都消失了。折腾到此，午饭都没有吃，真是欲哭无泪！

沉下心来 F12 看看报错，第一个报错如下：

![require,jquery 调用出错](https://iephen.pek3b.qingstor.com/b_image/20220216155418.jpg)

看到是引用的文件 CDN 缓存路径出错后，**找到官方路径和代码引用位置进行替换**后解决。

再次打开网站，终于折腾完毕，其他的报错暂且也不想再看了。

## 结语

太久不更新，问题就是多多呀。

主要是最近几年真的发生了太多事，精力实在有限。要不是有个什么契机，其实也是不想再去折腾了。~~毕竟我连常用指令都需要去找度娘解决，还有什么用呢？~~

这里我只是记录下折腾过程，免得日后什么时候弄起来遇到相同问题又要找度娘。

另外提醒下大家，千万不要学我“暴力 rm”、“未经确认就发布”等行为，一定要养成好习惯哟！~