---
layout: post
title: "有关 GitHub 的高级操作"
date: 2019-05-19 23:40:00 +0800
categories: git
tags: git GitHub
author: Bobby
---

* content
{:toc}

我们经常需要在 GitHub 上搜索一些经典代码或者资料，除了使用关键字查找以外，你还了解哪些更高级的更精确的查找方式呢？



### 使用 in 限制关键字的搜索范围

形如：`[keywords] in:name` 或 `[keywords] in:description`，`[keywords] in:readme`。

`[keywords] in:name`：项目名称中包含关键字的  
`[keywords] in:description`：项目描述中包含关键字的  
`[keywords] in:readme`：项目 readme 文件中包含关键字的  

组合使用方式：`[keywords] in:name,readme`，即项目名称和 readme 文件中包含关键字的。如查找项目名称和readme 文件中包含“spring-boot” 关键字的 repository。

![github-adv-opt-in](/assets/images/2019/05/github-adv-opt-in.jpg)

### 按照 Star 和 Fork 数量查找

形如：`[keywords] stars|forks:>number`， `[keywords] stars|forks:>=number` 或 `[keywords] stars|forks:[number1]..[number2]`。

如：查找 Star 数大于等于 5000 的 spring-boot 项目:

```
spring-boot stars:>=5000
```

如：查找 Fork 数介于 3000 到 5000 的 spring-boot 项目:

```
spring-boot forks:3000..5000
```

组合查询，查找 Fork 在 100 到 1000 之间并且 Star 数在 2000 以上的 spring-boot 项目：

```
spring-boot forks:100..1000 stars:>2000
```

![github-adv-opt-fork-and-star](/assets/images/2019/05/github-adv-opt-forkandstar.jpg)

注意：`:>`, `:>=`, `:[number1]..[number2]` 这些中间都没有空格。

### 使用 awesome 加强搜索

形如：`awesome [keywords]`，awesome 系列一般是用来查找学习、工具、书籍类相关的项目。

如：搜索优秀的 redis 相关的项目，包括框架、教程等，`awesome redis`:

![github-adv-opt-awesome](/assets/images/2019/05/github-adv-opt-awesome.jpg)

### 查找某个地区内使用某种开发语言的小伙伴

形如：`location:[地区]`，`language:[语言]`。如查找西安（Xi'an）使用 Java 语言的小伙伴：

```
location:Xi'an language:Java
```

![github-adv-opt-location language](/assets/images/2019/05/github-adv-opt-locat-lang.jpg)

### 高亮显示某一行或者某段代码

`代码地址#L[number]`，高亮显示某行，`代码地址#L[number1]-L[number2]`，高亮显示 number1 行到 number2 行。

比如“https://github.com/JiaoJianbo/learning/blob/master/spring-security-sso/spring-security-sso-server/src/main/java/com/bravo/demo/ssoserver/config/SsoAuthorizationServerConfig.java” 是我的 GitHub 仓库中的一个Java 文件，现在想让第 27 行高亮显示，那么给这个 URL 后面加 `#L27` 即可。

![github-adv-opt-highlight-line](/assets/images/2019/05/github-adv-opt-highlight-line.jpg)

同理， 想让第 22~36 行高亮显示，那么给这个 URL 后面加 `#L22-L36` 即可。

![github-adv-opt-highlight-multi-lines](/assets/images/2019/05/github-adv-opt-highlight-multi-lines.jpg)

### GitHub 快捷键

在某一个 Repository 中，按 `t`，所有代码会按字母顺序排列。如，访问“https://github.com/JiaoJianbo/springcloud-tutorial”，正常显示如下：

![github-adv-opt-before-shortcut](/assets/images/2019/05/github-adv-opt-before-shortcut.jpg)

按了字母 `t` 之后：

![github-adv-opt-after-shortcut](/assets/images/2019/05/github-adv-opt-after-shortcut.jpg)

更多快捷键操作，请参考: [https://help.github.com/en/articles/using-keyboard-shortcuts](https://help.github.com/en/articles/using-keyboard-shortcuts)

*引申*：到这里就不得不提一下有关 GitHub 的 Chrome 浏览器插件 **Octotree**。同样，码云也基于 Octotree 开发了相同的 Chrome 插件 **GitCodeTree**。它能让你的 Repository 中的代码，按照工程树结构显示，并且可以点击直接导航到某一文件。跟我们的 IDE 开发工具左侧的工程结构目录导航一样。

![github-adv-opt-octotree](/assets/images/2019/05/github-adv-opt-octotree.jpg)

### 参考资料：

[互联网大厂高频重点面试题第2季(下)](https://www.bilibili.com/video/av48965905/?p=54) -- 有关 GitHub 操作部分。  
[Using keyboard shortcuts](https://help.github.com/en/articles/using-keyboard-shortcuts)

**特别推荐**本视频的所有内容，以及上部 - [互联网大厂高频重点面试题第2季(上)](https://www.bilibili.com/video/av48961087/?p=1)，视频中的每个知识点都值得仔细研读。
