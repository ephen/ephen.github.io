---
title: 博客 Hexo 工作区同步至 Mac ，多个电脑可同时写博客啦～
date: 2022-2-20
categories: 
  - 博客折腾
tags: 
  - hexo
  - git
  - mac
toc: true
comments: true
disableNunjucks: false
---

一个月前，公司资产部在一年一度处理离职员工回收的资产时，我获得了一台 MacBook Air 。

由于过去从未用过 Mac ，在摸清了大致的用法后，突然想再折腾下博客。

之前我没有个人电脑，每次更换工作电脑（均为 Windows 操作系统）时，直接用 U 盘复制本地 Hexo 工作目录到新电脑上，安装相关依赖后使用。

这次准备把新到的 MacBook Air 作为我的个人电脑使用，然后让个人电脑和工作电脑都可以用来更新博客，所以需要考虑 Hexo 源文件多端同步了。

<!--more-->

## 为什么要同步 Hexo 源文件

用 Hexo 搭建博客的同学应该比较清楚： 当我们用 markdown 写好博文后，需要先执行 `hexo g` 命令生成 html 文件，再通过 `hexo d` 命令将生成的 html 文件发布到 Github Page ，这样 GitHub Page 即可直接当个人网站使用了。

所以 Github Page 上只有 html 格式静态网页，而我们通过 markdown 写的源文件，以及 hexo 主题文件和插件等信息都不会上传。

`hexo g` 的动作相当于网页编译行为，而当我们需要在其他电脑上也可以同步写文章时，就需要把 hexo 工作区进行同步。

## 使用 GitHub 存储 hexo 源文件

Github 是个很好的代码协同平台，我的博客本身也是放 GitHub Page ，因此我选择在 GitHub Page 项目源代码上新建一个分支来存放 hexo 源文件作为我的线上工作区。

登录 GitHub 后进入到我博客所在项目查看源代码，基于 `master` 新建一个名为 `suorce` 的分支，并将该分支设置为默认分支。

![GitHub 分支管理](https://imephen.pek3b.qingstor.com/b_image/20220220144314.png)

回到工作电脑上，新建一个工作区目录，执行 `git clone 项目地址 .` 将 github 上的代码下载下来。由于默认分支现在是 `source` ，因此本地工作区也默认为 `source` 分支。

由于远程分支是基于原分支复制的，因此内容和 master 一样，都是博客 html 文件。我除了保留此目录下隐藏的 `.git` 文件夹外，删除其他所有文件，再把之前工作目录下所有文件（含隐藏文件）复制过来。

![文件夹对比](https://imephen.pek3b.qingstor.com/b_image/20220220160320.png)

测试下新的工作目录是否能正确运行 hexo 及 Node.js 相关指令，并本地预览确认无误。

然后创建 `.gitignore` 文件，内容如下：

```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

> `.gitignore` 文件用于指明哪些文件不用上传或同步。
1. `node_modules/` 是 hexo 用到的 node.js 插件，这个可以在 node.js 环境中直接通过 `npm install` 命令直接安装，而所有用到的插件信息在目录下的 `package.json` 文件中有记录；
2. `public/` 是在执行 `hexo g` 命令后生成的全部网页文件及目录，这部分内容不稳定，当执行 `hexo c` 命令后，该目录就会被删除；
3. `.deploy_git/` 是在执行 `hexo d` 时从 `public/` 目录复制过来的，用于发布至 GitHub Page ，这部分内容最终会同步至原本的 `master` 分支。

做完这些操作后，依次执行以下命令来完成提交：

```
git add .
git commit -m "XXXX"
git push
```

此时再登录到 GitHub 网站上，即可看到最新的源代码保持与本地最新工作目录一致了（已忽略的文件夹除外）。

## Mac 环境安装及配置

### 环境依赖准备

打开个人电脑，通过终端命令确认系统自带 `git` 。

执行 `ssh-keygen -t rsa -C "github账号邮箱"` 生成ssh密钥后添加到 Github Settings --> SSH and GPG Keys 页面。

执行 `git config --global user.name "Ephen"` 和 `git config --global user.email 个人邮箱` 配置 Git 全局用户名及邮箱。

登录 [node.js 官网历史版本下载页](https://nodejs.org/en/download/releases/)搜索 v12.22.10 （保持与工作电脑上的版本一致）下载 `.pkg` 的程序安装包，按提示步骤完成安装。

![Node.js MAC 安装包下载](https://imephen.pek3b.qingstor.com/b_image/20220220153316.png)

### 项目同步

到个人目录创建一个分类文件夹，执行 `git clone 项目地址`命令将默认分支 `source` 下载到本地。

执行 `npm install hexo-cli -g` 安装 Hexo 。

> 源文件已经通过 git 下载，并直接使用的之前的配置，因此不需要执行 `hexo init` 来初始化博客了。

执行 `npm install` 将自动安装 `package.json` 文件中的全部插件。执行过程中会有一些报错，按照报错提示上强制继续即可。（忘记截图了...）

### 本地预览及环境变量

像在 Windows 上一样输入 `hexo s` 回车后发现执行不了，一开始还以为是之前 `npm install` 报错导致 hexo 没装上又重装了一遍，确认装上了后还是不能执行，估计就是环境变量问题了。

查阅[ Hexo 官方文档](https://hexo.io/docs/)后，发现可以使用 `npx hexo <command>` 命令，测试本地预览无误后开始折腾环境变量配置。

根据过去的 Unix/Linux 经验，通过执行 `export PATH="$PATH:./node_modules/.bin"` 后测试验证确实是缺失环境变量，于是试着将 PATH 加到 `~/.bash_profile` 文件中。退出用户后重新登录，发现并不能自动加载该文件。（由此证明，Hexo 上的官方文档 Guide 不对，需要更新）

再次求助于度娘，发现 Mac 上默认终端是 zsh （其实可以用一个 `echo $SHELL` 的命令即可查询），用户环境变量的文件是 `~/.zshrc`，于是执行以下命令后环境变量配置成功：

```
echo 'PATH="$PATH:./node_modules/.bin"' >> ~/.zshrc
```

## 折腾结束

通过前述设置后，每次写文章前先 `git pull` 下载源文件，写完文章后除了 Deploy 发布外，还需要记得同时更新下 Github 上的 source 主分支。

当然，如果还未写完就准备换电脑续写的话，就只用更新 source 而无需 deploy 发布。

本篇文章即通过个人电脑写一半后，到公司用工作电脑续写完成。