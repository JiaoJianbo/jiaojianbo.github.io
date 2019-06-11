---
layout: post
title: "自定义 Spring Boot Starter"
date: 2019-06-11 19:40:00 +0800
categories: Spring-Boot
tags: Spring-Boot
author: Bobby
---

* content
{:toc}

今天又复习了一下有关自定义 Spring Boot Starter 的内容，总体来说步骤如下所列。文中并没有贴出代码，详细的代码可以[点击这里](https://github.com/JiaoJianbo/learning/tree/master/springboot-starter-demo)到我的 GitHub 仓库。



### 自定义 Spring Boot Starter （场景启动器）

1. 该 Starter 需要使用的依赖有哪些？

    *这些依赖可以配置在 xxx-starter 工程下，如果 xxx-autoconfigure 工程也需要，那就直接配在 autoconfigure 工程中即可，不用再在 starter 工程下配置。*

2. 如何编写自动配置。（参考其他 AutoConfiguration 类）

	* `@Configuration`                                 指定该类是个配置类  
	* `@ConditionalOnXxx`                              在指定条件成立时，配置生效  
	* `@AutoConfigureOrder`                            指定自动配置的顺序  
	* `@AutoConfigureBefore`  
	* `@AutoConfigureAfter`  
	* `@ConfigurationProperties(prefix="my.bravo")`    结合 XxxProperties 类绑定相关配置  
	* `@EnableConfigurationProperties`                 启用 XxxProperties 配置类，并加入到容器中  
	* `@Bean`                                          给容器中添加指定的 Bean  

	将需要启动就加载的自动配置类，配置在  META-INF/spring.factories 文件中。例如：
	
	```properties
	# Auto Configure
	org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
	org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
	org.springframework.boot.autoconfigure.aop.AopAutoConfiguration
	```

3. 启动器（Starter）只用来做依赖导入，专门写一个自动配置模块，启动器依赖自动配置。这样提供给第三方时，只需要引入启动器即可。  

    *如，my-springboot-starter-test 中使用自定义 Starter 时，只引入了 my-springboot-starter，并没有再引入 my-springboot-starter-autoconfigure。*

4. 命名规约：xxx-starter, xxx-starter-autoconfigure

	* 官方：  
		- 前缀：spring-boot-starter-  
		- 模式：spring-boot-starter-模块名  
		- 例如：spring-boot-starter-web, spring-boot-starter-aop  
    
	* 自定义：  
		- 后缀：-spring-boot-starter  
		- 模式：模块名-spring-boot-starter  
		- 例如：mybatis-spring-boot-starter  
