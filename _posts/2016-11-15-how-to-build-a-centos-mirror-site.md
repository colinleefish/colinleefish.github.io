---
layout: post
title: "大规模封闭的生产环境？你需要一个镜像站"
tags: yum,repository,centos,infrastructure,rsync,mirrorsite,
unsplash_source_number: 8
excerpt: 搭建个镜像站，再不怕打补丁。
---

在网络上，每一个 Linux 分发版本的安装教程在文末都不忘提醒读者：“请使用 yum（或者 apt）为你的系统进行更新，安装最新的更新和补丁，以修复安全漏洞和获取最新功能。”更新的过程也基本上一番风顺：用户无需修改配置，便能轻易地连接到最快的站点，下载到最新的更新。仰仗互联网的开放和便捷，这一切都显得如此理所应当。

然而。

如果你是一个规模庞大且封闭的生产环境的管理员，你的设备没有公网的访问权限，获取更新、甚至哪怕是装一个 vim 的过程都可能变得十分曲折。你可能会想到从光盘中安装软件包，但遇到关键更新、或者严重 Bug 时，这种方法又失效了。

如果你也遇见了上面的问题，那么在你自己管理的环境中，很可能也需要一个随时更新的镜像站。本文就来分享一下，如何搭建一个镜像站，以及如何高效的利用这个镜像站。

鉴于生产环境中的 Linux 分发版本普及度，本文介绍的镜像站安装和使用主要针对 CentOS；Ubuntu 另有其独特的搭建思路，但因为应用较少，本文暂不涉及。

### 1. 搭建镜像站

#### 1.1 初识公共镜像站

在 CentOS 的官网上，可以找到所有各个镜像站的源头，也就是最权威的官方站点：[http://mirror.deppon.com](http://mirror.deppon.com)，里面保存了 CentOS 各个版本的镜像文件和 rpm 包。但是，世界上绝大多数的镜像站，同步的都是这个官方站点的一个子目录：[http://mirror.depon.com/centos](http://mirror.deppon.com/centos)。这个“centos”子目录有一个特点，就是只提供了各个大版本下面最新的镜像文件和最新的 rpm 包，比如当前的 5.11，6.8 和 7.2。旧版本的文件夹已经空了。如果需要下载旧版本的镜像文件，则需要去另外一个地方：[http://vault.centos.org](http://vault.centos.org)。这里面将各个版本的 CentOS 镜像都做了归档，可以通过链接和种子的方式下载到。

虽然镜像有小版本之分，但 rpm 安装包则是大版本内通用的，比如你的系统版本是 CentOS 6.3，使用 CentOS 6 的最新镜像源是完全没有问题的，即便最新版本已经到了 6.8。因此通用的镜像站，已经完全能够满足日常更新的需要了。

还有一件不得不提的事情：有一个叫做 [EPEL（Extra Packages for Enterprise Linux）](https://fedoraproject.org/wiki/EPEL)的软件源，是由 Fedora 用户组维护的面向红帽系列分发版的软件源。里面包含了诸多优质的软件，Nginx 就是其中之一。一般地，我们也会将这个软件源同步在自己的环境中。

#### 1.2 使用 Nginx, rsync 和 cron 搭建镜像站

下面我们就来聊一聊如何搭建一个简约的镜像站。整个过程大概仅需 20 分钟。

首先我们要安装 Nginx。而正如前文所说，安装 Nginx 首先需要配置 EPEL 源。然后再安装 Nginx。

```
$ sudo yum install -y epel-release
$ sudo yum install -y nginx
```

然后，我们要为镜像站的文件规划一个合适的存放路径，用来接收更新，以及对外提供服务。本例中，我们在根目录下创建一个新文件夹`/mirrors`，文件夹内的结构如下：

```
/mirrors
   |
   |--/data
   |     |
   |     |--/centos
   |     |
   |     |--/epel
   |
   |--/scripts
   |
   |--/log
```

在 Nginx 中，将 Nginx 的根目录设置为 `"/mirrors/data"`，同时还要开启 `autoindex`。

为我们的镜像站安好“新家”之后，接下来就要把他们“请来”了。这里我们使用了 `rsync`，一个 Linux 下的高效的文件同步工具。

在 CentOS 网站中，有一个专门提供第三方镜像站的页面：https://www.centos.org/download/mirrors/。用户可以在这个页面查询到有哪些镜像站，以及每一个镜像站是否支持 rsync —— 不是所有的镜像站都开启了这个功能。使用 HTTP 同步则会相对麻烦一点。

这里我们推荐您使用[清华大学](https://mirrors.tuna.tshinghua.edu.cn)和[中国科学技术大学](https://mirrors.ustc.edu.cn)的镜像站。这两个镜像站相对稳定，提供了多种丰富的镜像库，而且还支持 rsync 协议。我们这里就以清华大学的镜像站为例，编写如下两个 rsync 命令：

```
#!/bin/bash

/usr/bin/rsync -rltz4 --progress --delete --log-file=/var/log/mirrors/tuna-centos.log rsync://mirrors.tuna.tshinghua.edu.cn/centos/ /opt/mirrors/data/centos/
```

```
#!/bin/bash

/usr/bin/rsync -rltz4 --progress --delete --log-file=/var/log/mirrors/tuna-epel.log rsync://mirrors.tuna.tshinghua.edu.cn/epel/ /mirrors/data/epel/
```

#### 1.4 了解更多

### 2. 使用镜像站

### 2.1 “.repo” 文件范例

### 2.2 自动配置脚本

### 2.3 插件的取舍

---

以上便是我标记自己使用的云主机的方法，感谢阅读。您是否有更好的方法和实践？欢迎在下面留言。
