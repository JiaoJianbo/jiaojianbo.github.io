---
layout: post
title: "Quartz + H2 DB 时 Couldn't store job: Function EMPTY_BLOB() not found 错误解决办法"
date: 2019-07-13 11:40:00 +0800
categories: Windows
tags: Windows
author: Bobby
---

* content
{:toc}

本来项目中一直使用的是 Oracle 数据库，但是这几日开发环境中 Oracle 资源到期，暂时没有 Oracle 可用。为了正常开发，本地不得不使用 H2 数据库暂时替代一下。好在项目中使用的是 JPA，所以自己写的东西切换到 H2 基本没有什么严重的问题，但是集成的 Quartz 部分功能却遇到了一些异常。



### 背景

首先说一下涉及到的几个资源的大概版本。  
* JDK 1.8
* Spring Boot 2.1.x
* Quartz 2.3
* H2 1.4.199

其实，这里的问题跟是否集成 Spring Boot 关系不大。开始只是将配置中数据库的连接换成了 H2 的，在 Oracle 创建 Quartz 表的 SQL 脚本中的所有 VARCHAR2 类型改为 VARCHAR， 然后在 H2 上执行创建了相关的表和索引。

### 遇到问题

项目启动时，初始化 Quartz 的过程中就抛出了类似 “Couldn't store job: Function EMPTY_BLOB() not found” 的异常。另外还有，字段 IS_DURABLE（在表 QRTZ_JOB_DETAILS 中）长度不够的问题。

在网大概看了一下，有的说是低版本 Quartz 中的问题，要么升级版本，要么需要改源码。但是我觉得都到 2.x 的版本了，Quartz 不至于犯这种错误。于是，从头梳理了一下整个过程，最终解决了这几个问题，下面大概说一下具体步骤。

### 解决办法

这个解决办法是基于现有项目的，我们只需要关注重点的通用步骤。

1. pom.xml 中引入 H2 的依赖。

2. 修改 applcation.yml 或 applcation.properties 中 Datasource 的信息为 H2 的。

接下来是通用的步骤，跟是否集成 Spring Boot 没有多大关系。

3. 在 [Quartz 官方 GitHub 仓库](https://github.com/quartz-scheduler/quartz/tree/master/quartz-core/src/main/resources/org/quartz/impl/jdbcjobstore)中找到对应数据库的建表脚本，并执行，初始化数据表。

4. 在 quartz.properties 中，配置 `org.quartz.jobStore.driverDelegateClass` 为 `org.quartz.impl.jdbcjobstore.StdJDBCDelegate`.

```properties
#org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.oracle.OracleDelegate
org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
```

Quartz 提供的 `driverDelegateClass` 主要有以下几种，涵盖了大多数的关系型数据库。

| org.quartz.jobStore.driverDelegateClass |
| --- |
| org.quartz.impl.jdbcjobstore.StdJDBCDelegate (for fully JDBC-compliant drivers) |
| org.quartz.impl.jdbcjobstore.MSSQLDelegate (for Microsoft SQL Server, and Sybase) |
| org.quartz.impl.jdbcjobstore.PostgreSQLDelegate |
| org.quartz.impl.jdbcjobstore.WebLogicDelegate (for WebLogic drivers) |
| org.quartz.impl.jdbcjobstore.oracle.OracleDelegate |
| org.quartz.impl.jdbcjobstore.oracle.WebLogicOracleDelegate (for Oracle drivers used within Weblogic) |
| org.quartz.impl.jdbcjobstore.oracle.weblogic.WebLogicOracleDelegate (for Oracle drivers used within Weblogic) |
| org.quartz.impl.jdbcjobstore.CloudscapeDelegate |
| org.quartz.impl.jdbcjobstore.DB2v6Delegate |
| org.quartz.impl.jdbcjobstore.DB2v7Delegate |
| org.quartz.impl.jdbcjobstore.DB2v8Delegate |
| org.quartz.impl.jdbcjobstore.HSQLDBDelegate |
| org.quartz.impl.jdbcjobstore.PointbaseDelegate |
| org.quartz.impl.jdbcjobstore.SybaseDelegate |

### 参考资料：
[Quartz 2.3.0 Documentation](http://www.quartz-scheduler.org/documentation/quartz-2.3.0/)  
[Quartz Job Scheduler GitHub](https://github.com/quartz-scheduler)  
