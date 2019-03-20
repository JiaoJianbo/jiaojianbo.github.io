---
layout: post
title: "SSH 环境下使用 H2 Database 遇到的一些问题的总结"
date: 2019-03-20 18:00:00 +0800
categories: SSH
tags: Spring Hibernate H2
author: Bobby
---

* content
{:toc}

近日牙根发炎，脸肿，什么也干不了，索性花点时间整理一下以前做的一个 SSH 基础框架的项目，想上传到 github 上去供自己以后查阅参考。从头整理，顺便对一些依赖的框架版本也进行升级。如，将 Java 6 升级到 Java 8, Spring 3.1 升级到 Spring 4.3, Hibernate 3.6 升级到 Hibernate 5.0, 等等。为了达到“开箱即用”的效果，即代码拉下来以后对配置进行简单修改或零修改（但是 Java 8，Maven 的环境是要有的），就能让程序跑起来的，而不用再搭建复杂的数据库环境。我有意的将数据库换成了 H2 Database。但是其中还是遇到一些问题，觉得值得花时间总结一下。



### 执行初始化 SQL Script

想让程序在启动过程中执行一段初始化的 SQL 脚本其实方法挺多。比如使用 Listener。但是 Hibernate 提供了相应的配置，只要将 SQL 脚本整理好放在一个目录下就行。相应的 XML 配置为：

```xml
<prop key="hibernate.hbm2ddl.import_files">init-data.sql</prop>
```

Java 程序的写法大概如：

```java
jpaProperties.setProperty("hibernate.hbm2ddl.import_files", "init-data.sql");
```

`init-data.sql` 就放在 java/main/resources 目录下，前面也无需加`classpath:`前缀。

**Note**: 但是想要让该脚本顺利执行有个前提就是 Hibernate 配置 `hibernate.hbm2ddl.auto` 的值为`create`或`create-drop`。即：

```xml
<prop key="hibernate.hbm2ddl.auto">create-drop</prop>
```

### 初始化 SQL Script 执行出错

在开始测试初始化 SQL 脚本能否顺利执行，在 `init-data.sql` 写了一个简单的创建表的语句：

```sql
DROP TABLE IF EXISTS TEST;
CREATE TABLE TEST(
    ID INT PRIMARY KEY,
    NAME VARCHAR(255)
);
```

启动程序发现 SQL 脚本确实被检测到了，也去执行了，但是却报出语法错误：

```text
19:25:55.592 [main] INFO org.hibernate.tool.hbm2ddl.SchemaExport - HHH000476: Executing import script 'init-data.sql'
19:25:55.595 [main] ERROR org.hibernate.tool.hbm2ddl.SchemaExport - HHH000388: Unsuccessful: CREATE TABLE TEST(
19:25:55.595 [main] ERROR org.hibernate.tool.hbm2ddl.SchemaExport - Syntax error in SQL statement "CREATE TABLE TEST([*]"; expected "identifier"; SQL statement:
CREATE TABLE TEST( [42001-196]
19:25:55.595 [main] ERROR org.hibernate.tool.hbm2ddl.SchemaExport - HHH000388: Unsuccessful: ID INT PRIMARY KEY,
19:25:55.595 [main] ERROR org.hibernate.tool.hbm2ddl.SchemaExport - Syntax error in SQL statement "ID[*] INT PRIMARY KEY,"; expected "INSERT, {"; SQL statement:
ID INT PRIMARY KEY, [42001-196]
19:25:55.596 [main] ERROR org.hibernate.tool.hbm2ddl.SchemaExport - HHH000388: Unsuccessful: NAME VARCHAR(255)
19:25:55.596 [main] ERROR org.hibernate.tool.hbm2ddl.SchemaExport - Syntax error in SQL statement "NAME[*] VARCHAR(255)"; SQL statement:
NAME VARCHAR(255) [42000-196]
19:25:55.596 [main] ERROR org.hibernate.tool.hbm2ddl.SchemaExport - HHH000388: Unsuccessful: )
19:25:55.596 [main] ERROR org.hibernate.tool.hbm2ddl.SchemaExport - Syntax error in SQL statement ")[*]"; SQL statement:
) [42000-196]
19:25:55.597 [main] INFO org.hibernate.tool.hbm2ddl.SchemaExport - HHH000230: Schema export complete
```

