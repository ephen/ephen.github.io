---
title: 了不得了，使用 Caddy 反向代理 AxureCloud 还能自动提供 https 访问
date: 2025-2-20 
categories: 
  - [知识探索]
  - [博客折腾]
tags: 
  - caddy
  - deepseek
  - AI
  - Axure
  - https
toc: true
comments: true
---

## 折腾背景

2025 年 2 月开始，AxureCloud 官方不接受白嫖了，于是我打算自己搭一个个人自用的 AxureCloud 独立服务器。

看了下，现在市面上的云服务器价格都不便宜。我就想着，看能不能复用现有的资源，把 AxureCloud 搭建在个人博客所在服务器上，然后通过不同的域名访问。

不过，查阅了 《[Axure Cloud for Business On-Premises 官方部署文档](https://docs.axure.com/axure-cloud/business/install-on-premises/ )》后发现 AxureCloud 只能搭建于 Windows ，而我当前的博客服务器是 Linux 。正好，Windows 服务器还没咋玩过呢，那就弄个新的服务器试试。反正我的博客都是静态文件，应该用 Windows 也没什么问题。

虽然我都没折腾过 Windows，但好在现在 AI 技术发达，不懂的都问 AI 就行了。

作为一个多年更博主，记性越来越差了，顺便来博客记录下过程帮助长长脑子。

<!--more-->

## 部署 AxureCloud

### 服务器及软件准备

我在青云公有云 `Pek3` 区创建了一台配置为 2 核 4G 的 Windows Server 2019 云服务器，并绑定公网 IP 后，尝试下载 `AxureCloud for Business On-Premises` 官方安装包。

AxureCloud 官方安装包地址：[https://axure.cachefly.net/Enterprise/AxureCloud-Setup.msi](https://axure.cachefly.net/Enterprise/AxureCloud-Setup.msi) 

下载软件的过程中出现了一点小插曲：

> 由于低价的公网 IP 被设置了流量和带宽上限，导致下载上传均极其困难。我灵机一动想到了青云公有云还有对象存储，在同一个区域也许不通过公网传输，会不会快一点？于是，我尝试直接在本地下载安装包后上传到青云公有云 `Pek3` 区对象存储中，再到虚拟机中访问对象存储文件链接。接近 500MB 的文件，居然一眨眼的功夫（目测不超过 5 秒）就下载完成了！

AxureCloud 部署需要自行搭建数据库服务，官方支持 Mysql 或 MS SQL Server，我选择了 Mysql 。我如法炮制在本地下载了 `mysql-installer-community-8.0.41.0.msi` 后上传到青云公有云对象存储，提供虚拟机下载并安装 Mysql。

Mysql 官方下载地址：[https://cdn.mysql.com//Downloads/MySQLInstaller/mysql-installer-community-8.0.41.0.msi](https://cdn.mysql.com//Downloads/MySQLInstaller/mysql-installer-community-8.0.41.0.msi)

### 安装 Mysql

运行下载好的 `mysql-installer-community-8.0.41.0.msi` ，选择安装类型为 `Server only` ，点击 `Next` 。发现环境缺少 `VC 2019` 依赖，于是先去下载安装 `VC 2019` 。

`VC 2019-2022` 通用组件官方下载地址：[VC_redist.x64.exe](https://download.visualstudio.microsoft.com/download/pr/285b28c7-3cf9-47fb-9be8-01cf5323a8df/8F9FB1B3CFE6E5092CF1225ECD6659DAB7CE50B8BF935CB79BFEDE1F3C895240/VC_redist.x64.exe)

VC 安装完成后重新运行 `mysql-installer-community-8.0.41.0.msi` 以安装 Mysql ，还是选择 `Server only` ，然后依次按指引完成即可。

### 安装并启动 AxureCloud

执行下载好的 `AxureCloud-Setup.msi` ，根据 `Setup` 指定安装目录执行部署。流程结束后，会自动进入本地 AxureCloud Web 服务。

![AxureCloud Web 安装部署](https://iephen.pek3b.qingstor.com/b_image/202502-AxureCloud-Setup.png)

在 Web 页面上按提示配置好 Mysql 信息，并创建超级管理员用户，AxureCloud 就部署好了。

## 解决单服务器部署多站点的需求

最近 deepseek AI 挺火的，发现 Caddy 这个神器多亏了 deepseek 的帮忙。由于我不知道 AxureCloud 到底使用了什么技术栈提供的 web 服务，我只是简单的把我的需求告诉了它，它就回答我如何确认现有站点使用了什么技术栈并推荐了多种方式可以实现我的需求，包括 Caddy 。

![DeepSeek AI 解决单服务器部署多站点的需求](https://iephen.pek3b.qingstor.com/b_image/20250224140611.png)

![DeepSeek AI 介绍 Caddy](https://iephen.pek3b.qingstor.com/b_image/20250224140103.png)

### 安装 Caddy

官网下载 Caddy 可执行文件：[https://caddyserver.com/download.html](https://caddyserver.com/download.html)

创建一个 `C:/Caddy` 文件夹，把下载好的 `Caddy.exe` 放入该文件夹，并该文件夹路径添加到系统环境变量 PATH 中。

开启 CMD 窗口，执行 `caddy` 如果打印相关帮助，说明安装成功。

### Caddy 反向代理 AxCloud

由于 AxCloud Web 服务默认监听了 80 端口，我希望 Caddy 提供的服务能使用 `http` 的默认端口，因此需要将 AxCloud 的端口号调整下，然后再用 Caddy 反向代理过去。

通过查看 AxCloud 安装目录下的文件结构和配置内容，发现它使用了 `aspNetCore` 服务。我又借助了 DeepSeek 询问它属于什么技术栈，以及如何修改端口号等。

![DeepSeek AI 确认 aspNetCore 技术栈](https://iephen.pek3b.qingstor.com/b_image/20250224135450.png)

我按 DeepSeek 的提示找到了 AxureCloud 安装目录 `C:\AxureCloud\Share9\site` 下的 `appsettings.json` 文件，将 `urls` 配置项改为了 `http://*:5000` 。然后本地测试访问 `http://localhost:5000` 可成功打开 AxureCloud Web 服务。

接下来在 Caddy 中配置反向代理。还是让 DeepSeek 帮忙，我明确提供了我计划使用的域名（`axshare.ephen.me`）以及站点信息后，它告诉我需要在 Caddy 安装目录 `C:\Caddy` 下创建 `Caddyfile` 文件来配置，并直接提供了配置内容：

```
axshare.ephen.me {
    reverse_proxy localhost:5000
}
```

执行 `caddy run` 启动 Caddy 服务，日志显示服务启动，但是出现了 SSL 证书申领错误。想来也许是域名解析的问题，于是我去域名解析管理后台添加了当前服务器的公网 IP 到 `axshare.ephen.me` 的解析记录中，然后重新执行 `caddy run` 。

这次日志显示 SSL 证书申领成功，Caddy 服务启动成功。访问 https://axshare.ephen.me 即成功打开了 AxureCloud Web 服务。

### 下载博客文件并配置站点

我的博客是由本地 Hexo 驱动后，直接将静态文件托管在 GitHub 上的，由于众所周知的原因我当年并未直接将域名解析到 GitHub Page ，而是使用了一台 Linux 云服务器。当时我还写了一个 `shell` 脚本用于服务器端直接拉取 GitHub 上的博客文件后生成百度站点推送，详见：[全新出发，正式启用 Hexo！-RSS、站点地图及搜索引擎优化](/2016/hello-hexo/#RSS-、站点地图及搜索引擎优化)。

现在我想如法炮制，将这个脚本的功能移植到当前的 Windows 服务器上。但 Windows 原生并不支持 shell 脚本，也没有 `git` 。考虑到博客文件仅需要下载，因此我又借助了 DeepSeek AI 来询问如何把现有的 shell 脚本来通过 powershell 来实现，并提出不希望安装其他更多插件。

我完整的提供了 GitHub 下载链接和我计划安装的路径后，让 AI 帮我完成实现下载代码仓库、解压并去掉顶层目录后将内容移动到指定目录、生成站点推送 URL 等操作的 powershell 脚本。 AI 不仅给出了实现代码，还贴心的提示 Windows 上可能也没有 `curl` 命令，建议换用 `Invoke-WebRequest` 来实现百度站点推送。

以下为 AI 提供的 powershell 脚本完整内容：

```powershell
# 设置变量
$repoUrl = "https://github.com/ephen/ephen.github.io/archive/refs/heads/master.zip"
$downloadPath = "$PSScriptRoot\repo.zip"  # 下载到脚本运行目录
$tempExtractPath = "$PSScriptRoot\temp-extract"  # 临时解压目录
$siteRoot = "C:\wwwroot\ephen"          # 站点根目录
$siteDomain = "https://ephen.me"
$freshUrlsFile = "$PSScriptRoot\freshurls.txt"
$baiduApiUrl = "http://data.zz.baidu.com/urls?site=https://ephen.me&token=xxxxxxxxxx"  # 百度推送 API URL

# 1. 下载 GitHub 仓库 ZIP 文件
Write-Host "Downloading repository from GitHub ..."
Invoke-WebRequest -Uri $repoUrl -OutFile $downloadPath

# 2. 解压 ZIP 文件到临时目录
Write-Host "Extracting repository to temporary directory..."
Expand-Archive -Path $downloadPath -DestinationPath $tempExtractPath -Force

# 3. 获取解压后的顶层目录，然后将内容移动到站点根目录后删除临时目录
$topLevelDir = Get-ChildItem -Path $tempExtractPath -Directory | Select-Object -First 1
Get-ChildItem -Path $topLevelDir.FullName | Move-Item -Destination $extractPath -Force
Remove-Item -Path $tempExtractPath -Recurse -Force

# 4. 生成 freshurls.txt 文件
Write-Host "Generating fresh URLs..."
$htmlFiles = Get-ChildItem -Path $extractPath -Recurse -Filter *.html
$freshUrls = @()

foreach ($file in $htmlFiles) {
    # 获取文件的相对路径
    $relativePath = $file.FullName.Substring($extractPath.Length)
    $relativePath = $relativePath.Replace("\", "/")    
    # 拼接完整 URL
    $fullUrl = "$siteDomain/$relativePath"
    $freshUrls += $fullUrl
}

$freshUrls | Out-File -FilePath $freshUrlsFile -Encoding UTF8

# 5. 将 URL 列表推送到百度
Write-Host "Pushing URLs to Baidu..."
$freshUrlsContent = Get-Content -Path $freshUrlsFile -Raw
$headers = @{
    "Content-Type" = "text/plain"
}

Invoke-WebRequest -Uri $baiduApiUrl -Method Post -Body $freshUrlsContent -Headers $headers

Write-Host "Script completed successfully!"
exit 0
```

由于一开始我还没有在 Caddyfile 中配置博客信息，当调试 PowerShell 脚本时出现推送百度报错时我还以为是正常现象。但好在整个脚本运行中没有什么逻辑上的问题，并且确实都已正确生成了博客的静态文件。

于是我配置了 Caddyfile ，添加了博客的静态文件站点信息：

```
ephen.me {
    root * c:/wwwroot/ephen/
    file_server
}
```

有了之前的经验，我这次直接先改好了域名解析，然后执行 `caddy run` 遍一次性配置成功。再次尝试运行 PowerShell 脚本，发现百度推送仍然失败。于是我登录百度站长后台查看，发现原来是百度修改了 API 推送规则，每天免费推送配额仅 10 条了。

考虑到我本人基本上年更都做不到，如果只推送新增 Url ，10 条其实很够用了。于是我又让 AI 继续修改脚本，在保存文件时先做文件比较，只保存新增或有修改的文件，然后将这些变动的文件拼成 Url 放到推送列表。这样一来，百度推送终于测试成功。

以下是 DeepSeek AI 为实现此需求提供的关键代码：

```powershell
# 函数：计算文件的 SHA256 哈希值
function Get-FileHash256($filePath) {
    $hash = Get-FileHash -Path $filePath -Algorithm SHA256
    return $hash.Hash
}

...

# 遍历临时目录中的文件，比较并移动到站点根目录，将新文件或内容发生变化的文件覆盖
$changedFiles = @()
Get-ChildItem -Path $topLevelDir.FullName -Recurse | ForEach-Object {
    $relativePath = $_.FullName.Substring($topLevelDir.FullName.Length + 1)
    $destinationPath = Join-Path -Path $siteRoot -ChildPath $relativePath

    if (-not (Test-Path -Path $destinationPath) -or (Get-FileHash256 $_.FullName) -ne (Get-FileHash256 $destinationPath)) {
        Write-Host "Updating file: $relativePath"
        Copy-Item -Path $_.FullName -Destination $destinationPath -Force
        $changedFiles += $relativePath
    }
}

...

# 生成 freshurls.txt 时仅包含被替换或新增的文件对应的 URL
$freshUrls = @()
foreach ($file in $changedFiles) {
    $fullUrl = "$siteDomain/$($file.Replace('\', '/'))"
    $freshUrls += $fullUrl
}
```

虽然是已经实现了全部需求，但我运行几次之后发现，从下载到解压到文件比对，这个效率实在是太低了。然后尝试给 AI 提需求通过增加缓存哈希的方式解决效率问题，一直调试不成功。又想到，如果我删除了某些文章，这个脚本不会去删除服务器上的文件。

最终我还是决定下载安装 Git ，通过 Git 自身的版本控制信息来实现推送的 Url 列表。（也就是折腾来折腾去，还是回归了我原始的 shell 脚本。hhhhhhhh

我下载安装了 Git 之后，在预期的站点目录执行了 `git init` 和 `git clone ...`，然后脚本中就可以通过 `git pull origin master` 来获取内容变化了。

顺便让 AI 帮我检验，安装了 Git 是不是原来为 Linux 服务器写的脚本其实也可以直接在 Windows 上运行。AI 提示除了有个 `cat /dev/null >` 的命令由于 Windows 上没有 `/dev/null` 的设备，为了实现清空文件可以直接替换为 `>` ，其他的命令都可以直接运行。不过它还是给了更多的优化建议，最终 AI 帮我修改后的 shell 脚本如下：

```bash
#!/bin/sh
LoDIR=$(cd "$(dirname "$0")" && pwd | sed 's#/c/#C:/#')/ # 获取当前脚本的绝对路径并转换成 Windows 风格
GitDIR='C:/wwwroot/ephen/' # Git 仓库的路径，即站点根目录
SITE='ephen.me'

cd "${GitDIR}" || exit 1
git pull origin master > "${LoDIR}/.gitpull.log"
cd ..
> "${LoDIR}/freshurls.txt"
grep '/index.html' "${LoDIR}/.gitpull.log" | grep -v '\"' | awk '{print $1}' | while read -r line
do
    echo "https://${SITE}/${line}" >> "${LoDIR}/freshurls.txt"
done

curl -H 'Content-Type:text/plain' --data-binary "@${LoDIR}/freshurls.txt" "http://data.zz.baidu.com/urls?site=https://ephen.me&token=xxxxxxxxxxx"
```

### 用 Caddy 实现 www 转发

通过前面的配置，我已实现成功在一个 Windows 服务器同时搭建 AxureCloud 和个人博客，然后用不同的域名访问。目前我配置的博客域名为 `ephen.me` ，我希望通过 `www.ephen.me` 也可以访问，并自动跳转到当前的 `ephen.me` 。

于是 DeepSeek 推荐我在 Caddyfile 中又增加了一条配置：

```
www.ephen.me {
    redir https://ephen.me{uri} permanent
}
```

我在域名解析后台添加了 `www.ephen.me` 的 A 记录也指向这台服务器后，执行 `caddy run` 测试发现配置成功。

至此，完整的 Caddyfile 配置如下：

```
axshare.ephen.me {
    reverse_proxy localhost:5000
}

ephen.me {
    root * c:/wwwroot/ephen/
    file_server
}

www.ephen.me {
    redir https://ephen.me{uri} permanent
}
```

## 配置 caddy 服务自启动

既然是要做站点，我当然不希望通过一直挂着前台执行 `caddy run` 来提供 http 服务了。那么，能不能将 caddy 配置成 Windows 服务进程来实现呢？

AI 真的是个好工具！我就这么一问，它就给了我答案：

![DeepSeek AI 配置 Windows 服务](https://iephen.pek3b.qingstor.com/b_image/20250224135318.png)

于是，我通过在 cmd 窗口执行以下命令配置了一个 caddy 服务：

```
sc create caddy start= auto binPath= "C:\Caddy\caddy.exe run"
```

然后，通过 sc 原生的查询及启停命令 `sc.exe query caddy`、`sc.exe start caddy` 和 `sc.exe stop caddy` 来对这个新加的 caddy 服务做系统管控。当然，sc 命令还有其他更多功能，这里就不一一赘述了。

## 番外一：远程访问 mysql 数据库

前面搭建的 AxureCloud 选取了 mysql 数据库来存取数据，由于服务器各方面都已经搭建好配置好，如非必要我不想再进云服务器的 web 控制台。所以，期望能在本地直接远程访问数据库。

于是，我让 DeepSeek AI 教我配置了 mysql 的远程访问权限：

```powershell
# 登录到 MySQL 数据库
mysql -u root -p

# 进入数据库 command 后，切换到 mysql
> USE mysql;

# 创建 root 用户并设置密码
> create user root@'%' identified by '123456';

# 授予 root 用户所有权限
> grant all privileges on *.* to root@'%' with grant option;
> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
> FLUSH PRIVILEGES;
```

配置完成后，我就可以在本地通过 mysql 客户端工具（如 HeidiSQL ）访问服务器上的 mysql 数据库了。

## 番外二：远程执行博客同步脚本

既然不想再进服务器的 Web 控制台，那前面写的博客同步脚本也得要能远程执行。这事儿我也交给了 DeepSeek AI 来帮我解决：

![DeepSeek AI 指导如何远程执行 Windows 上的 bash shell 脚本](https://iephen.pek3b.qingstor.com/b_image/20250224134314.png)

于是，我根据 AI 的指引，在服务器上配置了 OpenSSH 服务并启动，之后就可以使用一条命令实现远程执行了 shell 脚本了。 

以下是服务器上的 SSH 服务配置过程（在 PowerShell 中执行）：

```powershell
# 检查OpenSSH功能是否可用
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Server*'

# 安装OpenSSH服务器
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 启动OpenSSH服务
Start-Service sshd

# 设置OpenSSH服务为自动启动
Set-Service -Name sshd -StartupType 'Automatic'

# 开放防火墙端口
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

然后，在本地 PC 开一个 Powershell 窗口执行以下命令：

```powershell
ssh Administrator@ephen.me "bash -lc 'cd /c/Caddy && sh ephen-sync.sh'"
```

回车后信任指纹，输入密码，就实现了远程执行服务器上的脚本了。

## 特别鸣谢

1. [DeepSeek - 探索未至之境](https://chat.deepseek.com/)
2. [Caddy 官方中文文档](https://caddy2.dengxiaolong.com/docs/)
3. [盐袋子 - AxureCloud 部署](https://www.besalt.top/article/p013)