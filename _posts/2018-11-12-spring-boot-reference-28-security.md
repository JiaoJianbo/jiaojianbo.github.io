---
title: Spring Boot Reference（节选）- 28. Security
date: 2018-11-12 18:30:00 +0800
layout: post
categories: Spring-Boot
tags: Spring-Boot
author: Bobby
---

* content
{:toc}

近日有些时间，就想把以前看过的文档和笔记整理一下。最早接触Spring-Boot的时候还是1.1.x的版本，当时它的官方文档大多也看了，整理了一些笔记。但是现在发现最新版已经到2.1.x了，新增的东西也挺多。鉴于2.x的版本目前使用的还不是很广泛，所以还是选择1.x中的最新版本——__1.5.17.RELEASE__。这里我只是核心章节的内容，重点部分做了一些记录，并没有逐字逐句去翻译，也是希望抓住重点部分。

官方文档地址：[https://docs.spring.io/spring-boot/docs/1.5.17.RELEASE/reference/htmlsingle/](https://docs.spring.io/spring-boot/docs/1.5.17.RELEASE/reference/htmlsingle/), 有些地方翻译或者记录的不够准确，可以对照原文看一下。



# Part IV. Spring Boot features

## 28. Security

如果Spring Security在Classpath下，那么web应用程序的所有HTTP endpoints默认就会由'Basic'的授权进行保护。通过添加`@EnableGlobalMethodSecurity`注解，你可以给方法级别添加安全控制。详情请参考[Spring Security Reference](https://docs.spring.io/spring-security/site/docs/4.2.9.RELEASE/reference/htmlsingle/#jc-method).

默认的`AuthenticationManager`有一个用户（用户名为'user'，密码是随机的，程序启动时可通过控制台INFO级别的log打印出来）。类似于：

```text
Using default security password: 78fa095d-3f4c-48b1-ad50-e24c31d5cf35
```

**注意:** 如果你调整过日志配置，那么请确保org.springframework.boot.autoconfigure.security的log级别是INFO，否则不能看到输出的密码。

当然你也可以通过配置`security.user.password`项，来修改默认密码。还有一些其他有用的属性都通过[SecurityProperties](https://github.com/spring-projects/spring-boot/blob/v1.5.17.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/SecurityProperties.java)具体化了（属性前缀为"security"）。

默认的security配置是在`SecurityAutoConfiguration`和它里面导入的类实现的（`SpringBootWebSecurityConfiguration`用于web security。而`AuthenticationManagerConfiguration`用于身份验证配置，而且该配置也与非web的应用程序有关）。你可以加入一个用`@EnableWebSecurity`注解的bean，来完全关闭这种默认的web应用程序安全配置（但是这个不能禁用身份验证管理器*Authentication Manager*的配置和Actuator的安全）。为了自定义通常使用外部属性或者`WebSecurityConfigurerAdapter`类型的bean（例如，添加基于form的login）。*注：Actuator请参考本文档的Part V. Spring Boot Actuator: Production-ready features*。  
**注意:** 如果添加了@EnableWebSecurity注解而且也禁用了Actuator 安全。那么除非你添加了一个自定义的WebSecurityConfigurerAdapter，要么整个应用程序就会使用一个默认的基于表单的登陆验证。

同样，你也可以通过添加一个`AuthenticationManager`类型的bean，来关闭authentication manager 的配置。或者通过在`@Configuration`类中的一个方法中织入`AuthenticationManagerBuilder`来配置一个全局的`AuthenticationManager`。有一些安全的应用程序在[Spring Boot samples](https://github.com/spring-projects/spring-boot/tree/v1.5.17.RELEASE/spring-boot-samples/)中来教你使用通用例子。

在一个web应用程序中有一些基本特性:

* 一个`AuthenticationManager` bean具有内置的存储和一个user（参见`SecurityProperties.User`的配置）。
* 忽略的path和一个公共的静态资源位置（/css/\*\*, /js/\*\*, /images/\*\*, /webjars/\*\* and \*\*/favicon.ico）。
* HTTP Basic security for all other endpoints.
* Security 事件发布到 Spring的`ApplicationEventPublisher` (成功的和没成功的授权以及拒绝访问)。
* Spring Security提供的公共的低级别的特性（HSTS, XSS, 缓存）默认都是打开的。
* 跨站请求伪造攻击（Cross Site Request Forgery，CSRF） 检查默认是关闭的。

以上所有的特性可以通过使用外部配置属性（`security.*`）进行关闭，打开或者修改。为了覆盖访问规则但不用修改其他的自动配置属性，可以添加一个使用`@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)`注解的`WebSecurityConfigurerAdapter`类型的bean，通过配置满足你的需求。  
**注意:** 默认的，一个`WebSecurityConfigurerAdapter`可以匹配任意path。但是如果你不想完全覆盖Spring Boot的自动配置的访问规则，那你这个Adapter就必须显式地配置那些你想覆盖的path。

### 28.1 OAuth2

如果spring-security-oauth2在你的项目的classspath下，那么你就可以使用一些自动配置来轻松搭建授权和资源服务器。详情请查看[Spring Security OAuth 2 Developer Guide](http://projects.spring.io/spring-security-oauth/docs/oauth2.html)。*（注：要明白什么是Authorization Server和 Resource Server，需要先对OAuth 2.0有个了解，官网: [https://oauth.net/2/](https://oauth.net/2/)）*

#### Authorization Server

要创建一个Authorization Server并且授予访问的token，需要使用`@EnableAuthorizationServer`注解，并且提供`security.oauth2.client.client-id`和`security.oauth2.client.client-secret`配置。那么在内存中会为你注册一个客户端。  
完了之后，您就可以使用客户端凭证来创建访问令牌，例如：

```
$ curl client:secret@localhost:8080/oauth/token -d grant_type=password -d username=user -d password=pwd
```

`/token` 终端（endpoint）的基本的授权凭证是`client-id` 和`c`lient-secret`。User的认证信息是基本的Spring Security user details（Spring Boot中默认的是“user”和一个随机密码）。

要关闭自动配置并自己配置Authorization Server的这些特性，只需添加一个`AuthorizationServerConfigurer`类型的`@Bean`。

#### Resource Server

要使用这个访问的token就需要一个Resource Server（它可以和Authorization Server是同一个）。创建一个Resource Server还是比较简单的，只需加入`@EnableResourceServer`注解，并且提供一些配置让server来解码access token。如果你的app也正好是个Authorization Server，那么它都已经知道如何解码了，那就不需要再做什么了。如果你的app是一个独立的服务，那么你需要一些其他的配置，以下选项之一:

* `security.oauth2.resource.user-info-uri` 来使用/me 资源(e.g. https://uaa.run.pivotal.io/userinfo on Pivotal Web Services (PWS)).  
* `security.oauth2.resource.token-info-uri` 来使用 token 解码 endpoint (e.g.https://uaa.run.pivotal.io/check_token on PWS).

如果你既定义了`user-info-uri`又定义了`token-info-uri`，那么你要设置一个标志，表明那个是首要使用（默认的是`prefer-token-info=true`）。

或者（既不是`user-info-uri`也不是`token-info-uri`）如果token是JWT的 *（注：JSON Web Token，官网：[https://jwt.io/](https://jwt.io/)）*，那么你可以配置`security.oauth2.resource.jwt.key-value`在本地解码（这里的key是验证密钥）。验证密钥值要么是对称密钥，要么是经过PEM编码的RSA公钥。如果您没有密钥，而且它是公共的，那么您可以使用配置`security.oauth2.resource.jwt.key-uri`，提供一个URI，以便去下载它（一个JSON对象包含一个名为“value” 的属性）。例如，在Pivotal Web Services（PWS）中，

```
$ curl https://uaa.run.pivotal.io/token_key
{"alg":"SHA256withRSA","value":"-----BEGIN PUBLIC KEY-----\nMIIBI...\n-----END PUBLIC KEY-----\n"}
```
另外，如果你的Authorization Server有个endpoint可以返回一组JSON Web Keys(JWKs)，那么你可以配置`security.oauth2.resource.jwk.key-set-uri`属性。例如，在PWS中，

```
$ curl https://uaa.run.pivotal.io/token_keys
{"keys":[{"kid":"key-1","alg":"RS256","value":"-----BEGIN PUBLIC KEY-----\nMIIBI...\n-----END PUBLIC
KEY-----\n"]}
```

**注意**：同时配置JWT和JWK属性会引起错误的，`security.oauth2.resource.jwt.key-uri`（或者`security.oauth2.resource.jwt.key-value`）和`security.oauth2.resource.jwk.key-set-uri`只能配置其一。

**警告**：如果你使用了`security.oauth2.resource.jwt.key-uri`或者`security.oauth2.resource.jwk.key-set-uri`配置。那么当你启动app的时候Authorization Server必须正在运行。如果不能得到这个key，那么会记录一条warning的log，并告诉你如何修复它。

OAuth2的资源被拥有`security.oauth2.resource.filter-order`的过滤器链保护，这个默认的过滤器链默认的在保护Actuator endpoints的过滤器之后（因此，除非你更改了顺序，要么Actuator endpoints还是使用HTTP Basic的认证方式）。
