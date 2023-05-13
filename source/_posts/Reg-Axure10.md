---
title: 不用破解注册，也许可以永久试用订阅式的 Axure RP 10 软件
date: 2023-5-12
categories:
    - [工具介绍]
    - [工作笔记]
tags:
    - Axure
    - PM
    - RunAsDate
    - libfaketime
toc: true
---

众所周知，Axure 9 升级到 Axure 10 后，就从原本的永久授权变更为订阅授权。同时 Axure 10 的订阅注册激活码与 Axure Cloud 的用户账号强绑定，激活时将根据登录账号在线跳转到官方网站实时获取注册状态。

一直以来，我还挺喜欢尝试新鲜事物的，尤其是各种软件、网站、科技产品等。Axure 作为互联网产品经理的必备软件，在 Axure 10 正式版诞生后，它默认提供一个月的试用期，于是我就早早的参与到试用大军中。

别说还挺好用的，尤其是动态面板多状态一起编辑，以及一键将表单转换成中继器并将中继器表格直接浮于内容区，真的有惊艳到我。（虽然工作中这俩功能其实并不常用hhh

尽管被惊艳到，但由于我已经习惯了 Axure 9 ，试用期一过就没有再去尝试了。并且由于高版本文件无法用低版本打开，所以也没敢将 Axure 10 作为正式的生产力工具。

前段时间心情不错，突然又想折腾一番～

<!--more-->

## 背景原理

既然可以免费试用一个月，那么验证时一定跟你打开软件的时间有关系。

Axure 这款软件性质是本地软件，正常来讲除了原型发布时其他时间均无需联网。

尝试了下：将电脑断网后通过离线修改 PC 本地的系统时间到可试用时间，再打开 Axure RP 10 确实可以继续试用。而且只要当前运行的软件不退出（窗口不关闭），即使恢复联网电脑系统时间被同步到正确到时间，依然还可以正常使用。

可是，咱总不能为了使用这款软件每次去断网修改系统时间后再联网吧？那可多麻烦。

就没有什么办法可以让这个软件启动的时候就自动完成这些操作吗？

答案当然是“有”，而且可能比咱想象的更好用。

## Windows 环境

要说我能找到这样的工具，得益于软件测试的出身。当遇到有关时间的测试时，总会去获得一些奇怪的知识。

Windows 兼容性高，逆向工程较成熟，反编译的成本相对来说比较低，所以能为软件修改启动时间的工具还挺多的。

下面就以 Axure 10 的使用为例，挑几个比较知名的工具介绍下。

### RunAsDate

官方网站：<https://www.nirsoft.net/utils/run_as_date.html>

RunAsDate 是一个绿色便携的软件，不用安装也不写注册表，就地执行即可。

为了方便永久试用目标软件，它可以在指定目标软件后创建新的目标软件快捷方式。此后点击该快捷方式即可运行目标软件，并且目标软件的系统启动时间就是你设置的时间。

PS：创建快捷方式后会在同目录生成 RunAsDate 的配置文件，同时也可以在同目录通过写一个语言配置文件达到为 RunAsDate 汉化的目的（官方网站上有提供汉化配置下载）。因此建议将 RunAsDate 工具及其配置放到一个独立的目录，然后可将该目录和目标软件目录放到一起。

### CrackLock

官方网站：<https://william.famille-blum.org/software/cracklock/>



## MAC 环境

### libfaketime

这是一个开源的程序库，其 GitHub 地址：<https://github.com/wolfcw/libfaketime>

这个库在 Linux 和 MAC 上都可以用，通常用来测试 Linux 上开发的软件。虽然 MAC 上可能会惯常用 UI 应用程序（就像 Axure 一样），其实也是可以找到对应的可执行程序来使用的。

话不多说，先安装起来。

打开终端，在本地创建一个目录后，`git clone https://github.com/wolfcw/libfaketime.git` 下载源码，然后进入目录执行 `sudo make install` 输入本机登录密码即可安装成功。

安装成功后，可通过 `faketime` 命令测试软件是否能达到效果。例如在 `date` 命令前置 `faketime` 以输出时间：

```
~ » faketime -f '2022-04-01 00:00:00' date                            
2022年 4月 1日 星期五 00时00分00秒 CST
```

然后同样的道理可以用到应用程序上，将上面的 `date` 命令替换为 App 的全路径可执行程序即可。例如可以试试将 Axure RP 10 的可执行路径 `/Applications/Axure\ RP\ 10.app/Contents/MacOS/Axure\ RP\ 10` 放到 `faketime` 后执行：

```
~ » faketime -f '@2022-04-01 00:00:00' /Applications/Axure\ RP\ 10.app/Contents/MacOS/Axure\ RP\ 10 
Environment version 6.0.0
Framework description .NET 6.0.0-rtm.21522.10
```

可以看到已正常启动软件，并且右上角提示还有 11 天过期（我是 2022 年 3 月份申请试用 Axure 10 的）：

![Axure RP 10](https://iephen.pek3b.qingstor.com/b_image/AxureRP10-184716.png)

可如果每次都要这样启动，是不是未免也太麻烦了点？

其实也不用。

我们进入到应用程序列表，双指点击 `Axure RP 10` 后单指点击“显示包内容”：

![Axure RP 10](https://iephen.pek3b.qingstor.com/b_image/Axure-190101.png)

打开后会看到当前目录下有一个名为 `info.plist` 的文件：

![Axure RP 10](https://iephen.pek3b.qingstor.com/b_image/Axure-191824.png)

然后编辑这个文件，在 `<dict>` 内加入如下代码后保存：

```xml
<key>LSEnvironment</key>
    <dict>
        <key>DYLD_FORCE_FLAT_NAMESPACE</key>
        <string>1</string>
        <key>DYLD_INSERT_LIBRARIES</key>
        <string>/usr/local/lib/faketime/libfaketime.1.dylib</string>
        <key>FAKETIME</key>
        <string>@2022-04-01 00:00:00</string>
        <key>FAKETIME_STOP_AFTER_SECONDS</key>
        <string>10</string>
    </dict>
```
![Axure RP 10](https://iephen.pek3b.qingstor.com/b_image/Axure-192011.png)

编辑完后重启电脑，或者打开终端执行命令： `/System/Library/Frameworks/CoreServices.framework/Frameworks/LaunchServices.framework/Support/lsregister -v -f /Applications/Axure\ RP\ 10.app`

设置完成后，打开启动台点击 Axure RP 10 的图标即可正常使用咯。

不过，此方法可能会在软件更新或升级后因重置了 `info.plist` 而失效，需要再次做上述操作后才可使用。

## 结语

以上方法仅供个人测试，商业使用还请寻找官方渠道获得授权。

由于各软件的验证机制的原因，不保证通用，也无法保证以上方法永久有效，还请低调试用。

PS：若试用不成功请自行解决或购买授权，本人不会提供任何技术解答。