---
layout: post
title: "Spring Boot 集成 MyBatis 延迟载失效？"
date: 2019-06-29 13:40:00 +0800
categories: MyBatis
tags: MyBatis Spring-Boot
author: Bobby
---

* content
{:toc}

今天在一个 Spring Boot 项目里，有个地方用到了延迟加载（也叫“懒加载”）。于是就顺手写了一个 test case，想看看这个延迟加载是不是真的按预想的那样，在需要的时候才去查数据库。



### 问题描述

但是结果并不像预想的那样。在执行 test case 的时候发现，原来认为会延迟执行的 SQL 在第一次就执行了。还是先上代码吧。简单起见，就只有 SysRole 和 User 两个类。在 SysRole 中有个 `private List<User> users;` 属性，就想让这个属性是延迟加载的。

SysRoleDao：

```java
public interface SysRoleDao {
    SysRole getRoleById(Integer id);

    SysRole getRoleByIdWithUsers(Integer id);

    List<User> getAllUserByRoldId(Integer id);
}
```

SysRoleMapper.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.bravo.demo.ssm.dao.SysRoleDao">
    <select id="getRoleById" resultType="com.bravo.demo.ssm.entity.SysRole">
        select * from sys_role where id = #{id}
    </select>
    
    <resultMap id="sysRoleResult" type="com.bravo.demo.ssm.entity.SysRole">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="status" property="status"/>
        <!-- 集合属性延迟加载 -->
        <collection property="users" select="com.bravo.demo.ssm.dao.SysRoleDao.getAllUserByRoldId" column="id" fetchType="lazy">
        </collection>
    </resultMap>
    <select id="getRoleByIdWithUsers" resultMap="sysRoleResult">
        select * from sys_role where id = #{id}
    </select>
    
    <select id="getAllUserByRoldId" resultType="com.bravo.demo.ssm.entity.User">
        select u.*
        from sys_role_user ru, user u
        where ru.uid = u.id
        and ru.rid = #{id}
    </select>
</mapper>
```

MyBatis 的全局配置：

```yaml
mybatis:
  type-aliases-package: com.bravo.demo.ssm.entity
  configuration:
    jdbc-type-for-null: NULL
    map-underscore-to-camel-case: true
    lazy-loading-enabled: true # 延迟加载
    aggressive-lazy-loading: false
```

Test case:

```java
@Test
public void testGetRoleByIdWithUsers() {
    SysRole role = sysRoleDao.getRoleByIdWithUsers(1);
    System.out.println(role);
    //System.out.println(role.getUsers());
}
```

从上面代码可以看出，为延迟加载做了双保险。在全局配置中启用，并且在 `collection` 元素中也使用了 `fetchType="lazy"` 属性。执行 test case，从 log 中可以看到，两次查询的 SQL 都执行了。

![log](/assets/images/2019/06/mybatis-lazyload-issue-log.jpg)

### 寻找解决方案

首先，仔细比对了一下以前的有关 MyBatis 延迟加载的做法，但是并没有发现什么不一样的地方。后来想到难道是因为集成了 Spring Boot 引起的，又翻了一下 Spring Boot 官方文档，没有发现相关内容。在网上也没看到有关集成 Spring Boot 或者 Spring 之后，MyBatis 延迟加载失效的 bug 问题。

再看了一下 test case，猜想难道是因为我输出了整个 `role` 对象导致的。于是将 test case 改为以下：

```java
@Test
public void testGetRoleByIdWithUsers() {
    SysRole role = sysRoleDao.getRoleByIdWithUsers(1);
    System.out.println(role.getName()); // 这里只输入 role 的 name 属性
    //System.out.println(role.getUsers());
}
```

再执行，此时真的只执行了第一条 SQL：

![log](/assets/images/2019/06/mybatis-lazyload-log.jpg)

感觉离真相近了一步，觉得应该是 `toString()` 方法中触发了延迟加载。查看 SysRole 类的 toString 方法：

```java
@Override
public String toString() {
    return "SysRole [id=" + id + ", name=" + name + ", status=" + status + "]";
}
```

可是看到，这个方法里面并没有用到 users 这个属性啊，按理说就不会去执行第二次查询。难道进了一条死胡同？总觉得有可能就是这个 `toString()` 方法触发了延迟加载，而跟方法里面是否用到这个属性没有关系。

### 解决办法

这时，突然印象当中在 MyBatis 的全局配置中有个类似 “lazyLoadTrigger” 的配置项。赶紧查看官方文档，果然让我发现了这么一个配置：

| 设置名 | 描述 | 有效值 | 默认值 |
| --- | --- | --- | --- |
| lazyLoadTriggerMethods | 指定哪个对象的方法触发一次延迟加载。 | 用逗号分隔的方法列表。 | equals,clone,hashCode,toString |

于是先修改一下配置，把 `toString` 移除，验证一下这个配置是否有效：

```yaml
mybatis:
  type-aliases-package: com.bravo.demo.ssm.entity
  configuration:
    jdbc-type-for-null: NULL
    map-underscore-to-camel-case: true
    lazy-loading-enabled: true # 延迟加载
    aggressive-lazy-loading: false
    lazy-load-trigger-methods: equals,clone,hashCode
