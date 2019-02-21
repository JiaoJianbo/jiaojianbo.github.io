---
layout: post
title: "Java 11 新特性"
date: 2019-02-22 11:00:00 +0800
categories: JDK
tags: Java JDK
author: Bobby
---

* content
{:toc}

Java 8 应该是目前使用最广泛的版本，但目前 Java 最新版本已经到 11.0.2。Java 11 也是继 Java 8 之后的又一个长期支持版本（Long-Term-Support，LTS），我相信过不了多久 Java 11 也将得到广泛使用。因此 Java 9 到 Java 11 的各个版本的新特性也不能不了解。



### Java 11 新特性

北京时间 2018 年 9 月 26 日，Oracle 官方宣布 Java 11 正式发布。这是继 Java 8 之后的又一个长期支持版本（Long-Term-Support，LTS）。

#### 改进 64 位架构的内联函数 (Improve Aarch64 Intrinsics)

改进现有的字符串和数组内部函数，并在 ARM64 位架构的处理器上，为 java.lang.Math 下的 sin、 cos 和 log 函数提供新的新的内部函数实现。

#### Epsilon — 一个无操作的垃圾收集器 (Epsilon: A No-Op Garbage Collector)

开发一个GC来处理内存分配，但不实现任何实际的内存回收机制。一旦可用的Java堆耗尽，JVM将关闭。EpsilonGC 看起来和感觉上都和其他OpenJDK GC 一样，使用参数 `-XX:+UseEpsilonGC` 启用。

#### 移除 Java EE 和 CORBA 模块 (Remove the Java EE and CORBA Modules)

Java EE 和 CORBA 两个模块在 JDK 9 中已经标记"deprecated"，在 JDK 11 中正式移除。JDK 中 deprecated 的意思是在不建议使用，在未来的 release 版本会被删除。

但是这个特性和 Java SE 关系不大。Java EE 可以单独引用，没必要把 Java EE 包含到 Java SE 中。

#### 标准的 HTTP 客户端 (HTTP Client (Standard))

HTTP Client 从 Java 9 开始引入孵化，到 Java 11 总算确定形成。替代了古老的 HttpURLConnection API，支持 HTTP/2 和 WebSocket，并且可以使用异步接口。

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.concurrent.CompletableFuture;

public class HttpApi {

    public static void main(String[] args) {
        HttpRequest request = HttpRequest.newBuilder().uri(URI.create("https://www.qq.com/")).GET().build();
        HttpResponse.BodyHandler<String> bodyHandler = HttpResponse.BodyHandlers.ofString();
        HttpClient client = HttpClient.newHttpClient();
        CompletableFuture<HttpResponse<String>> future = client.sendAsync(request, bodyHandler);
        future.thenApply(HttpResponse::body).thenAccept(System.out::println).join();
    }
}
```

#### 用于 Lambda 参数的局部变量语法 (Local-Variable Syntax for Lambda Parameters)

使用 var 声明局部变量类型的方式在 Java 10 的时候就已经有了。在 Java 11 中， Lambda 表达式中和也可以使用 var 关键字来标识变量，变量类型由编译器自行推断。

```java
import java.util.Arrays;

