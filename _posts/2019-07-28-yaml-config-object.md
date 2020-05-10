---
layout: post
title: "YAML 文件中配置对象"
date: 2019-07-28 11:40:00+0800
categories: Spring-Boot
tags: Spring-Boot
author: Bobby
---

* content
{:toc}

项目开发中一些参数值需要作为配置项，放在 YAML 文件中。对于简单类型以及 String 类型的值，配置在 YAML 中，然后使用 `@Value`注解绑定即可。但是对于一些复杂对象，或者自定义的对象又该如何配置和绑定呢？


### 问题场景

对于以下的几个参数都需要作为配置项：

```java
private boolean enabled;
private Long count;
private List<Employee> employeeList = new ArrayList<>();
```

其中 Employee 的定义如下：

```java
@Data
public class Employee {
    private String id;
    private String name;
}
```

于是 YAML 文件中就有了以下配置：

```yaml
myConfig:
  enabled: true
  count: 10
  employeeList:
    - {id: "001", name: "Ada"}
    - {id: "002", name: "Bob"}
```

查了一些资料，要得到 `List<Employee> employeeList` 属性，在 YAML 中应该按照上面的形式配置。现在只需要将配置跟参数绑定即可。对于前两个参数使用`@Value`注解很容易就可以取到参数值，但是对于复杂对象`@Value`却并不支持。

```java
@Value("${myConfig.enabled}")
private boolean enabled;

@Value("${myConfig.count}")
private Long count;

// 会抛异常，@Value 不支持复杂对象
//@Value("${myConfig.employeeList}")
//private List<Employee> employeeList = new ArrayList<>();
```

### 解决办法

对于复杂对象 Spring-Boot 提供了`org.springframework.boot.context.properties.ConfigurationProperties`注解，可以将复杂对象单独抽成一个配置对象。如：

```java
@ConfigurationProperties(prefix = "my-config")
@Component
@Data
public class MyConfig {
    private boolean enabled = false;

    private Long count = 0L;

    private List<Employee> employeeList = new ArrayList<>();
}
```

然后在需要使用这些配置值得地方，将 MyConfig 对象注入，然后通过 get 方法使用这些参数值。

```java
@Autowired
private MyConfig myConfig;
```

上面使用注解 `@ConfigurationProperties` 和 `@Component` 最终启用一个配置类。其实还有另一个方式，使用`@ConfigurationProperties` 和 `@EnableConfigurationProperties`, `@Configuration` 注解。如：

```java
@ConfigurationProperties(prefix = "my-config")
@Data
public class MyProperties {
    private boolean enabled = false;

    private Long count = 0L;

    private List<Employee> employeeList = new ArrayList<>();
}
```

```java
@EnableConfigurationProperties(MyProperties.class)
@Configuration
public class MyConfig {
}
```

在需要使用配置值的地方注入 MyProperties 对象即可。当然`@EnableConfigurationProperties`, `@Configuration`也可以和`@ConfigurationProperties`都加在 MyProperties 类上。

### 注意事项

* 首先要使用注解`@ConfigurationProperties`需要加入相关 jar 包。具体 Maven 依赖如下：

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-configuration-processor</artifactId>
  </dependency>
  ```

* 注解 @ConfigurationProperties 中的`prefix`值中不能包含大写字母。要么全部使用小些字母，如果参数中有驼峰命名格式，那么就用中划线`-`分隔每个单词，而且每个单词首字母都小写。如，我们使用`prefix = "my-config"`，那在 YAML 文件中可以使用`myConfig`或者`my-config`的格式。否则会有如下的错误信息：

  ```text
  Prefix must be in canonical form.
  ```
  ```text
  Canonical names should be kebab-case ('-' separated), lowcase alpha-numberic characters.
  ```

* Config 类以及用到的实体类（此处指 Employee）要符合 Java Bean 规范，都要包含无参构造函数以及 getter/setter 方法。否则不能注入参数值或对象，甚至还会抛出异常。这里我使用了 lombok 的 `@Data` 注解，会在编译时为类自动生成构造函数以及 getter/setter 方法。


### 一个更复杂的实例

Config 类以及相关的类

```java
@ConfigurationProperties(prefix = "service-url")
@Component
@Data
public class ServiceConfig {
  private List<EnvItem> envItemList;
}

@Data
public class EnvItem {
  private String envName;
  private List <ServiceItem> serviceList;
}

@Data
public class ServiceItem {
  private String name;
  private String method;
  private String url;
}
```

接下来是 YAML 文件中的配置

```yml
service-host:
  sit: 192.168.1.100
  uat: 192.168.100.10

service-url:
  env-item-list:
    - env-name: SIT
      service-list:
        - {name: "S1", method: "GET", url: "http://${service-host.sit}:8080/s1/user"}
        - {name: "S2", method: "GET", url: "http://${service-host.sit}:8081/s2/order"}
        - {name: "S3", method: "GET", url: "http://${service-host.sit}:8082/s3/pay"}
    - env-name: UAT
      service-list:
        - {name: "S1", method: "GET", url: "http://${service-host.uat}:8080/s1/user"}
        - {name: "S2", method: "GET", url: "http://${service-host.uat}:8081/s2/order"}
        - {name: "S3", method: "GET", url: "http://${service-host.uat}:8082/s3/pay"}
```

这里， `env-name` 和 `service-list` 组成一个 `EnvItem` 对象。

### 参考资料：
[SpringBoot中yaml配置对象](https://www.cnblogs.com/zhuxiaojie/p/6062014.html)  
[代码示例](https://github.com/JiaoJianbo/learning/tree/master/springboot-demo1)  
