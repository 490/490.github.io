---
title: Git入门
date: 2019-03-21 14:14:07
tags: 系统
---
# Git简介

  Git 其他版本管理系统的主要差别：对待数据的方式。Git采用的是直接记录快照的方式，而非差异比较。
 大部分版本控制系统（CVS、Subversion、Perforce、Bazaar 等等）都是以文件变更列表的方式存储信息，这类系统将它们保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异。
具体原理如下图所示，理解起来其实很简单，每个我们对提交更新一个文件之后，系统记录都会记录这个文件做了哪些更新，以增量符号Δ(Delta)表示。下图来源于Git官网。

<!--more-->

![image](http://490.github.io/images/20190321_143121.png)

**我们怎样才能得到一个文件的最终版本呢？**

很简单，高中数学的基本知识，我们只需要将这些原文件和这些增加进行相加就行了。

**这种方式有什么问题呢？**

比如我们的增量特别特别多的话，如果我们要得到最终的文件是不是会耗费时间和性能。

Git 不按照以上方式对待或保存数据。 反之，Git 更像是把数据看作是对小型文件系统的一组快照。 每次你提交更新，或在 Git 中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。 为了高效，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个 **快照流**。下图来源于Git官网。

![image](http://490.github.io/images/20190321_143144.png)

 Git 的三种状态

Git 有三种状态，你的文件可能处于其中之一：

•**已提交（committed）**：数据已经安全的保存在本地数据库中。•**已修改（modified）**：已修改表示修改了文件，但还没保存到数据库中。•**已暂存（staged）**：表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

由此引入 Git 项目的三个工作区域的概念：**Git 仓库(.git directoty) ** 、**工作目录(Working Directory)** 以及 **暂存区域(Staging Area)** 。下图来源于Git官网。

![image](http://490.github.io/images/20190321_143256.png)

**基本的 Git 工作流程如下：**

- 在工作目录中修改文件。
- 暂存文件，将文件的快照放入暂存区域。
- 提交更新，找到暂存区域的文件，将快照永久性存储到 Git 仓库目录。


# Git 使用快速入门

## 获取 Git 仓库

有两种取得 Git 项目仓库的方法。

•在现有目录中初始化仓库: 进入项目目录运行 `git init` 命令,该命令将创建一个名为 `.git` 的子目录。•从一个服务器克隆一个现有的 Git 仓库: `git clone [url]` 自定义本地仓库的名字: `git clone [url]` directoryname

## 记录每次更新到仓库

•**检测当前文件状态** : `git status`•**提出更改（把它们添加到暂存区**）：`git add filename` (针对特定文件)、`git add *`(所有文件)、`git add *.txt`（支持通配符，所有 .txt 文件）•**忽略文件**：`.gitignore` 文件•**提交更新:** `git commit -m "代码提交信息"` （每次准备提交前，先用 `git status` 看下，是不是都已暂存起来了， 然后再运行提交命令 `git commit`）•**跳过使用暂存区域更新的方式** : `git commit -a -m "代码提交信息"`。 `git commit` 加上 `-a` 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git add` 步骤。•**移除文件** ：`git rm filename` （从暂存区域移除，然后提交。）•**对文件重命名** ：`git mv README.md README`(这个命令相当于`mv README.md README`、`git rm README.md`、`git add README` 这三条命令的集合)

## 推送改动到远程仓库

*   如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：·`git remote add origin <server>` ,比如我们要让本地的一个仓库和 Github 上创建的一个仓库关联可以这样`git remote add origin https://github.com/Snailclimb/test.git`将

*   将这些改动提交到远端仓库：`git push origin master` (可以把 **master** 换成你想要推送的任何分支)

如此你就能够将你的改动推送到所添加的服务器上去了。

## 远程仓库的移除与重命名

•将 test 重命名位 test1：`git remote rename test test1`•移除远程仓库 test1:`git remote rm test1`

## 查看提交历史

在提交了若干更新，又或者克隆了某个项目之后，你也许想回顾下提交历史。 完成这个任务最简单而又有效的工具是 `git log` 命令。`git log` 会按提交时间列出所有的更新，最近的更新排在最上面。

**可以添加一些参数来查看自己希望看到的内容：**

只看某个人的提交记录：

```
git log --author=bob
```

## 撤销操作

有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。 此时，可以运行带有 `--amend` 选项的提交命令尝试重新提交：

```
git commit --amend
```

取消暂存的文件

```
git reset filename
```

撤消对文件的修改:

```
git checkout -- filename
```

假如你想丢弃你在本地的所有改动与提交，可以到服务器上获取最新的版本历史，并将你本地主分支指向它：

```
git fetch origin
```

## 分支

分支是用来将特性开发绝缘开来的。在你创建仓库的时候，**master** 是“默认的”分支。在其他分支上进行开发，完成后再将它们合并到主分支上。

我们通常在开发新功能、修复一个紧急 bug 等等时候会选择创建分支。单分支开发好还是多分支开发好，还是要看具体场景来说。

创建一个名字叫做 test 的分支

```
git branch test
```

切换当前分支到 test（当你切换分支的时候，Git 会重置你的工作目录，使其看起来像回到了你在那个分支上最后一次提交的样子。 Git 会自动添加、删除、修改文件以确保此时你的工作目录和这个分支最后一次提交时的样子一模一样）

```
git checkout test
```

![](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9TySunhaicSGCZFlichCE4c6iasUACWIULSx43rj0PkiaCveb3HfCFytfunS2ozZBmr3TqjO7qxibOf7iaUw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

你也可以直接这样创建分支并切换过去(上面两条命令的合写)

```
git checkout -b feature_x
```

切换到主分支

```
git checkout master
```

合并分支(可能会有冲突)

```
 git merge test
```

把新建的分支删掉

```
git branch -d feature_x
```

将分支推送到远端仓库（推送成功后其他人可见）：

```
git push origin 
```