public class LocalVar {
    public static void main(String[] args) {
        Arrays.asList("Java", "Python", "Ruby")
                .forEach((var s) -> {
                    System.out.println("Hello, " + s);
                });
    }
}
```

Java 10 让隐式类型变量可用于本地变量:

```java
var foo = new Foo();
for (var foo : foos) { ... }
try (var foo = ...) { ... } catch { ... }
```

lamdba表达式可能是隐式类型的，它形参的所有类型全部靠推到出来的。为了和本地变量保持一致，我们希望允许 var 作为隐式类型 lambda 表达式的形参：

```java
(var x, var y) -> x.process(y)
```

#### Unicode 10

支持 Unicode 的最新版本（[Unicode 10.0](http://www.unicode.org/versions/Unicode10.0.0/) 增加了 8518 个字符, 总计达到了 136,690 个字符. 并且增加了 4 个脚本, 总结 139 个脚本, 同时还有56个新的 emoji 表情符号。），主要在以下类中:

* java.lang 包下的 *Character* 和 *String*。
* java.awt.font 包下的 *NumericShaper*。
* java.text 包下的 *Bidi*, *BreakIterator* 和 *Normalizer*。

Unicode 是一个不断进化的工业标准，因此必须不断保持 Java 和 Unicode 最新版本同步。

#### 飞行记录器 (Flight Recorder)

为 Java 应用程序和 HotSpot JVM 的故障诊断提供一个低开销的数据收集框架。Flight Recorder 记录源自应用程序，JVM 和 OS 的事件。事件存储在一个文件中，该文件可以附加到错误报告中并由支持工程师进行检查，允许事后分析导致问题的时期内的问题。工具可以使用 API 从记录文件中提取信息。

#### 启动单一文件的源代码程序 （Launch Single-File Source-Code Programs）

以前我们运行一个 Java 源代码必须先编译，再运行，两步执行动作。

```bash
// 编译
javac HelloWorld.java
// 运行
java HelloWorld
```

而现在只需一步：

```bash
java HelloWorld.java
```

#### 低开销的堆性能分析 (Low-Overhead Heap Profiling)

提供一种低开销的 Java 堆分配采样方法，得到堆分配的 Java 对象信息，可通过 JVMTI 访问。

对用户来说，了解它们堆里的内存是很重要的需求。目前有一些已经开发的工具，允许用户窥探它们的堆，比如：Java Flight Recorder, jmap, YourKit, 以及 VisualVM tools。但是这工具都有一个很大的缺点：无法得到对象的分配位置。

#### 支持 TLS 1.3 (Transport Layer Security 1.3)

实现 TLS 协议 1.3 版本。（TLS 允许客户端和服务端通过互联网以一种防止窃听，篡改以及消息伪造的方式进行通信）。

TLS 1.3 是 TLS 协议的重大改进。与以前的版本相比，它提供了显着的安全性和性能改进。其他供应商的几个早期实现已经可用。我们需要支持 TLS 1.3 以保持竞争力并与最新标准保持同步。这个特性的实现动机和 Unicode 10 一样，也是紧跟历史潮流。

#### 可伸缩低延迟垃圾收集器 (ZGC: A Scalable Low-Latency Garbage Collector (Experimental))

ZGC：这应该是 JDK11 最为瞩目的特性，没有之一。但是后面带了 Experimental，说明还不建议用到生产环境。看看官方对这个特性的目标描述：

* GC暂停时间不会超过 10ms
* 既能处理几百兆小堆，也能处理几个 T 的大堆
* 和 G1 相比，应用吞吐能力不会下降超过 15%
* 为未来的 GC 功能和利用 colord 指针以及 Load barriers 优化奠定基础
* 初始只支持64位系统

GC 是 Java 主要优势之一。然而，当 GC 停顿太长，就会开始影响应用的响应时间。消除或者减少 GC 停顿时长，Java 将对更广泛的应用场景是一个更有吸引力的平台。此外，现代系统中可用内存不断增长，用户和程序员希望 JVM 能够以高效的方式充分利用这些内存，并且无需长时间的 GC 暂停时间。
ZGC 是一个并发，基于region，压缩型的垃圾收集器，只有 root 扫描阶段会 STW (Stop-the-world)，因此 GC 停顿时间不会随着堆的增长和存活对象的增长而变长。

ZGC 和 G1 停顿时间比较（时间越短越好）：

```text
ZGC
                avg: 1.091ms (+/-0.215ms)
    95th percentile: 1.380ms
    99th percentile: 1.512ms
  99.9th percentile: 1.663ms
 99.99th percentile: 1.681ms
                max: 1.681ms

