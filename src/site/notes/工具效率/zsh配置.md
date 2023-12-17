---
uid: null
aliases: null
tags:
  - 工具效率
source: null
created: 2023-01-22 13:33:08
updated: 2023-04-18 15:48:11
title: zsh 配置
dg-publish: true
---

# zsh 配置

## Zsh

先安装 zsh，建议把 zsh 设置成默认 shell
```
chsh -s $(which zsh)
```
## Zinit 安装

使用 zinit 来管理插件，其链接为: [zdharma-continuum/zinit: 🌻 Flexible and fast ZSH plugin manager (github.com)](https://github.com/zdharma-continuum/zinit)

按照下述步骤操作，安装常用插件即可

```
bash -c "$(curl --fail --show-error --silent --location https://raw.githubusercontent.com/zdharma-continuum/zinit/HEAD/scripts/install.sh )"

// reload zsh

zinit self-update

# Plugin history-search-multi-word loaded with investigating.
zinit load zdharma-continuum/history-search-multi-word

# Two regular plugins loaded without investigating.
zinit light zsh-users/zsh-autosuggestions
zinit light zdharma-continuum/fast-syntax-highlighting

# Snippet
zinit snippet https://gist.githubusercontent.com/hightemp/5071909/raw/

# Load powerlevel10k theme
zinit ice depth"1" # git clone depth
zinit light romkatv/powerlevel10k

# Load pure theme
zinit ice pick"async.zsh" src"pure.zsh" # with zsh-async library that's bundled with it.
zinit light sindresorhus/pure

# Load starship theme
# line 1: `starship` binary as command, from github release
# line 2: starship setup at clone(create init.zsh, completion)
# line 3: pull behavior same as clone, source init.zsh
zinit ice as"command" from"gh-r" \
          atclone"./starship init zsh > init.zsh; ./starship completions zsh > _starship" \
          atpull"%atclone" src"init.zsh"
zinit light starship/starship
```

主题
```
zinit ice depth=1; zinit light romkatv/powerlevel10k
```