---
layout: post
title: "Vagrant: 一种性感的虚拟机使用方式"
tags: vagrant,virtualbox,ansible,provisioning,test environment
unsplash_id: _fYBsiQzMHI
excerpt: Vagrant 能够让用户以一种可定义、可分享的方式来创建虚拟机，并且还能在公网范围进行 Web 或 SSH 共享。“环境不同”导致运行失败的情况再也不会发生了。
---

开发需要在各种系统上进行开发任务，运维则需要在各种系统上学习工具使用。因此，虚拟机恐怕也是 IT 人员最常使用的工具之一了。最常用的虚拟化工具有 VMware Workstation 和 VirtualBox 等等，虽然底层的实现各有不同，但具体的使用方法则非常相近：打开工具，为虚拟机创建一块存储空间，配置一下性能参数，在界面中安装系统，接下来就可以使用了。很多时候，为了满足虚拟机重复使用的需要，各种虚拟机工具中还会带有“模版”功能和“快照”功能，前者可以方便地让用户创建出标准的虚拟机，后者可以让用户快速的回复到以前的状态。

虚拟化为 IT 人员提供了极大的便利。但是，人们很快便不满足此了。具体说来，使用虚拟机的时候，用户往往会遇到下面的问题：

- 虚拟化工具学习成本：有的时候，配置虚拟机的步骤比较繁琐，普通用户上手可能比较困难。
- 环境无法共享：一个人创建的环境和另一个人创建的环境很难完全一致，在 Alice 的虚拟机中跑的好好的代码，到了 Bob 的机器上可能怎么也运行不起来。
- 虚拟化平台不统一：不同用户、不同场景使用的平台可能不一样，在不改变工作流程的前提下，很难跨跃多个平台来进行虚拟化管理和资源使用。

