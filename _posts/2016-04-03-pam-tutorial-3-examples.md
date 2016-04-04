---
layout: post
title: "PAM 教程：三、Linux-PAM 小实验"
tags: Linux, PAM, Authentication, Tutorial
unsplash_source_number: 3
excerpt: 本系列讲解了 Linux-PAM 的工作机制和配置方式。本文是该系列的第三篇：《Linux-PAM 小实验》，首先用几个小实验来验证第二篇中提到的部分内容，然后尝试用 Google Authenticator 为一台 CentOS 设备添加了一次性密码的功能，还用 pam_time.so 模块禁止了用户在夜间登录。
---

本系列讲解了 Linux-PAM 的工作机制和配置方式，并利用几个 Linux-PAM 模块做一些有趣的小实验。附录中介绍了一些常用的 Linux-PAM 模块。

本文的目标读者是期望了解 PAM 认证机制的 Linux 用户或者系统管理员。如果您是开发人员，希望编写一个使用 PAM 认证的应用程序，或者是为 PAM 写插件的开发人员，本文的内容可能并不能满足您的需求，请参阅[《Linux-PAM应用开发指南》](http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_ADG.html)（英文）和[《Linux-PAM 模块开发指南》](http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_MWG.html)（英文）。

本文是该系列的第三篇：《Linux-PAM 小实验》。

### 目录

1. [PAM 简介]({% post_url 2016-03-30-pam-tutorial-1-intro %})

2. [Linux-PAM 的配置文件]({% post_url 2016-04-02-pam-tutorial-2-config-files %})

3. [Linux-PAM 小实验]({% post_url 2016-04-03-pam-tutorial-3-examples %})（本文）

4. [Linux-PAM 模块一览]({% post_url 2016-04-04-pam-tutorial-4-module-references %})

5. [参考文献]({ post_url 2016-04-05-pam-tutorial-5-bibliography %})

---


### 3. Linux-PAM 小实验

在本篇中，我们会先用几个小实验来验证第二节中讲到的控制模式（control），然后会使用几个比较好玩又实用的模块来加固我们的系统。

笔者使用的系统为 CentOS 7.2.1511 x64 版本，运行在 VirtualBox 中，IP 地址为 `192.168.5.181`，主机名为 `colin-mac-ctos-1`。

##### 验证 `required` 和 `requisite` 的区别

在笔者使用的 CentOS 版本中，`/etc/pam.d/system-auth` 是一个有关密码验证的配置文件，`su` 命令把它作为验证过程中的一个子栈。因此，修改该文件便相当于修改了 `su` 的验证配置。`system-auth` 文件中 `auth` 工作组的内容如下：

```
auth        required      pam_env.so
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
auth        required      pam_deny.so
```

这里我们重点关注第二行和第四行。第二行中的 `pam_unix.so` 会询问用户名和密码，并和 `/etc/passwd` 以及 `/etc/shadow` 作比对；第四行 `pam_deny.so` 会无条件地返回 `err` 从而导致认证失败。我们为了演示，将第四行剪切，挪至第一行和第二行之间。

```
auth        required      pam_env.so
auth        required      pam_deny.so
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
```

根据我们在第二篇介绍过的内容可知，当认证流程走到第二行时，由于 `pam_deny.so` 返回失败结果，且控制模式为 `required`，因此整个流程栈走到第二行的时候已经注定失败了。但由于 `required` 不会马上使流程终止，因此这个流程还会走下去。我们来尝试一下，用 `su` 命令从普通用户切换到 `root`。

<script type="text/javascript" src="https://asciinema.org/a/c9pokom8w0vr13zaj3lpttkmu.js" id="asciicast-c9pokom8w0vr13zaj3lpttkmu" async></script>

笔者输入了正确的密码，依然未能通过验证。失败的结果在提示输入密码之前就已经注定了。

现在我们将第二行的 `required` 修改为 `requisite`，再尝试一次。此时的配置文件如下。

```
auth        required      pam_env.so
auth        requisite     pam_deny.so
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
```

<script type="text/javascript" src="https://asciinema.org/a/5emvn9xrpjeswml9qkm8wp72n.js" id="asciicast-5emvn9xrpjeswml9qkm8wp72n" async></script>

可以发现，我们这次连输入密码的机会都没有了。因为 `requisite` 模式直接导致了流程栈的退出。

---
[上一篇]({% post_url 2016-04-02-pam-tutorial-2-config-files %}) / [下一篇]({% post_url 2016-04-04-pam-tutorial-4-module-references %})
