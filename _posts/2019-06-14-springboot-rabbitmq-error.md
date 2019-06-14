---
layout: post
title: "Spring Boot 集成 RabbitMQ 遇到“An unexpected connection driver error occured”错误"
date: 2019-06-14 23:40:00 +0800
categories: Spring-Boot
tags: Spring-Boot RabbitMQ
author: Bobby
---

* content
{:toc}

今天玩 Spring Boot 集成 RabbitMQ 时，碰到一个 “An unexpected connection driver error occured” 错误。



很久之前就看过了 Spring Boot 集成 RabbitMQ 的视频了，但是一直没有引入到现有的工程中。今天本机安装好 RabbitMQ 后，决定引入现有的 Spring Boot 工程中，试玩一下。写了一个简单的 test case。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootRabbitMQTest {
	@Autowired
	private RabbitTemplate rabbitTemplate;

	@Test
	public void sendDirectMsg() {
		String exchange = "exchange.direct";
		String routingKey = "bobby.queue";

		Map<String, Object> msgObj = new HashMap<>();
		msgObj.put("msg", "这是一个direct 消息");
		msgObj.put("list", Arrays.asList("嘿嘿嘿", 123, true));
		
		rabbitTemplate.convertAndSend(exchange, routingKey, msgObj);
	}
}
```

看似很简单的一个例子，结果一运行就报出了下面的错误。

```text
INFO 8164 --- [           main] c.b.d.s.a.SpringBootRabbitMQTest         : Started SpringBootRabbitMQTest in 18.832 seconds (JVM running for 20.407)
ERROR 8164 --- [ 127.0.0.1:5672] c.r.c.i.ForgivingExceptionHandler        : An unexpected connection driver error occured

java.net.SocketException: Socket Closed
	at java.net.SocketInputStream.socketRead0(Native Method) ~[?:1.8.0_191]
	at java.net.SocketInputStream.socketRead(SocketInputStream.java:116) ~[?:1.8.0_191]
	at java.net.SocketInputStream.read(SocketInputStream.java:171) ~[?:1.8.0_191]
	at java.net.SocketInputStream.read(SocketInputStream.java:141) ~[?:1.8.0_191]
	at java.io.BufferedInputStream.fill(BufferedInputStream.java:246) ~[?:1.8.0_191]
	at java.io.BufferedInputStream.read(BufferedInputStream.java:265) ~[?:1.8.0_191]
	at java.io.DataInputStream.readUnsignedByte(DataInputStream.java:288) ~[?:1.8.0_191]
	at com.rabbitmq.client.impl.Frame.readFrom(Frame.java:91) ~[amqp-client-4.0.3.jar:4.0.3]
	at com.rabbitmq.client.impl.SocketFrameHandler.readFrame(SocketFrameHandler.java:164) ~[amqp-client-4.0.3.jar:4.0.3]
	at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:571) [amqp-client-4.0.3.jar:4.0.3]
	at java.lang.Thread.run(Thread.java:748) [?:1.8.0_191]

ERROR 8164 --- [:0:0:0:0:1:5672] c.r.c.i.ForgivingExceptionHandler        : An unexpected connection driver error occured

java.net.SocketException: Socket Closed
	at java.net.SocketInputStream.socketRead0(Native Method) ~[?:1.8.0_191]
	at java.net.SocketInputStream.socketRead(SocketInputStream.java:116) ~[?:1.8.0_191]
	at java.net.SocketInputStream.read(SocketInputStream.java:171) ~[?:1.8.0_191]
	at java.net.SocketInputStream.read(SocketInputStream.java:141) ~[?:1.8.0_191]
	at java.io.BufferedInputStream.fill(BufferedInputStream.java:246) ~[?:1.8.0_191]
	at java.io.BufferedInputStream.read(BufferedInputStream.java:265) ~[?:1.8.0_191]
	at java.io.DataInputStream.readUnsignedByte(DataInputStream.java:288) ~[?:1.8.0_191]
	at com.rabbitmq.client.impl.Frame.readFrom(Frame.java:91) ~[amqp-client-4.0.3.jar:4.0.3]
	at com.rabbitmq.client.impl.SocketFrameHandler.readFrame(SocketFrameHandler.java:164) ~[amqp-client-4.0.3.jar:4.0.3]
	at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:571) [amqp-client-4.0.3.jar:4.0.3]
	at java.lang.Thread.run(Thread.java:748) [?:1.8.0_191]
```

大概看一下是驱动错误，首先就想到是不是 Spring Boot 和 RabbitMQ 的版本不匹配引起的。我用的是 Spring Boot 1.5.9 Release 和 RabbitMQ 3.7.15。Spring Boot 少说也是一年前的版本了，而 RabbitMQ 的版本还是比较新的，才是近期发布的。

看了一下 Spring Boot 默认引入的 JAR，有个 spring-rabbit-1.7.4.RELEASE.jar。大概看了一下源码，但也没有发现 RabbitMQ 的具体版本。百度了一下，每个人遇到的问题也都不太一样。但是其中发现[一篇文章](https://blog.csdn.net/zht741322694/article/details/82801873)里面贴的错误信息中提到了 ‘spring-rabbit-1.7.4.RELEASE.jar’，于是看了一下，他是由于权限问题导致的。但是我用的默认 guest 用户是具有管理员权限的啊，再检查一下配置文件，发现原来刚才的配置还没有写完，用户名和密码为空。大写的尴尬！！！

```yaml
  rabbitmq:
    host: localhost
    port: 5672
    virtual-host: /
    username: 
    password: 
```

赶紧添加上用户名、密码，保存，再运行。终于看到一个绿条。

之所以想记录一下这个错误，是由于它不像我们平时连接数据库那样，如果用户名密码错误，会报类似 “Connection refused” 的错误。而它这里的 “connection driver error” 容易让人想到是由于驱动错误导致的。
