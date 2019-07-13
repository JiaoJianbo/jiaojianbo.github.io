---
layout: post
title: "非管理员账号设置 Windows 10 环境变量"
date: 2019-07-13 11:40:00 +0800
categories: Windows
tags: Windows
author: Bobby
---

* content
{:toc}

这里主要说的是**非管理员账号**，如何设置自己的环境变量。



在公司使用的电脑是没有管理员权限的，而作为一个开发人员难免有时候需要手动配置环境变量。经过实践大概有以下几种方式可行：

### 1. 运行命令

这种方式忘记以前是在哪看到的。即按 Win + R，打开运行，输入 `rundll32.exe sysdm.cpl,EditEnvironmentVariables`，回车。便会弹出配置用户环境变量的界面。

![Run command](/assets/images/2019/07/windows-run-evn-config.jpg)

![Control Panel](/assets/images/2019/07/windows-env-config.jpg)

### 2. 在用户账号界面配置

按 Win + R 打开运行，输入 `control`，回车。打开控制面板，进入“用户帐户”。

![User Account](/assets/images/2019/07/windows-cp-useracc.jpg)

可以看到，这里修改环境变量是不需要管理员权限的。

![User Account Env](/assets/images/2019/07/windows-account-evn.jpg)

### 3. 使用 SETX 命令

SETX 不同于 SET 命令，创建或修改的变量将在以后的命令窗口可用，是永久有效的。详细用法可以使用命令 `setx /?` 查看帮助信息。默认情况下写入的是用户变量，要想写系统环境变量，加参数 /M。而我们只能设置用户变量，就使用最简单的格式 `SETX [变量名] [变量值]` 即可，变量值最好用英文双引号引住。例如：

![Windows SETX](/assets/images/2019/07/windows-setx.jpg)

![Windows SETX](/assets/images/2019/07/windows-setx-env.jpg)

### 参考资料：
[Win10环境变量怎么设置？在Windows 10中创建环境变量的3种方法(详细)](https://www.jb51.net/os/win10/663281.html)  