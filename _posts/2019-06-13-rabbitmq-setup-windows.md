---
layout: post
title: "Windows 下 RabbitMQ 的安装及初始配置"
date: 2019-06-13 23:40:00 +0800
categories: MQ
tags: RabbitMQ
author: Bobby
---

* content
{:toc}

记录一下 RabbitMQ 在 Windows 下的安装过程和一些初始配置。



### Windows 下 RabbitMQ 安装及初始配置

RabbitMQ 是什么？这里就不多说了，只引用官网上的一句话 “RabbitMQ 是被部署最广泛的开源消息代理 （RabbitMQ is the most widely deployed open source message broker）”。

#### 准备安装包

由于 RabbitMQ 是使用 Erlang 语言开发的的，所以 RabbitMQ 的运行需要依赖 Erlang 环境。RabbitMQ 和 Erlang 版本兼容关系查看[这里](https://www.rabbitmq.com/which-erlang.html)。官网部分截图如下：

![RabbitMQ and Erlang/OTP Compatibility Matrix](/assets/images/2019/06/rabbitmq-erlang-compab-matrix.jpg)

根据上面兼容关系，我选择 RabbitMQ 3.7.15 和 Erlang/OTP 21.3。首先到[这里](http://www.erlang.org/downloads)下载 Erlang/OTP 安装包，然后到 RabbitMQ [官网](https://www.rabbitmq.com/)去下载 RabbitMQ 安装包。

![RabbitMQ and Erlang/OTP Packages](/assets/images/2019/06/rabbitmq-erlang-pack.jpg)

#### 开始安装

Erlang 的安装没有特别之处，除过选择安装目录以外，基本可以一直 “Next” 直到结束。但是注意一点，RabbitMQ 官网提到，Erlang 必须以管理员身份运行安装。否则，会缺少 RabbitMQ installer 所需的注册表信息。

接下来开始安装 RabbitMQ，也是除过选择安装目录以外，基本可以一直 “Next” 直到结束。

![RabbitMQ setup 1](/assets/images/2019/06/rabbitmq-setup-1.jpg)

![RabbitMQ setup 2](/assets/images/2019/06/rabbitmq-setup-2.jpg)

中间出现防火墙拦截警告，勾选我的网络状况，选择“允许访问(A)”即可。

![Windows Defender](/assets/images/2019/06/windows-defender.jpg)

#### 初始配置

首先进入 RabbitMQ 安装目录的 sbin 目录下，或者直接点击刚才安装的快捷方式。

![RabbitMQ Cli 1](/assets/images/2019/06/rabbitmq-cli-prompt.jpg)

![RabbitMQ Cli 2](/assets/images/2019/06/rabbitmq-cli-prompt-2.jpg)

输入 `rabbitmq-plugins enable rabbitmq_management` 启用管理插件。

![RabbitMQ Enable Management Plugin](/assets/images/2019/06/rabbitmq-enable-mgmt.jpg)

输入 `net stop RabbitMQ && net start RabbitMQ` 重启 RabbitMQ 服务，注意要以管理员身份运行。

![RabbitMQ Restart](/assets/images/2019/06/rabbitmq-restart.jpg)

此时，可以通过浏览器访问 RabbitMQ 的控制台了，默认地址 http://localhost:15672/，默认用户名和密码都是 `guest`。

![RabbitMQ management](/assets/images/2019/06/rabbitmq-mgmt-console.jpg)

接下来，继续创建用户。

##### 创建用户分配角色

输入 `rabbitmqctl list_users`, 查看已有用户列表。目前仅有一个 guest 用户，是管理员角色。

![RabbitMQ list users](/assets/images/2019/06/rabbitmq-list-user.jpg)

使用 `rabbitmqctl add_user <username> <password>`, 创建一个名为 username 的新用户。

![RabbitMQ add user](/assets/images/2019/06/rabbitmq-adduser.jpg)

添加了一个新用户 Bobby，但是还没有任何角色。RabbitMQ 主要有几种角色（administrator, monitoring, policymaker, management, others），详细信息下去再去了解。现在设置新增的用户 Bobby 为管理员角色。输入 `rabbitmqctl set_user_tags Bobby administrator`。

![RabbitMQ set user tags](/assets/images/2019/06/rabbitmq-set-tags.jpg)

其实， set_user_tags 还可以同时设置多个 tag， 如 `rabbitmqctl set_user_tags Bobby administrator monitoring`。

后面，还有更改密码和删除密码的操作，如修改默认用户 guest 的默认密码，或者直接删除默认的 guest 用户。

更改密码：`rabbitmqctl change_password <username> <newPassword>`

删除用户：`rabbitmqctl delete_user <username>`

##### 权限设置

这里的权限指的是对 exchange, queue 的操作权限，包括配置权限，读写权限。

输入 `rabbitmqctl list_user_permissions guest` 查看 guest 用户的权限。

![RabbitMQ set user tags](/assets/images/2019/06/rabbitmq-list-permi.jpg)

设置权限的命令形如 “rabbitmqctl set_permissions -p VHostPath username ConfigPerm WritePerm ReadPerm”，如给 Bobby 用户设置权限 `rabbitmqctl set_permissions -p / Bobby ".*" ".*" ".*"`，这样外部程序才能通过 Bobby 用户远程访问 RabbitMQ。

![RabbitMQ set user tags](/assets/images/2019/06/rabbitmq-set-perm.jpg)

查看所有用户（指定的 VirtualHost）的权限: `rabbitmqctl list_permissions [-p VHostPath]`

清除用户的权限信息：`rabbitmqctl clear_permissions [-p VHostPath] <username>`

至此，RabbitMQ 的安装和基本设置就可以结束了。下面就可以开始使用 RabbitMQ 了，可以通过管理界面设定 exchange, queue 还有 binding 信息。还可以直接通过程序的方式操作 MQ。对了，RabbitMQ 的默认端口是 5672。

### 参考资料：
[RabbitMQ Erlang Version Requirements](https://www.rabbitmq.com/which-erlang.html)  
[RabbitMQ Installing on Windows](https://www.rabbitmq.com/install-windows.html)  
[RabbitMQ Configuration](https://www.rabbitmq.com/configure.html)  
[windows下 安装 rabbitMQ 及操作常用命令](https://www.cnblogs.com/ericli-ericli/p/5902270.html)  
