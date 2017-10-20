---
layout: post
title: "大规模封闭的生产环境？你需要一个镜像站"
tags: yum,repository,centos,infrastructure,rsync,mirrorsite
unsplash_id: B5j_W25e1JU
excerpt: 搭建个镜像站，再不怕打补丁。
---

在网络上，每一个 Linux 分发版本的安装教程在文末都不忘提醒读者：“请使用 yum（或者 apt）为你的系统进行更新，安装最新的更新和补丁，以修复安全漏洞和获取最新功能。”更新的过程也基本上一番风顺：用户无需修改配置，便能轻易地连接到最快的站点，下载到最新的更新。仰仗互联网的开放和便捷，这一切都显得如此理所应当。

然而。

如果你是一个规模庞大且封闭的生产环境的管理员，你的设备没有公网的访问权限，获取更新、甚至哪怕是装一个 vim 的过程都可能变得十分曲折。你可能会想到从光盘中安装软件包，但遇到关键更新、或者严重 Bug 时，这种方法又失效了。

如果你也遇见了上面的问题，那么在你自己管理的环境中，很可能也需要一个随时更新的镜像站。本文就来分享一下，如何搭建一个镜像站，以及如何高效的利用这个镜像站。

鉴于生产环境中的 Linux 分发版本普及度，本文介绍的镜像站安装和使用主要针对 CentOS；Ubuntu 另有其独特的搭建思路，但因为应用较少，本文暂不涉及。

### 1. 搭建镜像站

#### 1.1 初识公共镜像站

