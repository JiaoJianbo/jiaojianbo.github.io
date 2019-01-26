---
layout: post
title: "在集成 Spring-Security 的环境中，使用 Postman 测试 RESTful 接口如何发送 CRSF Token"
date: 2019-01-26 11:00:00 +0800
categories: Security
tags: Spring Security RESTful
author: Bobby
---

* content
{:toc}

最近在学习 RESTful 接口的相关理论，自己也在动手搭建一个微服务的框架，开发了些 RESTful 的接口。使用 Postman 手动测试这些接口时，GET 方式的请求输入认证信息后可以正常调用，而 POST, PUT 和 DELETE 方式的请求在调用时，会报 403 Forbidden 的错误，错误信息 "Invalid CSRF Token 'null' was found on the request parameter '_csrf' or header 'X-XSRF-TOKEN'."。



首先，CSRF（[Cross Site Request Forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery)，也叫作 XSRF的）是什么这里就不赘述了，自己可以查找相关资料。我使用的 Spring-Boot-1.5.9.RELEASE 搭建的开发环境，安全方面集成的是 Spring-Security，暂时采用的是 HTTP Basic 认证方式。

好了，让我再回到上面的问题。CSRF Token 是由服务端先给客户端的，客户端再次请求时将这个 Token 再发送回去。只有服务端验证通过，请求才能被顺利执行。而且这个 Token 每次是会变的。那么我们使用 Postman 如何在每次发送请求时（GET 请求默认不需要 CSRF Token）得到这个动态的 Token？又如何将这个 Token 再返回给服务端呢？

在网上找了些资料，自己通过实验，终于解决了这个问题。

##### 1.在Postman 中创建 Environment

  在 Postman 中先创建一个环境（Environment），以后的请求都基于这个环境。
  <img src="/assets/images/2019/01/postman-create-env.jpg" alt="create an env" />

##### 2.在该环境（Environment）中创建变量（如csrf-token），表示 CSRF Token 的值

  <img src="/assets/images/2019/01/postman-add-variable.jpg" alt="add variable" />

##### 3.创建一个 GET 请求，用来从服务端获取 CSRF Token

  基于刚才的环境（Environment）创建一个 GET 请求，用来从服务端获取 token 值。注意这个请求的 Authorization tab 里面要输入用户的认证信息，在 Test tab 里面输入以下代码（我的 Postman 是独立安装版，Version 6.7.1，低版本的可能代码写法跟以下不同），用来从服务端的 response cookie 中取出 token 值，然后再赋给上面创建的变量。

  ```
  pm.environment.set("csrf-token", decodeURIComponent(pm.cookies.get("XSRF-TOKEN")));
  ```

  <img src="/assets/images/2019/01/postman-retrieve-token.jpg" alt="retrieve token value" />

##### 4.新建待测试的 POST，PUT，DELETE 请求，在 header 中加入 Token 值，返回给服务端

  基于刚才的环境（Environment）创建待测试的 POST，PUT，DELETE 请求，在请求头 header 中加入一个`X-XSRF-TOKEN`的键，它的值就是token 变量的值`{ {xsrf-token} }`（两个大括号之间没有空格）。  
  <img src="/assets/images/2019/01/postman-set-token.jpg" alt="set header" />

##### 5.再测试 POST，PUT，DELETE 请求时，服务端正常响应

  <img src="/assets/images/2019/01/postman-success.jpg" alt="response ok" />

#### 以上过程中，有几点需要注意：

1. 上面第 3 步中，如何保证服务端的 response cookie 中含有 token 值？
在服务端自定义的`WebSecurityConfigurer`中，启用 CSRF，并且 token repository 使用`CookieCsrfTokenRepository`。如果需要前端页面使用 JavaScript 从 cookie 中取出 token 值，还需要设置 token repository `withHttpOnlyFalse()`。至于 cookie 中 token value 的 key 为什么是 `XSRF-TOKEN`？这是 Spring Security 中定义的默认值。
要检查 token 值获取到了没，可以查看 response cookie，也可以用鼠标悬停在变量上查看当前值。
    <img src="/assets/images/2019/01/postman-check-token1.jpg" alt="check token value" />

    <img src="/assets/images/2019/01/postman-check-token2.jpg" alt="check token value" />

2. 上面第 4 步中，发送 token 时，header 中 key 为什么是`X-XSRF-TOKEN`？这也是 Spring Security 中定义的默认值。另外 token 值也可以通过一个名为 `_csrf` 的隐藏域发送。这些其实通过最上面的报错信息中也可以看出。

3. 如果执行 POST，PUT，DELETE 请求时，报 token 值无效或者为空的错误，那么请先执行一下 GET 请求再试。

<br/>
[参考]  
[How do I send spring csrf token from Postman rest client?](https://stackoverflow.com/questions/27182701/how-do-i-send-spring-csrf-token-from-postman-rest-client)  
[我的 WebSecurityConfigurer](https://github.com/JiaoJianbo/learning/blob/master/spring-mybatis/src/main/java/com/bravo/demo/ssm/security/WebSecurityConfig.java)  
[Spring-Security 中 CSRF](https://docs.spring.io/spring-security/site/docs/4.2.3.RELEASE/reference/htmlsingle/#csrf)  
[CsrfFilter.java](https://github.com/spring-projects/spring-security/blob/4.2.x/web/src/main/java/org/springframework/security/web/csrf/CsrfFilter.java)  
[CookieCsrfTokenRepository.java](https://github.com/spring-projects/spring-security/blob/4.2.x/web/src/main/java/org/springframework/security/web/csrf/CookieCsrfTokenRepository.java)
