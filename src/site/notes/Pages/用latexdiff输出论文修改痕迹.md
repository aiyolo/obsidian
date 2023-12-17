---
aliases: null
tags:
  - latex
source: null
created: 2020-10-22 20:55:00
updated: 2023-03-07 16:23:35
uid: null
title: 用 latexdiff 输出论文修改痕迹
dg-publish: true
---

# 用 Latexdiff 输出论文修改痕迹

第一步，我们先在 overleaf 上创建一个示例项目，文件名为 main.tex，pdf 渲染效果如下

![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303071545790.webp)

第二步，我们随便在 main.tex 上做点修改，把修改后的文件取个名字叫做 revise.tex，修改后的效果如下

![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303071545791.webp)

乍一看不太容易看出来哪里作出了修改，我们现在用 latexdiff 处理一下，latexdiff 在 tex 套装里默认安装了，可以在本地执行，网上也有在线版的，但是可能不安全。

现在在本地终端里执行一下命令：

```text
latexdiff main.tex revise.tex > diff.tex
```

在 overleaf 上看一下输出的 diff.tex 文件，可以看到修改的痕迹还是比较直观的，nice

![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303071545792.webp)
