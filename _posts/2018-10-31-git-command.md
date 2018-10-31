---
layout: post
title:  "Git常用命令"
date:   2018-10-31 22:40:00 +0800
categories: git
tags: git
author: Bobby
---

* content
{:toc}

整理出一些常用的git命令。



**Clone一个仓库**

使用SSH，`git clone git@<domain>.com:<username>/<repository>.git` 

或者 `git clone ssh://<username>@<domain>.com/<repository>.git`。例如：`git clone git@github.com:JiaoJianbo/test.git`

使用HTTPS，`git clone https://<domain>.com:<username>/<repository>.git` 例如：`git clone https://github.com/JiaoJianbo/test.git`

将所有改动加入暂存区 `git add .`，执行前一定先使用 `git status` 查看一下。

从暂存区恢复到工作区 `git reset HEAD <filename>`，所有的改动还在。

还原**工作区**中的某个文件改动 `git reset HEAD <filename>` 或者 `git checkout -- <filename>` 注意`--`后面有空格，其实就是让这个文件回到最近一次`git commit`或`git add`时的状态，无论工作区是修改还是删除。

还原**工作区**中所有文件的改动 `git reset --hard HEAD`

撤销commit的修改（但是没有push） `git reset HEAD^`



