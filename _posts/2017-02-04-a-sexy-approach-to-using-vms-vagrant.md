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

本文便旨在向读者介绍性感的 Vagrant，带领读者进行 Vagrant 快速上手，然后以搭建一台 ELK 服务器的完整案例来展示 Vagrant 的便捷和强大。

部分操作步骤和脚本来自[官方文档](https://www.vagrantup.com/docs/)，下文不会单独指出。

笔者使用的环境为 macOS Sierra 10.12.3 ，Vagrant 1.9.1 以及 VirtualBox 5.1.14。

### 1. 前期准备和安装

Vagrant 将其支持的各种虚拟化平台称为 “Provider”。我们在本次的快速上手中选择了 VirtualBox 作为后端虚拟化平台。因此，您首先需要安装最新版本的 VirtualBox 来搭配 Vagrant。请访问[这里](https://www.virtualbox.org/wiki/Downloads)下载最新版的 VirtualBox。

安装 Vagrant 的方法也很简单，在[此处](https://www.vagrantup.com/downloads.html)下载安装即可。需要注意的是，如果您使用的是 macOS 或 Linux，您可能希望使用 Brew 或者 YUM 等软件管理器来安装，然而官方并不推荐这样做：在第三方软件管理器的软件仓库中，Vagrant 可能版本较低，或者缺少依赖包。因此还是建议用户直接在官方网站上下载安装。

在安装之后，用户的家目录下会多出一个给 Vagrant 使用的隐藏文件夹 `.vagrant.d`。这里将保存 Vagrant 所需的所有基础配置。如果将其删除，再次运行 Vagrant 时，Vagrant 会认为这是一次全新安装，然后再次将该文件夹创建出来。

### 2. 创建项目文件夹和 Vagrantfile

接下来我们就可以为创建虚拟机做准备了。首先，我们要为虚拟机创建一个主目录，用来存放与该服务器相关的文件等；然后在这个文件夹中创建 Vagrant 的主配置文件：Vagrantfile。

Vagrantfile 是用来描述虚拟机状态的主文件，其中包括虚拟机镜像、机器配置、网络配置、以及初始化脚本等。通过使用 `vagrant init` 命令，我们便可以创建出一份标准的 Vagrantfile。

```Shell
$ mkdir vagrant_getting_started
$ cd vagrant_getting_started
$ vagrant init
```

这样一来，我们便创建出 `Vagrantfile` 这个文件了。

### 3. 获取虚拟机模版：Box

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

### 4. 启动虚拟机以及使用 SSH 连接

接下来我们便可以启动虚拟机了。

```
$ vagrant up
```

这个过程中完全不需要打开 VirtualBox，Vagrant 会自己去调用 VirtualBox，完成所有创建的工作。

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

此时，这台虚拟机便会被删除。但是，我们使用过的 Box 并不会被删除掉。它还会保存在 `~/.vagrant.d` 中，以便我们再次使用。如果需要删除某一个 Box，使用 `vagrant box remove` 加上 Box 的名称即可。

### 5. 共享文件夹

Vagrant 提供了在宿主机和虚拟机之间的一个共享文件夹功能。如果你使用的是 VirtualBox，这个功能要求虚拟机中安装有 VirtualBox Guest Addition。我们刚刚使用的 “bento/centos-7.2” Box 中已经预装了该工具，所以可以直接使用。

宿主机方面，共享文件夹便是我们刚才使用的 Vagrantfile 所在的项目文件夹；虚拟机方面，共享文件夹的路径位于 `/vagrant`。

注意：但如果你是在 “bento/centos-7.3” Box 中使用共享目录功能，请不要尝试升级内核。在我自己的测试过程中，更新内核会导致 VirtualBox Guest Addition 失效，导致共享文件夹无法使用。

Vagrant 还提供了其他的同步方法，包括 rsync，NFS 等等。但这些或多或少需要在宿主机上面做一些相应的配置，所以这里还是推荐使用 VirtualBox Guest Addition。

### 6. 网络

Vagrant 可以帮助我们配置端口映射。比如我们刚刚使用的 `vagrant ssh` 命令，实际上连接的也是我们宿主机的 2222 端口，只是由 Vagrant 操纵 VirtualBox 将虚拟机的 22 端口映射了过来而已。

此外，我们还可以在 Vagrantfile 中自行配置希望映射的端口，写法也非常简单：

```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.3"
  config.vm.network :forwarded_port, guest: 80, host: 4567
end
```

这样一来，如果在这个虚拟机里开启了跑在 80 端口的 HTTP 服务，便可以在宿主机的 `http://127.0.0.1:4567` 访问到了。

网卡也可以通过 Vagrantfile 来配置。上面的内容都是针对于使用 VirtualBox NAT 的网络选项来说的，这种配置适用于一些比较简单的场景。如果有配置 Public IP 或 Private IP 的需求，也可以写在 Vagrantfile 中：

```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.3"
  config.vm.network :forwarded_port, guest: 80, host: 4567
  config.vm.network "private_network", ip: "192.168.10.101"
  config.vm.network "public_network"
end
```

在 Vagrant 和 VirtualBox 的搭配中，Vagrant 所谓的 `private_network` 会自动和 VirtualBox 的 “Host-only network” 相对应，是一个仅包含了宿主机和虚拟机的网络，且需要预先在 VirtualBox 的网络配置中创建出来；而 `public_network` 和 “Bridged network” 相对应，在网络中和宿主机处于平等地位。

![VirtualBox 的网络配置，图示中显示的是“Host-only network”](http://i.imgur.com/QSG0eDd.png)
*VirtualBox 的网络配置，图示中显示的是“Host-only network”*

### 7. 系统初始化

如果 Vagrant 仅能做到如此，对于解放生产力来说还是不够的。因此 Vagrant 内置了一套系统初始化功能，能够让用户在创建虚拟机时便将初始化工作一并完成。这套功能在 Vagrant 中被称为 “Provision”。

Provision 支持多种初始化脚本。用户可以把需要执行的 Shell 命令写在 Vagrantfile 中，也可以单独写一个 Bash 脚本。Vagrant 还支持 Ansible、CFEngine、Chef、Puppet 等众多配置管理工具。Vagrant 介绍了使用 Bash 脚本的[例子](https://www.vagrantup.com/docs/getting-started/provisioning.html)，我们这里用 Ansible 来做一个范例，来展示一下如何利用 Provision 功能。

如果希望使用 Ansible，前提是宿主机上已安装好了 Ansible，macOS 下使用 Brew 或者 pip 均可，这里暂且不表。接下来，我们在这个虚拟机项目的文件夹中创建一个 Ansible Playbook，将其命名为 `elk-playbook.yml`。

<script src="https://gist.github.com/colinleefish/800cefbd0796230ce3e92b2eef30fc2a.js"></script>

接下来，我们编辑 Vagrantfile，告诉 Vagrant 在创建虚拟机之后执行我们的 Playbook。

```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.3"
  config.vm.provision "ansible" do |ansb|
    ansb.playbook = "elk-playbook.yml"
    ansb.sudo = true
  end
end
```

然后我们便可以执行 `vagrant up` 了。在正常显示虚拟机创建过程提示后，将会显示的 Ansible 执行过程。

因为要下载的内容比较多，安装时间可能会比较长。在成功执行完成 Ansible Playbook 之后，一个完整安装了 ELK 环境的虚拟机便创建完成了！

### 8. 全网共享

有了这些功能，Vagrant 在日常开发和学习环境中已经带给用户极大的便利了。然而这还不算完，Vagrant 还有一个不得不说的犀利功能：全网共享。

比如，你在你自己的虚拟机中运行着自己开发的网站，希望展示另一个城市的朋友。有了 Vagrant，你可以轻松地创建出一个临时的地址，朋友只要访问这个地址，便可以直接访问到你虚拟机上的网站来；不仅是网站，通过这个地址 SSH 到你的虚拟机也可以；甚至，任何一个端口，经过配置之后，都可以映射到这个公网的地址，共享给他人进行访问，称得上完全意义上的“全网全局共享”。

使用这个功能的前提是用户已经注册了 HashiCorp 的 “Atlas” 账号。HashiCorp 是发布和维护 Vagrant 工具的公司，Atlas 是这家公司的一套企业级 IT 产品，但创建账号、使用共享功能是完全免费的。

首先，我们要在 Shell 中登录 Atlas 账号。

```
$ vagrant login
Username or Email: colinleefish@gmail.com
Password (will be hidden):
You are now logged in!
```

然后就可以使用共享功能了。

```
$ vagrant share
...
==> default: Your Vagrant Share is running!
==> default: URL: http://frosty-weasel-0857.vagrantshare.com
...
```

如果你的虚拟机中的 80 端口正在提供 HTTP 服务，上面的链接便可以直接访问到你的网站。

如果你希望朋友通过 SSH 连接到你的服务器，在共享命令中加入 `--share` 参数即可。

```
$ vagrant share --ssh
==> default: Detecting network information for machine...
    default: Local machine address: 192.168.163.152
    default: Local HTTP port: 4567
    default: Local HTTPS port: disabled
    default: SSH Port: 22
==> default: Generating new SSH key...
    default: Please enter a password to encrypt the key:
    default: Repeat the password to confirm:
    default: Inserting generated SSH key into machine...
==> default: Checking authentication and authorization...
==> default: Creating Vagrant Share session...
    default: Share will be at: itty-bitty-polar-8667
==> default: Your Vagrant Share is running!
    default: Name: itty-bitty-polar-8667
...
```

将最下面的随机名字（`itty-bitty-polar-8667`）发给你的朋友，你的朋友在他的 Shell 中执行 `vagrant connect --ssh itty-bitty-polar-8667` 即可连接到你的虚拟机上来了。

```
$ vagrant connect --ssh itty-bitty-polar-8667
Loading share 'itty-bitty-polar-8667'...
The SSH key to connect to this share is encrypted. You will
require the password entered when creating the share to
decrypt it. Verify you have access to this password before
continuing.

Press enter to continue, or Ctrl-C to exit now.
Password for the private key:
Executing SSH...
Welcome to Ubuntu 12.04.3 LTS (GNU/Linux 3.8.0-29-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
Last login: Fri Mar  7 17:44:50 2014 from 192.168.163.1
vagrant@vagrant:~$
```

当然，你和你的朋友能使用这个功能的前提是你们都安装了 Vagrant。

……既然 Vagrant 这么强大，有什么理由不推荐给他呢？


---

有关 Vagrant 的特点和使用方法，我们就先介绍到这里，希望对读者有所帮助。感谢阅读，也欢迎留言或者和我交流。
