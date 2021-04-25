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

docker run hello-world






