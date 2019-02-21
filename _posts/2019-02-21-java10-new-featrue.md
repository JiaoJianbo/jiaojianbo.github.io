---
layout: post
title: "Java 10 新特性"
date: 2019-02-21 11:00:00 +0800
categories: JDK
tags: Java JDK
author: Bobby
---

* content
{:toc}

Java 8 应该是目前使用最广泛的版本，但目前 Java 最新版本已经到 11.0.2。Java 11 也是继 Java 8 之后的又一个长期支持版本（Long-Term-Support，LTS），我相信过不了多久 Java 11 也将得到广泛使用。因此 Java 9 到 Java 11 的各个版本的新特性也不能不了解。



### Java 10 新特性

2018 年 3 月 21 日发布了第十个大版本。

#### 局部变量类型推断 (Local-Variable Type Inference)

新的语法将减少 Java 代码的冗长度，同时保持对静态类型安全性的承诺。局部变量类型推断主要是向 Java 语法中引入在其他语言（比如 C#、JavaScript）中很常见的保留类型名称 var。但需要特别注意的是：var 不是一个关键字，而是一个保留字。只要编译器可以推断此种类型，开发人员不再需要专门声明一个局部变量的类型，也就是可以随意定义变量而不必指定变量的类型。

局部变量类型推断示例:

```java
var list = new ArrayList<String>();
var stream = list.stream();
```

上面这段例子，在以前版本的 Java 语法中初始化列表的写法为：

```java
List<String> list = new ArrayList<String>();
Stream<String> stream = getStream();
```

Java 7 之后版本类型初始化示例:

```java
List<String> list = new LinkedList<>();
Stream<String> stream = getStream();
```

但这种 var 变量类型推断的使用也有局限性，仅局限于具有初始化器的局部变量、增强型 for 循环中的索引变量以及在传统 for 循环中声明的局部变量，而不能用于推断方法的参数类型，不能用于构造函数参数类型推断，不能用于推断方法返回类型，也不能用于字段类型推断，同时还不能用于捕获表达式（或任何其他类型的变量声明）。

#### 整合 JDK 代码仓库 (Consolidate the JDK Forest into a Single Repository)

在已发布的 Java 版本中，JDK 的整套代码根据不同功能已被分别存储在多个 Mercurial 存储库，这八个 Mercurial 存储库分别是：root、corba、hotspot、jaxp、jaxws、jdk、langtools、nashorn。

JDK 10 中将所有现有存储库合并到一个 Mercurial 存储库中。这种合并的一个次生效应是，单一的 Mercurial 存储库比现有的八个存储库要更容易地被镜像(作为一个 Git 存储库)，并且使得跨越相互依赖的变更集的存储库运行原子提交成为可能，从而简化开发和管理过程。

#### 统一的垃圾回收接口 (Garbage-Collector Interface)

在当前的 Java 结构中，组成垃圾回收器（GC）实现的组件分散在代码库的各个部分。尽管这些惯例对于使用 GC 计划的 JDK 开发者来说比较熟悉，但对新的开发人员来说，对于在哪里查找特定 GC 的源代码，或者实现一个新的垃圾收集器常常会感到困惑。更重要的是，随着 Java modules 的出现，我们希望在构建过程中排除不需要的 GC，但是当前 GC 接口的横向结构会给排除、定位问题带来困难。

为解决此问题，需要整合并清理 GC 接口，以便更容易地实现新的 GC，并更好地维护现有的 GC。Java 10 中，hotspot/gc 代码实现方面，引入一个干净的 GC 接口，改进不同 GC 源代码的隔离性，多个 GC 之间共享的实现细节代码应该存在于辅助类中。这种方式提供了足够的灵活性来实现全新 GC 接口，同时允许以混合搭配方式重复使用现有代码，并且能够保持代码更加干净、整洁，便于排查收集器问题。

#### 并行全垃圾回收器 G1 (Parallel Full GC for G1)

G1 垃圾回收器是 Java 9 中 Hotspot 的默认垃圾回收器，是以一种低延时的垃圾回收器来设计的，旨在避免进行 Full GC，但是当并发收集无法快速回收内存时，会触发垃圾回收器回退进行 Full GC。之前 Java 版本中的 G1 垃圾回收器执行 GC 时采用的是基于单线程标记扫描压缩算法（mark-sweep-compact）。为了最大限度地减少 Full GC 造成的应用停顿的影响，Java 10 中将为 G1 引入多线程并行 GC，同时会使用与年轻代回收和混合回收相同的并行工作线程数量，从而减少了 Full GC 的发生，以带来更好的性能提升、更大的吞吐量。

Java 10 中将采用并行化 mark-sweep-compact 算法，并使用与年轻代回收和混合回收相同数量的线程。具体并行 GC 线程数量可以通过：-XX：ParallelGCThreads 参数来调节，但这也会影响用于年轻代和混合收集的工作线程数。


<br/>
[参考]  
[Java 10 新特性介绍](https://www.ibm.com/developerworks/cn/java/the-new-features-of-Java-10/index.html)  
[What's New in JDK 10 - New Features and Enhancements](https://www.oracle.com/technetwork/java/javase/10-relnote-issues-4108729.html#NewFeature)  
[OpenJDK JDK 10](http://openjdk.java.net/projects/jdk/10/)  