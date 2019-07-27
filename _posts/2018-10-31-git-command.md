---
layout: post
title: "Git常用命令"
date: 2018-10-31 22:40:00 +0800
categories: git
tags: git
author: Bobby
---

* content
{:toc}

整理出一些常用的git命令。看这份文档之前，要求您对git的基本概念要清楚，比如"暂存区", "工作区", "本地分支", "远程分支"，等等。



使用SSH Clone一个仓库，`git clone git@<domain>.com:<username>/<repository>.git` 或者 `git clone ssh://<username>@<domain>.com/<repository>.git`。 例如：`git clone git@github.com:JiaoJianbo/test.git`

使用HTTPS Clone一个仓库，`git clone https://<domain>.com:<username>/<repository>.git` 例如：`git clone https://github.com/JiaoJianbo/test.git`

使用 HTTPS 除了速度慢以外，还有个最大的麻烦是每次 clone 和 push 都必须输入口令，但是在某些只开放 HTTP(S) 端口的公司内部就无法使用 SSH 协议就只能用 HTTPS 方式。

Clone 指定branch，使用`-b`参数，`git clone -b <branch name> <git URL>`

Clone 指定tag，使用`--branch`参数，`git clone --branch <tag name> <git URL>`

将所有改动加入暂存区 `git add .`，执行前一定先使用 `git status` 查看一下。

**从暂存区恢复到工作区** `git reset HEAD <filename>`，对文件做的最新的所有改动都还在（包括已加入到暂存区的和加入暂存区后在工作区新做的修改）。

**撤销工作区的改动** `git checkout -- <filename>` 注意`--`后面有空格。其实就是让这个文件回到最近一次`git commit` 或者 `git add`时的状态，无论工作区是修改还是删除。但是前提是该文件曾经被`add`过或者`commit`过，一个未曾加入过版本控制的新文件则不起作用。

**撤销工作区和暂存区**中**所有文件**的改动 `git reset --hard HEAD`, 即将HEAD指到最近的一次commit。**注意**：那些已加入到暂存区还没有提交的改动也会被撤销，当然对未曾加入过版本控制的新文件则不影响。如果误操作了，不想撤销暂存区的改动，那么去搜索一下`git fsck --lost-found`的具体用法。

查看所有的commit记录 `git log` 或者 `git log --pretty=oneline`，还有 `git log --graph --pretty=oneline --abbrev-commit`

撤销commit的修改（但是没有push） `git reset HEAD^` 或者 `git reset --hard HEAD^`， *注：`HEAD^`是上一个版本，`HEAD^^`是上两个版本， `HEAD~10`是上10个版本*。撤销后已修改的内容还在，只是回退到工作区。

回退到某个版本 `git reset --hard <commit id>`， commit id 没必须要写全，一般写前五位就可以了，commit id 可以使用 `git log` 或者 `git reflog` 去查看。

删除某个文件的 git 记录以及磁盘上文件本身 `git rm <filename>`， 或者分两步，先手动删除磁盘上的文件，然后再`git add <filename>` 将 git 记录删除。但是直到 commit 才算正式生效。

删除 git 跟踪记录，但是想保留磁盘上的文件本身，使用 `git rm --cached`，下面有详细介绍。

查看某个文件由谁在什么时候做了什么改动 `git blame <filename>`

查看工作区中某个文件跟版本库中的差别 `git diff HEAD -- <filename>`。`git diff` 比较的是**工作区尚未暂存的**文件的改动，即比较的是工作区和暂存区之间的差异。如果之前没有暂存过，那比较的就是工作区跟版本库（最后一次 commit）之间的差异。因此，在执行完 add 操作后再执行 `git diff` 什么也没有。`git diff --cached` 或者 `git diff --staged` 比较的是**暂存区**和版本库中的差别。

创建并切换到新分支 `git checkout -b <branch-name>`, `git branch` 查看分支，`git branch <branch-name>`创建分支， `git checkout <branch-name>`切换分支

删除分支 `git branch -d <branch-name>`

拉取远端分支（本地不存在的） `git checkout -b <local-branch-name> origin/<remote-branch-name>`，如果拉取不成功，可以先尝试 `git fetch`

推送一个本地分支到远端 `git push origin <branch-name>`

推送一个本地分支（远端不存在的） `git push --set-upstream origin <branch-name>`

建立本地分支与远端分支的链接 `git branch --set-upstream-to=origin/<remote-branch-name> <local-branch-name>`

查看所有本地分支和远程分支的对应关系 `git branch -vv`

查看所有分支的当前状态 `git branch -av`

查看远程库的详细信息 `git remote -v`

多分支工作时注意以下场景：

场景一：

1. 在分支 A 修改，不 commit （可以使用 git add 加入暂存区）。
2. 此时在分支 A 的基础上，创建分支 B。之前在分支 A 上修改的内容也会带到分支 B。
3. 在分支 B 上接着修改，但仍然不提交。又切回分支 A。
4. 此时，在分支 B 做的修改，在分支 A 仍然可以看到。

