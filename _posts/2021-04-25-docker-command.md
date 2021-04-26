---
layout: post
title: "Podman 或 Docker 常用命令"
date: 2021-04-25 11:00:00 +0800
categories: Docker
tags: Docker
author: Bobby
---

* content
{:toc}

主要记录一些常用的 Docker 命令。




## Build and Ship any Application Anywhere.

podman login [repository_url]

podman pull [image]

podman images

mkdir -p /home/tom/container/journey

podman run -dit --name=journey-server -v /home/tom/container/logfile:/var/log/journey:Z [image]

podman ps -a

-------------

docker images

docker images tomcat

docker run hello-world

`docker run [OPTIONS] IMAGE [COMMAND] [ARG ...]`

OPTIONS 说明：有些是一个减号，有的是两个减号。

--name="Container Name": 指定容器名。

-d: 后台运行容器，并返回容器 ID，也即启动守护式容器。

-i: 以交互模式运行容器，通常与 -t 同时使用

-t: 为容器重新分配一个伪输入终端

-P: 随机端口映射

-p: 指定端口映射，通常有如下四种：

    ip:hostPort:containerPort

    ip::containerPort
  
    hostPort:containerPort
  
    containerPort

两种方式退出容器：
1. exit
2. ctrl + P + Q

docker run -it -p 8888:8080 tomcat

docker run -it -P tomcat

docker ps 中会显示端口映射信息

docker ps -l

docker ps -n 3

`docker exec -it <CONTAINER-ID> /bin/bash`

cd /usr/local/tomcat/webapps

rm -rf docs

提交一个容器的副本，使之成为一个新的镜像。

docker commit -m="comments" -a="author" Container-ID 要创建的镜像名:[tag name]

docker commit -m="tomcat without docs" -a="Bobby" 499cd373a543 com/bravo/tomcat2:1.0.0

docker images 便可以查看新修改的镜像

运行新的镜像：

docker run -it -p 7777:8080 com/bravo/tomcat2:1.0.0

------

### Dockerfile

1. 手动编写一个 dockerfile 文件，当然，必须要符合 file 的规范。
2. 有这个文件后，使用 `docker build -t` 命令执行，获得一个自定义的镜像。
3. 使用 `docker run` 命令执行这个镜像。

去 docker Hub 查看已有镜像的 Dockerfile.

Dockerfile 内容的基础知识：

1. 每条保留字指令都必须为大写字母且后面要跟至少一个参数。
2. 指令按照从上到下，顺序执行。
3. 注释使用 #
4. 每条指令都会创建一个新的镜像层，并对镜像进行提交。

Dockerfile 执行的大致流程：

1. docker 从基础镜像运行一个容器。
2. 执行一条指令，并对容器做出修改。
3. 执行类似 docker commit 的操作，提交一个新的镜像层。
4. docker 再基于刚提交的镜像，运行一个新容器。
5. 执行 dockerfile 中的下一条指令，直到所有指令都执行完成。

Dockerfile 保留字指令：

1. FROM:

2. MAINTAINER

3. RUN

4. EXPOSE

5. WORKDIR

6. ENV

7. ADD:

8. COPY

9. VOLUME

10. CMD

11. ENTRYPOINT

12. ONBUILD
......


docker history IMAGE-ID

