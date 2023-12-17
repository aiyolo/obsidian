---
aliases: null
tags: deep_learning
source: null
uid: null
title: Colab 使用教程
updated: 2023-03-09 17:24:08
created: 2020-05-13 13:24:46
author: aiyolo
dg-publish: true
---

# Colab 使用教程

## 在 colab 中使用 google drive

- 谷歌硬盘的挂载位置 `/content/drive/My Drive`
- 新建的 notebook 默认位于在此文件夹下
- 导入的公开的 notebook 位于 Colab Notebooks 文件夹下 ![]( https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303091719356.png)

## 切换笔记本所在位置

```
import os
os.chdir('/content/drive/My Drive')
```

## 在 colab 中使用 conda

```
!pip install -q condacolab
import condacolab
condacolab.install()
```

## 使用 shell

```
!pip install colab-xterm
%load_ext colabxterm
%xterm

// 或者
!bash
```

## 环境变量

```
%env OPENAI_API_KEY={your_key}
```

## References

- [How to install / use Conda on Google Colab (inside-machinelearning.com)](https://inside-machinelearning.com/en/how-to-install-use-conda-on-google-colab/)
