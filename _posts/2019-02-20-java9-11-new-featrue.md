---
layout: post
title: "Java 9 到 Java 11 各个版本的新特性"
date: 2019-02-20 11:00:00 +0800
categories: JDK
tags: Java JDK
author: Bobby
---

* content
{:toc}

Java 8 应该是目前使用最广泛的版本，但目前 Java 最新版本已经到 11.0.2。Java 11 也是继 Java 8 之后的又一个长期支持版本（Long-Term-Support，LTS），我相信过不了多久 Java 11 也将得到广泛使用。因此 Java 9 到 Java 11 的各个版本的新特性也不能不了解。



### Java 9 新特性

JDK9 在2017年9月21日发布。

#### JDK 和 JRE 的改变

在 Java SE 9之前，JDK 构建系统用于生成两种类型的运行时映像 —— Java 运行时环境（JRE）和 Java 开发工具包（JDK）。JRE 是 Java SE 平台的完整实现，JDK 包含了 JRE 和开发工具和类库。下图显示了Java SE 9之前的JDK安装中的主目录：

* 在 Java SE 9之前，JDK中：

  <img src="/assets/images/2019/02/java-pre-9.png" alt="Before Java 9" />

  bin 目录用于包含命令行开发和调试工具，如 javac, jar 和 javadoc。它还用于包含 Java 命令来启动 Java 应用程序。  
  include 目录包含在编译本地代码时使用的 C/C++ 头文件。  
  lib 目录包含 JDK 工具的几个 JAR 和其他类型的文件。它有一个 tools.jar 文件，其中包含 javac 编译器的 Java 类。  
  jre\bin 目录包含基本命令，如 java 命令。在 Windows 平台上，它包含系统的运行时动态链接库（DLL）。  
  jre\lib 目录包含用户可编辑的配置文件，如 `.properties` 和 `.policy` 文件。  
  jre\lib\approved 目录包含允许使用标准覆盖机制的 JAR。这允许在 Java 社区进程之外创建的实施标准或独立技术的类和接口的更高版本被并入Java 平台。这些 JAR 被添加到 JVM 的引导类路径中，从而覆盖了 Java 运行时中存在的这些类和接口的任何定义。  
  jre\lib\ext 目录包含允许扩展机制的 JAR。该机制通过扩展类加载器（该类加载器）加载了该目录中的所有 JAR，该引导类加载器是系统类加载器的子进程，它加载所有应用程序类。通过将 JAR 放在此目录中，可以扩展 Java SE 平台。这些 JAR 的内容对于在此运行时映像上编译或运行的所有应用程序都可见。  
  jre\lib 目录包含几个 JAR。如 rt.jar 文件包含运行时的 Java 类和资源文件。许多工具依赖于 rt.jar 文件的位置。  
  jre\lib 目录包含用于非 Windows 平台的动态链接本地库。  
  jre\lib 目录包含几个其他子目录，其中包含运行时文件，如字体和图像。  

* 在Java SE 9 的JDK中：

  Java SE 9 调整了 JDK 的目录层次结构，并删除了 JDK 和 JRE 之间的区别。下图显示了 Java SE 9 中 JDK 安装的目录。JDK 9 中的 JRE 安装不包含 include 和 jmods 目录:

  <img src="/assets/images/2019/02/java-after-9.png" alt="After Java 9" />

  没有名为 jre 的子目录。  
  bin 目录包含所有命令。在 Windows 平台上，它继续包含系统的运行时动态链接库。  
  conf 目录包含用户可编辑的配置文件，例如以前位于 jre\lib 目录中的 `.properties` 和 `.policy` 文件。  
  include 目录包含要在以前编译本地代码时使用的 C/C++ 头文件。它只存在于 JDK 中。  
  jmods 目录包含 JMOD 格式的平台模块。创建自定义运行时映像时需要它。它只存在于 JDK 中。  
  legal 目录包含法律声明。  
  lib 目录包含非 Windows 平台上的动态链接本地库。其子目录和文件不应由开发人员直接编辑或使用。

