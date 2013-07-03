---
layout: post
title: "Tex中插入外部URL作为参考文献"
description: ""
category: tool
tags: [tex]
---

在BibTex中没有一个专门的参考文献类型是为website引用使用的。不过幸运的是，在参考文献类型中有一个Misc的类型中有一个`howpublished`的属性，我们可以利用这个属性和`\url`命令混用来实现我们想要的功能。

下面我们举一个例子，我把文件的名字命名为为urlbib：
	
	\documentclass{article}
	\usepackage{url}
	
	\begin{document}
	\section*{Testing the URL citations}
	
	Line spacing (the space between each line) in \LaTeX\ documents may be changed via the \verb! \linespread{spacing} ! command.~\cite{snapshots-for-hdfs}
	
	\bibliographystyle{plain}
	\bibliography{urlbib}
	
	\end{document}
在例子中我指定的`documentclass`为`article`，如果你是`report`等其他的也没问题。其中最主要的就是需要引入url的package。然后把文件存为urlbib.tex，然后我们再完成bib库的制作：

	@Misc{snapshots-for-hdfs,
	howpublished = {\url{http://hortonworks.com/blog/snapshots-for-hdfs/}},
	note = {Accessed April 25th, 2012},
	title = {Snapshots for HDFS},
	author = {Hari Mankude}
	}
然后存储为urlbib.bib为文件名的文件。

这样基本的准备工作就准备完成了，剩下的就是需要编译参考资料库以及tex文件了。需要注意的是我使用的Mac　OSX，我使用了xelatex来编译，这里我就只是给出xelatex的例子：

	xelatex　urlbib
	bibtex   urlbib
	xelatex　urlbib
	xelatex　urlbib
需要注意的是：命令出连续出现了3次`xelatex　urlbib`，不是写错，是必须这样的流程，不信你可以试试，少一步的话最后总是不完美。还有就是如果你在文档中使用了`minted`的包做代码高亮的话，执行的命令只是需要把`xelatex`修改为`xelatex -shell-escape`即可。

##　最后
* 下面的代码可以使得参考文献加入目录中

	\cleardoublepage
	\phantomsection
	\addcontentsline{toc}{chapter}{参考文献}

* 修改参考文献的名字，只需要在文中添加`\renewcommand{\bibname}{参考文献}`即可。如果还有其他需要修改可以参考这个文章[How to change LaTeX’s “fixed names”](http://www.tex.ac.uk/cgi-bin/texfaq2html?label=fixnam)
* 本篇文章主要参考了[Referencing website URLs with LaTeX (BibTeX)](http://timmurphy.org/2010/04/04/referencing-website-urls-with-latex-bibtex/)，大部分内容都是翻译自这篇文章，感谢原文作者。
{% include JB/setup %}
