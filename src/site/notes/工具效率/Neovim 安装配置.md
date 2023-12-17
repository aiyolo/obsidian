---
uid: null
aliases: null
tags:
  - 工具效率
publish: https://blog.csdn.net/aiyolo/article/details/128567132?spm=1001.2014.3001.5501
source: null
created: 2023-01-05 16:47:11
updated: 2023-03-07 16:25:54
title: Neovim 安装配置
dg-publish: true
---

# Neovim 安装配置

最近试用了几种不同的 neovim 配置，经过多方比较，最终留下了目前在 github 中 star 排名第一的 LunarVim。整体来说不需要太多的操作，主要需要一个好的上网环境。

配置完成之后，它有以下几点比较吸引我。

- 支持语法提示，错误提示  
![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303071628806.png)
- 可以跳转到定义，包括库文件  
![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303071628807.png)

现将折腾经历记录如下。

## 安装 Neovim

建议从源码安装，在 ubuntu 里使用 apt 安装的 neovim 版本太低，不符合 Lunarvim 的要求

```
wget https://github.com/neovim/neovim/releases/download/stable/nvim-linux64.deb
sudo apt install ./nvim-linux64.deb
```

## 安装 Rust

完整安装 LunarVim 需要使用 cargo 安装一些插件

```
curl https://sh.rustup.rs -sSf | sh
```

## 安装 LunarVim

链接：[Installation | LunarVim](https://www.lunarvim.org/docs/installation)

```
// 安装
bash <(curl -s https://raw.githubusercontent.com/lunarvim/lunarvim/master/utils/installer/install.sh)
```

如果按照成功，会提示以下信息

```
Thank you for installing LunarVim!!
You can start it by running: /home/aiyolo/.local/bin/lvim
Do not forget to use a font with glyphs (icons) support [https://github.com/ryanoasis/nerd-fonts]
```

卸载命令

```
bash ~/.local/share/lunarvim/lvim/utils/installer/uninstall.sh
```

## 使用 LunarVim

使用的话要使用 lvim 命令进入编辑器，在编辑器里的 normal 模式下，输入 `:<TAP>` 即冒号 +TAB 键会所有支持的命令，常见的命令如下

```
// 在编辑器里更新 
:LvimUpdate 

// 或者在命令行更新
lvim +LvimUpdate +q

// 更新Lvim核心插件
:LvimSyncCorePlugins

// 安装语言服务器，可以替换成其他语言
:LspInstall c++

// install Tressitter
TSInstall cpp
```

lvim 的配置文件的位置在

```
~/.config/lvim/config.lua
```

## 其他替代选择

[LunarVim/nvim-basic-ide: 🪨 This is my attempt at a basic stable starting point for a Neovim IDE. (github.com)](https://github.com/LunarVim/nvim-basic-ide)

```
git clone https://github.com/LunarVim/nvim-basic-ide.git ~/.config/nvim
:checkhealth
sudo apt install xsel # for X11
sudo apt install wl-clipboard # for wayland
sudo apt install python-venv
pip install pynvim
npm i -g neovim
sudo apt install ripgrep
:TSInstall c++ 
:LSPInstall cpp
:DIInstall ccpp_vsc

```

### Packer.vim 安装插件

进入 ` ~/.config/nvim/lua/user/plugins.lua `

```
// 在下面添加需要的插件
-- My plugins here
use "olimorris/onedarkpro.nvim"
```

### 使用语言服务器

在 `mason.lua` 里添加下面一行

```
local servers = {
	"sumneko_lua",
	"cssls",
	"html",
	"tsserver",
	"pyright",
	"bashls",
	"jsonls",
	"yamlls",
    "clangd" -- 添加
}
```

### 快捷键

```
c-\ 打开终端
```
