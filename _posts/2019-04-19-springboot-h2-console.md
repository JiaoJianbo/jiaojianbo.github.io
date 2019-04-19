---
layout: post
title: "Spring Boot 中引入 Spring Security 后如何正常访问 H2 Console"
date: 2019-04-19 11:00:00 +0800
categories: SSH
tags: Spring-Boot H2 Security
author: Bobby
---

* content
{:toc}

Spring Boot 项目中引入 Spring Security 后如何正常访问 H2 Console?



### SpringBoot 中引入 Spring Security 后如何正常访问 H2 Console

我们知道如果在 Spring Boot 项目中使用 H2 DataBase，那么默认的就可以通过 `${context-path}/h2-console` 的路径访问 H2 console 了。在开发中还是挺方便的，不用再单独启动一个 H2 客户端。

但是当项目找中引入 Spring Security 之后，却发现每次访问 H2 console 都要求先登录。那么有没有办法不用登录就可以直接访问 H2 console 呢？当然有，只需要将这个路径加入到 Spring Security 忽略验证的路径集合中即可。

```java
http.authorizeRequests().antMatchers("/h2-console/**").permitAll();
```

至此，如果你的 Spring Security 没有启用 CSRF (XSRF) 防护，那基本上应该可以正常使用了。但是如果启用了 CSRF 防护，那么可能会遇到类似下面的结果（这是我自定义的错误页面，大家重点看错误信息）。

![H2 Console CSRF error](/assets/images/2019/04/h2-console-csrf.jpg)

同样的，还需要将 H2 console 的访问路径继续添加到 CSRF 防护忽略的路径集合里。即，

```java
http.csrf().ignoringAntMatchers("/h2-console/**");
```

到这里是否问题已经解决呢？我们再试。结果可以正常登录，但是却出下面的界面，┓(;´_｀)┏

![H2 Console CSRF login error](/assets/images/2019/04/h2-console-csrf-login-error.jpg)

打开浏览器控制台，发现下面的问题，

![H2 Console CSRF login browser error](/assets/images/2019/04/h2-console-csrf-login-browser-error.jpg)

似曾相识，参见上一篇文章 [Spring Security 3 升级到 4 的过程中遇到的一些问题 #默认的请求响应安全头变化](/2019/03/25/spring-security-upgrade-note/)

因此，还需要修改 Spring Securiy 对 iframe/frame 的安全策略，

```java
http.headers().frameOptions().sameOrigin();
```

到此，H2 console 总算是能正常访问了。

![H2 Console Login Success](/assets/images/2019/04/h2-console-login-success.jpg)


### 参考资料：

[SpringBoot 使用 Spring Security 开启了 CSRF 防跨站攻击防护后 POST 方法无效](https://liuyanzhao.com/7610.html)  
[解决在项目里引入Spring Security后iframe或者frame所引用的页无法显示的问题](https://blog.csdn.net/lvchengbo/article/details/54911056)  
