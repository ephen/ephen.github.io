---
title: NodeJS 和 Hexo 版本升级记，继续填坑博客配置
date: 2022-2-25
categories: 
  - 博客折腾
tags: 
  - hexo
  - node
  - npm
  - stylus
toc: true
comments: true
---

继填坑《[博客更新后遗症](/2022/blog-sequela/)》后，心里总是隐隐有些不安。毕竟作为一个喜欢探索的人，根本无法容忍版本降级。

于是决定再重新把 NodeJS 装回最新版本看看到底会发生什么，以及能不能不通过降版的方式解决。

先仔细检查下当前的版本配置。惊奇的发现 hexo 版本并没有升级！（~~果然之前的一顿操作太秀逗了~~）

> 之前的`npm install -g hexo-cli` 命令应该是只升级了 `hexo-cli` ，我本地的 hexo 版本才 `V3.9.0` 。

好吧，那就一次性都升级了再说。

<!--more-->

## 升级 hexo 及全部插件

先安装 `npm-check` 和 `npm-upgrade` ：

```
npm install -g npm-check npm-upgrade
```

安装完后，执行 `npm-check` 即可检查本地各插件版本情况。

执行 `npm-upgrade` 可根据当前版本和最新版本比较，让用户确认和选择是否升级。
若用户确认升级，则会自动把 `package-lock.json` 和 `package.json` 文件内容进行更新后保存，然后执行：

```
npm update -g --save 
```

上述命令执行完毕，则所有通过 `npm-upgrade` 确认的插件全部都升级到最新（包括 Hexo）。

升级完后通过 `hexo version` 验证 Hexo 版本，已经更新到最新的 `6.0.0` 啦！

## 升级 NodeJS 和 npm