#### 模块化 （Modularity）

Modularity 提供了类似于 OSGI 框架的功能，模块之间存在相互的依赖关系，可以导出一个公共的 API，并且隐藏实现的细节，Java 提供该功能的主要的动机在于，减少内存的开销。我们大家都知道，在 JVM 启动的时候，至少会有 30～60MB 的内存加载。主要原因是 JVM 需要加载 rt.jar，不管其中的类是否被 classloader 加载，第一步整个 jar 都会被 JVM 加载到内存当中去。模块化可以根据模块的需要加载程序运行需要的 class。

在引入了模块系统之后，JDK 被重新组织成 94 个模块。Java 应用可以通过新增的 **jlink** 工具，创建出只包含所依赖的 JDK 模块的自定义运行时镜像。这样可以极大的减少 Java 运行时环境的大小。采用模块化系统的应用程序只需要这些应用程序所需的那部分JDK模块，而非是整个JDK框架了。

#### 版本字符串格式改变（Version-String Format）

新的版本字符串格式为：$MAJOR.$MINOR.$SECURITY.$PATCH

$MAJOR: 主版本号  
$MINOR: 次版本号  
$SECURITY: 安全相关更新版本号  
$PATCH: 补丁版本号，

当安全版本号、次版本号或主版本号增加时，补丁版本号都要重置为零。当主版本号增加时，次版本号和安全版本号元素的后续版本号将设置为零。但是，当次版本号增加时，后续的安全版本号不需要设置为零。无论次版本号是什么，更高的安全版本值都表示更安全的版本。

#### 多版本兼容 JAR

多版本兼容 JAR 功能能让你创建仅在特定版本的 Java 环境中运行库程序时选择使用的 class 版本：

```text
multirelease.jar
├── META-INF
│   └── versions
│       └── 9
│           └── multirelease
│               └── Helper.class
├── multirelease
    ├── Helper.class
    └── Main.class
```

在上述场景中， multirelease.jar 可以在 Java 9 中使用, 不过 Helper 这个类使用的不是顶层的 multirelease.Helper 这个 class, 而是处在“META-INF/versions/9”下面的这个。这是特别为 Java 9 准备的 class 版本，可以运用 Java 9 所提供的特性和库。同时，在早期的 Java 诸版本中使用这个 JAR 也是能运行的，因为较老版本的 Java 只会看到顶层的这个 Helper 类。

#### 访问资源

Java 提供了一种通过在类路径上定位资源来访问资源的位置无关的方式。需要与在 JAR 中打包类文件相同的方式打包资源，并将 JAR 添加到类路径。通常，类文件和资源打包在同一个 JAR 中。

* 在 JDK 9 之前访问资源

  在 JDK 9 之前，可以使用以下两个类中的方法来访问资源：  
  java.lang.Class  
  java.lang.ClassLoader  
  两个类中有两种不同的命名实例方法：  
  *URL getResource(String name)*  
  *InputStream getResourceAsStream(String name)*  
  两种方法都会以相同的方式找到资源，它们的差异仅在于返回类型。

  ClassLoader类包含三个额外的查找资源的静态方法：  
  *static URL getSystemResource(String name)*  
  *static InputStream getSystemResourceAsStream(String name)*  
  *static Enumeration\<URL\> getSystemResources(String name)*

* 在 JDK 9 中访问资源

  在 JDK 9 之前，可以从类路径上的任何 JAR 访问资源。在 JDK 9 中，类和资源封装在模块中。为了有限地访问模块中的资源，做了一些妥协，但是仍然强制执行模块的封装。JDK 9 包含三类资源查找方法：  
  java.lang.Class  
  java.lang.ClassLoader  
  java.lang.Module  
  Class 和 ClassLoader 类没新增任何新的方法。Module 类包含一个*getResourceAsStream(String name)*方法，如果找到该资源，返回一个 InputStream，否则返回 null。

#### JShell

