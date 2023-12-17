---
aliases: null
tags: note 
source: null
created: 2022-08-19 16:12:21
updated: 2023-03-02 14:15:11
uid: null
title: Vscode 免输入密码链接远程主机
dg-publish: true
---

# Vscode 免输入密码链接远程主机

该过程只需要两步：

1、在本地主机生成密钥对

```bash
(local_host) ➜  .ssh ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/aiyolo/.ssh/id_rsa):
...
(local_host) ➜  .ssh ls
config      id_rsa      id_rsa.pub  known_hosts
```

2、把公钥 `id_rsa.pub` 文件的内容添加到远程主机文件：`~/.ssh/authorized_keys`。

```text
(remote_host) ➜  .ssh vim authorized_keys
```

note：该文件默认情况下不存在，需要手动创建，为了简单起见，先用密码连接到远程主机，再在远程主机里用 vim 编辑该文件，复制粘贴 `id_rsa.pub` 文件内容到 `~/.ssh/authorized_keys`
