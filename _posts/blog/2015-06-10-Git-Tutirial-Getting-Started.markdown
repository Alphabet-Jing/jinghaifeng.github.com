---
layout: post
keywords: blog
description: blog
title: "Git使用教程——起步"
categories: [Git]
tags: [Git]
---
{% include JB/setup %}



#[原文地址](https://www.atlassian.com/git/tutorials/setting-up-a-repository/git-config)
![](https://www.atlassian.com/git/images/tutorials/getting-started/setting-up-a-repository/hero.svg)

#创建一个仓库

这个教程针对一些最常用`Git`命令，提供了简洁的描述。首先，`创建一个仓库`章节说明了，开始新的版本控制计划时，你需要的所有工具。然后，剩余的章节介绍你日常将使用的`Git`命令。

通过本章，你应能创建一个`Git`仓库，记录项目快照，并查看项目历史。

---

##git init
`git init`命令将创建一个新的`Git`仓库。它常被用来将一个已存在的、未纳入版本控制的项目，转换成一个`Git`仓库，或者直初始化一个新的空仓库。在已初始化的仓库之外，大部分的`Git`命令是不可用的。因此，`git init`通常是你创建新项目时，首先使用的命令。

执行`git init`将在项目的根目录中创建一个`.git`子目录，其中包含这个仓库所需的所有元数据。除去`.git`目录，整个项目保持不变（和`SVN`不一样，`Git`不需要在每个子目录下都创建`.git`目录）。

###**用法**

	git init

将当前目录转换成`Git`仓库。在当前目录下，创建`.git`目录，使之可以记录项目版本。

	git init <directory>
	
在制定的目录中，创建一个空得的`Git`仓库。执行此命令后，将会创建一个叫`<directory>`得新目录，其中只有`.git`子目录。

	git init --bare <directory>
	
初始化一个空的`Git`仓库，但会忽略工作树。共享的仓库都应使用`--bare`标记来创建。通常，这些仓库初始化使用`--bare`标志，并以`.git`结束。例如：这个叫做`my-project`的裸仓库，应当被存储在一个叫做`my-project.git`的目录中。

###**讨论**

对比`SVN`，`git init`命令能非常简单地创建一个新的版本控制项目。`Git`不需要你，为了创建仓库，从而导入文件，检查工作拷贝。你需要做的只是切换到项目目录，执行`git init`命令，然后你将拥有一个功能齐全的`Git`仓库。然而，对大部分的项目而言，`git init`只会在创建中心仓库时，使用一次——开发者们一般不会使用`git init`来创建他们的本地仓库。相反，他们经常使用`git clone`来拷贝已存在的仓库到他们本地设备上。

####**裸仓库(Bare Repositories)**
`--bare`标志会创建一个没有工作目录的仓库，使之能在仓库中编辑文件、提交修改。中央仓库应当总被创建为`裸仓库`。因为在推送分支到非裸仓库时，有覆盖修改的可能性。考虑到`--bare`标志可以使仓库成为一种存储设备，而不是开发环境。这意味着，对于所有的虚拟工作流，中心仓库都应是`bare`，而开发者的本地仓库则应是`non-bare`。

![](https://www.atlassian.com/git/images/tutorials/getting-started/setting-up-a-repository/01.svg)

###**例子**

自从`git clone`成为一种更方便的创建本地项目拷贝的方法后，`git init`的主要用途变成了创建中心仓库：

	ssh <user>@<host>
	cd path/above/repo
	git init --bare my-project.git
	
首先，通过SSH进入包含中心仓库的服务器。然后，找到项目将被存储的位置。最后，使用`--bare`标志创建一个中心存储仓库。开发着们将在这之后使用`[clone](/tutorials/setting-up-a-repository/git-clone) my-project.git`，在他们的开发设备上创建一个拷贝。

---

##git clone

`git clone`命令能复制已存在的`Git`仓库。这有几分类似于`svn checkout`，除了“工作拷贝”是一个完整的`Git`仓库——拥有自己的历史，能够管理文件，是完全独立于原始仓库。

方便起见，克隆时会自动创建叫一个做`origin`的远程连接，指向原始仓库。这将使得本地仓库与远程仓库能够轻松地交互。

###用法

	git clone <repo>
	
将处于`<repo>`的仓库克隆到本地设备上。通过`HTTP`或`SSH`可将原始仓库部署到本地文件系统或者远程设备上。

	git clone <repo> <directory>
	
将`<repo>`仓库克隆到本地`<directory>`目录下。

###**讨论**

如果项目已经被建立在中心仓库中，`git clone`命令式是用户获得一份开发拷贝的最常用方法。和`git init`相似，克隆一般只是一次性操作——一旦开发者拥有了一份工作拷贝，所有的版本控制操作和协作，都可通过被墓地仓库实现。

####**仓库间协作*

理解`Git`的`工作拷贝`概念是非常重要的，它与你从`SVN`仓库中`check out`一份工作拷贝是完全不一样的。与`SVN`不一样的是，`Git`的工作拷贝与中心仓库之间是没有区别的——它们都是完整的`Git`仓库。

这些特性使`Git`的团队协作与`SVN`从根本上区别开。`SVN`的协作是依赖于中心仓库与工作拷贝之间的关系，与之相反，`Git`的协作模型是基于仓库与仓库之间的交互。你使用`push`和`pull`进行仓库间的提交，取代了通过校验一份工作拷贝到`SVN`中心仓库。

![svn-repo](https://www.atlassian.com/git/images/tutorials/getting-started/setting-up-a-repository/03.svg)

![git-repo](https://www.atlassian.com/git/images/tutorials/getting-started/setting-up-a-repository/02.svg)

当然，这不意味着会妨碍你赋予一些特殊的意义给`Git`仓库。例如，标明一个`Git`仓库作为中心仓库，即可使用`Git`复制一份集中化的工作流。这些技巧是通过一些约定而非是`VCS`自身的硬性要求。

###**例子**

这个例子演示了，如何使用名叫`john`的`SSH`，获得一份存储于`example.com`的中心仓库的本地拷贝。

	git clone ssh://john@example.com/path/to/my-project.git 
	cd my-project
	# Start working on the project
	
首先，第一个命令，在本地`my-project`目录中，初始化了一个新的`Git`仓库，并把中心仓库的内容填充进去。然后，你能切换进项目中，开始编辑文件，提交快照，与其他的仓库进行交互。同时需要注意的是，从被克隆的仓库的`.git`扩展名是被忽略的。这表明了本地拷贝的`non-bare`状态。

---

##**git config**

·git config`命令可使你通过命令行配置你的`Git`安装（或个体仓库）。这个命令能定义从用户信息到首选项到仓库行为等，所有数据。下面列举了几个常用的配置操作。

###**用法**

	git config user.name <name>
	
在当前仓库中，定义作者名称——将被用于所有的提交中。通常，你将使用`--global`标志来配置当前用户。

	git config --global user.name <name>

通过当前用户，定义了所有提交时的作者名称。

	git config --global user.email <email>
	
通过当前用户，定义了提交时的email。

	git config --global alias.<alias-name> <git-command>
	
定义了一个`Git`命令的别名。

	git config --system core.editor <editor>

在本地定义了一个文本编辑器，在提交命令时，会被用到。这`<editor>`参数是所需编译器的打开命令，例如，vi。

	git config --global --edit
	
通过编辑器打开全局配置，进行手动编辑。

###**讨论**

所有的配置操作都将存放在纯文本文件中，因此，`git config`仅是一个方便的命令行接口。通常，你只需在新的开发设备上配置一次即可，而大部分情况，你将使用`--global`标志。

`Git`保存了配置选项在三个独立的文件中，使之对独立仓库，用户，和整个系统有不同的作用范围。

* <repo>/.git/config ——仓库的特定设置。
* ~/.gitconfig ——用户的特定设置，也是使用`--global`之后的选项。
* $(prefix)/etc/gitconfig —— 系统范围的设置。

当这些选项出现冲突时，本地设置覆盖用户设置，用户设置覆盖系统设置。如果你打开任一文件，你将看到的如下：

	[user] 
	name = John Smith
	email = john@example.com
	[alias]
	st = status
	co = checkout
	br = branch
	up = rebase
	ci = commit
	[core]
	editor = vim
	
你可以手动编辑这些值，与使用`git config`命令效果一致。

###**例子**

在安装`Git`之后，你首先应该设置你的`name`/`email`同时定制一些默认操作。如下，是一般的默认初始操作：

	# Tell Git who you are
	git config --global user.name "John Smith"
	git config --global user.email john@example.com

	# Select your favorite text editor
	git config --global core.editor vim

	# Add some SVN-like aliases
	git config --global alias.st status
	git config --global alias.co checkout
	git config --global alias.br branch
	git config --global alias.up rebase
	git config --global alias.ci commit

之前的操作，将会影响`~/.gitconfig`文件。