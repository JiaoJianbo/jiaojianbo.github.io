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

## 预定义的控制流

## 超媒体技术

超媒体技术分两种：链接和表单。链接反应了当前资源与某个目标资源之间的联系，并用 URI 来标识目标资源。表单分为两种。最简单的称为应用表单（application form），这种表单告诉客户端如何处理应用状态（application state）。应用表单是一种处理那些“名称符合一定模式”的资源的方式，可以把它看成一种有不止一个目标的链接。还有一种表单称之为资源表单（resource form），这种表单告诉客户端如何构造一个表示（representation）来更新资源状态。当然，GET 和 DELETE 请求是无需发送表示的。

链接与应用表单实现了连通性，以及 Roy Fielding 博士论文里提到的“将超媒体作为应用状态的引擎”。

### URI 模板

例如：https://s3.amazonaws.com/{name-of-bucket}/{name-of-object}。URI 模板为我们提供了一种准确无误地给 URI “填空”的方式。

### XHTML4

#### XHTML4 里的链接

有许多 HTML 标签可以用于设置超链接（比如 img），不过主要还是 link 和 a 这两个标签。跟 link 和 a 标签有关的 3 个重要属性是：href, rel 和 rev。

#### XHTML4 里的表单

form 标签。

### XHTML5

### WADL

Web 应用描述语言（Web Application Description Language, WADL）是一种用于“表达HTTP资源的行为”的 XML 词汇（参见 WADL Java 客户端的开发网站 [https://wadl.dev.java.net/](https://wadl.dev.java.net/)）。WADL 是依照 Web 服务描述语言（Web Service Description Language, WSDL）的方式来命名的。

WADL 是一套标准的词汇，它的作用跟 APP 服务文档差不多，不过它能用于任何资源。你可以提供一个描述了你的服务所暴露的每个资源的 WADL 文档。WADL 简化了 Web 服务客户端的编写。

# 第十章 面向资源的架构 vs 大 Web 服务

## 大 Web 服务试图解决哪些问题

大 Web 服务试图解决的主要问题，正是这种面向过程的、需要中介的分布式服务的设计。由于种种原因，这种应用往往在商业与政府应用中比较流行，而在技术和学术领域就不行了。

## SOAP

诸多 WS-* 规范都是在 SOAP 的基础上构建的。经过 SOAP 信封包装后的一个 XML 文档示例：

```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope">
    <soap:Body>
        <hello-world xmlns="http://example.com"/>
    </soap:Body>
</soap:Envelope>
```

SOAP 信封必须跟它所包含的文档具有相同的字符编码。关于 SOAP 主体（Body）部分的最后一点说明：你可以用基于 XML Schema Data Types 的数据类型信息来标注参数。例如：`<ie xsi:type="xsd:string">latin1</ie>`。

SOAP 为报头实体定义了两个属性：actor 和 mustUnderstand。假如你事先知道你的消息抵达目标之前将经过若干个媒介，你可以用 actor 来标识（通过 URI）一个报头的目标。mustUnderstand 属性用于对这些媒介（或最终目标）做一些要求。假如某个媒介无法理解发给它的报头，而且该报头的 mustUnderstand 为真的话，那么它必须拒绝此消息——即使它觉得自己能处理该条消息也不行。以一个跟两阶段提交（two-phase commit）操作有关的报头为例，假如目标不能理解两阶段提交，那么该操作就不应继续下去。

SOAP 确实具有一样它所声称的优点，即传输无关性（transport independence）。报头放在消息里，这意味着它们独立于用来传输消息的协议。SOAP 信封不是只能放在 HTTP 信封里发送，你还可以通过 Email、即时消息、原始的 TCP 或任何其它协议来传输 SOAP 消息。公开采用 SMTP 传输的情况不多，有些公司只是在内部采用 JMS 传输，大部分 SOAP 消息还是通过 HTTP 传输的。

## WSDL

## UUID

## 安全性

## 可靠消息传递

## 事务

两阶段提交。补偿事务。

## BPEL, ESB 和 SOA

业务流程执行语言（Business Process Execution Language, BPEL）是一种用以描述牵扯多方的业务流程的 XML 语法。这些流程可以通过软件和 Web 服务进行自动编配（orchestration）。

企业服务总线（Enterprise Service Bus, ESB）有各种各样的定义，不过一般都包括发现、负载均衡、路由、桥接、转换及 Web 服务请求管理等方面。这常常引起运营与开发的分离，使他们各自都得以简化并易于管理。

BPEL 和 ESB 的不足在于，它们都容易增加与公共第三方中间件的耦合度与依赖程度。它们的一个优势，在于你对中间件可以有多种选择。

面向服务的架构（Service-Oriented Architecture, SOA）也许是这几个词中定义最不明确的一个。而且没有办法能够检验一个给定实现是否算作 SOA。SOA 是独立于服务的技术架构的。在面向资源的环境、RPC 服务的环境及异构环境下均能实现 SOA。

# 第十一章 将 Ajax 应用作为 REST 客户端

Ajax 应用的不利方面在于，每个应用的状态都是同一个 URI —— 即最终用户访问的第一个 URI。这破坏了可寻址性（addressability）和无状态性（statelessness）。虽然幕后的 Web 服务也许是可寻址的和无状态的，但最终用户无法把一个特定状态加入收藏夹，而且浏览器的“后退”按钮也失去了原有的作用。

# 第十二章 REST 式服务框架

## Ruby on Rails

## Restlet

<img src="/assets/images/2019/02/restlet-class-hierarchy.jpg" alt="Restlet Class Hierarchy" />

可参考：[Restlet 开发实例 https://www.cnblogs.com/Scott007/p/3300128.html](https://www.cnblogs.com/Scott007/p/3300128.html)

## Django

# 延伸阅读：

[各种 JAX-RS 实现框架的比较](https://www.ibm.com/developerworks/web/library/wa-apachewink3/)

[比较各JAX-RS实现：CXF,Jersey,RESTEasy,Restlet](http://www.cnblogs.com/felixjia/p/3608718.html)
