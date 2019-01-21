---
layout: post
title: "《RESTful Web Services中文版》读书笔记（一）"
date: 2019-01-18 21:00:00 +0800
categories: RESTful
tags: RESTful, REST
author: Bobby
---

* content
{:toc}

最近还是继续在学习 REST、 微服务等相关知识，想从理论方面对这些知识有个更深入、更全面的了解。《RESTful Web Services中文版》是目前正在看的。对其中有些内容作了摘录，便于以后查阅和复习。
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

无状态性（statelessness）意味着：有一种状态，是服务器不应该保存的。实际上状态分两种：应用状态（Application State）与资源状态（Resource State）。前者应该保存在客户端，后者应该保存在服务端。

