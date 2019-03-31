---
layout: post
title: Git 基础学习（二）：基本工作流
category: 学习笔记
tags: Git
mathjax: false
issue_id: 6
---

* content
{:toc}

本文是廖雪峰的 Git 教程 [1] 的学习笔记的第二部分，介绍了 Git 的基本操作，主要包括 Git 的初始化、配置以及基本工作流。

<!--more-->

廖的教程的内容比较浅，但是足够让人对 Git 有一个基本的了解。作为内容深度的补充，我参考了包括《Pro Git》[2] 在内的一些其他资料，对部分细节问题做了更深入的探讨。

在阅读本文时，可以参考廖的教程中相关内容的示例，以加深理解。

## 初始化与配置

### git init

使用 Git 管理项目，第一步就是创建一个仓库（版本库，repository）。

这一操作其实十分简单：在需要使用 Git 管理的目录下，执行 `git init` 就可以了。该命令会使得该目录下生成一个隐藏的 `.git` 目录，这个 `.git` 目录下存放的就是 Git 用于版本管理的数据。

### git config

创建了仓库后，如果有必要，还需要配置好 Git 会用到的相关参数。

命令 `git config [<options>]` 用于配置 Git 的环境变量，直接输入 `git config` 会显示出所有可用的参数。

需要指出的是，Git 有三个级别的配置文件，它们的作用域是不同的。

- `--system` 对系统中的所有用户生效
- `--global` 对当前用户的所有仓库生效
- `--local` 仅对当前仓库生效

对于一个仓库，如果在不同级别的配置文件里存在同名变量，那么哪个配置文件里的值会被使用呢？

事实上，作用域更小的配置文件会起作用。对于学过编程的同学来说，这件事十分自然。如果你没有学过编程，那么不妨设想大作用域的配置相当于一般规则，小作用域的配置相当于特例，特例所定义的规则自然优先级更高。

下面给出一个最常用的配置实例：为当前用户的所有仓库统一配置提交者的姓名和邮箱。

```bash
$ git config --global user.name "name"              # 配置姓名
$ git config --global user.email "email@email.com"  # 配置邮箱

$ git config --list                                 # 查看所有配置信息
```

## 基本工作流

在一个已经配置好的仓库里，我们进行的最多的工作就是“修改文件-记录版本”的循环，这也是 Git 最基础的使用方式。掌握好这一组基本操作，我们就拥有了使用 Git 来帮助管理项目的能力了。

### git status & git diff

在修改完文件后，准备记录版本之前，很可能由于这次修改在时间上的跨度，我们已经不能确认自己具体修改了哪些文件的哪些内容。如果两眼一闭，不加检查，很可能会提交一些意料之外的改动。因此，虽然这步操作是可选项，但是我强烈建议在进行版本相关的操作之前，一定要 review 各文件的改动内容。

在 Git 里，通过 `git status` 可以查看项目的改动状态。运行该命令，将显示项目中存在改动的文件，以及这些改动所处的状态：已被添加到 Index，或者仍在 Working Directory。

进一步，可以通过 `git diff` 来显示一个文件具体的改动内容，根据官方文档 [3]，

- `git diff` 比较 Working Directory 和 Index，显示未添加到 Index 的改动；
- `git diff --staged/cached` 比较 Index 和 HEAD，显示在 commit 时会提交的改动；
- `git diff [options] [<commit>] [--] [<path>…​]` 比较 Working Directory 相对 `<commit>` 的改动。

### git add

在 Working Directory 里，有复数个文件被改动是极为正常的情况。然而，并非所有的改动我们都希望提交到下一个版本，比如说本地的临时修改，或者至少是暂时在这个版本不希望提交的内容。因此，我们要将需要提交的改动从 Working Directory 给添加到 Index 中。

在 Git 里，通过 `git add [<pathspec>]` 可以把某一路径下的改动从 Working Directory 添加到 Index 中。如果确认所有的改动都是需要的，也可以一次性添加当前目录或者当前项目中的所有改动。从 Stack Overflow [4] 中了解到，在 Git 2.0 中，

> - `git add .` and `git add -A .` add new/modified/deleted files in the current directory
> - `git add --ignore-removal .` adds new/modified files in the current directory
> - `git add -u .` adds modified/deleted files in the current directory
> - without the dot, add all files in the project regardless of the current directory

需要额外注意的是，如果你把一个文件 `git add` 后，在这个文件上再次做了修改，那么新的改动是不包含在 Index 中的。你需要再次 add 这个文件，才能把新的改动也添加到 Index 里。

### git commit

在经过若干次 add 后，即可通过 `git commit` 将 Index 中的改动提交到版本库，固定为当前分支的下一个版本。

在进行 commit 操作时，需要提供 message 以说明此次 commit。commit message 的填写十分重要，在以后浏览版本历史的时候，这个 message 可以极大方便我们了解每次 commit 的内容，而不是非得去逐项查看更改。这相当于代码的注释，甚至更为重要。

下面给出一个提供 message 的 commit 实例。

```bash
$ git commit -m "message about this commit"
```

### git rm

有些存在于版本库中的文件，可能过了若干版本后，我们已经不再需要了。如果想要把它从版本库中移除，我们可以先在系统中把文件删掉，然后把这个修改 add 到 Index。

Git 也提供了另外一个命令专门用来删除文件，这个命令就是 `git rm`。它会使文件的状态变成 untracked 并提交到 Index。根据 Pro Git 第 2.2 节 [2]，

- `git rm` 会同时删除硬盘上的文件；
- `git rm --cached` 会保留硬盘上的文件；
- 如果文件已被添加到 Index，那么不能直接通过 `git rm` 移除，需要使用 `git rm -f`。

## 参考资料

[1] [Git教程 - 廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

[2] [Pro Git. 2nd Edition. Scott Chacon, Ben Straub.](https://git-scm.com/book/en/v2)

[3] [Git - git-diff Documentation](https://git-scm.com/docs/git-diff)

[4] [git add - Difference between "git add -A" and "git add ." - Stack Overflow](https://stackoverflow.com/questions/572549/difference-between-git-add-a-and-git-add)
