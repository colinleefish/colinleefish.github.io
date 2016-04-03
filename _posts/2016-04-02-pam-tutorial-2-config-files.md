---
layout: post
title: "PAM 教程：二、Linux-PAM 的配置文件"
tags: Linux, PAM, Authentication, Tutorial
unsplash_source_number: 2
excerpt: 本系列讲解了 Linux-PAM 的工作机制和配置方式。本文是该系列的第二篇：《PAM 的配置文件综述》，主要介绍了 PAM 的配置文件格式，type、control、stack 的概念，以及配置项的逻辑关系。
---

本系列讲解了 Linux-PAM 的工作机制和配置方式，并利用几个 Linux-PAM 模块做一些有趣的小实验。附录中介绍了一些常用的 Linux-PAM 模块。

本文的目标读者是期望了解 PAM 认证机制的 Linux 用户或者系统管理员。如果您是开发人员，希望编写一个使用 PAM 认证的应用程序，或者是为 PAM 写插件的开发人员，本文的内容可能并不能满足您的需求，请参阅[《Linux-PAM应用开发指南》](http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_ADG.html)（英文）和[《Linux-PAM 模块开发指南》](http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_MWG.html)（英文）。

本文是该系列的第二篇：《Linux-PAM 的配置文件》。

### 目录

1. [PAM 简介]({% post_url 2016-03-30-pam-tutorial-1-intro %})

2. [Linux-PAM 的配置文件]({% post_url 2016-04-02-pam-tutorial-2-config-files %})（本文）

3. [Linux-PAM 小实验]({% post_url 2016-04-03-pam-tutorial-3-examples %})

4. [Linux-PAM 模块一览]({% post_url 2016-04-04-pam-tutorial-4-module-references %})

---


### 2. Linux-PAM 的配置文件综述

PAM 的各个模块一般存放在 `/lib/security/` 或 `/lib64/security/` 中，以动态库文件的形式存在（可参阅 `dlopen(3)`），文件名格式一般为 `pam_*.so`。PAM 的配置文件可以是 `/etc/pam.conf` 这一个文件，也可以是 `/etc/pam.d/` 文件夹内的多个文件。如果 `/etc/pam.d/` 这个文件夹存在，Linux-PAM 将自动忽略 `/etc/pam.conf`。

`/etc/pam.conf` 类型的格式如下：

```
服务名称  工作类别  控制  模块路径  模块参数
```

`/etc/pam.d/` 类型的配置文件通常以每一个使用 PAM 的程序的名称来命令。比如 `/etc/pam.d/su`，`/etc/pam.d/login` 等等。还有些配置文件比较通用，经常被别的配置文件引用，也放在这个文件夹下，比如 `/etc/pam.d/system-auth`。这些文件的格式都保持一致：

```
工作类别  控制模式  模块路径  模块参数
```

不难看出，文件夹形式的配置文件中只是没有了服务名称这一列：服务名称已经是文件名了。

由于很难在时下的发行版本中找到使用 `/etc/pam.conf` 这一独立文件作为 PAM 配置的例子，此处仅就 `/etc/pam.d/` 格式举例。在笔者安装的 CentOS 7.2.1511 中，`/etc/pam.d/login` 的内容如下：

```conf
#%PAM-1.0
auth [user_unknown=ignore success=ok ignore=ignore default=bad] pam_securetty.so
auth       substack     system-auth
auth       include      postlogin
account    required     pam_nologin.so
account    include      system-auth
password   include      system-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
session    optional     pam_console.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    include      system-auth
session    include      postlogin
-session   optional     pam_ck_connector.so
```

`#` 表示注释。

每一行代表一条规则。但也可以用 `\` 来放在行末，来连接该行和下一行。

例子的最后一行开头有一个短横线 `-`，意思是如果找不到这个模块，导致无法被加载时，这一事件不会被记录在日志中。这个功能适用于那些认证时非必需的、安装时可能没被安装进系统的模块。

#### 工作类别（type）、流程栈（stack）和控制模式（control）

我们在[第一篇]({% post_url 2016-03-30-pam-tutorial-1-intro %})中接触了 Linux-PAM 的四种工作类别（type）。在上面的例子中，工作类别作为第一列出现。

讲到这里，我们有必要引入 PAM 的 **流程栈（stack）** 概念：它是认证时执行步骤和规则的堆叠。在某个服务的配置文件中，它体现在了配置文件中的自上而下的执行顺序中。栈是可以被引用的，即在一个栈（或者流程）中嵌入另一个栈。我们之后和它会有更多的接触。

第二列为 **控制模式（control）** ，用于定义各个认证模块在给出各种结果时 PAM 的行为，或者调用在别的配置文件中定义的认证流程栈。该列有两种形式，一种是比较常见的“关键字”模式，另一种则是用方括号包含的“返回值=行为”模式。

“关键字”模式下，有以下几种控制模式：

- `required`：如果本条目没有被满足，那最终本次认证一定失败，但认证过程不因此打断。整个栈运行完毕之后才会返回（已经注定了的）“认证失败”信号。
- `requisite`：如果本条目没有被满足，那本次认证一定失败，而且整个栈立即中止并返回错误信号。
- `sufficient`：如果本条目的条件被满足，且本条目之前没有任何`required`条目失败，则立即返回“认证成功”信号；如果对本条目的验证失败，不对结果造成影响。
- `optional`：该条目仅在整个栈中只有这一个条目时才有决定性作用，否则无论该条验证成功与否都和最终结果无关。
- `include`：将其他配置文件中的流程栈包含在当前的位置，就好像将其他配置文件中的内容复制粘贴到这里一样。
- `substack`：运行其他配置文件中的流程，并将整个运行结果作为该行的结果进行输出。该模式和 `include` 的不同点在于认证结果的作用域：如果某个流程栈 `include` 了一个带 `requisite` 的栈，如果这个 `requisite` 失败将直接导致认证失败，同时退出栈；而某个流程栈 `substack` 了同样的栈时，`requisite` 的失败只会导致这个子栈返回失败信号，母栈并不会在此退出。

“返回值=行为”模式则更为复杂，其格式如下：

```
[value1=action1 value2=action2 ...]
```