可是这个 SQL 语句通过 H2 Console 根本没有问题，而且我们直观的看这个 SQL 语句本身也没有什么语法错误的。最后终于在 Stack Overflow 找到了[解决办法](https://stackoverflow.com/questions/17926093/h2-sql-grammar-exception)。即加入下面这个配置，或者一条 SQL 语句只写一行。

```xml
<prop key="hibernate.hbm2ddl.import_files_sql_extractor">org.hibernate.tool.hbm2ddl.MultipleLinesSqlCommandExtractor</prop>
```

显示 SQL 脚本已执行，而且没什么问题。

```text
......
21:14:42.510 [main] INFO org.hibernate.tool.hbm2ddl.SchemaExport - HHH000476: Executing import script 'init-data.sql'
21:14:42.540 [main] INFO org.hibernate.tool.hbm2ddl.SchemaExport - HHH000230: Schema export complete
......
```

### 使用 H2 Console 查看数据

既然数据已经初始化好了，那么我们就可以通过 H2 console 去检查数据了。但是登陆进去，却发现什么都没有（也许是我的方法不对）。

![H2 Console local](/assets/images/2019/03/h2-console-local-check.jpg)

再次复习了一下 H2 的相关知识，H2 有多种使用模式，我个人认为主要分两种：一种是嵌入式（或者文件模式），就是可持久化的，最终在磁盘上有个数据文件。而另一种是内存模式，随着服务启动生成，数据库关闭，数据丢失。而每种模式有都可以分为本地连接和方式和远程连接方式。各种连接模式的 URL 写法也各不相同：（更详细的各种 URL 写法参见[官网](http://www.h2database.com/html/features.html#database_url)）

```properties
### File mode
# 默认情况下只允许有一个客户端连接到H2数据库，有客户端连接到H2数据库之后，此时数据库文件就会被锁定
jdbc.url=jdbc:h2:file:~/${jdbc.dbname}
# 基于Service的形式进行连接的，因此允许多个客户端同时连接到H2数据库
jdbc.url=jdbc:h2:tcp://${jdbc.host}/~/${jdbc.dbname}
### Memory mode 
# 这种模式下，打开Console控制台是个空白的数据库
jdbc.url=jdbc:h2:mem:jeef
# 只有使用远程模式时，单独打开Console控制台才可以看见相关数据
jdbc.url=jdbc:h2:tcp://${jdbc.host}/mem:${jdbc.dbname}
```

所以，如果我使用内存模式，就应该选择这种`jdbc.url=jdbc:h2:tcp://${jdbc.host}/mem:${jdbc.dbname}` URL 连接方式（官网上这种方式也叫 Server mode 下的远程连接方式，即 Server mode (remote connections) using TCP/IP）。我们再看一下：

![H2 Console remote login](/assets/images/2019/03/h2-console-remote-check-login.jpg)

![H2 Console remote](/assets/images/2019/03/h2-console-remote-check.jpg)

最终也在官网找到了相关说法，应该使用 Server mode (remote connections)。[Link→](http://www.h2database.com/html/features.html#in_memory_databases)

---

### 题外话

终于发现在 Markdown 文档里引用图片，可以不再使用 HTML 的 `img` 标签了 \\(^o^)/。以前做法：

```html
<img src="/assets/images/2019/03/h2-console-remote-check.jpg" alt="H2 Console remote" />
```

现如今做法：

```markdown
![H2 Console remote](/assets/images/2019/03/h2-console-remote-check.jpg)
```

---

### 有关 H2 Database 的使用，可以参考下面的一些资料：

[官网](http://www.h2database.com/html/main.html)  
[Java嵌入式数据库H2学习总结(一)——H2数据库入门](https://www.cnblogs.com/xdp-gacl/p/4171024.html)  
[Java嵌入式数据库H2学习总结(二)——在Web应用程序中使用H2数据库](https://www.cnblogs.com/xdp-gacl/p/4171278.html)  
[Java嵌入式数据库H2学习总结(三)——在Web应用中嵌入H2数据库](https://www.cnblogs.com/xdp-gacl/p/4190424.html)  
[Stack Overflow - H2 SQL Grammar Exception](https://stackoverflow.com/questions/17926093/h2-sql-grammar-exception)