在 CentOS 的官网上，可以找到所有各个镜像站的源头，也就是最权威的官方站点：[http://mirror.centos.org](http://mirror.centos.org)，里面保存了 CentOS 各个版本的镜像文件和 rpm 包。但是，世界上绝大多数的镜像站，同步的都是这个官方站点的一个子目录：[http://mirror.centos.org/centos](http://mirror.deppon.org/centos)。这个“centos”子目录有一个特点，就是只提供了各个大版本下面最新的镜像文件和最新的 rpm 包，比如当前的 5.11，6.8 和 7.2。旧版本的文件夹已经空了。如果需要下载旧版本的镜像文件，则需要去另外一个地方：[http://vault.centos.org](http://vault.centos.org)。这里面将各个版本的 CentOS 镜像都做了归档，可以通过链接和种子的方式下载到。

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
   |--/repo
   |
   |--/log
```

在 Nginx 中，将 Nginx 的根目录设置为 `"/mirrors/data"`，同时还要开启 `autoindex`。下面是一个 Nginx 配置文件样例，将下面这段配置复制在 `/etc/nginx/conf.d/mirror-site.conf` 中，重启 Nginx 后即可生效。

```
server {
    listen       80;
    server_name  {服务器的名称};


    # 镜像站的主要数据
    location / {
        root        /mirrors/data/;
        autoindex   on;
    }

    # repo文件
    location /repo {
        root          /mirrors;
        autoindex     on;
    }

    # 自动配置脚本
    location /sh {
        root          /mirrors;
    }
}
```

为我们的镜像站安好“新家”之后，接下来就要把他们“请来”了。这里我们使用了 `rsync`，一个 Linux 下的高效的文件同步工具。

在 CentOS 网站中，有一个专门提供第三方镜像站的页面：[https://www.centos.org/download/mirrors/](https://www.centos.org/download/mirrors/)。用户可以在这个页面查询到有哪些镜像站，以及每一个镜像站是否支持 rsync —— 不是所有的镜像站都开启了这个功能。使用 HTTP 同步则会相对麻烦一点。

这里我们推荐您使用[清华大学](https://mirrors.tuna.tshinghua.edu.cn)和[中国科学技术大学](https://mirrors.ustc.edu.cn)的镜像站。这两个镜像站相对稳定，提供了多种丰富的镜像库，而且还支持 rsync 协议。我们这里就以清华大学的镜像站为例，编写如下两个 rsync 命令：

**/mirrors/scripts/tuna-centos.sh**

```
#!/bin/bash

/usr/bin/rsync -rltz4 --progress --delete --log-file=/mirrors/log/tuna-centos.log rsync://mirrors.tuna.tsinghua.edu.cn/centos/ /mirrors/data/centos/
```

**/mirrors/scripts/tuna-epel.sh**

```
#!/bin/bash

/usr/bin/rsync -rltz4 --progress --delete --log-file=/mirrors/log/tuna-epel.log rsync://mirrors.tuna.tsinghua.edu.cn/epel/ /mirrors/data/epel/
```

我们使用的参数中，`r`表示递归同步，也就是将文件夹内的所有文件都同步，`l`表示同步链接文件，`t`表示保留时间戳，`z`表示开启压缩；`--progress`和`--delete`分别表示使用现实详细进度，和删除源地址中没有的文件（也就是和上游保持完全的一致），`--log-file`能指定一个日志位置。

我们将其保存成两个 Shell 脚本。放在 /mirrors/scripts 文件夹中。

随意执行一下，可以看见 rsync 已经在欢快地进行同步了。

<script type="text/javascript" src="https://asciinema.org/a/9ok3w1rzo8t1xsx575zuu1rer.js" id="asciicast-9ok3w1rzo8t1xsx575zuu1rer" async></script>

注意，这里仅是一个演示，一切的操作都是在 root 用户名下进行的；如果您是在实际环境中搭建，强烈建议您使用普通用户进行操作。

接下来我们将这个脚本放在 cron 中定时执行即可。

```
[root@localhost ~]$ crontab -l
0 23 * * 1,3,5 sh /mirrors/scripts/epel-ustc.sh >> /dev/null
0 23 * * 2,4,6 sh /mirrors/scripts/centos-ustc.sh >> /dev/null
```

第一次同步需要耗费的时间较长。如果您的网络质量不理想，还有可能断掉，所以请在第一次同步时多加注意。

好了，现在你的专属镜像站已经搭建完成了。如果你愿意，还可以为它制作一份首页和帮助文档，方便他人使用。

#### 1.4 了解更多

清华大学的 TUNA 为他们的镜像站制作了一个非常漂亮的[前端](https://github.com/tuna/mirror-web)，以及一个 Golang 写成的[同步工具](https://github.com/tuna/tunasync)，作为 cron 的替代品。代码托管在了 Github。

### 2. 使用镜像站

#### 2.1 “.repo” 文件范例

有了镜像站只是第一步，接下来我们就要聊一聊如何使用了。

众所周知，YUM 源的信息都会存放在 `/etc/yum.repos.d/` 这个文件夹内，以后缀为 `.repo` 的形式存在。我们可以预先写好标准的 `.repo` 文件，在部署机器的时候直接下载，省时省力。

最简单的方法是找一个系统自带的 `.repo` 文件。然后我们把它改成我们的内容。下面就是用在 CentOS 7 的原始 `CentOS-Base.repo` 文件的一部分。

```
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

每一段的以下两个地方都需要我们修改。

1. 注释掉 `mirrorlist` 一行。这个地方会帮我们寻找最快的镜像站，但仅适用于能连外网的情况。我们已经有了我们的镜像站，所以这一行已经不需要了；还要
2. 打开 `baseurl` 这一行。并且将域名修改为我们刚刚建立的镜像站地址。其他的东西就不用修改了。

如果有的段落里有 `enabled=0` 的，也无需做改动。仅在需要时临时打开即可。

有的运维人员可能喜欢将 `gpgcheck` 设置为 `0`，也就是不用密钥校验软件包的真实性。但这样的做法有非常大的风险性，因此我们还是建议开启这个功能。

EPEL 的 `.repo` 文件获取方式也很简单。只要执行了 `yum install epel-release`。你的 `/etc/yum.repos.d/ `文件夹下就会有 `epel.repo` 和 `epel-testing.repo` 这两个文件。要修改的内容和上面的大致相同。但有一点区别是，如果使用了 EPEL 源，且打开了 `gpgckech` 在首次使用时，YUM 会询问用户是否导入配置中说明的密钥。

#### 2.2 自动配置脚本

我们准备好了 `.repo` 文件，可是每次配置起来依然是一件麻烦事。索性写一个脚本吧。

<script src="https://gist.github.com/colinleefish/0087e06722cd4377dc46f635eb686ece.js"></script>

我们将这个脚本就命名为 `sh`，放在 `/mirrors` 下。以后在配置的时候。只要执行

```
curl -sS http://mirrors.example.com/sh | bash
```

就可以快速配置好 YUM 源了。

#### 2.3 插件的取舍

在我们使用了内部的镜像站之后，便可以禁用掉一些不再适用的 YUM 插件了。比如 `fastestmirror`：如果配置了 `mirrorlist`，这个插件会帮我们测算最快的镜像站。但我们此时已经无需再测速了，在 `/etc/yum/pluginconf.d/fastestmirror.conf` 中，将 `enabled` 改为 `0`即可。

---

以上便是一些搭建和使用镜像站的心得，感谢阅读。欢迎在下面留言。
