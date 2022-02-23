---
title: 怎么避免无效的软件测试场景？
id: 105
date: 2015-03-27
categories:
  - [心情驿站]
  - [工作笔记]
tags:
  - 测试
toc: false
---

博主最近在知乎上赚点赞，随便进了一个问题随便的回答了下。

原文链接：[https://www.zhihu.com/question/28911306/answer/42860809](https://www.zhihu.com/question/28911306/answer/42860809) （欢迎小伙伴飘过去点个赞！~）

为了不让博客太空旷，遂搬来这里。回答仅是个人见解，有其他想法或是说得不对的地方请留言讨论。

以下是原问题以及我的回答。

<!--more-->

> ## 怎么避免无效的软件测试场景？

> 很多时候，我们测试的时候会考虑方方面面的问题，这固然好，有足够的覆盖，但往往发现有些朋友设计的时候，很多case其实是完全可以避免的，或者不可能发生的场景，有些是即使发生了，对客户也没任何影响，有人有类似的总结吗？比如怎么避免无效场景。</div>

**辛德瑞拉CiCi：**

软件测试本来就是会假定所有情况都有可能存在的，在单元测试中需要对代码的每一个逻辑条件进行测试，集成测试也是需要对每一个组件逻辑进行测试，只要是逻辑存在的那么都是有效的，每一个逻辑错误都有可能成为软件缺陷的根源。

只有在α和β测试中会更考虑实际的使用情况，但这些的前提是已经对整个产品进行过全面的单元测试、集成测试、系统测试的情况下。

对于撸主的问题，不妨拆解为以下两个问题。
1. 如何尽可能少的设计用例覆盖尽可能多的测试面？
2. 如何全面理解用户需求，做好用户测试？ （都是从用户测试的层面来说的，单元测试、集成测试、系统测试的请忽略）

### 第一个问题，可以从以下几个方面改进

1. 了解被测软件的业务逻辑，根据场景业务逻辑线设计用例。全部用例结合到一起，尽可能保证让每一个业务逻辑分支被覆盖到至少一次。
2. 用例设计完后与相关业务人员进行用例评审，发起评审时提前让大家看一下，并提示主要关注用例预置条件和预期结果是否正确、用例是否有错漏或冗余。
3. 执行用例时先挑出最可能发现问题的用例做冒烟测试（也就是场景评级），而且冒烟过程中你也会发现会有其他需要修改的地方。

### 第二个问题，属于用户感知层面的。如果你实在假装用户做不到，那么不妨自己做一个真用户，看看是不是会发现不一样的东西。