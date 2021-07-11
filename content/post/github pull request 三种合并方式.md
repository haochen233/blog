---
title: "Github Pull Request 三种合并方式"
date: 2021-07-11T16:33:51+08:00
draft: true
author: haochen
categories:
- git
tags:
- "github"
- "pull reuqest"
- "rebase"
---

### 前言

最近呢在github上开源了一个golang实现的[socks5](https://github.com/haochen233/socks5)协议的仓库（PS:如果觉得不错点个 star呢，谢谢）。其中关于pull request（下文就简称**pr**）合并这一块，准备使用`git rebase` 来得到一条没有分叉的提交历史。

### rebase合并冲突的问题

起初，正常使用**github rebase and merge**来合并**pr**，并没有注意到异常情况。但后来随着其他开发者的加入，合并的**pr**越来越多；突然在一次**pr**合并时发现了其中有已经合入过的**commits**，并且**rebase** 提示有冲突。  

然后就发现了**github rebase** 与 ·git rebase·的 不同之处。**github rebase** 会把pr中的每个提交的提交者都修改为 **author + reviewer **，且重新生成**commit id**。

例，请注意**pr**中的**commit id**，和**rebase** 合并后的 **commit id**

合并前：

![](/img/post/github-pull-request-三种合并方式/pre-pr.png)

合并后：

![](/img/post/github-pull-request-三种合并方式/pr-commit.png)

使用 **github rebase and merge** 会修改 **pr**中commits的作者信息和commit id。

### 造成提交历史混乱

在这个**pr**合并后，如果这个开发者不基于最新的上游分支（**upstream**）分叉新创建分支，而是继续且长期使用本地分支；如果要更新当前分支，那么这个开发者必须使用 `git rebase` 来变基到**upstream**分支上。

为什么不使用`git merge`呢？如果使用`git merge`命令就会出现重复的提交历史，因为**upstream** 分支上是**commit id**是已经变化的提交（虽然提交的内容是一模一样，但还是会被人为是不同的两次提交）。

使用`git merge`，如下：

![](/img/post/github-pull-request-三种合并方式/merge.png) 

可以看到重复的提交。假如此开发者提交一次后，在推送到远端，并创建**pr**；该**pr**中原本应该只有一次提交，但是却多出来2次重复的提交。这就导致了历史的混乱。如下，红框中为重复的提交：

![](/img/post/github-pull-request-三种合并方式/dup.png) 

而使用`git rebase`，`git` 可以检测到重复的提交，即你本地的提交与远端**github rebase**后的提交是一回事（只是**github rebase** 更改了你的**commit id**），故在本地使用`git rebase`可以直接将本地分支更新为upstream上最新的（本地的commit id等信息被upstream 上的直接覆盖）。



此例中，虽然**pr**中显示有4个提交，但是实际有影响的只有最新的那一个提交（重复的提交只会影响历史，并不影响代码内容，因为重复提交的更改已经合并过了），但是将重复的提交历史合入仓库中，势必会影响代码仓库的历史追溯（可以使用**squash and merge** 来压缩这一次**pr**）。但是，开发者打开**pr**查看历史会感到迷惑，所以应该要避免这种情况。



### github上其余两种合并方式

#### merge pull request
commit id 等都不变，且会多生成一个merge pull request的提交。


#### squash and merge
将一个pr中的所有commit压缩为一个commit再合并。假如原本pr中有5次提交，压缩并合并后只会有一个特定格式的commit。

格式如下：															

| 提交数   | 摘要                                     | 描述                                   |
| -------- | ---------------------------------------- | -------------------------------------- |
| 一个提交 | 单个提交的提交消息标题，后接拉取请求编号 | 单个提交的提交消息正文                 |
| 多个提交 | 拉取请求标题，后接拉取请求编号           | 按日期顺序列出所有被压缩提交的提交消息 |



### 结论



一定要特别注意

- 作为代码审查者一定要认真审视pr所包含的内容及带来的影响。

- 作为开发者，最好不要长期使用一个分支；因为在更新本地分支时，不论你用**merge or rebase** 都会带来新的重复的提交（以前的提交都被压缩为一个新的提交），如果继续用该分支新建**pr**的话极有可能引入重复的提交。
- 在一次**pr**合并后就删除来源分支。