后来，上面的问题终于有了解决的方法：**[Vagrant](https://www.vagrantup.com)**。

Vagrant 是一套用 Ruby 编写的工具，它为用户提供了一种标准化、可定义、可分享的虚拟化平台使用模式。使用 Vagrant，用户可以使用文本文件来“定义”一个虚拟机，快速创建出自己想要的环境，还可以将其分享给其他人；此外，Vagrant 还支持多款虚拟化平台，VMware、Virtualbox、AWS 甚至是 Docker 都可以，用户再也不需要为对接多个虚拟化平台而犯愁了。

本文旨在向读者介绍 Vagrant，带领读者进行 Vagrant 快速上手，然后以搭建一台 ELK 服务器的完整案例来展示 Vagrant 的便捷和强大。

### 1. Vagrant 快速上手 

在这里，我们将 Vagrant 进行一次简单的快速上手，在阅读完这个部分之后，您将能够自己通过命令创建一台虚拟机，并执行常见的操作。

部分操作步骤和脚本来自[官方文档](https://www.vagrantup.com/docs/)，下文不会单独指出。

笔者使用的环境为 macOS Sierra 10.12.3 ，Vagrant 1.9.1 以及 VirtualBox 5.1.14。

#### 1.1 前期准备和安装

Vagrant 将其支持的各种虚拟化平台称为 “Provider”。我们在本次的快速上手中选择了 VirtualBox 作为后端虚拟化平台。因此，您首先需要安装最新版本的 VirtualBox 来搭配 Vagrant。请访问[这里](https://www.virtualbox.org/wiki/Downloads)下载最新版的 VirtualBox。

安装 Vagrant 的方法也很简单，在[此处](https://www.vagrantup.com/downloads.html)下载安装即可。需要注意的是，如果您使用的是 macOS 或 Linux，您可能希望使用 Brew 或者 YUM 等软件管理器来安装，然而官方并不推荐这样做：在第三方软件管理器的软件仓库中，Vagrant 可能版本较低，或者缺少依赖包。因此还是建议用户直接在官方网站上下载安装。

在安装之后，用户的家目录下会多出一个给 Vagrant 使用的隐藏文件夹 `.vagrant.d`。这里将保存 Vagrant 所需的所有基础配置。如果将其删除，再次运行 Vagrant 时，Vagrant 会认为这是一次全新安装，然后再次将该文件夹创建出来。

#### 1.2 创建项目文件夹和 Vagrantfile

接下来我们就可以为创建虚拟机做准备了。首先，我们要为虚拟机创建一个主目录，用来存放与该服务器相关的文件等；然后在这个文件夹中创建 Vagrant 的主配置文件：Vagrantfile。

Vagrantfile 是用来描述虚拟机状态的主文件，其中包括虚拟机镜像、机器配置、网络配置、以及初始化脚本等。通过使用 `vagrant init` 命令，我们便可以创建出一份标准的 Vagrantfile。

```Shell
$ mkdir vagrant_getting_started
$ cd vagrant_getting_started
$ vagrant init
```

这样一来，我们便创建出 `Vagrantfile` 这个文件了。

#### 1.2 获取虚拟机模版：Box

有了虚拟机的配置文件还不够，我们要告诉 Vagrant 使用哪一个虚拟机模版。以前我们会自己从头创建虚拟机，然后自己制作成模版；而在 Vagrant 的世界，模版名称叫做 “Box”。此外，Vagrant 还有一个专门的[模版仓库](https://atlas.hashicorp.com/boxes/search)，提供一些权威机构或用户创建的模版供用户下载。

让我们用 Vagrant 下载一份 CentOS 7.3 的 Box。这里我们选用了 [Bento 项目](https://github.com/chef/bento)提供的镜像。这个项目由 Chef 公司发起，旨在为 Vagrant 制作各种操作系统的标准 Box。

```
$ vagrant box add bento/centos-7.3
```

首次下载时间较长。在国内，速度可能会比较慢。如果有搭梯子的方法，建议各位在 Shell 中使用 HTTP 代理下载。

下载完成之后，这份 CentOS 7.3 的 Box 会被保存在 `~/.vagrant.d/boxes/bento-VAGRANTSLASH-centos-7.3`中。再继续往下寻找，会找到`.vmdk`格式的虚拟机文件。这就是我们需要的虚拟机模版了。

Vagrant Box 的命名规则为“用户名/项目名”，比如 `bento/centos-7.3`，和 Github 的风格很像。

打开我们创建的“Vagrantfile”文件，告诉 Vagrant 我们要使用刚刚下载的 `bento/centos-7.3`。

```Ruby
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.3"
end
```

如果我们在这里填了一个没有预先下载好的 Box 名称，Vagrant 会先去官方的 Box 库将其下载下来，保存在上面提到的目录中。

Box 还支持版本标记，如果我们希望使用某一个 Box 的特定版本，我们也可以在 Vagrantfile 中指定。例如：

```Ruby
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.3"
  config.vm.box_version = "2.3.1"
end
```

#### 1.3 启动虚拟机以及使用 SSH 连接

接下来我们便可以启动虚拟机了。

```
$ vagrant up
```

这个过程中完全不需要打开 Virtualbox，Vagrant 会自己去调用 Virtualbox，完成所有创建的工作。

创建完成之后，我们可以通过 Vagrant 提供的 SSH 直接连接到服务器中，Vagrant 已帮我们配置好端口映射和密钥，非常方便。

```
$ vagrant ssh
```

按照平时的方式退出 SSH 便可退回到原来的 Shell。

如果希望将这台服务器关机，可以执行：

```
$ vagrant halt
```

在现在的这个目录下再次执行 `vagrant up` 即可将其重新开启。

如果希望删除这台虚拟机，也非常方便：

```
$ vagrant destroy
```

此时，这台虚拟机便会被删除。但是，我们使用过的 Box 并不会被删除掉。它还会保存在 `~/.vagrant.d` 中，以便我们再次使用。如果需要删除某一个 Box，可以使用 `vagrant box remove` 命令，后接 Box 名称即可。

#### 1.4 

### 2. 利用 Vagrant 配置一台 ELK 服务器

#### 2.1 

#### 2.2 使用 Ansible Playbook 安装 ELK

#### 2.3 

---

以上便是一些搭建和使用镜像站的心得，感谢阅读。欢迎在下面留言。
