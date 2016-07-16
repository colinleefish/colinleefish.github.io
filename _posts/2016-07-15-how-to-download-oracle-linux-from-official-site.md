---
layout: post
title: "如何从 Oracle 官网下载 Oracle Linux"
tags: oracle linux, linux
unsplash_source_number: 6
excerpt: 本文介绍了如何在 Oracle 官网下载 Oracle Linux，并根据国内网络现状给出了应对办法。
---

Oracle Linux 是 Oracle 公司基于 Red Hat Enterprise Linux 重新打包的 Linux 分发版本。这个分发版本保持了和 RHEL 系列较高的兼容性，并且拥有大多数主流服务器厂商的硬件支持。Oracle Linux 提供 RHEL 原生的内核，还提供了 Oracle 自行打包的 *Unbreakable Enterprise Kernel*，加入了对 OLTP，InfiniBand，SSD，异步IO等支持。

可能有些人会误以为 Oracle Linux 也是收费软件，但事实并非如此：下载、使用 Oracle Linux，以及使用它的[YUM源](http://public-yum.oracle.com)都是[免费的](http://www.oracle.com/us/technologies/linux/competitive-335546.html)。寻求[官方技术支持](http://www.oracle.com/us/technologies/027617.pdf)才需要订阅付费项目。在国内，有公司选用了 Oracle Linux 来运行 Oracle 数据库。

Oracle Linux 紧跟 RHEL 的发布周期，在 RHEL 发布新版本之后很快便会发布，速度甚至超过了 CentOS。可以说也是一个非常积极的 RHEL 系分发版。但在国内，可能由于知名度不如 CentOS，以及鲜有稳定的下载源，获取该系列镜像文件有不小的困难。由此，本文介绍了如何在 Oracle 官网下载 Oracle Linux，并根据国内网络现状给出了应对办法。

（请注意，Oracle 的官方网站历经数次变动，本文中介绍的下载方式也很可能在未来出现变化。因此还请与时俱进，以 Oracle 官网为准。）

### 1. Oracle Linux 的获取

#### 1.1 注册 Oracle 账号

Oracle 的官方网站为：[https://www.oracle.com/](https://www.oracle.com/)。在该网站下载产品一般都需要 Oracle 的账号。我们这里就先来注册一个。

![Oracle 官网](https://i.imgur.com/FgpHhwA.png)

在 Oracle 的首页，点击**“Sign IN/Register”**。转到登录页面。

![登录页面](https://i.imgur.com/ajmeMgb.png)

在登录页面的右侧，点击灰色的**“Create Account”**按钮。

![填入注册信息](https://i.imgur.com/77iNMWQ.png)

在此处填入注册信息，然后点击红色的**“Create Account”**按钮。完成注册之后，刚刚填写的注册邮箱会收到一封确认邮件。建议在注册之后立即前往邮箱确认，否则无法进行后续的下载。

![确认注册邮件](https://i.imgur.com/Yu8Vinq.png)

点击**“Verify Email Address”**，完成注册的全过程。

![注册完成](https://i.imgur.com/R9d7LkM.png)

#### 1.2 登录并找到下载位置

完成了注册环节，现在便可以前往下载了。回到刚刚的登录页面，使用注册的邮箱登录。

![登录](https://i.imgur.com/8yVae1V.png)

然后在首页找到**“Download”**选项卡。鼠标悬停在上面便会展开 Oracle 提供的下载，我们在这里点选**“Linux and Oracle VM”**，进入 Oracle 的交付云站点（Delivery Cloud）。

![Download 选项卡](https://i.imgur.com/S8wlK1P.png)

在 Oracle Delivery Cloud 中可以根据分类搜索需要的内容。我们在搜索框中输入**“Linux”**，搜索框会自动响应，弹出可选的下载内容。

![搜索 Linux](https://i.imgur.com/7sFwvWx.png)

然后在后面选择处理器架构，这里我们选着“x86 64 bit”。

![处理器架构](https://i.imgur.com/MksaM5Tr.png)

完成下载内容和处理器架构的选择后，在列表里便会显示出可以提供下载的内容。此时便可以点击**“Continue”**了。

![完成下载项选择，进入下一步](https://i.imgur.com/rAP6Lee.png)

默认情况下，下载列表中仅展示了当前的最新版本。

![仅显示最新版本的下载列表](https://i.imgur.com/tyQynsN.png)

如果需要下载历史版本，可以点选版本名下方的蓝色小字**“Select Alternate Release...”**。

![历史版本列表](https://i.imgur.com/E3jRkhe.png)

选择完毕之后，可以点选**“Continue”**，进入下载前的最后阶段。首先要同意服务条款。

![服务条款](https://i.imgur.com/PLghYUP.png)

然后就进入了最终的下载界面。在这里您可以点击**“Download”**，但由于国内的网络环境限制，下载速度很难得到保证，还经常容易断线。好在 Oracle 很贴心的提供了一个叫做“**WGET Options”**的下载方式。这种下载方式提供给用户一个 Shell 脚本，用户在自己的 Linux 机器上执行这个脚本，输入账号和密码，便可以下载了。等于是把用户验证和下载的工作从网页端移到了命令行端。我们此处便选用这种下载方式。在窗口的右下角，我们点击**“WGET Options”**。

![最终的下载页面和WGET方式](https://i.imgur.com/8SF4Fv5.png)

然后在弹出的窗口点击**“Download .sh”**。接下来便可以开始下载了。

![下载WGET方式的Shell脚本](https://i.imgur.com/hxY5hPo.png)

#### 1.3 使用 WGET 下载方式在 Linux 命令行中下载

我们将下载好的脚本传入自己的 Linux 环境中，便可以执行下载了。但是，刚刚下载到的脚本在执行后会提示用户输入账号和密码，当我们输入之后，敲下回车，后台开始下载时，可能还会卡住不动。因此我们在这里简单修改一下脚本，把账号密码写进去。

在下面的展示中，我们演示了如何关闭账号密码的输入，以及如何手动添加自己的账号密码。

<script type="text/javascript" src="https://asciinema.org/a/0k5tpa0c7jmq3o3ghrxmjo2df.js" id="asciicast-0k5tpa0c7jmq3o3ghrxmjo2df" async></script>

开始执行该脚本。这里我们将该脚本放入后台执行，确保在退出 Shell 之后仍可以进行下载。

<script type="text/javascript" src="https://asciinema.org/a/cn1o2e9tn2q85kq4ajavvvarb.js" id="asciicast-cn1o2e9tn2q85kq4ajavvvarb" async></script>

此时 Oracle Linux 已经开始在后台下载了。

#### 1.4 关于在中国大陆下载速度慢的问题

如果遇到在大陆下载速度慢的问题，我们建议购买一台位于国外的云主机，本例中便使用了一台 [DigitalOcean](https://www.digitalocean.com/?refcode=467ce24ba101) 位于新加坡的云主机实例。在下载完成之后，可以在该机器上安装 HTTP 服务器，再通过 HTTP 下载到本地。经过实测，从云主机下载到本地的速度可以稳定在 1MB/s ，有时能达到 5MB/s（民用宽带，100Mbps）。

![通过云主机下载的速度相对让人满意](https://i.imgur.com/OqBrqKK.png)

### 2. Oracle Linux 的 YUM 源

正如文章开头所讲到的一样，Oracle 官方很贴心的提供了一个 YUM 源，并且可以免费使用。地址为：[http://public-yum.oracle.com/](http://public-yum.oracle.com/)。该网站上有具体的配置文档，请参阅以获得更多内容。

<div style="position: relative; width: 100%; height: 0; padding-bottom: 75%;"><iframe src="https://vizzlo.com/embed/colinlee/untitled-document" style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border:none; overflow:hidden;" allowTransparency="false" scrolling="no" frameborder="0"></iframe ></div>

---

以上便是一次 Oracle Linux 的完整下载过程等内容。欢迎在下方的评论区留言。