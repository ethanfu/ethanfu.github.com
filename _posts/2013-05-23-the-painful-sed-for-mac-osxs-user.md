---
layout: post
title: "Mac用户的Sed之殇"
description: ""
category: Mac
tags: [sed]
---
{% include JB/setup %}

现在越来越多的程序员开始转型到Mac OSX，我也是其中一个。之所以投入Mac的阵营的原因有很多，但是好看的UI以及能够相对比较完美的使用shell应该是很多人的重要理由。既然每个程序员都会使用shell，那么sed应该是其中必不可少的一个了。

sed全称是stream editor，流编辑器，是Unix常见的命令行程序，主要用于解析和转换文本，如果想看详细介绍的话，参看[sed wiki](http://en.wikipedia.org/wiki/Sed)。sed是有多个分支的，最原始的版本就是贝尔实验室的版本，作者是[Lee E. McMahon](http://en.wikipedia.org/wiki/Lee_E._McMahon)，然后BSD和GNU都分别是一个，应该还有其他的，我也没太仔细考证。

只是看标题的话，其实自己都感觉有点标题党了，其实也无非就是想说一件事，就是Mac没使用GNU的sed给我们工作还是带了了少许不便吧，毕竟大家还是GNU的sed是使用的最多的。上面提到了sed是有多个分支的，由于Mac OSX使用了FreeBSD的内核，所以Mac OSX默认使用的是BSD sed，那么与大部分Linux默认使用的GNU sed是不太一样的。例如我们要替换file文件的所有ss为dd这个功能，如果我们在Ubuntu上我们可以如下写：

	sed -i 's/ss/dd/g' file
	
但是在Mac OSX上你需要如下写法才可：
	
	sed -i '' 's/ss/dd/g' file

这样就导致你写的shell可能在团队内部是不太兼容的，当使用Ubuntu的用户就没法使用你的脚本，除非做少许修改，就是把`''`去掉。

说了半天才把问题引出来实在不好意思，我们开始进入正题，今天我主要想解决的问题就是怎么刚才描述的问题给搞定。我推荐的方案就是就是把Mac的sed更换为GNU sed，那我们就基于这个方案做一些阐述。

不过通常来说，我们不应该替换掉系统默认的工具。主要是因为可能有系统管理的脚本或者是一些第三方包可能会依赖于系统默认的工具，所以如果你想使用GNU的工具来替代现在操作系统默认安装的工具的话，可能会带来不必要的麻烦。尤其是当你在Mac OS X上使用本来是为Linux或者BSD设计的软件的时候，因为这些类型的程序更可能依赖于shell scripts和系统命令，而不是调用OS X APIs。

在这个情况下，我们可以安装GNU的工具安装在另外一个目录中，而不是覆盖系统已安装的工具，然后我们在环境变量中把我们这个目录的优先级提前，这样系统就会在查找的时候会先定位到我们自己的目录。那么这样我们可以利用[Homebrew](http://mxcl.github.io/homebrew/index_zh-cn.html)来安装GNU的工具，执行以下命令：

	brew install coreutils
	brew install gsed

[GNU Coreutils](http://www.gnu.org/software/coreutils/)安装了大部分GNU的软件，包括wc，date等，但是因为不包括sed，所以还得跟上后边一句。软件安装以后都会在软件名称的基础上加上`g`的前缀。

如果每次都输入`gsed`的话还是比较麻烦的，并且兼容性上依然存在问题，每次使用Linux的人还是得和你不同的版本的脚本才可以。这样我们可以在`~/.bash_profile`上添加以下代码

	PATH="$(brew --prefix coreutils)/libexec/gnubin:$PATH"
	ln -s $(which gsed) $(brew --prefix coreutils)/libexec/gnubin/sed

这样完成之后source下`~/.bash_profile`就可以在Mac OS X上使用GNU版本的sed了，不过其他的软件的已经安装，不过还是有`g`的前缀，所以根据自己需求调整吧。


