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

**从暂存区恢复到工作区** `git reset HEAD <filename>`，对文件做的最新的所有改动都还在（包括已加入到暂存区的和加入暂存区后在工作区新做的修改）。

**撤销工作区的改动** `git checkout -- <filename>` 注意`--`后面有空格。其实就是让这个文件回到最近一次`git commit` 或者 `git add`时的状态，无论工作区是修改还是删除。但是前提是该文件曾经被`add`过或者`commit`过，一个未曾加入过版本控制的新文件则不起作用。

**撤销工作区和暂存区**中**所有文件**的改动 `git reset --hard HEAD`, 即将HEAD指到最近的一次commit。**注意**：那些已加入到暂存区还没有提交的改动也会被撤销，当然对未曾加入过版本控制的新文件则不影响。如果误操作了，不想撤销暂存区的改动，那么去搜索一下`git fsck --lost-found`的具体用法。

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

`git merge dev` 将dev分支合并到当前分支，或者在dev分支上执行 `git rebase <target-branch>` 也会将dev分支合并到目标分支上，如果有冲突，解决完冲突，再执行 `git rebase --continue`。**注意：两个merge和rebase执行的时候，所在当前分支的区别。还有，一定不要对一个公共的分支做rebase操作** rebase操作可以把本地未push的分叉提交历史整理成直线。

舍弃rebase `git rebase --abort`
