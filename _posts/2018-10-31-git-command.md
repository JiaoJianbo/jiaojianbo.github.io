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

整理出一些常用的git命令。看这份文档之前，要求您对git的基本概念要清楚，比如"暂存区", "工作区", "本地分支", "远程分支"，等等。



**Clone一个仓库**

使用SSH，`git clone git@<domain>.com:<username>/<repository>.git` 

或者 `git clone ssh://<username>@<domain>.com/<repository>.git`。例如：`git clone git@github.com:JiaoJianbo/test.git`

使用HTTPS，`git clone https://<domain>.com:<username>/<repository>.git` 例如：`git clone https://github.com/JiaoJianbo/test.git`

将所有改动加入暂存区 `git add .`，执行前一定先使用 `git status` 查看一下。

从暂存区恢复到工作区 `git reset HEAD <filename>`，所有的改动还在。

还原**工作区**中的某个文件改动 `git reset HEAD <filename>`，从暂存区放回到工作区

`git checkout -- <filename>` 注意`--`后面有空格，丢弃工作区的改动。其实就是让这个文件回到最近一次`git commit` 或 `git add`时的状态，无论工作区是修改还是删除。

还原**工作区**中所有文件的改动 `git reset --hard HEAD`

查看所有的commit记录 `git log` 或者 `git log --pretty=oneline`，还有 `git log --graph --pretty=oneline --abbrev-commit`

撤销commit的修改（但是没有push） `git reset HEAD^` 或者 `git reset --hard HEAD^`， *注：`HEAD^`是上一个版本，`HEAD^^`是上两个版本， `HEAD~10`是上10个版本*

回退到某个版本 `git reset --hard <commit id>`， commit id 没必须要写全，一般写前五位就可以了，commit id 可以使用 `git log` 或者 `git reflog` 去查看。

删除某个文件 `git rm <filename>` 或者 手动删除，然后再`git add <filename>`

查看某个文件由谁在什么时候做了什么改动 `git blame <filename>`

查看工作区中某个文件跟版本库中的差别 `git diff HEAD -- <filename>`

创建并切换到新分支 `git checkout -b <branch-name>`, `git branch` 查看分支，`git branch <branch-name>`创建分支， `git checkout <branch-name>`切换分支

删除分支 `git branch -d <branch-name>`

拉取远端分支（本地不存在的） `git checkout -b <local-branch-name> origin/<remote-branch-name>`，如果拉取不成功，可以先尝试 `git fetch`

推送一个本地分支（远端不存在的） `git push --set-upstream origin <branch-name>` 

建立本地分支与远端分支的链接 `git branch --set-upstream-to=origin/<remote-branch-name> <local-branch-name>`

查看所有本地分支和远程分支的对应关系 `git branch -vv`

查看所有分支的当前状态 `git branch -av`

查看远程库的详细信息 `git remote -v`

多人合作时，每次要push时，注意先 pull 再 merge 再push

`git merge dev` 将dev分支合并到当前分支，或者在dev分支上执行 `git rebase <target-branch>` 也会将dev分支合并到目标分支上，如果有冲突，解决完冲突，再执行 `git rebase --continue`。**注意：两个merge和rebase执行的时候，所在当前分支的区别。还有，一定不要对一个提交到公共仓库的分支做rebase操作。**

舍弃rebase `git rebase --abort`
