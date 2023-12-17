---
aliases: null
tags: 
source: null
created: 2020-06-19 22:39:00
updated: 2023-03-07 15:49:46
uid: null
title: 将带有 bib 参考文献的 tex 文档转化成 docx 格式文件
dg-publish: true
---

# 将带有 Bib 参考文献的 Tex 文档转化成 Docx 格式文件

## 需求

英文论文投稿一般要投 tex 格式，参考文献是 bib 格式，但是导师习惯在 docx 上修改，所以需要将带 bib 参考文件的 tex 格式转成 docx 格式

note：只想预览效果可以直接看图，不看内容

## 工具

- pandoc

pandoc 是一款免费的格式转换工具

## 操作方法

我们准备一个 tex 文件 _untitled.tex_ 和一个 bib 文件 _citations.bib_ ，它们位于同一层目录，文件内容放在本文末尾。

tex 编译成 pdf 后是下面的效果

![](https://pic1.zhimg.com/80/v2-c31a5ac20196fbf9f439c05541ef11b8_1440w.webp)

将 tex 文件转成 docx 文件，只要使用下面的命令，其中 output.docx 是输出文件

```text
pandoc Untitled.tex -o output.docx --bibliography citations.bib
```

我们看一下 output.docx 的内容

![](https://pic3.zhimg.com/80/v2-b891af50681a38bd6a41ca0f3c7b9d62_1440w.webp)

**可以看到，公示能正常转换，但是引用没有序号，参考链接也没有序号**

另外我在 [这篇回答](https://www.zhihu.com/question/31850346) 里看到使用 pandoc-crossref 转换，其操作为

```text
pandoc  Untitled.tex -o output.docx --filter pandoc-crossref --bibliography=citations.bib
```

这样做需要安装 pandoc-crossref, 和 pandoc-citeproc，mac 下可以直接使用 brew 命令安装

## 附件

以下是 untitled.tex 的文件内容

```tex
\documentclass{article}

\begin{document}
This document is an example of BibTeX using in bibliography management. Three items 
are cited: \textit{The \LaTeX\ Companion} book \cite{latexcompanion}, the Einstein
journal paper \cite{einstein}, and the Donald Knuth's website \cite{knuthwebsite}. 
The \LaTeX\ related items are \cite{latexcompanion,knuthwebsite}. 

Here is a formula: 
$$\sum_{i=1}^{n} a_i = \pi $$
\medskip

\bibliographystyle{unsrt}
\bibliography{citations}

\end{document}

```

以下是 citations.bib 文件内容

```text
@article{einstein,
    author =       "Albert Einstein",
    title =        "{Zur Elektrodynamik bewegter K{\"o}rper}. ({German})
        [{On} the electrodynamics of moving bodies]",
    journal =      "Annalen der Physik",
    volume =       "322",
    number =       "10",
    pages =        "891--921",
    year =         "1905",
    DOI =          "http://dx.doi.org/10.1002/andp.19053221004"
}

@book{latexcompanion,
    author    = "Michel Goossens and Frank Mittelbach and Alexander Samarin",
    title     = "The \LaTeX\ Companion",
    year      = "1993",
    publisher = "Addison-Wesley",
    address   = "Reading, Massachusetts"
}

@misc{knuthwebsite,
    author    = "Donald Knuth",
    title     = "Knuth: Computers and Typesetting",
    url       = "http://www-cs-faculty.stanford.edu/\~{}uno/abcde.html"
}
```
