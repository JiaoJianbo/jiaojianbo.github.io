---
layout: post
title: "《RESTful Web Services 中文版》读书笔记（三）"
date: 2019-02-28 11:00:00 +0800
categories: RESTful
tags: RESTful REST
author: Bobby
---

* content
{:toc}

最近还是继续在学习 REST、 微服务等相关知识，想从理论方面对这些知识有个更深入、更全面的了解。《RESTful Web Services 中文版》是目前正在看的。对其中有些内容作了摘录，便于以后查阅和复习。
其中，引用的书中的内容，版权归作者所有。如有侵权，请联系删除。



# 第九章 服务的技术构件

## 表示格式

### XHTML

媒体类型：application/xhtml+xml。HTML 用于 human web，而 XHTML 用于 programmable web。XHTML 在 HTML 的基础上施加了一些约束，这些约束使得 XHTML 文档同时也是有效的 XML 文档。

### 含有微格式的 XHTML

媒体类型：application/xhtml+xml。微格式属于轻量级的标准，它对 XHTML 加以扩展，给 HTML 标签增添特定领域的语义。微格式并没有重新发明数据存储技术，而是采用现有的 HTML 标签。语义内容通常由属性（class, rel 和 rev 等）的自定义值来表达。

官方微格式：hCalendar, hCard, rel-license, rel-nofollow, rel-tag, VoteLinks, XFN (XHTML Friends Network), XMDP (XHTML Meta Data Profiles), XOXO (Extensible Open XHTML Outlines)。另外还有一些（*当时的*）微格式草案：geo, hAtom, hResume, hReview, xFolk。

### Atom

媒体类型：application/atom+xml。Atom 是一种用于描述具有时间戳的条目列表（lists of timestamped entries）的 XML 词汇。Atom 有专门用于表达相关语义的标签，如：作者、参与者、语言、版权信息、标题、类别，等等。

Atom 列表被称为提要（feed），Atom 列表里的各项被称为条目（entry）。有些提要采用的是某个版本的 RSS。RSS 是跟 Atom 具有同样功能的另一套 XML 词汇。

一个 Atom 文档，本质上是一个以发布资源的列表。你可以用 Atom 来表达图片库、音乐专辑或搜索结果列表。

如果你的应用基本符合 Atom 的模式（schema），但需要增加几个标签，这没问题。你可以在 Atom 提要里嵌入来自其他命名空间（namespace）的 XML 标签。你也可以定义自己的命名空间，并在 Atom 提要里嵌入来自该命名空间的标签。Atom 跟 XHTML 微格式一样，你可以在 Atom 提要里使用来自 Atom 以外的标签，而不会破坏 Atom 的有效性。

### OpenSearch
