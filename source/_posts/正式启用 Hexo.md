---
title: 全新出发，正式启用Hexo！
permalink: hello-hexo
date: 2016-07-26
categories: 
  - 博客折腾
tags: 
  - hexo
  - sitemap
  - git
toc: true
toc_list_number: true
comments: true
top: true
---

**郑重声明：原网站 <http://www.chinatesters.com> 已正式停止更新，内容已全部转移至这里，未来所有的更新也都在这里。新的站点，新的开始，新的旅程！~**
<!--more-->

----

## 缘起

最早看到有人推荐 Hexo 已经是很久以前甚至不太记得是什么时候的事了，印象中那会儿才弄懂什么静态网站、动态网站之类的词语，只听说 Hexo 是个基于 node.js 开发的静态博客框架。当时有点感兴趣，但因为没有时间又担心技术不足就没有再去做过多了解。

今年 5 月，公司内部测试尝试量子加速的加速效果时，其产品经理告诉我量子加速对静态资源的加速效果是尤其明显的，可以搭建个静态博客比如 Hexo 什么的和 Wordpress 对比试试看。至此，我开启了 Hexo 折腾模式，但因为没接触过 git 又不了解 node.js ，等我搭建好时已经浪费了很多时间。而我依然很忙，加之又想换域名又想导入原博客的内容还想找个看得顺眼的主题，觉得折腾起来实在有些麻烦就先搁着，等什么时候有空了在做。

直到有一天……

> 淦！ CloudXNS 监控提醒我网站宕掉了……
> -“运维兄弟，我的虚拟机咋了？ ip 是 xxx.xxx.xxx.xxx ”
> -“我问了没人回应我就下架了呢，这个是你的啊？”
> -“对呀对呀，我不在你们群里，可以恢复么？我都没备份……”
> -“……”

最后，公司运维小哥费了很大劲周末加班给我恢复了数据。为了避免未来数据备份的麻烦，我决定开始正式整理好 Hexo 并启用 Github page 服务了。

## 环境准备（折腾前必须掌握的）

有一些知识和条件是需要准备的，我这里简单说下，具体可依赖搜索引擎在不会的时候轻松完成。~~其实我能说是快忘掉了好吗？~~

### 开通 Github Page 服务

在自己的 github 主页中创建一个名为`用户名.github.io`的 Repository 即可。

### 本地安装 Git 并了解一些常用 git 命令