再使用 `npm-check` 检查，发现一些 Hexo 专属插件得不到验证。那我无法确认肯定是不甘心的嘛！于是求助朋友 [@Sukka](https://blog.skk.moe/) 后，他告诉我在 Hexo 上建议使用 `npm outdated` 命令来检查。

执行后发现没反应，苏卡卡提示我要 npm v7 版本以上才可以使用，这才想起我 NodeJS 还未升级。于是上官网下载最新支持 LTS 的版本 `v16.14.0` 重新安装。然后执行以下命令升级 npm ：

```
npm i npm@latest -g
```

最后执行 `npm outdated` 命令检查 Hexo 插件版本情况，发现还有两个插件需要更新：

![plugins_outdated](https://imephen.pek3b.qingstor.com/b_image/20220225154316.jpg)

去 Hexo 官网搜索这两个插件的作用，我发现我根本就没有用到 rsync 同步，于是将 `hexo-deployer-rsync` 插件卸载掉了。

## 修复并解决升级带来的问题

### 百度站点地图插件无法升级

接上一章节，由于想到国内还是以百度为主，于是升级插件：

```
npm update hexo-generator-baidu-sitemap
```

发现升级不成功，根据 Log 提示执行 `npm audit fix` 无法解决。再执行 `npm audit fix --force` 发现，把我的 Hexo 版本又降回了 `v3.9.0` ！于是又重来 `npm-upgrade` ，最终反反复复升级、降级了好几次，暴脾气灰姑娘直接就原地爆炸了。

于是访问百度站长平台去研究下站点收录的方式，以确定该插件在我博客上存在的价值。强行通过以下两点自证该插件价值为 0 后，**将之删除**：

1. 我博客上存在 `hexo-generator-sitemap` 插件，生成的站点地图（ `sitemap.xml` ）有被百度抓取收录，并且收录数量比百度专属插件生成的 `baidusitemap.xml` 还多几个（但其实对比 Google 后发现，百度对站点地图的支持并不友好，相比之下收录少得可怜）。
2. 我每次写完博文提交后，都会通过脚本（详见《[全新出发，正式启用 Hexo ！](/2016/hello-hexo/#RSS-、站点地图及搜索引擎优化)》文中所述）调 API 手动更新收录，这个收录是最全最实时的，比 SiteMap 好用。

删除插件及相关配置后，看着 `npm outdated` 命令执行后的干净对话框，心想终于清净了。赶紧看看本地博客运行的情况~

### external_link 配置报错

执行 `hexo clean` 后，在执行 `hexo g` 生成文件时，Hexo 就给了我第一个下马威：

```
INFO  Validating config
WARN  Deprecated config detected: "external_link" with a Boolean value is deprecated. See https://hexo.io/docs/configuration for more details.
```

尽管只是个 WARN 报错，但我自然是不甘心的。想来我从 `v3.9.0` 一步跨越到 `v6.0.0` ，仅大版本就跳了三级。

于是去找下 Hexo 官方有没有 Release Notes，一个个的看。终于在 `2020-7-29` 发布的 [Hexo 5.0.0 Released](https://hexo.io/news/2020/07/29/hexo-5-released/) 中找到了关于 `external_link` 配置的大变化。

同时阅读错误信息中提示的官方文档后，将 Hexo 根目录下的配置文件 `_config.yml` 中的 `external_link: true` 修改成如下配置：

```yml
external_link: 
  enable: true # Open external links in new tab
  field: site # Apply to the whole site
  exclude: ''
```

问题解决。再次运行没有出现报错~

### 文章路径发生变化

执行 `hexo s` 查看本地站点，点到文章《[博客更新后遗症](/2022/blog-sequela)》阅读时，发现其中引用的《[全新出发，正式启用 Hexo ！](/2016/hello-hexo/)》链接点击报错 `404` 。

仔细一看，我原本每篇博文都是放到年份文件夹中，而新的链接都直接用了我博文中标记的 `permalink` ，因此导致路径变化。

于是尝试修改文章中的 `permalink` 无果（加 `/:year/` 参数无法被识别到），又去求助于苏卡卡同学。苏卡卡同学很疑惑，说 Hexo 的老版本里面并没有在博文中使用 `permalink` 标签，让我直接去改 `_config.yml` 文件。

我查看了 `_config.yml` 文件后，发现其中配置并没有问题。既然如此，我删掉其中一篇博文的 `permalink` 标签后测试，发现年份是回来了，但博文路径变成了中文。这下明白了，原来 `_config.yml` 文件中 `permalink` 的 `:title` 变量取的是我的 Markdown 源文件的文件名。

于是修改所有的博文，删掉每篇博文中的 `permalink` 标签，然后将 `.md` 源文件名字改为对应被删除的 `permalink` 值。

如下图所示：

![修改博文源文件 Permalink ](https://imephen.pek3b.qingstor.com/b_image/20220223113316.jpg)

至于为什么我那么老的 Hexo 版本就在博文中有 `permalink` ？我仔细回想了下，应该是当年从 WordPress 迁移博文时，用的那个 `hexo-migrator-wordpress` 插件的杰作。当时觉得这个标记挺好用的，所以就在后来的每篇博文中沿用了。

### 主题调用的副标题和作者信息丢失

想来上方的链接都能出错，再也不敢掉以轻心。于是我将我线上正在运行的站点和本地生成的站点同时在浏览器打开并排仔细对比，映入眼帘就发现左边栏的差异：

![Subtile&Author Lost](https://imephen.pek3b.qingstor.com/b_image/20220223113045.jpg)

想起我之前在《[又来折腾博客了](/2019/blog-update/)》一文中提到的设置版权信息也有用到作者名字，遂点开任意一篇博文滚到页面底部看到“文章作者”后边也是空的。

查看本地代码中引用的是 `theme.author` ，但是在主题的 `_config.yml` 中并没有搜到 `author` 配置。查找我当前的文字内容，发现其来源于 Hexo 的 `_config.yml` 文件中的站点配置：

```yml
# Site
title: Ephen‘s Blog
subtitle: 奇葩一朵， IT 废材
description:
author: Ephen
language: zh-Hans
timezone:
```

于是将 `subtitle` 和 `author` 复制到主题的配置文件中，测试通过。

### 翻页转义字符显示异常

继续~~“欣赏”~~对比网站，发现页面底部文章列表下方的 Prev 和 Next 两个按钮显示出问题了，字符 `«` 和 `»` 没有正常显示，变成了 `&laquo;` 和 `&raquo;` 。

求助谷姐，找到了网友的解决方案（参考链接：https://www.chenb.top/2018/11/01/blog-way/）。

修改文件 `archive.ejs` ，找到此处源码，增加一行 `escape: false,` 来解决。

```
  <% if (page.total > 1){ %>
    <nav id="page-nav">
      <%- paginator({
        escape: false,
        prev_text: '&laquo; Prev',
        next_text: 'Next &raquo;'
      }) %>
    </nav>
  <% } %>
```

### Warning: Accessing non-existent property...

站点总算展示~~完美~~了，回过头来看到在运行 `hexo s` 时还有一些 Warning 报错：

```
$ hexo s
INFO  Validating config
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/ . Press Ctrl+C to stop.
(node:1656) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
(Use `node --trace-warnings ...` to show where the warning was created)
(node:1656) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
(node:1656) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
(node:1656) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
(node:1656) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
(node:1656) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
```

虽然是 Warning 不影响，但看着就是难受啊。经过度娘和谷姐的双重帮助下，大家的解决方案要么是降级 Nodejs 要么就是升级 Hexo 、升级插件 `hexo-renderer-stylus` 。

降级我知道肯定没毛病，但我不就白折腾了吗？而我的 Hexo 版本和 `hexo-renderer-stylus` 插件版本已经是最新了，各种暴力重装也没法解决。也试过网友们说的加 `"stylus": "^0.54.8"` 、`exports.lineno = null;` 等操作，依然还存在这个问题。

只好自己上阵弄了。运行 `npx cross-env NODE_OPTIONS="--trace-warnings" hexo s` 拿到报错详情：

```
$ npx cross-env NODE_OPTIONS="--trace-warnings" hexo s
INFO  Validating config
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/ . Press Ctrl+C to stop.
(node:11088) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
    at emitCircularRequireWarning (node:internal/modules/cjs/loader:707:11)
    at Object.get (node:internal/modules/cjs/loader:721:5)
    at Boolean.Node [as constructor] (D:\Hexo\ephen.github.io\node_modules\nib\node_modules\stylus\lib\nodes\node.js:42:23)
    at new Boolean (D:\Hexo\ephen.github.io\node_modules\nib\node_modules\stylus\lib\nodes\boolean.js:23:8)
    at Object.<anonymous> (D:\Hexo\ephen.github.io\node_modules\nib\node_modules\stylus\lib\nodes\index.js:57:16)
    at Module._compile (node:internal/modules/cjs/loader:1103:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1155:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Module.require (node:internal/modules/cjs/loader:1005:19)
    at require (node:internal/modules/cjs/helpers:102:18)
    at Object.<anonymous> (D:\Hexo\ephen.github.io\node_modules\nib\node_modules\stylus\lib\lexer.js:13:13)
    at Module._compile (node:internal/modules/cjs/loader:1103:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1155:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
...
```

找到错误信息中提到的 4 个文件的具体错误位置一个个的查看与报错相关联的内容，最终删除了 `.\node_modules\nib\node_modules\stylus\lib\nodes\node.js` 文件中的一段代码后解决。

修改前这段代码为：

```
var Node = module.exports = function Node(){
  this.lineno = nodes.lineno || 1;
  this.column = nodes.column || 1;
  this.filename = nodes.filename;
};
```

修改后：

```
var Node = module.exports = function Node(){};
```

我也不知道是啥意思，反正观摩了半天没发现出什么问题，等出了问题再来回溯吧。

## 后记：一点小插曲

其实这次还解决了一些 Web F12 发现的报错问题，都比较小也不太典型，因此就此略过。

另外，在查阅 Hexo 官方各大版本升级公告时，发现还有主题配置方式的大改变，所幸我的站点尚未被影响到。查问题时有段时间以为是我主题配置的问题，准备迁移到最新的配置方式上去。但是感觉搞起来特别麻烦，准备弃坑 Hexo 了，最后还是沉下心来把问题一个个的解决。

现在站点运行还算稳，就先这样吧。等以后什么时候有空了，再来看看主题的事。
