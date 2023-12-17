---
uid: 
aliases: null
tags:
  - obsidian
source: null
created: 2023-02-09 10:29:59
updated: 2023-03-02 12:23:08
title: Obsidian 使用 Livesync 同步数据
dg-publish: true
---

# Obsidian 使用 Livesync 同步数据

之前一直使用 icloud 同步我的 obsidian 笔记，同时定期使用 git 备份笔记。但是前段时间因为在 ios 上误删了一个文件夹，导致我的 icloud 桌面端和手机端的笔记不一致。使用过 icloud 同步笔记的同学，想必也都经历过一份文件在桌面端和手机端同步时，出现多个副本的情况。还有一种同步方案是使用 remotely sync，该插件目前的使用体验也是一言难尽。

最近了解了一下 livesync, 试用了一下目前感觉体验非常好。其优点包括：

- 免费
- 支持实时同步
- 卡同步的情况比较少

现在将我配置的过程记录下来分享给大家。整个过程主要参考了：

- [Fly.io for self hosting CouchDB · Discussion #85 · vrtmrz/obsidian-livesync (github.com)](https://github.com/vrtmrz/obsidian-livesync/discussions/85)
- [Obsidian 免费的实时同步服务 (irithys.com)](https://irithys.com/p/obsidian-livesync/)

注意，以下配置过程需要提前准备一张 visa 的信用卡，绑定到 flyio 账号，不会扣费。需要绑定账号信用卡是因为 flyio 担心免费资源被滥用，详情见 [Is it free ? Getting Error: We need your payment information to continue! - Questions / Help - Fly.io](https://community.fly.io/t/is-it-free-getting-error-we-need-your-payment-information-to-continue/8871)

## 1 配置过程

### 1.1 服务端配置

#### 1.1.1 fly.io

注册 https://fly.io/ 账号，可以使用 github 账号，然后在本地安装 flyio 命令行，请根据自己的系统在官网找到对于的安装命令

```
// 安装flyio cli
iwr https://fly.io/install.ps1 -useb | iex
```

登录账号

```
// 登录
flyctl auth login 
```

使用 couchdb 镜像

```
flyctl launch --image couchdb
```

注意！如果你的账号没有绑定银行卡，那么此步会让你添加银行卡

```
PS D:\flyio\couchdb> flyctl launch --image couchdb
Creating app in D:\flyio\couchdb
Using image couchdb
? Choose an app name (leave blank to generate one): couchdb

? Choose an app name (leave blank to generate one): couchdb
automatically selected personal organization: aiyolo
? Choose a region for deployment: Tokyo, Japan (nrt)
Error We need your payment information to continue! Add a credit card or buy credit: https://fly.io/dashboard/aiyolo/billing
```

再次尝试命令，输入服务器的位置，这个离自己越近越好，其他都选择 No

```
PS D:\flyio\couchdb> flyctl launch --image couchdb
Creating app in D:\flyio\couchdb
Using image couchdb
? Choose an app name (leave blank to generate one):

? Choose an app name (leave blank to generate one):
automatically seleolo
? Choose a region for deployment: Hong Kong, Hong Kong (hkg)
Created app frosty-glitter-2627 in organization personal
Admin URL: https://fly.io/apps/frosty-glitter-2627
Hostname: frosty-glitter-2627.fly.dev
? Would you like to set up a Postgresql database now? No
? Would you like to set up an Upstash Redis database now? No
Wrote config file fly.toml
? Would you like to deploy now? No
Your app is ready! Deploy with `flyctl deploy`
```

创建卷，其中 --region 后面填刚才选择的服务器位置，--size 后面填容器卷的大小，比如 2 表示 2g，注意！flyio 免费提供的空间为 3g

```
PS D:\flyio\couchdb> fly volumes create --region hkg couchdata --size 2
        ID: vol_x915grndpj14n70q
      Name: couchdata
       App: frosty-glitter-2627
    Region: hkg
      Zone: be08
   Size GB: 2
 Encrypted: true
Created at: 09 Feb 23 02:59 UTC
```

打开文件夹下的 `fly.toml`，对照下面内容修改, 主要是 ` env `, ` mounts `, ` experimental `, ` services. ports ` 这几个地方。  
注意！最好添加一个端口号为 5984 的服务端口，不然同步容易发生中断

```
app = "purple-wind-1262"
kill_signal = "SIGINT"
kill_timeout = 5
processes = []

[build]
  image = "couchdb"

[env]
  COUCHDB_USER="your_name" ## 修改

## 修改
[mounts]
  source="couchdata"
  destination="/opt/couchdb/data"

[experimental]
  allowed_public_ports=[] ## 修改
  auto_rollback = true

[[services\|services]]
  http_checks = []
  internal_port = 5984 ## 修改
  processes = ["app"]
  protocol = "tcp"
  script_checks = []
  [services.concurrency]
    hard_limit = 25
    soft_limit = 20
    type = "connections"

  [[services.ports\|services.ports]]
    force_https = true
    handlers = ["http"]
    port = 80

  [[services.ports\|services.ports]]
    handlers = ["tls", "http"]
    port = 443
    
## 修改
  [[services.ports\|services.ports]]
    handlers = ["tls", "http"]
    port = 5984

  [[services.tcp_checks\|services.tcp_checks]]
    grace_period = "1s"
    interval = "15s"
    restart_limit = 0
    timeout = "2s"
```

设置密码

```
flyctl secrets set COUCHDB_PASSWORD=your_password
```

部署

```
flyctl deploy
```

如果报错了，执行以下命令之后，再重新部署

```
fly ips allocate-v4
```

如果以上都顺利的话，那么 deploy 之后，命令行能正常退出，然后执行以下命令，在浏览器打开 app 管理端

```
flyclt open
```

在 `https://XXX.dev ` 后面加上 ` /_utils `，使用刚才配置的用户名和密码登录到管理界面，

- 在菜单第二项里找到 `Configure a Single Node ` 并点击
- 在菜单第四项里找到 `CORS `, 点击 enable，并在下面选择 `all damains `

至此服务端的配置就完成了

#### 1.1.2 IBM Cloud (Optional)

[注册 IBM Cloud](https://cloud.ibm.com/registration)

[obsidian-livesync/setup_cloudant.md at main · vrtmrz/obsidian-livesync (github.com)](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/setup_cloudant.md)

### 1.2 桌面端配置

首先下载在 obsidian 插件商店搜索 livesync 插件。第一次配置时，我们在 `setup wizard` 中直接点击下面选项即可  
![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303031135209.png)  
然后在 `remote database configuration` 中，依次填写服务端的配置信息，database 可以随便取个名字，它会自动在云端创建  
![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303031135210.png)

填写完后，点击 test 不出意外会显示 `connected to obsidian`，点击 check 之后，出现的 `?` 后点击 `fix` 直到全部变成对钩

![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303031135211.png)

然后开启在编辑器内显示同步进度（`show status inside editors`），这样做方便在手机端观察同步进度  
![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303031135212.png)  
最后一步，点击 `live sync`  
![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303031135213.png)

不出意外的话，状态栏的同步进度条开始动了, 点击左侧的按钮可以看到同步状态，像我这样配置日志里不会显示请求连接失败，否则的话可能会卡同步了

![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303031135214.png)

### 1.3 手机端配置

在桌面端的 `setup wizzard` 点击 `copy setup uri`，然后在弹出的对话框中输入任意密码，输入完后把生成的 uri 粘贴到手机端

提前手机端下载 livesync，并且启用，在浏览器中输入生成的 uri，会跳转到 obsidian

在弹出的对话框中选择导入桌面端的 livesync 配置即可完成配置。

此时返回主界面，应该可以看到同步进度了  
![]()  
![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303031135215.png)

至此整个过程完成。

唯一比较遗憾的大概是云端的存储容量有限，如果有一个 nas，在 nas 上配置 couchdb，那就更好了。

## 2 参考资料

1. [Fly. io for self hosting CouchDB · Discussion #85 · vrtmrz/obsidian-livesync (github. com)]( https://github.com/vrtmrz/obsidian-livesync/discussions/85 )
2. [Obsidian 免费的实时同步服务 (irithys.com)](https://irithys.com/p/obsidian-livesync/)
