# Git入门指南
本教程的目的是为那些初学者尽快熟悉 ***Git***，提供了一个良好的起点。  
接下来，我们会讲一些 ***Git*** 的基本用法，那些你将在90%的时间都在使用的命令。这些东东能给一个不错的使用的基础，也许这些命令就是你将使用的全部命令。这些大约会你30分钟的时间来读。  

## GIT对象模型
### SHA
所有用来表示项目历史信息的文件,是通过一个40个字符的（40-digit）“对象名”来索引的，对象名看起来像这样:  

```
6ff87c4664981e4397625791c8ea3bbb5f2279a3
```  

你会在Git里到处看到这种“40个字符”字符串。每一个“对象名”都是对“对象”内容做SHA1哈希计算得来的，（SHA1是一种密码学的哈希算法）。这样就意味着两个不同内容的对象不可能有相同的“对象名”。  

这样做会有几个好处：  

* Git只要比较对象名，就可以很快的判断两个对象是否相同。  
* 因为在每个仓库（repository）的“对象名”的计算方法都完全一样，如果同样的内容存在两个不同的仓库中，就会存在相同的“对象名”下。  
* Git还可以通过检查对象内容的SHA1的哈希值和“对象名”是否相同，来判断对象内容是否正确。  

### 对象

每个对象(object) 包括三个部分：**类型**，**大小**和**内容**。大小就是指内容的大小，内容取决于对象的类型，有四种类型的对象："blob"、"tree"、 "commit" 和"tag"。

* “**blob**”用来存储文件数据，通常是一个文件。
* “**tree**”有点像一个目录，它管理一些“**tree**”或是 “**blob**”（就像文件和子目录）
* 一个“**commit**”只指向一个"tree"，它用来标记项目某一个特定时间点的状态。它包括一些关于时间点的元数据，如时间戳、最近一次提交的作者、指向上次提交（commits）的指针等等。
* 一个“**tag**”是来标记某一个提交(commit) 的方法。

几乎所有的Git功能都是使用这四个简单的对象类型来完成的。它就像是在你本机的文件系统之上构建一个小的文件系统。

### 与SVN的区别

Git与你熟悉的大部分版本控制系统的差别是很大的。也许你熟悉Subversion、CVS、Perforce、Mercurial 等等，他们使用 “增量文件系统” （Delta Storage systems）, 就是说它们存储每次提交(commit)之间的差异。Git正好与之相反，它会把你的每次提交的文件的全部内容（snapshot）都会记录下来。这会是在使用Git时的一个很重要的理念。

### Blob对象

一个blob通常用来存储文件的内容.

