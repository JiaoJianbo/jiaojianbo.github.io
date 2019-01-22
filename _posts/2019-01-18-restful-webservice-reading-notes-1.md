---
layout: post
title: "《RESTful Web Services 中文版》读书笔记（一）"
date: 2019-01-18 21:00:00 +0800
categories: RESTful
tags: RESTful REST
author: Bobby
---

* content
{:toc}

最近还是继续在学习 REST、 微服务等相关知识，想从理论方面对这些知识有个更深入、更全面的了解。《RESTful Web Services 中文版》是目前正在看的。对其中有些内容作了摘录，便于以后查阅和复习。
其中，引用的书中的内容，版权归作者所有。如有侵权，请联系删除。



## 前言

**本书主要内容**：

* 强调 Web 基础技术—— HTTP 应用协议、URI 命名标准，以及 XML 语言的强大能力。
* 介绍面向资源的架构（Resource-Oriented Architecture, ROA）,即一组用于设计 REST 式 Web 服务的原则。
* 揭示 REST 式设计为何比 RPC 式设计更简单、更具多功能性和可伸缩性。
* 给出 REST 式 Web 服务的真实案例，比如 Amazon S3 (Simple Storage Service) 和 Atom 发布协议。
* 讨论了各种流行的编程语言的 Web 服务客户端。
* 展示了如何用三种流行的框架—— Ruby on Rails, Restlet (Java) 和 Django (Python) 实现 REST 式服务。
* 聚焦实际问题，比如如何设计与实现 REST 式 Web 服务及客户端。

## 第一章 Programmable Web 及其分类

Programmable Web 跟 Human Web 相对。Human Web 指的传统的面向人类用户使用的 Web，而 Programmable Web 指的是面向计算机程序使用的 Web，如 Web Service。

Programmable Web 是基于 HTTP 和 XML 技术的。当然也有使用 HTML, JSON, 纯文本和二进制文档的。但是使用较多的还是 XML。

**HTTP 请求的各个主要部分**：

* HTTP 方法 (HTTP method): 也被称作“HTTP 动词 (HTTP verb)”或“HTTP 动作 (HTTP action)”。如 "GET", "POST"。
* 路径 (Path): URI (URL) 里主机名（hostname）后面的部分。
* 请求报头 (request headers): 它们是一组关键字-值对 (key-value pairs)，起元数据的作用。如：Host, User-Agent, Accept, 等等。应用程序也可以自定义自己的报头。
* 实体主体 (entity-body)，也称作文档 (document) 或表示 (representation)。放在信封里的文档。一般来说 GET 请求都没有实体主体，完成请求的所有信息都在path 和 header 里面。

**HTTP 响应可分为三部分**：

* HTTP 响应代码 (HTTP response code): 通知客户端“HTTP 请求是成功还是失败”的数字代码，并告诉客户端应如何对待信封及信封里的内容。
* 响应报头 (response headers)：跟请求报头一样。
* 实体主体 (entity-body) 或表示 (representation)。