场景二：
1. 在分支 A 修改，不 commit （可以使用 git add 加入暂存区）。
2. 此时在分支 A 的基础上，创建分支 B。之前在分支 A 上修改的内容也会带到分支 B。
3. 在分支 B 上接着修改，然后 commit。
4. 切回分支 A。此时分支 A 上的内容是最后一次 commit 的，切换到分支 B 之前做的修改全没了。

**结论**：工作区和暂存区，没有分支的概念，对所有分支是共享的。只有 commit 后，修改的内容才会进入具体分支版本库，并且清空工作区和暂存区。

多人合作时，每次要 push 时，注意先 pull， 再 merge， 再 push。

`git merge dev` 将 dev 分支合并到当前分支，或者在 dev 分支上执行 `git rebase <target-branch>` 也会将 dev 分支合并到目标分支上，如果有冲突，解决完冲突，再执行 `git rebase --continue`。**注意：两个 merge 和 rebase 执行的时候，所在当前分支的区别。还有，一定不要对一个公共的分支做 rebase 操作** rebase 操作可以把本地未 push 的分叉提交历史整理成直线。

舍弃 rebase `git rebase --abort`

2019-03-29 更新
---

前面上传项目时，将 eclipse 自动生成的 `.project` 和 `.classpath` 也都上传了。现在由 eclipse 换成 STS，结果这两种文件都发生了改变。
![git status](/assets/images/2019/03/2019-03-29_git-status.jpg)

想忽略这两种文件的改动，于是乎将 `.project` 和 `.classpath` 加入 `.gitignore` 文件，结果并没有起作用。又想到莫非是这些文件在子目录下，于是改为 `**/.project` 和 `**/.classpath`，但是仍然没有起作用。

最后，终于找到了一个解决办法。使用 `git rm --cached [filepath]` 命令。
![git rm cached](/assets/images/2019/03/2019-03-29_git-rm-cached.jpg)

同样，也删除 .classpath 之后，最终显示修改的文件就是我想要提交的了。
![git rm cached](/assets/images/2019/03/2019-03-29_git-status-final.jpg)

**参考**：  
[git rm与git rm --cached](https://www.cnblogs.com/toward-the-sun/p/6599656.html)  
[Git忽略提交规则 - .gitignore配置运维总结](https://www.cnblogs.com/kevingrace/p/5690241.html)


Git tag 操作
---

列出已有的 tag：`git tag`  
打一个 tag（默认打在最后一次 commit 上）：`git tag -a [tagName] -m "my comments"`  
给指定的某个 commit 号加tag：`git tag -a [tagName] 9fceb02 -m "my comments"`  
查看指定 tag 的详细信息：`git show [tagName]`  
推送单个本地 tag：`git push origin [tagName]`  
推送本地所有 tag：`git push origin --tags`  
删除本地 tag：`git tag -d [tagName]`  
删除远端 tag：`git tag push origin :refs/tags/[tagName]`  

Git stash 操作
---

**应用场景**：  
1. 当你现在正在分支 A 上开发，但是突然需要修复一个 bug，但是目前分支 A 上的改动你又不想丢弃或者提交（参考上面的场景一、二，如果工作区和暂存区的改动不提交或者丢弃，都会被带到其他分支，并将最终被提交）。那么此时就可以使用 `git stash` 将所有工作区和暂存区的改动存储起来，然后再切换到其他分支去修复 bug。

2. 由于疏忽，本应该在个人自己分支开发的内容，却在公共的 dev 分支上进行了开发，需要重新切回到自己分支上进行开发。可以用 `git stash` 将内容保存至堆栈中，切回到自己分支后，使用 `git stash pop` 或者 `git stash apply` 恢复已修改的内容即可。

将所有未提交的修改（工作区和暂存区）保存至堆栈中：`git stash`，此时工作区和暂存区清空  
为 git stash 添加一些注释：`git stash save <comments>`  
查看当前 stash 中的所有内容：`git stash list`  
将当前 stash 中的内容弹出，并应用到当前分支上: `git stash pop`, 该命令当前最近保存的（先进后出）内容恢复到工作区，并删除保存记录。  
`git stash apply` 将堆栈中的内容应用到当前目录，不同于 git stash pop，该命令不会将内容从堆栈中删除，也就说该命令能够将堆栈的内容多次应用到工作目录中，适应于多个分支的情况。  
`git stash drop <stash-name>` 从堆栈中移除某个指定的 stash 记录。  
`git stash clear` 清除堆栈中的所有内容。  
`git stash show -p stash@{0} | git apply -R` 取消之前所应用的 stash 的修改  
`git stash branch` 基于最近的 stash 创建分支。

**参考**：  
[3 Git 工具 - 储藏（Stashing）](https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E5%82%A8%E8%97%8F%EF%BC%88Stashing%EF%BC%89)  
[git stash详解](https://blog.csdn.net/stone_yw/article/details/80795669) 