![](http://gitbook.liuhui998.com/assets/images/figure/object-blob.png)

你可以使用git show命令来查看一个blob对象里的内容。假设我们现在有一个Blob对象的SHA1哈希值，我们可以通过下面的的命令来查看内容：

```
$ git show 6ff87c4664

 Note that the only valid version of the GPL as far as this project
 is concerned is _this_ particular version of the license (ie v2, not
 v2.2 or v3.x or whatever), unless explicitly otherwise stated.
...
```

一个"blob对象"就是一块二进制数据，它没有指向任何东西或有任何其它属性，甚至连文件名都没有.

因为blob对象内容全部都是数据，如两个文件在一个目录树（或是一个版本仓库）中有同样的数据内容，那么它们将会共享同一个blob对象。Blob对象和其所对应的文件所在路径、文件名是否改被更改都完全没有关系。

### Tree 对象

一个tree对象有一串(bunch)指向blob对象或是其它tree对象的指针，它一般用来表示内容之间的目录层次关系。

![](http://gitbook.liuhui998.com/assets/images/figure/object-tree.png)

git show命令还可以用来查看tree对象，但是git ls-tree能让你看到更多的细节。如果我们有一个tree对象的SHA1哈希值，我们可以像下面一样来查看它：

```
$ git ls-tree fb3a8bdd0ce
100644 blob 63c918c667fa005ff12ad89437f2fdc80926e21c    .gitignore
100644 blob 5529b198e8d14decbe4ad99db3f7fb632de0439d    .mailmap
100644 blob 6ff87c4664981e4397625791c8ea3bbb5f2279a3    COPYING
040000 tree 2fb783e477100ce076f6bf57e4a6f026013dc745    Documentation
100755 blob 3c0032cec592a765692234f1cba47dfdcc3a9200    GIT-VERSION-GEN
100644 blob 289b046a443c0647624607d471289b2c7dcd470b    INSTALL
100644 blob 4eb463797adc693dc168b926b6932ff53f17d0b1    Makefile
100644 blob 548142c327a6790ff8821d67c2ee1eff7a656b52    README
...
```

就如同你所见，一个tree对象包括一串(list)条目，每一个条目包括：mode、对象类型、SHA1值 和名字(这串条目是按名字排序的)。它用来表示一个目录树的内容。

一个tree对象可以指向(reference): 一个包含文件内容的blob对象, 也可以是其它包含某个子目录内容的其它tree对象. Tree对象、blob对象和其它所有的对象一样，都用其内容的SHA1哈希值来命名的；只有当两个tree对象的内容完全相同（包括其所指向所有子对象）时，它的名字才会一样，反之亦然。这样就能让Git仅仅通过比较两个相关的tree对象的名字是否相同，来快速的判断其内容是否不同。

注意：所有的文件的mode位都是644 或 755，这意味着Git只关心文件的可执行位.

### Commit对象

"commit对象"指向一个"tree对象", 并且带有相关的描述信息.

![](http://gitbook.liuhui998.com/assets/images/figure/object-commit.png)

你可以用 --pretty=raw 参数来配合 git show 或 git log 去查看某个提交(commit):

```
$ git show -s --pretty=raw 2be7fcb476
commit 2be7fcb4764f2dbcee52635b91fedb1b3dcf7ab4
tree fb3a8bdd0ceddd019615af4d57a53f43d8cee2bf
parent 257a84d9d02e90447b149af58b271c19405edb6a
author Dave Watson <dwatson@mimvista.com> 1187576872 -0400
committer Junio C Hamano <gitster@pobox.com> 1187591163 -0700

    Fix misspelling of 'suppress' in docs

    Signed-off-by: Junio C Hamano <gitster@pobox.com>
```

你可以看到, 一个提交(commit)由以下的部分组成:

* 一个 tree 对象: tree对象的SHA1签名, 代表着目录在某一时间点的内容.

* 父对象 (parent(s)): 提交(commit)的SHA1签名代表着当前提交前一步的项目历史. 上面的那个例子就只有一个父对象; 合并的提交(merge commits)可能会有不只一个父对象. 如果一个提交没有父对象, 那么我们就叫它“根提交"(root commit), 它就代表着项目最初的一个版本(revision). 每个项目必须有至少有一个“根提交"(root commit). 一个项目可能有多个"根提交“，虽然这并不常见(这不是好的作法).

* 作者 : 做了此次修改的人的名字,　还有修改日期.

* 提交者（committer): 实际创建提交(commit)的人的名字, 同时也带有提交日期. TA可能会和作者不是同一个人; 例如作者写一个补丁(patch)并把它用邮件发给提交者, 由他来创建提交(commit).

* 注释: 用来描述此次提交.

注意: 一个提交(commit)本身并没有包括任何信息来说明其做了哪些修改; 所有的修改(changes)都是通过与父提交(parents)的内容比较而得出的. 值得一提的是, 尽管git可以检测到文件内容不变而路径改变的情况, 但是它不会去显式(explicitly)的记录文件的更名操作.　(你可以看一下 git diff 的 -M 参数的用法)

一般用 git commit 来创建一个提交(commit), 这个提交(commit)的父对象一般是当前分支(current HEAD),　同时把存储在当前索引(index)的内容全部提交.

### 对象模型

现在我们已经了解了3种主要对象类型(blob, tree 和 commit), 好现在就让我们大概了解一下它们怎么组合到一起的.

如果我们一个小项目, 有如下的目录结构:

```
$>tree
.
|-- README
`-- lib
    |-- inc
    |   `-- tricks.rb
    `-- mylib.rb

2 directories, 3 files
```

如果我们把它提交(commit)到一个Git仓库中, 在Git中它们也许看起来就如下图:

![](http://gitbook.liuhui998.com/assets/images/figure/objects-example.png)

你可以看到: 每个目录都创建了 **tree对象** (包括根目录), 每个文件都创建了一个对应的 **blob对象** . 最后有一个 **commit对象** 来指向根tree对象(root of trees), 这样我们就可以追踪项目每一项提交内容.

### 标签对象

一个标签对象包括一个对象名(译者注:就是SHA1签名), 对象类型, 标签名, 标签创建人的名字("tagger"), 还有一条可能包含有签名(signature)的消息. 你可以用 git cat-file 命令来查看这些信息:

```
$ git cat-file tag v1.5.0
object 437b1b20df4b356c9342dac8d38849f24ef44f27
type commit
tag v1.5.0
tagger Junio C Hamano <junkio@cox.net> 1171411200 +0000

GIT 1.5.0
-----BEGIN PGP SIGNATURE-----
Version: GnuPG v1.4.6 (GNU/Linux)

iD8DBQBF0lGqwMbZpPMRm5oRAuRiAJ9ohBLd7s2kqjkKlq1qqC57SbnmzQCdG4ui
nLE/L9aUXdWeTFPron96DLA=
=2E+0
-----END PGP SIGNATURE-----
```

## 安装Git
### 从源代码开始安装

如果你在一个其基于Unix的系统中，你可以从Git的官网上下载它的源代码,并运行像下面的几行命令,你就可以安装:

```
$ make prefix=/usr all ;# as yourself 
$ make prefix=/usr install ;# 以root权限运行
```

你需一些库: expat,curl, zlib, 和 openssl; 除了expat 外，其它的可能在你的机器上都安装了。

### Linux

如果你用的是Linux，你可以用你的本地包管理系统(native package management system)来安装.

```
$ yum install git-core  #译者注，在redhat等系统下用yum

$ apt-get install git-core  #译者注，在debian, ubuntu等系统下用apt-get
```

### Mac OS X

git可能在你的机器安装了，如果没有，请升级到最新系统。

### Windows

在Windows下安装Git是很简单的，你只要下载msysGit就可以了。

## 安装与初始化
### Git 配置

使用Git的第一件事就是设置你的名字和email,这些就是你在提交commit时的签名。

```
$ git config --global user.name "Scott Chacon"
$ git config --global user.email "schacon@gmail.com"
```

执行了上面的命令后,会在你的主目录(home directory)建立一个叫 ~/.gitconfig 的文件. 内容一般像下面这样:

```
[user]
        name = Scott Chacon
        email = schacon@gmail.com
```

译者注:这样的设置是全局设置,会影响此用户建立的每个项目.

如果你想使项目里的某个值与前面的全局设置有区别(例如把私人邮箱地址改为工作邮箱);你可以在项目中使用git config 命令不带 --global 选项来设置. 这会在你项目目录下的 .git/config 文件增加一节[user]内容(如上所示).

## 获得一个Git仓库

既然我们现在把一切都设置好了，那么我们需要一个Git仓库。有两种方法可以得到它：一种是从已有的Git仓库中　clone (克隆，复制)；还有一种是新建一个仓库，把未进行版本控制的文件进行版本控制。

### Clone一个仓库

为了得一个项目的拷贝(copy),我们需要知道这个项目仓库的地址(Git URL). Git能在许多协议下使用，所以Git URL可能以ssh://, http(s)://, git://,或是只是以一个用户名（git 会认为这是一个ssh 地址）为前辍. 有些仓库可以通过不只一种协议来访问，例如，Git本身的源代码你既可以用 git:// 协议来访问：

```
git clone git://git.kernel.org/pub/scm/git/git.git
```

也可以通过http 协议来访问:

```
git clone http://www.kernel.org/pub/scm/git/git.git
```

git://协议较为快速和有效,但是有时必须使用http协议,比如你公司的防火墙阻止了你的非http访问请求.如果你执行了上面两行命令中的任意一个,你会看到一个新目录: 'git',它包含所有的Git源代码和历史记录.

在默认情况下，Git会把"Git URL"里目录名的'.git'的后辍去掉,做为新克隆(clone)项目的目录名: (例如. git clone http://git.kernel.org/linux/kernel/git/torvalds/linux-2.6.git 会建立一个目录叫'linux-2.6')

### 初始化一个新的仓库

现在假设有一个叫”project.tar.gz”的压缩文件里包含了你的一些文件，你可以用下面的命令让它置于Git的版本控制管理之下.

```
$ tar xzf project.tar.gz
$ cd project
$ git init
```

Git会输出:

```
Initialized empty Git repository in .git/
```

如果你仔细观查会发现project目录下会有一个名叫”.git” 的目录被创建，这意味着一个仓库被初始化了。

## 正常的工作流程

修改文件，将它们更新的内容添加到索引中.

```
$ git add file1 file2 file3
```

你现在为commit做好了准备，你可以使用 git diff 命令再加上 --cached 参数 ,看看哪些文件将被提交(commit)。

```
$ git diff --cached
```

(如果没有--cached参数，git diff 会显示当前你所有已做的但没有加入到索引里的修改.) 你也可以用git status命令来获得当前项目的一个状况:

```
$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#   modified:   file1
#   modified:   file2
#   modified:   file3
#
```

如果你要做进一步的修改, 那就继续做, 做完后就把新修改的文件加入到索引中. 最后把他们提交：

```
$ git commit
```

这会提示你输入本次修改的注释，完成后就会记录一个新的项目版本.

除了用git add 命令，我还可以用

```
$ git commit -a
```

这会自动把所有内容被修改的文件(不包括新创建的文件)都添加到索引中，并且同时把它们提交。

这里有一个关于写commit注释的技巧和大家分享:commit注释最好以一行短句子作为开头，来简要描述一下这次commit所作的修改(最好不要超过50个字符)；然后空一行再把详细的注释写清楚。这样就可以很方便的用工具把commit注释变成email通知，第一行作为标题，剩下的部分就作email的正文.

### Git跟踪的是内容不是文件

很多版本控制系统都提供了一个 "add" 命令：告诉系统开始去跟踪某一个文件的改动。但是Git里的 ”add” 命令从某种程度上讲更为简单和强大. git add 不但是用来添加不在版本控制中的新文件，也用于添加已在版本控制中但是刚修改过的文件; 在这两种情况下, Git都会获得当前文件的快照并且把内容暂存(stage)到索引中，为下一次commit做好准备。

## 分支与合并基础

一个Git仓库可以维护很多开发分支。现在我们来创建一个新的叫”experimental”的分支：

```
$ git branch experimental
```

如果你运行下面这条命令：

```
$ git branch
```

你会得到当前仓库中存在的所有分支列表：

```
  experimental
* master
```

“experimental” 分支是你刚才创建的，“master”分支是Git系统默认创建的主分支。星号(“*”)标识了你当工作在哪个分支下，输入：

```
$ git checkout experimental
```

切换到”experimental”分支，先编辑里面的一个文件，再提交(commit)改动，最后切换回 “master”分支。

```
(edit file)
$ git commit -a
$ git checkout master
```

你现在可以看一下你原来在“experimental”分支下所作的修改还在不在；因为你现在切换回了“master”分支，所以原来那些修改就不存在了。

你现在可以在“master”分支下再作一些不同的修改:

```
(edit file)
$ git commit -a
```

这时，两个分支就有了各自不同的修改(diverged)；我们可以通过下面的命令来合并“experimental”和“master”两个分支:

```
$ git merge experimental
```

如果这个两个分支间的修改没有冲突(conflict), 那么合并就完成了。如有有冲突，输入下面的命令就可以查看当前有哪些文件产生了冲突:

```
$ git diff
```

当你编辑了有冲突的文件，解决了冲突后就可以提交了：

```
$ git commit -a
```

这时你就可以删除掉你的 “experimental” 分支了(如果愿意)：

```
$ git branch -d experimental
```

git branch -d只能删除那些已经被当前分支的合并的分支. 如果你要强制删除某个分支的话就用git branch –D；下面假设你要强制删除一个叫”crazy-idea”的分支：

```
$ git branch -D crazy-idea
```

分支是很轻量级且容易的，这样就很容易来尝试它。

### 如何合并

你可以用下面的命令来合并两个分离的分支：git merge:

```
$ git merge branchname
```

这个命令把分支"branchname"合并到了当前分支里面。如有冲突(冲突--同一个文件在远程分支和本地分支里按不同的方式被修改了）；那么命令的执行输出就像下面一样

```
$ git merge next
 100% (4/4) done
Auto-merged file.txt
CONFLICT (content): Merge conflict in file.txt
Automatic merge failed; fix conflicts and then commit the result.
```

在有问题的文件上会有冲突标记，在你手动解决完冲突后就可以把此文件添 加到索引(index)中去，用git commit命令来提交，就像平时修改了一个文件 一样。

如果你用gitk来查看commit的结果，你会看到它有两个父分支：一个指向当前 的分支，另外一个指向刚才合并进来的分支。

### 解决合并中的冲突

如果执行自动合并没有成功的话，git会在索引和工作树里设置一个特殊的状态， 提示你如何解决合并中出现的冲突。

有冲突(conflicts)的文件会保存在索引中，除非你解决了问题了并且更新了索引，否则执行 git commit都会失败:

```
$ git commit
file.txt: needs merge
```

如果执行 git status 会显示这些文件没有合并(unmerged),这些有冲突的文件里面会添加像下面的冲突标识符:

```
<<<<<<< HEAD:file.txt
Hello world
=======
Goodbye
>>>>>>> 77976da35a11db4580b80ae27e8d65caf5208086:file.txt
```

你所需要的做是就是编辑解决冲突，（接着把冲突标识符删掉），再执行下面的命令:

```
$ git add file.txt
$ git commit
```

注意：提交注释里已经有一些关于合并的信息了，通常是用这些默认信息，但是你可以添加一些你想要的注释。

上面这些就是你要做一个简单合并所要知道的，但是git提供更多的一些信息来 帮助解决冲突。

### 撒销一个合并

如果你觉得你合并后的状态是一团乱麻，想把当前的修改都放弃，你可以用下面的命令回到合并之前的状态：

```
$ git reset --hard HEAD
```

或者你已经把合并后的代码提交，但还是想把它们撒销：

```
$ git reset --hard ORIG_HEAD
```

但是刚才这条命令在某些情况会很危险，如果你把一个已经被另一个分支合并的分支给删了，那么 以后在合并相关的分支时会出错。

### 快速向前合并

还有一种需要特殊对待的情况，在前面没有提到。通常，一个合并会产生一个合并提交(commit), 把两个父分支里的每一行内容都合并进来。

但是，如果当前的分支和另一个分支没有内容上的差异，就是说当前分支的每一个提交(commit)都已经存在另一个分支里了，git 就会执行一个“快速向前"(fast forward)操作；git 不创建任何新的提交(commit),只是将当前分支指向合并进来的分支。

## 查看历史 －Git日志

git log命令可以显示所有的提交(commit)。 ......

```
$ git log v2.5..        # commits since (not reachable from) v2.5
$ git log test..master  # commits reachable from master but not test
$ git log master..test  # commits reachable from test but not master
$ git log master...test # commits reachable from either test or
                        #    master, but not both
$ git log --since="2 weeks ago" # commits from the last 2 weeks
$ git log Makefile      # commits that modify Makefile
$ git log fs/           # commits that modify any file under fs/
$ git log -S'foo()'     # commits that add or remove any file data
                        # matching the string 'foo()'
$ git log --no-merges   # dont show merge commits
```

当然你也可以组合上面的命令选项；下面的命令就是找出所有从"v2.5“开 始在fs目录下的所有Makefile的修改.

```
$ git log v2.5.. Makefile fs/
```

Git会根据git log命令的参数，按时间顺序显示相关的提交(commit)。

```
commit f491239170cb1463c7c3cd970862d6de636ba787
Author: Matt McCutchen <matt@mattmccutchen.net>
Date:   Thu Aug 14 13:37:41 2008 -0400

    git format-patch documentation: clarify what --cover-letter does

commit 7950659dc9ef7f2b50b18010622299c508bfdfc3
Author: Eric Raible <raible@gmail.com>
Date:   Thu Aug 14 10:12:54 2008 -0700

    bash completion: 'git apply' should use 'fix' not 'strip'
    Bring completion up to date with the man page.
```

你也可以让git log显示补丁(patchs):

```
$ git log -p

commit da9973c6f9600d90e64aac647f3ed22dfd692f70
Author: Robert Schiele <rschiele@gmail.com>
Date:   Mon Aug 18 16:17:04 2008 +0200

    adapt git-cvsserver manpage to dash-free syntax

diff --git a/Documentation/git-cvsserver.txt b/Documentation/git-cvsserver.txt
index c2d3c90..785779e 100644
--- a/Documentation/git-cvsserver.txt
+++ b/Documentation/git-cvsserver.txt
@@ -11,7 +11,7 @@ SYNOPSIS
 SSH:

 [verse]
-export CVS_SERVER=git-cvsserver
+export CVS_SERVER="git cvsserver"
 'cvs' -d :ext:user@server/path/repo.git co <HEAD_name>

 pserver (/etc/inetd.conf):
```

### 日志统计

如果用--stat选项使用'git log',它会显示在每个提交(commit)中哪些文件被修改了, 这些文件分别添加或删除了多少行内容.

```
$ git log --stat

commit dba9194a49452b5f093b96872e19c91b50e526aa
Author: Junio C Hamano <gitster@pobox.com>
Date:   Sun Aug 17 15:44:11 2008 -0700

    Start 1.6.0.X maintenance series

 Documentation/RelNotes-1.6.0.1.txt |   15 +++++++++++++++
 RelNotes                           |    2 +-
 2 files changed, 16 insertions(+), 1 deletions(-)
```

### 格式化日志

你可以按你的要求来格式化日志输出。‘--pretty'参数可以使用若干表现格式，如‘oneline':

```
$ git log --pretty=oneline
a6b444f570558a5f31ab508dc2a24dc34773825f dammit, this is the second time this has reverted
49d77f72783e4e9f12d1bbcacc45e7a15c800240 modified index to create refs/heads if it is not 
9764edd90cf9a423c9698a2f1e814f16f0111238 Add diff-lcs dependency
e1ba1e3ca83d53a2f16b39c453fad33380f8d1cc Add dependency for Open4
0f87b4d9020fff756c18323106b3fd4e2f422135 merged recent changes: * accepts relative alt pat
f0ce7d5979dfb0f415799d086e14a8d2f9653300 updated the Manifest file
```

或者你也可以使用 'short' 格式:

```
$ git log --pretty=short
commit a6b444f570558a5f31ab508dc2a24dc34773825f
Author: Scott Chacon <schacon@gmail.com>

    dammit, this is the second time this has reverted

commit 49d77f72783e4e9f12d1bbcacc45e7a15c800240
Author: Scott Chacon <schacon@gmail.com>

    modified index to create refs/heads if it is not there

commit 9764edd90cf9a423c9698a2f1e814f16f0111238
Author: Hans Engel <engel@engel.uk.to>

    Add diff-lcs dependency
```

你也可用‘medium','full','fuller','email' 或‘raw'. 如果这些格式不完全符合你的相求， 你也可以用‘--pretty=format'参数(参见：git log)来创建你自己的"格式“.

```
$ git log --pretty=format:'%h was %an, %ar, message: %s'
a6b444f was Scott Chacon, 5 days ago, message: dammit, this is the second time this has re
49d77f7 was Scott Chacon, 8 days ago, message: modified index to create refs/heads if it i
9764edd was Hans Engel, 11 days ago, message: Add diff-lcs dependency
e1ba1e3 was Hans Engel, 11 days ago, message: Add dependency for Open4
0f87b4d was Scott Chacon, 12 days ago, message: merged recent changes:
```

另一个有趣的事是：你可以用'--graph'选项来可视化你的提交图(commit graph),就像下面这样:

```
$ git log --pretty=format:'%h : %s' --graph
* 2d3acf9 : ignore errors from SIGCHLD on trap
*   5e3ee11 : Merge branch 'master' of git://github.com/dustin/grit
|\  
| * 420eac9 : Added a method for getting the current branch.
* | 30e367c : timeout code and tests
* | 5a09431 : add timeout protection to grit
* | e1193f8 : support for heads with slashes in them
|/  
* d6016bc : require time for xmlschema
```

它会用ASCII字符来画出一个很漂亮的提交历史(commit history)线。

### 日志排序

你也可以把日志记录按一些不同的顺序来显示。注意，git日志从最近的提交(commit)开始，并且从这里开始向它们父分支回溯。然而git历史可能包括多个互不关联的开发线路，这样有时提交(commits)显示出来就有点杂乱。

如果你要指定一个特定的顺序，可以为git log命令添加顺序参数(ordering option).

按默认情况，提交(commits)会按逆时间(reverse chronological)顺序显示。

但是你也可以指定‘--topo-order'参数，这就会让提交(commits)按拓朴顺序来显示(就是子提交在它们的父提交前显示). 如果你用git log命令按拓朴顺序来显示git仓库的提交日志，你会看到“开发线"(development lines)都会集合在一起.

```
$ git log --pretty=format:'%h : %s' --topo-order --graph
*   4a904d7 : Merge branch 'idx2'
|\  
| *   dfeffce : merged in bryces changes and fixed some testing issues
| |\  
| | * 23f4ecf : Clarify how to get a full count out of Repo#commits
| | *   9d6d250 : Appropriate time-zone test fix from halorgium
| | |\  
| | | * cec36f7 : Fix the to_hash test to run in US/Pacific time
| | * | decfe7b : fixed manifest and grit.rb to make correct gemspec
| | * | cd27d57 : added lib/grit/commit_stats.rb to the big list o' files
| | * | 823a9d9 : cleared out errors by adding in Grit::Git#run method
| | * |   4eb3bf0 : resolved merge conflicts, hopefully amicably
| | |\ \  
| | | * | d065e76 : empty commit to push project to runcoderun
| | | * | 3fa3284 : whitespace
| | | * | d01cffd : whitespace
| | | * | 7c74272 : oops, update version here too
| | | * | 13f8cc3 : push 0.8.3
| | | * | 06bae5a : capture stderr and log it if debug is true when running commands
| | | * | 0b5bedf : update history
| | | * | d40e1f0 : some docs
| | | * | ef8a23c : update gemspec to include the newly added files to manifest
| | | * | 15dd347 : add missing files to manifest; add grit test
| | | * | 3dabb6a : allow sending debug messages to a user defined logger if provided; tes
| | | * | eac1c37 : pull out the date in this assertion and compare as xmlschemaw, to avoi
| | | * | 0a7d387 : Removed debug print.
| | | * | 4d6b69c : Fixed to close opened file description.
```

你也可以用'--date-order'参数，这样显示提交日志的顺序主要按提交日期来排序. 这个参数和'--topo-order'有一点像，没有父分支会在它们的子分支前显示，但是其它的东东还是按交时间来排序显示。你会看到"开发线"(development lines)没有集合一起，它们会像并行开发(parallel development)一样跳来跳去的：

```
$ git log --pretty=format:'%h : %s' --date-order --graph
*   4a904d7 : Merge branch 'idx2'
|\  
* | 81a3e0d : updated packfile code to recognize index v2
| *   dfeffce : merged in bryces changes and fixed some testing issues
| |\  
| * | c615d80 : fixed a log issue
|/ /  
| * 23f4ecf : Clarify how to get a full count out of Repo#commits
| *   9d6d250 : Appropriate time-zone test fix from halorgium
| |\  
| * | decfe7b : fixed manifest and grit.rb to make correct gemspec
| * | cd27d57 : added lib/grit/commit_stats.rb to the big list o' file
| * | 823a9d9 : cleared out errors by adding in Grit::Git#run method
| * |   4eb3bf0 : resolved merge conflicts, hopefully amicably
| |\ \  
| * | | ba23640 : Fix CommitDb errors in test (was this the right fix?
| * | | 4d8873e : test_commit no longer fails if you're not in PDT
| * | | b3285ad : Use the appropriate method to find a first occurrenc
| * | | 44dda6c : more cleanly accept separate options for initializin
| * | | 839ba9f : needed to be able to ask Repo.new to work with a bar
| | * | d065e76 : empty commit to push project to runcoderun
* | | | 791ec6b : updated grit gemspec
* | | | 756a947 : including code from github updates
| | * | 3fa3284 : whitespace
| | * | d01cffd : whitespace
| * | | a0e4a3d : updated grit gemspec
| * | | 7569d0d : including code from github updates
```

最后，你也可以用 ‘--reverse'参数来逆向显示所有日志。

## 比较提交 - Git Diff

你可以用 git diff 来比较项目中任意两个版本的差异。

```
$ git diff master..test
```

上面这条命令只显示两个分支间的差异，如果你想找出‘master’,‘test’的共有 父分支和'test'分支之间的差异，你用3个‘.'来取代前面的两个'.' 。

```
$ git diff master...test
```

git diff 是一个难以置信的有用的工具，可以找出你项目上任意两点间 的改动，或是用来查看别人提交进来的新分支。

### 哪些内容会被提交(commit)

你通常用git diff来找你当前工作目录和上次提交与本地索引间的差异。

```
$ git diff
```

上面的命令会显示在当前的工作目录里的，没有 staged(添加到索引中)，且在下次提交时 不会被提交的修改。

如果你要看在下次提交时要提交的内容(staged,添加到索引中),你可以运行：

```
$ git diff --cached
```

上面的命令会显示你当前的索引和上次提交间的差异；这些内容在不带"-a"参数运行 "git commit"命令时就会被提交。

```
$ git diff HEAD
```

上面这条命令会显示你工作目录与上次提交时之间的所有差别，这条命令所显示的 内容都会在执行"git commit -a"命令时被提交。

### 更多的比较选项

如果你要查看当前的工作目录与另外一个分支的差别，你可以用下面的命令执行:

```
$ git diff test
```

这会显示你当前工作目录与另外一个叫'test'分支的差别。你也以加上路径限定符，来只 比较某一个文件或目录。

```
$ git diff HEAD -- ./lib 
```

上面这条命令会显示你当前工作目录下的lib目录与上次提交之间的差别(或者更准确的 说是在当前分支)。

如果不是查看每个文件的详细差别，而是统计一下有哪些文件被改动，有多少行被改 动，就可以使用‘--stat' 参数。

```
$>git diff --stat
 layout/book_index_template.html                    |    8 ++-
 text/05_Installing_Git/0_Source.markdown           |   14 ++++++
 text/05_Installing_Git/1_Linux.markdown            |   17 +++++++
 text/05_Installing_Git/2_Mac_104.markdown          |   11 +++++
 text/05_Installing_Git/3_Mac_105.markdown          |    8 ++++
 text/05_Installing_Git/4_Windows.markdown          |    7 +++
 .../1_Getting_a_Git_Repo.markdown                  |    7 +++-
 .../0_ Comparing_Commits_Git_Diff.markdown         |   45 +++++++++++++++++++-
 .../0_ Hosting_Git_gitweb_repoorcz_github.markdown |    4 +-
 9 files changed, 115 insertions(+), 6 deletions(-)
```

有时这样全局性的查看哪些文件被修改，能让你更轻轻一点。

## 分布式的工作流程

假设Alice现在开始了一个新项目，在/home/alice/project建了一个新的git 仓库(repository)；另一个叫Bob的工作目录也在同一台机器，他要提交代码。

Bob 执行了这样的命令:

```
$ git clone /home/alice/project myrepo
```

这就建了一个新的叫"myrepo"的目录，这个目录里包含了一份Alice的仓库的 克隆(clone). 这份克隆和原始的项目一模一样，并且拥有原始项目的历史记 录。

Bob 做了一些修改并且提交(commit)它们:

```
(edit files)
$ git commit -a
(repeat as necessary)
```

当他准备好了，他告诉Alice从仓库/home/bob/myrepo中把他的修改给拉 (pull)下来。她执行了下面几条命令:

```
$ cd /home/alice/project
$ git pull /home/bob/myrepo master
```

这就把Bob的主(master)分支合并到了Alice的当前分支里了。如果Alice在 Bob修改文件内容的同时也做了修改的话，她可能需要手工去修复冲突. (注意："master"参数在上面的命令中并不一定是必须的，因为这是一个 默认参数)

git pull命令执行两个操作: 它从远程分支(remote branch)抓取修改 的内容，然后把它合并进当前的分支。

如果你要经常操作远程分支(remote branch),你可以定义它们的缩写:

```
$ git remote add bob /home/bob/myrepo
```

这样，Alic可以用"git fetch"" 来执行"git pull"前半部分的工作， 但是这条命令并不会把抓下来的修改合并到当前分支里。

```
$ git fetch bob
```

我们用git remote命令建立了Bob的运程仓库的缩写，用这个(缩写) 名字我从Bob那得到所有远程分支的历史记录。在这里远程分支的名 字就叫bob/master.

```
$ git log -p master..bob/master
```

上面的命令把Bob从Alice的主分支(master)中签出后所做的修改全部显示出来。

当检查完修改后,Alice就可以把修改合并到她的主分支中。

```
$ git merge bob/master
```

这种合并(merge)也可以用pull来完成，就像下面的命令一样：

```
$ git pull . remotes/bob/master
```

注意：git pull 会把远程分支合并进当前的分支里，而不管你在命令 行里指定什么。

其后，Bob可以更新它的本地仓库--把Alice做的修改拉过来(pull):

```
$ git pull
```

如果Bob从Alice的仓库克隆(clone)，那么他就不需要指定Alice仓库的地 址；因为Git把Alice仓库的地址存储到Bob的仓库配库文件，这个地址就是 在git pull时使用：

```
$ git config --get remote.origin.url
/home/alice/project
```

(如果要查看git clone创建的所有配置参数，可以使用"git config -l", git config 的帮助文件里解释了每个参数的含义.)

Git同时也保存了一份最初(pristine)的Alice主分支(master)，在 "origin/master"下面。

```
$ git branch -r
  origin/master
```

如果Bob打算在另外一台主机上工作，他可以通过ssh协议来执行"clone" 和"pull"操作：

```
$ git clone alice.org:/home/alice/project myrepo
```

git有他自带的协议(native protocol),还可以使用rsync或http; 你可以点 这里 git pull 看一看更詳細的用法。

Git也可以像CVS一样来工作：有一个中心仓库，不同的用户向它推送(push) 自己所作的修改；你可以看看这里： git push gitcvs-migration.

### 公共Git仓库

另外一个提交修改的办法，就是告诉项目的维护者(maintainer)用 git pull 命令从你的仓库里把修改拉下来。这和从主仓库"里更新代码类似，但是是从 另外一个方向来更新的。

如果你和维护者(maintainer)都在同一台机器上有帐号，那么你们可以互相从对 方的仓库目录里直接拉(pull)所作的修改；git命令里的仓库地址也可以是本地 的某个目录名：

```
$ git clone /path/to/repository
$ git pull /path/to/other/repository
```

也可以是一个ssh地址：

```
$ git clone ssh://yourhost/~you/repository
```

如果你的项目只有很少几个开发者，或是只需要同步很少的几个私有仓库， 上面的方法也许够你用的。

然而，更通用的作法是维护几个不同的公开仓库(public repository). 这样可以把每个人的工作进度和公开仓库清楚的分开。

你还是每天在你的本地私人仓库里工作，但是会定期的把本地的修改推(push) 到你的公开仓库中；其它开发者就可以从这个公开仓库来拉(pull)最新的代码。 如果其它开发者也有他自己的公共仓库，那么他们之间的开发流程就如下图 所示：

```
                        you push
  your personal repo ------------------> your public repo
    ^                                     |
    |                                     |
    | you pull                            | they pull
    |                                     |
    |                                     |
        |               they push             V
  their public repo <------------------- their repo
```

### 将修改推到一个公共仓库

通过http或是git协议，其它维护者可以抓取(fetch)你最近的修改，但是他们 没有写权限。这样，这需要将本地私有仓库的最近修改上传公共仓库中。

译者注: 通过http的WebDav协议是可以有写权限的,也有人配置了git over http.

最简单的办法就是用 git push命令 和ssh协议; 用你本地的"master" 分支去更新远程的"master"分支，执行下面的命令:

```
$ git push ssh://yourserver.com/~you/proj.git master:master
```

或是:

```
$ git push ssh://yourserver.com/~you/proj.git master
```

和git-fetch命令一样git-push如果命令的执行结果不是"快速向前"(fast forward) 就会报错; 下面的章节会讲如何处理这种情况.

推(push)命令的目地仓库一般是个裸仓库(bare respository). 你也可以推到一 个签出工作目录树(checked-out working tree)的仓库，但是工作目录中内 容不会被推命令所更新。如果你把自己的分支推到一个已签出的分支里，这 会导致不可预知的后果。

在用git-fetch命令时，你也可以修改配置参数，让你少打字:)。

下面这些是例子:

```
$ cat >>.git/config <<EOF
[remote "public-repo"]
    url = ssh://yourserver.com/~you/proj.git
EOF
```

你可以用下面的命令来代替前面复杂的命令:

```
$ git push public-repo master
```

你可以点击这里: git config，查看remote..url, branch..remote, 和remote..push等选项的解释.

### 当推送代码失败时要怎么办

如果推送(push)结果不是"快速向前"(fast forward),那么它 可能会报像下面一样的错误：

```
error: remote 'refs/heads/master' is not an ancestor of
local  'refs/heads/master'.
Maybe you are not up-to-date and need to pull first?
error: failed to push to 'ssh://yourserver.com/~you/proj.git'
```

这种情况通常由以下的原因产生：

- 用 `git-reset --hard` 删除了一个已经发布了的一个提交，或是

- 用 `git-commit --amend` 去替换一个已经发布的提交，或是

- 用 `git-rebase` 去rebase一个已经发布的提交.　 
你可以强制git-push在上传修改时先更新，只要在分支名前面加一个加号。

```
$ git push ssh://yourserver.com/~you/proj.git +master
```

## Git标签
### 轻量级标签

我们可以用 git tag不带任何参数创建一个标签(tag)指定某个提交(commit):

```
$ git tag stable-1 1b2e1d63ff
```

这样，我们可以用stable-1 作为提交(commit) "1b2e1d63ff" 的代称(refer)。

前面这样创建的是一个“轻量级标签"，这种分支通常是从来不移动的。

如果你想为一个标签(tag)添加注释，或是为它添加一个签名(sign it cryptographically), 那么我们就需要创建一个 ”标签对象".

### 标签对象

如果有 "-a", "-s" 或是 "-u " 中间的一个命令参数被指定，那么就会创建 一个标签对象，并且需要一个标签消息(tag message). 如果没有"-m " 或是 "-F " 这些参数，那么就会启动一个编辑器来让用户输入标签消息(tag message).

译者注：大家觉得这个标签消息是不是提交注释(commit comment)比较像。

当这样的一条命令执行后，一个新的对象被添加到Git对象库中，并且标签引用就指向了 一个标签对象，而不是指向一个提交(commit). 这样做的好处就是：你可以为一个标签 打处签名(sign), 方便你以后来查验这是不是一个正确的提交(commit).

下面是一个创建标签对象的例子:

```
$ git tag -a stable-1 1b2e1d63ff
```

标签对象可以指向任何对象，但是在通常情况下是一个提交(commit). (在Linux内核代 码中，第一个标签对象是指向一个树对象(tree),而不是指向一个提交(commit)).

### 签名的标签

如果你配有GPG key,那么你就很容易创建签名的标签.首先你要在你的 .git/config 或 ~.gitconfig里配好key.

下面是示例:

```
[user]
    signingkey = <gpg-key-id>
```

你也可以用命令行来配置:

```
$ git config (--global) user.signingkey <gpg-key-id>
```

现在你可以直接用"-s" 参数来创“签名的标签”。

```
$ git tag -s stable-1 1b2e1d63ff
```

如果没有在配置文件中配GPG key,你可以用"-u“ 参数直接指定。

```
$ git tag -u <gpg-key-id> stable-1 1b2e1d63ff
```

## 更深入的学习
如果想深入学习git，[可以点这里](http://gitbook.liuhui998.com/index.html)

legit https://github.com/kennethreitz/legit

## Books

* [progit](http://git-scm.com/book/zh/v1)
* [Git 版本控制](http://www.amazon.cn/dp/B00U42VM7Y/ref=cm_sw_r_si_2_dp_rCJRvb0YJ2Y7D)