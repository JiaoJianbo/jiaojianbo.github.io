---
title: Spring Boot Reference（节选）- 23. SpringApplication
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

## 23. SpringApplication

### 23.1. Startup failure

SpringApplication提供了一个方便的方法，从启动main()方法引导启动一个Spring应用程序。
针对启动失败，Spring boot提供了许多FailureAnalyzer的实现类，提供具体的错误信息和解决办法。当然你也可以很容易的添加自己的实现类。另外也可以开启debug模式，或者debug日志。
开启dbug属性:`java -jar myproject-0.0.1-SNAPSHOT.jar --debug`

### 23.2. 自定义Banner

在classpath下面添加一个`banner.txt`，或者通过`banner.location`指定banner文件的位置。通过`banner.charset`设置banner文件的编码，默认是UTF-8。另外还可以在classpath中加入`banner.gif`, `banner.jpg` 或 `banner.png`图片文件，或者通过`banner.image.location`指定。`banner.txt`中可以加入一些变量，例如`${application.title}`。具体参见spring-boot-reference文档。
另外开可以通过编程的方式设置banner，通过调用方法`SpringApplication.setBanner(…)`。实现接口`org.springframework.boot.Banner`中的`printBanner()`方法。
通过设置`spring.main.banner-mode`属性决定在控制台（属性值设置为console），日志（属性值为log）中输出banner，或者直接关掉（off）。比如在YAML文件中的设置。（__注意属性值要加双引号__）

<img src="/assets/images/2018/10/banner_off.png" alt="banner_off" witdth="" height=""/>

### 23.3. 自定义SpringApplication

可以参考SpringApplication javadoc对默认设置做一些修改。如禁用banner，
```
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
}
```
当然也可以配置在application.properties或YAML文件中。

### 23.4. Fluent builder API

例如这样的构造器API，
```
new SpringApplicationBuilder()
    .sources(Parent.class)
    .child(Application.class)
    .bannerMode(Banner.Mode.OFF)
    .run(args);
```
具体详见`SpringApplicationBuilder` [javadoc](https://docs.spring.io/spring-boot/docs/1.5.17.RELEASE/api/org/springframework/boot/builder/SpringApplicationBuilder.html).

### 23.5. Application events and listeners

有些event是在`ApplicationContext`创建前就触发的，因此这些event对应的listener就不能使用`@Bean`去注册了。需要使用方法`SpringApplication.addListeners(…)`  或者`SpringApplicationBuilder.listeners(…)`。

如果你想这些listener被自动注册，而不管Application是如何创建的。那么你可以在你的项目中添加一个`META-INF/spring.factories`文件，然后在其中引用这些listener。Key值为`org.springframework.context.ApplicationListener`。如，
```
org.springframework.context.ApplicationListener=com.example.project.MyListener1,\
com.example.project.MyListener2,\
com.example.project.MyListener3
```
在应用程序运行时，Application event是按如下顺序发送的：
1.	首先在运行开始时发送`ApplicationStartingEvent`，早于除过the registration of listeners and initializers以外的其他任何处理程序。
2.	当一个`Environment`在一个已知的Context中被使用到时，发送一个`ApplicationEnvironmentPreparedEvent`，且早于Context的创建。
3.	在所有定义的bean被load完以后，refresh开始之前，发送一个`ApplicationPreparedEvent`。
4.	在refresh之后，所有相关的callback已经被处理，表明application已经ready to service requests，此时发送一个`ApplicationReadyEvent`。
5.	当在启动过程中用异常发生时，会发送一个`ApplicationFailedEvent`。

虽然你不会直接使用这些event，但是知道它们的存在会带来一些便利。其实Spring Boot在内部依靠event来处理各种任务。

### 23.6. Web environment

`SpringApplication`会争取创建正确的`ApplicationContext`。根据你是否创建的是web application，默认创建`AnnotationConfigApplicationContext`或者`AnnotationConfigEmbeddedWebApplicationContext`。当然你也可是通过方法`setWebEnvironment(boolean webEnvironment)`修改这种默认实现。也可以通过方法`setApplicationContextClass(…)`完全控制`ApplicationContext`的类型。

**Tip:**
在Junit test中使用`SpringApplication`时，经常会用到`setWebEnvironment(false)`方法.

### 23.7. Accessing application arguments

如果你想访问`SpringApplication.run(…)`方法传入的参数，你可以给你的bean类注入一个`org.springframework.boot.ApplicationArguments`类型的bean。`ApplicationArguments`接口提供访问`Stings[]`参数还有`option` 和`non-option` 参数的方法。

```
import org.springframework.boot.*
import org.springframework.beans.factory.annotation.*
import org.springframework.stereotype.*
@Component
public class MyBean {
    @Autowired
    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        // if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
    }
}
```
**Tip:** Spring Boot也会在`Environment`中注入一个`CommandLinePropertySource`。这就允许你通过`@Value`注解注入单个application arguments。

### 23.8. Using the ApplicationRunner or CommandLineRunner

如果您需要在`SpringApplication`启动后运行一些特定的代码，你可以实现`ApplicationRunner`或者`CommandLineRunner`接口。这两个接口都只提供了一个`run(...)`方法，并且会在`SpringApplication.run(…)`完成以前被调用。

```
@Component
public class MyBean implements ApplicationRunner{
    public void run(ApplicationArguments args) {
        // Do something...
    }
}
// ---------------------------------------- //
@Component
public class MyBean implements CommandLineRunner {
    public void run(String... args) {
        // Do something...
    }
}

```
如果你有多个这些bean的实现，你可以通过实现`org.springframework.core.Ordered`接口或者直接使用`org.springframework.core.annotation.Order`注解，标明这些bean的执行顺序。

### 23.9. Application exit

每一个`SpringApplication`都会为JVM注册一个shutdown hook，确保`ApplicationContext`在程序退出时优雅的关闭。所有标准的Spring生命周期callbacks都可以使用（如，实现`DisposableBean`接口，或者使用`@PreDestroy`注解）。

另外，如果需要在`SpringApplication.exit()` 退出时返回一个特定的exit code。bean也可以实现`org.springframework.boot.ExitCodeGenerator`接口，那么这个exit code会作为一个状态码传递给`System.exit()`。
```
@SpringBootApplication
public class ExitCodeApplication {
    @Bean
    public ExitCodeGenerator exitCodeGenerator() {
        return new ExitCodeGenerator() {
            @Override
            public int getExitCode() {
            return 42;
            }
        };
    }

    public static void main(String[] args) {
        System.exit(SpringApplication
            .exit(SpringApplication.run(ExitCodeApplication.class, args)));
    }
}
```
同样，exception也可以实现ExitCodeGenerator接口。当发生异常时，Spring Boot可以提供这个exit code.

### 23.10. Admin features

可以通过设置·spring.application.admin.enabled·的属性值来启用admin相关的特性。这会在·MBeanServer·平台上暴露·SpringApplicationAdminMXBean·。你可以使用这个特性远程管理你的Spring Boot application。

**Tip:** 如果你想知道application运行在哪个HTTP端口，可以通过获取key为的`local.server.port`属性查看。