G1
                avg: 156.806ms (+/-71.126ms)
    95th percentile: 316.672ms
    99th percentile: 428.095ms
  99.9th percentile: 543.846ms
 99.99th percentile: 543.846ms
                max: 543.846ms
```

因为 ZGC 还处于实验阶段，所以需要通过 JVM 参数 UnlockExperimentalVMOptions 来解锁这个特性。

```text
-XX:+UnlockExperimentalVMOptions -XX:+UseZGC
```

#### 弃用 Nashorn JavaScript 引擎 (Deprecate the Nashorn JavaScript Engine)

弃用 Nashorn JavaScript脚本引擎、API 和jjs工具，以便在将来的版本中删除它们。Nashorn JavaScript 引擎是作为Rhino脚本引擎的替代品在 Java 8 引入的。

#### 弃用 Pack200 工具和 API (Deprecate the Pack200 Tools and API)

弃用 pack200 和 unpack200 工具，Pack200 API 在 java.util.jar 中。Pack200 是JAR文件的压缩方案，在 Java SE 5.0 中引入的。

#### 其他 API 变化

java.util.Collection 接口添加新的默认方法 *toArray(IntFunction)*。是对现有的 *toArray(T[])* 方法的重载。

```java
// 旧的方法:传入String[]:
String[] oldway = list.toArray(new String[list.size()]);

// 新的方法:传入IntFunction:
String[] newway = list.toArray(String[]::new);
```

String 新增了 *strip()* 方法。和 *trim()* 相比，*strip()* 可以去掉 Unicode 空格，例如，中文空格：

```java
String s = " Hello, JDK11!\u3000\u3000";
System.out.println("     original: [" + s + "]");
System.out.println("         trim: [" + s.trim() + "]");
System.out.println("        strip: [" + s.strip() + "]");
System.out.println(" stripLeading: [" + s.stripLeading() + "]");
System.out.println("stripTrailing: [" + s.stripTrailing() + "]");
```

输出如下：

```text
     original: [ Hello, JDK11!　　]
         trim: [Hello, JDK11!　　]
        strip: [Hello, JDK11!]
 stripLeading: [Hello, JDK11!　　]
stripTrailing: [ Hello, JDK11!]
```

还新增 *isBlank()* 方法，可判断字符串是不是“空白”字符串。这个方法在诸多 StringUtils 工具类中都有。

```java
String s = " \u3000"; // 由一个空格和一个中文空格构成
System.out.println(s.isEmpty()); // false
System.out.println(s.isBlank()); // true
```

还新增 *lines()* 方法，可以非常方便地按行分割字符串：

```java
String s = "Java\nPython\nRuby";
s.lines().forEach(System.out::println);
```

还新增 *repeat()* 方法，可以指定重复次数：

```java
System.out.println("-".repeat(10)); // 打印----------
```

java.nio.file.Files 类新增 *writeString* 和 *readString* 两个静态方法，可以直接把 String 写入文件，或者把整个文件读出为一个 String：

```java
Files.writeString(
        Path.of("./", "tmp.txt"), // 路径
        "hello, jdk11 files api", // 内容
        StandardCharsets.UTF_8); // 编码

String s = Files.readString(
        Paths.get("./tmp.txt"), // 路径
        StandardCharsets.UTF_8); // 编码
```

这两个方法可以大大简化读取配置文件之类的问题。

<br/>
[参考]  
[JDK11新特性解读](https://www.liaoxuefeng.com/article/0015419379727788f4e146b6fb1409dbaa7ad35db2560fc000)  
[What's New in JDK 11 - New Features and Enhancements](https://www.oracle.com/technetwork/java/javase/11-relnote-issues-5012449.html#NewFeature)  
[OpenJDK JDK 11](http://openjdk.java.net/projects/jdk/11/)  
[JDK11新特性解读](https://blog.csdn.net/shenyonggang/article/details/82856349)