Python 中的读取-求值-打印循环（Read-Evaluation-Print Loop, REPL）很方便。它的目的在于以即时结果和反馈的形式。许多语言已经具有交互式编程环境，Java 现在加入了这个俱乐部。您可以从控制台启动 jshell，并直接启动输入和执行 Java 代码。jshell 的即时反馈使它成为探索 API 和尝试语言特性的好工具。除了表达式之外，还可以创建 Java 类和方法。jshell 也有基本的代码完成功能。

<img src="/assets/images/2019/02/jshell.jpg" alt="jshell" />

#### 集合工厂方法

Java 9 增加了List.of(), Set.of(), Map.of() 和 Map.ofEntries() 等工厂方法来创建不可变集合。

```java
List strs = List.of("Hello", "World");
List ints = List.of(1, 2, 3);
Set strs = Set.of("Hello", "World");
Set ints = Set.of(1, 2, 3);
Map maps = Map.of("Hello", 1, "World", 2);
```

除了更短和更好阅读之外，这些方法也可以避免您选择特定的集合实现。 事实上，从工厂方法返回已放入数个元素的集合实现是高度优化的。这是可能的，因为它们是不可变的：**在创建后，继续添加元素到这些集合会导致 “UnsupportedOperationException”**。

#### 改进的 Stream API

在 Java 9 中 Stream 接口中添加了 4 个新的方法：dropWhile, takeWhile, ofNullable。还有个 iterate 方法的重载方法，可以让你提供一个 Predicate 来指定什么时候结束迭代：

```java
IntStream.iterate(1, i -> i < 100, i -> i + 1).forEach(System.out::println);
```

第二个参数是一个 Lambda，它会在当前 IntStream 中的元素到达 100 的时候返回 true。因此这个简单的示例是向控制台打印 1 到 99。  
除了对 Stream 本身的扩展，Optional 和 Stream 之间的结合也得到了改进。现在可以通过 Optional 的新方法 *stream* 将一个 Optional 对象转换为一个(可能是空的) Stream 对象：

```java
Stream<Integer> s = Optional.of(1).stream();
```

在组合复杂的 Stream 管道时，将 Optional 转换为 Stream 非常有用。

#### 私有接口方法

Java 8 为我们带来了接口的默认方法。如果在接口上有几个默认方法，代码几乎相同，会发生什么情况？通常，您将重构这些方法，调用一个可复用的私有方法。但默认方法不能是私有的。将复用代码创建为一个默认方法不是一个解决方案，因为该辅助方法会成为公共 API 的一部分。使用 Java 9，您可以向接口添加私有辅助方法来解决此问题：

```java
public interface MyInterface {
    void normalInterfaceMethod();

    default void interfaceMethodWithDefault() { init(); }
    default void anotherDefaultMethod() { init(); }

    // This method is not part of the public API exposed by MyInterface
    private void init() { System.out.println("Initializing"); }
}
```

#### HTTP/2

JDK9 之前提供 HttpURLConnection API 来实现 HTTP 访问功能，但是这个类基本很少使用，一般都会选择 Apache 的 Http Client，此次在Java 9 的版本中引入了一个新的 package: java.net.http，用于代替老旧的 HttpURLConnection API，并提供对 WebSocket 和 HTTP/2 的支持。注意：新的 HttpClient API 在 Java 9 中以所谓的孵化器模块交付。也就是说，这套 API 不能保证 100% 完成。不过你可以在 Java 9 中开始使用这套 API：

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest req = HttpRequest.newBuilder(URI.create("http://www.google.com"))
                             .header("User-Agent","Java")
                             .GET()
                             .build();
HttpResponse<String> resp = client.send(req, HttpResponse.BodyHandler.asString());
HttpResponse<String> resp = client.send(req, HttpResponse.BodyHandler.asString());
```

#### Javadoc 的变化

Javadoc 的增强包括以下内容: 简化的 Doclet API、Javadoc 搜索、Javadoc 的输出现在符合兼容 HTML5 标准以及支持模块系统中的文档注释。

<img src="/assets/images/2019/02/jdk9-api.jpg" alt="JDK9 API" />

#### 异常处理try升级

Java 7 之前的处理方式:

```java
InputStreamReader reader = null;

