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

[OpenSearch](http://www.opensearch.org/) 是一种常被嵌入 Atom 文档的 XML 词汇。它用于表达搜索结果列表。其想法是：把整个搜索结果列表作为一个 Atom 提要，并把搜索结果列表里的各条记录作为 Atom 条目。

### SVG

媒体类型：image/svg+xml。SVG （Scalable Vector Graphics, 可缩放向量图形）是一套令程序可以理解并处理图形的 XML 词汇。SVG 图像可以任意缩放，而不损失任何细节。SVG 图像可以被编辑和调整，而且可以把部分 SVG 图像提取出来，合并到其他图像中去。简言之，SVG 令我们可以像处理其他类型的文档一样来处理图像文件。

### 表单编码的键-值对

媒体类型：application/x-www-form-urlencoded。它主要在客户端向服务器提交表示时采用。HTML 表单将默认采用这种格式，而且 Ajax 应用也容易构造这种格式。不过，这种格式也可以在服务器给客户端返回表示时采用。

### JSON

媒体类型：application/json。JSON（JavaScript Object Notation, JavaScript 对象表示法）是一种用于表达一般的数据结构的序列化格式。它要比相应的 XML 文档更轻量级、更具可读性。

### RDF 和 RDFa

资源描述框架（Resource Description Framework, RDF）（http://www.w3.org/RDF/）是一种表达关于资源的知识的方式。在 RDF 里，URI 不仅包括以 http: 开头的URI，还可以是像 isbn:（用于标识图书）或 urn:（可用于标识任何事物）这种抽象的 URI模式。  
[RDFa](http://rdfa.info/about)，它是一种像微格式一样在 XHTML 里嵌入 RDF 的标准。

### 编码问题

在美国，一般采用 UTF-8, US-ASCII 或 Windows-1252 等编码。在西欧，一般采用 ISO 8859-1 编码。在 Web 上，HTML 的默认编码为 ISO 8859-1，这种编码跟 Windows-1252 差不多，但略有不同。日文文档一般采用 EUC-JP, Shift_JIS 或 UTF-8 编码。大部分编码都是互不兼容的。人们提出了 Unicode，这是一种表达所有人类文字体系的方法。Unicode 不是一种字符编码，但是 UTF-8 和 UTF-16 都可以用于 Unicode。在处理多语言数据时，最好的做法就是**为所有数据采用同一种编码**。

RFC 4627 指出，JSON 文档里的字符必须是 Unicode 字符，并采用任一种 UTF-* 编码。在实践上，这意味着要么采用 UTF-8，要么采用具有字节顺序标记（Byte-Order Mark, BOM）的 UTF-16。因为有这一约束，所以客户端通过 JSON 文档前面四个字符就能断定其字符编码了（详见 RFC-4267），并不需要 JSON 文档明确指出文档编码。