响应报头 Content-Type 给出实体主体的**媒体类型** (media type)。关于媒体类型有一个标准 [http://www.iana.org/assignments/media-types/](http://www.iana.org/assignments/media-types/)。也称作“MIME类型”，“内容类型（content type）”或“数据类型（data type）”。

SOAP 服务一般是把方法信息放在 entity body 和 HTTP header 里。

常见的 Web 服务架构主要有三种：REST 式架构、RPC 式架构和 REST-RPC混合架构。

REST 式架构意味着，方法信息（method information）都在 HTTP 方法（HTTP method）里；面向资源的架构（ROA）意味着，作用域信息（scoping information）都在URI里。

RPC 式 Web 服务（RPC-style Web Service）通常从客户端收到一个充满数据的信封，然后发回一个同样充满数据的信封。RPC 式架构意味着，方法信息和作用域信息都在信封或报头里。HTTP 是一种常见的信封格式，另一种常见的信封格式是 SOAP（把 SOAP 信封放在 HTTP 信封里，在 HTTP 上传送 SOAP文档）。XML-RPC 是最典型的 RPC 架构例子。

WSDL (Web Service Description Language, Web 服务描述语言) 是一套描述 SOAP Web 服务的 XML 词汇。

WADL (Web Application Description Language, Web 应用描述语言) 是一套描述 REST 式 Web 服务的 XML 词汇。

HTTP+POX: “HTTP 加普通老式 XML（Plain Old XML, POX）”

## 第二章 编写 Web 服务客户端

所有的 Web 服务请求都涉及到这三个步骤：

1. 准备好要放入 HTTP 请求的数据：HTTP 方法、URI、HTTP 报头，以及（对于采用 PUT 或 POST 方法的请求而言）需要放进请求实体主体里的文档。
2. 把这些数据格式化为一个 HTTP 请求，然后把该 HTTP 请求发给正确的 HTTP 服务器。
3. 把服务器返回的数据（响应代码，报头及实体主体）解析为一个数据结构，供程序使用。

服务器应该是理想主义的；客户端必须是使用主义的——这是 Postel 法则“严以律己，宽以待人（Be conservative in what you do; be liberal in which you accept from others）”的一个变形。

要构建一个完全通用的 Web 客户端，需要具备以下特征的 HTTP 库：

* 它必须支持 HTTPS 和 SSL 证书验证。
* 它必须支持至少五个 HTTP 方法：GET, HEAD, POST, PUT, DELETE。有些库仅仅支持 GET 和 POST。
* 它必须允许程序员定制 PUT 和 POST 请求的实体主体里的数据。
* 它必须允许程序员定制请求的 HTTP 报头。
* 它必须允许程序员获取 HTTP 响应代码和响应报头，还有响应的实体主体。
* 它必须支持通过 HTTP 代理（Proxy）进行 HTTP 通信。

HTTP 库的一些可选特性：

* HTTP 库应该请求压缩格式的数据，并在收到数据时自动解压。相关的 HTTP 请求报头是 Accept-Encoding，相关的 HTTP 响应报头是 Encoding。
* HTTP 库应该自动把返回的响应缓存起来。相关的 HTTP 请求报头是 ETag 和 If-Modified-Since，相关的 HTTP 响应报头是 ETag 和 Last-Modified。
* HTTP 库应该透明地支持常见的 HTTP 认证形式，如：基本认证（Basic）、摘要认证（Digest）及 WSSE 认证。如果能够支持（或有插件支持）特定机构专有的认证方法（比如 Amazon 的）也是很有用的。相关的 HTTP 请求报头是 Authorization，相关的 HTTP 响应报头（主张认证的一方）是 WWW-Authenticate。
* HTTP 库应该能透明地做 HTTP 重定向。同时避免无限重定向和循环重定向。
* HTTP 库应该能解析并创建 HTTP cookie 字符串，而不是只能由程序员手工设置 Cookie 报头。对 REST 来说这点不是很重要。

几个 Java HTTP 客户端库的功能对照表：

|     | HttpURLConnection | HttpClient | Restlet |
| --- | :---: | :---: | :---: |
| HTTPS | 支持 | （同左） | （同左） |
| HTTP 动词 | 所有 | （同左） | （同左） |
| 自定义数据 | 支持 | （同左） | （同左） |
| 自定义报头 | 支持 | （同左） | （同左） |
| 代理 | 支持 | （同左） | （同左） |
| 压缩 | 不支持 | 不支持 | 支持 |
| 缓存 | 支持 | 不支持 | 支持 |
| 认证方式 | 基本、摘要, NTLM | （同左） | 基本，Amazon |
| Cookies | 支持 | （同左） | （同左） |
| 重定向 | 支持 | （同左） | （同左） |

消息认证代码（Message Authentication Code, MAC）

## 第四章 面向资源的架构

ROA 的特性：可寻址性（addressability）、无状态性（statelessness）、连通性（connectedness）和统一接口（uniform interface）。

REST 并不是一种架构，而是一组设计原则。作为一组设计原则，REST 是非常通用的。它并不限定于 Web。REST 不依赖于 HTTP 机制或 URI 结构。

资源必须至少有一个 URI。 URI 既是资源的名称，也是资源的地址。URI 应该是自描述性的。

一个资源有多个 URI 的好处是，客户端对该资源的引用变得容易。坏处是，同一资源具有多个 URI 将产生“稀释效应”：有的客户端用这个 URI，有的客户端用那个 URI，而且无法自动验证这些 URI 指向同一个资源。
一种方案是：假如一个资源有多个 URI，那么选择其中一个作为该资源的“规范” URI。当客户端请求该规范 URI 时，服务器返回响应代码 200（“OK”），并附上正确数据；当客户端请求其他 URI 时，服务器返回响应代码 303 （“See Also”），并给出该资源的规范 URI。虽然客户端无法仅凭外观得知这两个 URI 是否同一个资源，但它可以分别向这两个 URI 发出一个 HEAD 请求，看是否其中一个 URI 重定向到另一个，或者二者都重定向到第三个 URI。
还有一个方案，就是对这些 URI 一视同仁，做同样的响应；但是对于非规范 URI 的请求，将在响应报头 Content-Location 里给出“规范” URI。

资源是通过 URI 暴露出来的，所以一个可寻址的应用会为它可能提供的每一则信息都发布一个 URI。

**无状态性**（statelessness）意味着每个 HTTP 请求都是完全孤立的。当客户端发出一个 HTTP 请求时，请求里包含服务器实现（fulfill）该请求所需的全部信息，服务器不依赖任何之前请求提供的信息。假如本次请求需要之前某个请求提供的信息，那么客户端应当把那个信息也包括在本次请求里。
无状态性还引入了一些新特性。在负载均衡（load-balanced）服务器上分配无状态的应用将容易许多。因为各个请求之间没有相互依赖，所以它们可被放在不同的服务器上处理，而不需要服务器之间做任何协作。

**无状态性**（statelessness）意味着：有一种状态，是服务器不应该保存的。实际上状态分两种：**应用状态**（Application State）与**资源状态**（Resource State）。前者应该保存在客户端，后者应该保存在服务端。我们各自的客户端分别保存着各自的应用状态。资源状态对于每个客户端都是相同的，它应该保存在服务端。无状态性要免除**会话复制**（session replication）和**会话亲缘性**（session affinity, 有的地方也叫**会话粘滞**）。只有当你的资源状态需要被划分到不同机器上时，你才需要考虑数据复制（data replication）的问题。

资源是**表示**（representations）的来源，表示只是关于资源当前状态的一些数据。即使在一个对象的诸多表示中，已经有一个表示包含实际数据了，它也可以还有其他包含元数据的表示。
表示除了可以从服务器传给客户端，也可以从客户端传给服务器（比如创建和更新）。

一个资源有多种表示的时候，服务器应该把哪个表示返回给客户端呢？有种方案叫做**内容协商**（Content Negotiation），客户端在请求此 URI 时，会在 HTTP 请求里提供一个专门的报头（header），用于告知服务器客户端需要的是哪种表示。

**超媒体即应用状态引擎**（Hypermedia As The Engine Of Application State, HATEOAS）。这条公理的意思是：HTTP 会话的当前状态不是作为资源状态（resource state）保存在服务器上的，而是被客户端作为应用状态（application state）来跟踪的。客户端应用状态在服务器提供的“超媒体（hypermedia）”（即超文本表示里的链接和表单）的指引下发生变迁。
服务器通过超媒体告诉客户端当前状态有哪些后续状态可以进入。链接起的就是这种推进状态的作用，它指引你如何从当前状态进入下一个可能的状态。我们称这种“具有链接”的特性为**连通性**（connectedness）。假如你通过跟随链接和提交表单就能改变一个 Web 服务的状态，那么这个 Web 服务就是连通的（connected）。

获取一个只包含元数据（metadata）的表示，用 HTTP HEAD 方法。客户端可以用 HEAD 请求来检查一个资源是否存在，或者查看关于该资源的其他信息，而不必获取整个表示（representation）。HEAD 请求就是比 GET 请求少返回实体主体而已。

查看一个资源支持那些 HTTP 方法，用 HTTP OPTIONS 方法。OPTIONS 请求的响应里含有 HTTP Allow 报头，它给出该资源所支持 HTTP 方法的集合。下面是一个 Allow 报头的例子：

```text
Allow: GET, HEAD
```

另外还有 TRACE 和 CONNECT 这两个 HTTP 方法。TRACE 用于调试代理（proxy），CONNECT 用于通过 HTTP 代理来转发其他一些协议。

在一个 REST 式设计中，POST 通常被用于创建从属资源（subordinate resources）——即从属于“父”资源（另一个资源）而存在的资源。
POST 和 PUT 的区别就在于：假如是客户端负责决定新资源采用使用 URI，那就用 PUT；假如是服务器负责新资源采用什么 URI，那就用POST。*（注：因为我们创建对象时的 ID，一般都是在服务端生成的，所以大多数时候 POST 用来创建资源，PUT 用来更新资源）*
用 POST 方法创建资源时，客户端不必知道新资源的确切 URI。在大多数情况下，客户端只要知道“父”资源（parent resource）或“工厂”资源（factory resource）的 URI 就行了。服务器对这种 POST 请求的响应，通常具有 HTTP 状态代码 201 （“Created”），而且会在其中的 Location 报头里面包含新建从属资源的 URI。
给一个资源发 POST 请求，未必总是创建一个全新的从属资源。有时，当向一个资源 POST 数据时，它把你发来的信息附加到它自身的状态上，而不是用该信息新建一个资源。如一个日志服务，进行追加日志。

重载的 POST (overloaded POST)。*（注：在 HTML5 之前，form 表单只支持 GET 和 POST 两种方法。在 Spring-Boot 环境中可以使用一个名为 `_method` 隐藏域和 `HiddenHttpMethodFilter`）*

**安全性**（Safety）: GET, HEAD 请求应该都是安全的（safe）。不应该改变服务器状态。

**幂等性**（Idempotence）：GET, HEAD, DELETE, PUT 请求应该都是幂等的（idempotent）。*（即一个请求执行一次和执行多次的结果是一样的）*

安全性和幂等性可以令客户端在非可靠的网络上实现可靠的 HTTP 请求。POST 既不是安全也不是幂等的。
