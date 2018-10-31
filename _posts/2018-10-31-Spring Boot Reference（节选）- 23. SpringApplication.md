---
title: Spring Boot Reference（节选）- 23. SpringApplication
layout: post
categories: Spring-Boot
tags: Spring-Boot
---
近日有些时间，就想把以前看过的文档和笔记整理一下。最早接触Spring-Boot的时候还是1.1.x的版本，当时它的官方文档大多也看了，整理了一些笔记。但是现在发现最新版已经到2.1.x了，新增的东西也挺多。鉴于2.x的版本目前使用的还不是很广泛，所以还是选择1.x中的最新版本——__1.5.17.RELEASE__。这里我只是核心章节的内容，重点部分做了一些记录，并没有逐字逐句去翻译，也是希望抓住重点部分。

官方文档地址：[https://docs.spring.io/spring-boot/docs/1.5.17.RELEASE/reference/htmlsingle/](https://docs.spring.io/spring-boot/docs/1.5.17.RELEASE/reference/htmlsingle/), 有些地方翻译或者记录的不够准确，可以对照原文看一下。

# Part IV. Spring Boot features

## 23. SpringApplication

### 23.1. Startup failure

SpringApplication提供了一个方便的方法，从启动main()方法引导启动一个Spring应用程序。
针对启动失败，Spring boot提供了许多FailureAnalyzer的实现类，提供具体的错误信息和解决办法。当然你也可以很容易的添加自己的实现类。另外也可以开启debug模式，或者debug日志。
开启dbug属性:`java -jar myproject-0.0.1-SNAPSHOT.jar --debug`

### 23.2. 自定义Banner

在classpath下面添加一个banner.txt，或者通过banner.location指定banner文件的位置。通过banner.charset设置banner文件的编码，默认是UTF-8。另外还可以在classpath中加入banner.gif, banner.jpg 或 banner.png图片文件，或者通过banner.image.location指定。banner.txt中可以加入一些变量。具体参见spring-boot-reference文档。
另外开可以通过编程的方式设置banner，通过调用方法SpringApplication.setBanner(…)。实现接口`org.springframework.boot.Banner`中的`printBanner()`方法。
通过设置`spring.main.banner-mode`属性决定在控制台（属性值设置为console），日志（属性值为log）中输出banner，或者直接关掉（off）。比如在YAML文件中的设置。（__注意属性值要加双引号__）
<img src="/assets/images/2018/10/banner_off.png" alt="banner_off" witdth="" height=""/>