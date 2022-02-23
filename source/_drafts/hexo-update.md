`npm install -g hexo-cli` 命令未升级 hexo
hexo 原始版本 V3.9.0 ，新版本 V6.0.0

先把 npm-check 更新到最新：
    npm install -g npm-check

执行 npm-check 可检查本地插件版本情况

更新 npm-upgrade ：
    npm install -g npm-upgrade

执行 npm-upgrade 可根据当前版本和最新版本比较，让用户确认和选择是否升级，用户确认升级即把 package-lock.json 和 package.json 文件内容更新，然后执行 npm update -g --save 即可把选择的插件全部升级到最新（包括 hexo）。

external_link 配置报错，去检查 hexo 版本升级的新特性（https://boriskp.github.io/upgrade-hexo-to-v5-1-1/）

查阅官方大版本升级公告：
https://hexo.io/news/2021/12/26/hexo-6-0-0-released/
https://hexo.io/news/2020/07/29/hexo-5-released/

修改 permalink ，现在文章里面的 permalink 标记被 hexo 用了，将文章里面的标记删除，文件名修改成英文

添加 subtile 和 author 标签到主题

hexo 环境建议用 `npm outdated` 命令检查，不要用 `npm-check`

npm v7 以上才能用 `npm outdated`，先 npm i npm@latest -g 升级 npm

nodejs 版本太低不支持，重新升级 nodejs ，再升级 npm

升级 node 后有 WARN 报错：重装 hexo-renderer-stylus 插件，无法解决。

移除 "hexo-deployer-rsync" 和 "hexo-generator-baidu-sitemap" 插件

hexo 转义字符显示异常：https://www.chenb.top/2018/11/01/blog-way/

主题修改，提交到官方，修改本地主题配置。