```

再执行 test case（输出整个 role 对象），果然没有触发第二次查询：

![log](/assets/images/2019/06/mybatis-lazyload-log2.jpg)

看来就是因为输出 role 对象时，调用了 toString() 方法，从而触发的延迟加载问题，并不是 MyBatis 延迟加载失效。不过，`lazyLoadTriggerMethods` 在平时使用时保留默认值就够了，没有特殊需要不需要重写。在实际工作使用中，我们要有这个意识，知道这几个方法会触发延迟加载，而这个延迟加载有没有必要，根据需要再进行合理配置。

### 一探源码

查看源码，可以发现在两个类中用到了 `lazyLoadTriggerMethods` 配置。一个是 `org.apache.ibatis.executor.loader.cglib.CglibProxyFactory` 另一个是 `org.apache.ibatis.executor.loader.javassist.JavassistProxyFactory`，两者的作用大体相当。在这两个类中都有个内部类 `EnhancedResultObjectProxyImpl`，CglibProxyFactory 中的实现了 `MethodInterceptor` 接口，其中有个 `intercept` 方法。而 JavassistProxyFactory 中的则实现了 `MethodHandler` 接口，其中有个 `invoke` 方法。可以把它们看做最后处理 ResultSet 的拦截器链中的一个，两者在处理逻辑上几乎一样。以 CglibProxyFactory 中的 EnhancedResultObjectProxyImpl 为例，在它的 intercept 方法中有一段：

```java
if (lazyLoader.size() > 0 && !FINALIZE_METHOD.equals(methodName)) {
  if (aggressive || lazyLoadTriggerMethods.contains(methodName)) {
	lazyLoader.loadAll();
  } else if (PropertyNamer.isSetter(methodName)) {
	final String property = PropertyNamer.methodToProperty(methodName);
	lazyLoader.remove(property);
  } else if (PropertyNamer.isGetter(methodName)) {
	final String property = PropertyNamer.methodToProperty(methodName);
	if (lazyLoader.hasLoader(property)) {
	  lazyLoader.load(property);
	}
  }
}
```
从第二个 if 判断中就可以看出，如果配置项 `aggressiveLazyLoading` 为 `true`，或者当前的方法名包含在 `lazyLoadTriggerMethods` 配置项中，那么就会加载所有延迟加载的内容（这里的 `lazyLoader` 是一个 `org.apache.ibatis.executor.loader.ResultLoaderMap` 对象）。

### 参考资料：
[mybatis &#x2013; MyBatis 3 | 配置](http://www.mybatis.org/mybatis-3/zh/configuration.html#settings)  
MyBatis 3.4.6 源码