到 [Git 官网](https://git-scm.com/downloads)下载 Git 并安装， windows 安装过程中记得勾选 git bash ，这个是执行 git 命令的入口。

安装好后，启动 git bash 通过执行命令 `ssh-keygen -t rsa -C "github账号邮箱"` 生成 ssh 密钥用来关联 github 账号。

到系统用户文档页（ Linux 系统请去 `~/.ssh` 目录找）找到 id_rsa 文件，打开将内容复制，登陆 github 账号设置中添加 SSH Key 并复制进去。

### 安装 node.js

Hexo 就是基于 node.js 的，所以这个必须得安装了，安装了它才能执行后面的 npm 命令。
可以自行到 [Node.js 官网](https://nodejs.org)下载自己需要的版本， Windows 建议下载最多用户使用的稳定版，我试过最新版好像有点问题。

### 安装 Hexo

Git 和 node.js 安装好后，在本地创建一个文件夹例如 `E:\Hexo` 。
进入该文件夹，打开 git bash ，执行命令 `npm install -g hexo-cli` 即可将 Hexo 安装到该目录。

> Linux 安装过程请直接参考[ Hexo 官方操作文档](https://hexo.io/docs/)，照给出的命令执行就好了。当然，如果要放到 github page 上同样需要生成 ssh key 与 github 账号关联并创建一个 github 主页。

然后用如下命令启动博客：

```
$ hexo init
$ npm install
$ hexo server
```

启动后就可以在本机通过 `http://loaclhost:4000` 查看效果了。

## 折腾模式开启

本次折腾，主要做了以下事情：

1. 主题安装，选用了 [yilia](https://github.com/litten/hexo-theme-yilia) 主题，但又根据 [yelee](https://github.com/MOxFIVE/hexo-theme-yelee.git) 主题代码做了微量调整；
2. 导入原来 Wordpress 博客中的文章、页面和评论；
3. github 、 coding 、云主机多点部署并对新购买的域名做分区解析；
4. 生成 RSS 订阅、站点地图并将链接主动提交到百度站长平台等利于搜索引擎的优化工作。

### 主题安装

执行命令 `git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia` 和 `git clone https://github.com/MOxFIVE/hexo-theme-yelee.git` 下载上述两个主题，然后复制 yilia 创建名为 ephen 的主题副本，修改 Hexo 博客根目录（我的是 `E:\Hexo` ）的 _config.xml 文件将 theme 修改为 ephen 。

修改完工后删掉下载的两个主题目录。~~诶诶，怎么有种卸磨杀驴的感觉？~~

本主题的主要修改点：

1. 增加 Telegram 图标，和 Telegram 配置；
2. 增加显示和隐藏文章目录的支持；
3. 原来的评论框代码不够整齐好看，把 yelee 的评论框移植了过来；
4. 增加文章搜索的支持（需要安装插件： `npm install hexo-generator-search --save` ，并添加插件设置）；
5. 增加文章左右边栏按钮；
6. 其他看起来不太整齐的小部分代码；

另外，我还想收藏下这个： <http://www.tuicool.com/articles/fYZ7Zrj> 什么时候有空了再改。

> 如果有人喜欢我改后的主题，可以在下面留言

### 导入WordPress的页面/文章和评论

导入文章和页面需要安装这个插件： `npm install hexo-migrator-wordpress --save`

登陆 Wordpress 管理控制台，导出 WXR 文件，如下图所示：

![导出 wp 文件](https://pek3b.qingstor.com/imephen/20190426162220.png)

将导出的文件保存为 `wordpress.xml` ，执行命令 `hexo migrate wordpress wordpress.xml` 即可完成文章导入。

> 注意：

> 1. 如果新站和老站的域名和链接不同，导入之前需要先修改 xml 文件中类似 `<link>` 和 `<...url>` 之类的 Label 内容，以保持和预期站点一致
> 2. 导入文件如果是中文标题，可能会出现文件乱码，只能手工去修改了。完成导入后插件的光荣任务结束，处女座星人可以选择卸载它了： `npm remove hexo-migrator-wordpress`
> 3. 这个操作并不会导入评论，导入评论请继续往下看

由于 Hexo 的评论都是使用的社会化评论框，所以评论需要往社会化评论框（我用的是 Disqus ）中导入才能显示。这类工具一般都是支持转移的，向 Disqus 导入评论的操作如下：

1. 登陆 WP 控制台，导出时选择“文章”，导出文件命名为 `comments.xml` ，注意检查下文件中每篇文章的 URL 是不是和预期的 URL 一致（不一致会导致评论无法显示！）
2. 登陆 Disqus 后台，进入这个页面： `https://用户名.disqus.com/admin/discussions/import/platform/wordpress/`
3. 选择刚才编辑好的文件导入

这样，就完成了文章和评论的迁移了。

> 不过有点小遗憾，评论者头像都丢失，变更为默认头像了。

### 多点部署和分区解析

博客发布到 github page 和 coding page 需要安装这个插件： `npm install hexo-deployer-git --save`

然后到博客根目录找到 `_config.yml` 文件打开，将以下代码添加到最后：

```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo:
    github: https://github.com/用户名/用户名.github.io.git
    coding: https://git.coding.net/用户名/用户名.git
  branch: master
```

设置好后，只需要在 Hexo 根目录执行 `hexo g` 后再执行 `hexo d` 便可完成页面的部署。

和 Github 上一样， Coding 上也要预先安装 SSH Key 并开通 Page 服务。SSH Key 不需要重新生成，将之前在 github 上用的内容复制过去即可；开通 Page 服务时注意将部署分支修改成 master （默认分支是 Coding-page ）。

然后去 Github 和 Coding 相应 Page 项目设置中绑定自己的域名，如图所示：

![ Github 绑定域名](https://pek3b.qingstor.com/imephen/20190426162332.png)

![ Coding 绑定域名](https://pek3b.qingstor.com/imephen/20190426162355.png)

如果希望在自己的云主机中也部署一份，那么可登陆主机生成 SSH Key，将 Key 添加到 Github 或者 Coding 中（我添加到了 Github ），到网站文件根目录关联项目链接后执行 `git pull origin master` 即可完成站点部署。

最后，登陆 CloudXNS 做域名解析，我的解析设置如下图：

![ CloudXNS 域名解析](https://pek3b.qingstor.com/imephen/20190426162423.png)

### RSS 、站点地图及搜索引擎优化

这里不用多说，就是几个插件的安装和设置了。它们分别是： `hexo-generator-feed` ， `hexo-generator-sitemap` 和 `hexo-generator-baidu-sitemap` 。（两个 sitemap 一个是谷歌格式，一个是百度格式）

RSS 不需要做其他的操作，插件安装好后每次 `hexo g` 就会生成对应的 RSS 文件，需要时引用这个文件路径即可（比如主题设置中）。

针对 Sitemap ，需要到博客根目录去修改 `_config.yml` 文件，在 `plugins：` 下添加以下内容：

```
#sitemap
sitemap:
	path: sitemap.xml
baidusitemap:
	path: baidusitemap.xml
```

分别登陆谷歌站长平台和百度站长平台添加自己的新域名和站点地图。

按照百度站长平台的指引添加站点地图的位置：

![百度站点地图](https://pek3b.qingstor.com/imephen/20190426162507.png)

也可以到自己的云主机平台上执行命令手动提交，这种提交方式百度响应是最快的了。

因此我在我的云主机上写了个脚本，用于每次同步网站内容后将新链接提交到百度：

```bash
#!/bin/sh

DIR='/var/www/html'
SITE='http://ephen.me/'

cd ${DIR}/
git pull origin master > ${DIR}/.gitpull.log
cd ..
cat /dev/null > ${DIR}/freshurls.txt 
grep '/index.html' .gitpull.log | grep -v '\"' | awk '{print $1}' |while read line
do
	echo  "${SITE}${line}" >> ${DIR}/freshurls.txt 
done

curl -H 'Content-Type:text/plain' --data-binary @freshurls.txt "http://data.zz.baidu.com/urls?site=ephen.me&token=xxxxxxxxxx"

exit
```

如果有人也想使用此脚本，替换自己的目录、站点链接以及百度站长 TokenID 即可。

> 注意：该脚本不支持 URL 链接为中文的情况！！

## 结束语

这次折腾花了不小的精力，毕竟是自己的事情，所以为不占用工作时间多次加班到很晚才走。

本来不想记录的，担心下次出什么状况又要花大力气折腾，所以趁热记录下，同时也希望我的折腾过程能帮助到其他的新手。