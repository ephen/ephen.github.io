---
title: 给网站添加备案号展示
date: 2017-2-22 
categories: 
  - 博客折腾
tags: 
  - hexo
  - 备案
toc: true
comments: true
---

## 折腾原因

由于在公司折腾 `CloudXNS` 和`牛盾`，有个备案比较好开展工作。正好上次公司 CDN 网监来查，要求我把备案展示出来。想想这事情也该办了，搞起……

唔，还是提一嘴，姐无任何前端知识储备，大牛路过请点右上红叉关闭。

## 目标

1. 备案号放至左侧边栏下方，链接到工信部网站；
2. 备案号可进入主题 `_config.yml` 中配置修改；
3. 文章页显示目录时不做展示；
4. 放弃适配移动端。

<!--more-->

## 折腾过程

### 添加备案号并写入配置文件

这部分比我想象中简单多了。照配置文件的 `AboutMe` 抄一抄居然就完成了。

`left-col.ejs` 文件中添加以下代码：

```js
<% if (theme.beian){ %>
    <a class="beian" href="http://www.miitbeian.gov.cn/state/outPortal/loginPortal.action" target="_blank"><%=theme.beian%></a>
<%}%>
```

`_config.yml` 中添加：

```yml
## 是否展示备案号
### beian: false	# 不开启
beian: 鄂ICP备15003996号
```

功能测试通过！

### 备案号模块样式

样式主体是照抄的博客说明模块，以下是样式代码，续写到 `main.styl` 文件中的 `#header` 部分：

```css
.beian{
	position: relative;
	bottom: 15px;
	font-weight: 300;
	cursor: pointer;
	text-transform: uppercase;
	float:none;
	text-align: center;
	display: -webkit-box;
	-webkit-box-orient: horizontal;
	-webkit-box-pack: center;
	-webkit-box-align: center;
	li{
		cursor: default;
		a{
			font-size: 14px;
			min-width: 300px;
		}
	}
}
```

`bottom` 表示在页面底端保留的高度，配合 `position: relative;` 以保持在页面底端。

### 显示目录时隐藏备案号

本来不想做这个的，后来发现目录在时摆不好这个位置，同时如果目录过长过多会影响页面的美观，因此还是隐藏比较好。

目录是否显示的原本样式是在 `toc.ejs` 里面添加的 `<style></style>` 标签，在标签中添加：

```css
#header .beian {
display: none;
}
```

就可以了。

### 点击“显示目录”和“隐藏目录”时切换动画

侧边栏目录初始状态动画展现在 `pc.js` 文件中，把 `.beian` 元素加进去：

```js
var miniArchives = function(){
        $(".post-list").addClass("toc-article");
        $("#post-nav-button > a:nth-child(2)").click(function() {
            $("#post-nav-button .fa-bars,#post-nav-button .fa-times").toggle();
            $(".post-list").toggle(300);
            if ($(".toc").length > 0) {
                $("#toc, #tocButton").toggle(200, function() {
                    if ($(".switch-area").is(":visible")) {
                        $("#toc, .switch-btn, .switch-area, .beian").toggle();
                        $("#tocButton").attr("value", yiliaConfig.toc[0]);
                        }
                    });
            }
            else {
                $(".switch-btn, .switch-area, .beian").fadeToggle(300);
            }
        });
}()
```

然后 `toc.js` 文件中有原本就有其他元素的切换动画，同样加个 `.beian` 即可：

```js
$("#tocButton").click(function() {
    if ($("#toc").is(":hidden")) {
        $("#tocButton").attr("value", valueHide);
        $("#toc").slideDown(320);
        $(".switch-btn, .switch-area, .beian").fadeOut(300);
    }
    else {
        $("#tocButton").attr("value", valueShow);
        $("#toc").slideUp(350);
        $(".switch-btn, .switch-area, .beian").fadeIn(500);
    }
})
```

### 测试成功

测试时chrome调试外还特地拿移动设备也试了下，并没有备案号，符合预期。

简单分析看了下，发现移动端的功能基本集中于 `mobile.js` 文件中，这个文件我并没有添加备案的相关信息。

## 总结

蛮早就想搞这个备案号展示的，可是因为害怕搞不定拖了两三个月，可真正做起来不到半天就搞完了。所以，哪有那么难？

总的来说，只要有想法，瞎抄都能抄到答案，差别只在于愿不愿意去做。

很多其他事情大概也是一样的：**没做的时候以为是大山，做了才发现很简单。**