try {
    reader = new InputStreamReader(System.in);
    reader.read();
} catch(IOException e) {
    e.printStackTrace();
} finally {
    if(reader != null){
        try {
            reader.close();
        } catch(IOException e) {
            e.printStackTrace();
        }
    }
}
```

Java 7,8 共同的 try-with-resource 处理方式:

```java
// Java 7 和 8 每一个流打开的时候都要关闭，但是在try的括号中进行关闭
try (InputStreamReader reader = new InputStreamReader(System.in)) {
    reader.read();
} catch(IOException e) {
    e.printStackTrace();
}
```

Java 9 的处理方式：

```java
// Java 9 每一个流打开的时候都要关闭，但是在try的括号中进行关闭
// 在 Java 8 的基础上进一步升级，直接在try括号中写入变量就好。如果有多个流，就用分号分隔
// try (reader; writer){}
InputStreamReader reader = new InputStreamReader(System.in);

try (reader) {
    reader.read();
} catch(IOException e) {
    e.printStackTrace();
}
```

#### 压缩字符串 —— String 底层存储结构更换

Java8之前 String 的底层结构类型都是 char[], 但是 Java9 就替换成 byte[] 这样来讲，更节省了空间和提高了性能。

之所以替换是因为之前一直是最小单位是一个 char，用到两个 byte。但是 Java8 是基于 latin1 的，而这个 latin1 编码可以用一个 byte 标识，所以当你的数据明明可以用到一个 byte 的时候，我们用到了一个最小单位 char 两个 byte，就多出了一个 byte 的空间。所以 Java9 在这一方面进行了更新，现在的 Java9 是基于 ISO/latin1/UTF-16, latin1 和 ISO 用一个 byte标识，UTF-16 用两个 byte 标识，Java9会自动识别用哪个编码，当数据用到一个 byte，就会使用 iSO 或者 latin1，当空间数据满足两个 byte 的时候，自动使用 UTF-16，节省了很多空间。

#### 统一 JVM 日志

Java 9 中 ，JVM 有了统一的日志记录系统，可以使用新的命令行选项`-Xlog`来控制 JVM 上 所有组件的日志记录。该日志记录系统可以设置输出的日志消息的标签、级别、修饰符和输出目标等。

#### Java9的垃圾收集机制

Java 9 移除了在 Java 8 中 被废弃的垃圾回收器配置组合，同时把 G1 设为默认的垃圾回收器实现。替代了之前默认使用的 Parallel GC，对于这个改变，evens的评论是酱紫的：这项变更是很重要的，因为相对于 Parallel 来说，G1 会在应用线程上做更多的事情，而 Parallel 几乎没有在应用线程上做任何事情，它基本上完全依赖 GC 线程完成所有的内存管理。这意味着切换到 G1 将会为应用线程带来额外的工作，从而直接影响到应用的性能。

### Java 10 新特性



### Java 11 新特性


<br/>
[参考]  
[Java 9 逆天的十大新特性](https://blog.csdn.net/mxw2552261/article/details/79080678)  
[JDK9 新特性](http://swiftlet.net/archives/category/jdk9)  
[Java Platform, Standard Edition What’s New in Oracle JDK 9](https://docs.oracle.com/javase/9/whatsnew/toc.htm#JSNEW-GUID-C23AFD78-C777-460B-8ACE-58BE5EA681F6)  
[Java 9 新特性，看这里就明白了](https://baijiahao.baidu.com/s?id=1593429162250494010&wfr=spider&for=pc)  
[Java Doclet API](https://docs.oracle.com/javase/9/docs/api/jdk/javadoc/doclet/package-summary.html)  

[What's New in JDK 10 - New Features and Enhancements](https://www.oracle.com/technetwork/java/javase/10-relnote-issues-4108729.html#NewFeature)  

[JDK11新特性解读](https://www.liaoxuefeng.com/article/0015419379727788f4e146b6fb1409dbaa7ad35db2560fc000)