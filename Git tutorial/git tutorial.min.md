# Git入门指南(简明版)

## 概述

Git是一个版本控制工具，十分流行十分屌，下面讲述一些常用命令和一些好习惯。

## Clone命令

```shell
$ git clone <git address>
```

`clone`命令能从远端服务器下载指定的仓库到本地，因为Git在设计之初就认为本地和远端的仓库是不同步的，所以Git十分适合团队开发。

## branch命令

```shell
$ git branch
* master
* another-branch
```

`branch`命令可以对分支进行操作，分支是Git的精髓，任何修改是基于分支之上的。

### 列出所有分支

```shell
$ git branch
* master
* another-branch
```

不加参数调用`branch`可以列出本地所有分支，`-r`分支可以列出所有远端的分支，`-a`可以列出所有本地和远端的分支。

**注意：**这里列出的远端分支并不是当时远端仓库实际上的情况，这是在前一次同步时远端仓库的情况，这也是Git分离远端和本地仓库所产生的，这保证了你的修改永远基于一个之前存在的远端分支。

### 创建分支

```shell
$ git branch <new-branch-name>
```

在命令后面直接加上新的分支名，即可创建新分支。

**注意：**创建新分支之后并不会切换过去。

### 删除分支

```shell
$ git branch -d <branch-name>
```

在命令后面加`-d`可以删除指定分支

## checkout命令

```shell
$ git checkout <branch-name>
```

`checkout`命令可以切换当前工作的分支。可以使用`-b <new-branch-name>`来创建并切换到新的分支。

**注意：**切换分支时会切换到目标分支最新的版本上，例如在分支A上创建了分支B，并对分支B进行了一系列修改，这些修改已经提交（commit），那么切换回分支A时仿佛丢失这些提交，又回到了当时创建分支B的状态，不过不用担心，当你切换到分支B时，这些提交又回来了，这说明了Git保存了所有提交的状态，无论你切换到哪个分支，都能一瞬间回到分支最新的状态。

## status命令

```shell
$ git status
尚未暂存以备提交的变更：
  （使用 "git add <file>..." 更新要提交的内容）
  （使用 "git checkout -- <file>..." 丢弃工作区的改动）

	修改：     data/map.js
	修改：     data/map/domain.js
	修改：     data/map/init.js

未跟踪的文件:
  （使用 "git add <file>..." 以包含要提交的内容）

	data/data/monster.js
	data/map/monster.js
```

`status`命令可以看到当前分支的修改状况，分为修改、创建、删除三类情况。

## add命令

```shell
$ git add <file-full-name>
```

`add`命令可以将修改的文件加入索引，这些索引内的文件将被下一次提交（commit）收录并提交。

可以使用`add .`来将所有修改加入索引。

## reset命令

```shell
$ git reset <file-full-name>
```

`reset`命令跟`add`命令正好相反，将索引中指定的文件修改去除。

`reset`命令还常用于回退指定提交，在`commit`命令中说明。

## commit命令

```shell
$ git commit
```

`commit`命令将待提交索引中的文件打包成一个版本，并提交到本地库，这一指令无法回退，每个commit会产生一个commit id，这个id是唯一的。

若是发现执行了一个错误的commit，可以使用`reset <commit id>`来回退到指定id的版本修改。

## log命令

```shell
$ git log
```

`log`命令将显示当前分支本地所有的commit列表，用于管理提交，可以配合`reset`来回退版本。

## fetch命令

```shell
$ git fetch <remote-name> [<branch-name>]
```

`fetch`命令将从指定的仓库取到所有更新，并下载到本地，也可以制定一个`branch-name`来取回特定分支的所有更新。

**注意：**这个命令只是取到所有更新记录，并不会将这些修改合并到本地文件。

这个命令也是Git设计的精髓，这保证了本地库和远端库分离，并且这个分布式版本控制软件设计的如此巧妙，使得远端库不仅仅可以是服务器，它可以是你的一块硬盘，或者一个U盘，甚至是另一个Git项目。

这个命令被包含在`pull`命令的前半部分，在`pull`命令中说明。

## merge命令

```shell
$ git merge <remote-branch-name>[:<target-branch-name>]
```

`merge`命令将指定的分支合并到另一个分支中，不指定目标分支则合并到当前分支。

这个命令被包含在`pull`命令后半部分，在`pull`命令中说明。

## pull命令

```shell
$ git pull <remote-name> <remote-branch-name>[:<target-branch-name>]
```

`pull`命令将`fetch`命令和`merge`命令合并到了一起。先从远端仓库取得更新，然后合并到指定分支。

**注意：**并不是说`pull`命令能代替`fetch`和`merge`，只是说在理想情况下可以这么做，如果产生冲突和一些其他个别情况，还是需要分别执行上述两个命令。

## push命令

```shell
$ git push <remote-name> <local-branch-name>[:<remote-branch-name>]
```

`push`命令将本地修改提交到远端，如果不指定远端分支名，则使用本地分支名，如果远端不存在目标分支名，则新建一个分支。

这个命令和`pull`（`fetch`和`merge`）将本地和远端仓库同步起来，这就是Git的全貌。
