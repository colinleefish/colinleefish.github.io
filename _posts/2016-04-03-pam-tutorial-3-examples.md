---
layout: post
title: "PAM 教程：三、Linux-PAM 小实验"
tags: Linux, PAM, Authentication, Tutorial
unsplash_source_number: 3
excerpt: 本系列讲解了 Linux-PAM 的工作机制和配置方式。本文是该系列的第三篇：《Linux-PAM 小实验》，首先用一个小实验来感受 required 和 requisite 的区别，然后尝试用 Google Authenticator 为一台 CentOS 设备添加了一次性密码的功能，还用 pam_time.so 模块禁止了用户在夜间登录。
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

在笔者使用的 CentOS 版本中，`/etc/pam.d/system-auth` 是一个通用的配置文件，`su` 等命令把它作为验证过程中的一个子栈。因此，修改该文件便相当于修改了 `su` 等命令的验证配置。`system-auth` 文件中 `auth` 工作组的内容如下：

```
auth        required      pam_env.so
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
auth        required      pam_deny.so
```

这里我们重点关注第二行和第四行。第二行中的 `pam_unix.so` 会询问用户名和密码，并和 `/etc/passwd` 以及 `/etc/shadow` 做对比；第四行 `pam_deny.so` 会无条件地返回 `err` 从而导致认证失败。我们为了演示，将第四行剪切，挪至第一行和第二行之间。

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

##### 使用 Google Authenticator 给 CentOS 添加一次性密码

Google Authenticator 项目是一个用户认证方面的解决方案，使用了[开放认证联盟（OATH）](http://www.openauthentication.org/)制订的开放标准。支持基于步数的[`HOTP` 算法](https://tools.ietf.org/html/rfc4226)和基于时间的[`TOTP` 算法](https://tools.ietf.org/html/rfc6238)。详细的文档可以参阅该项目的 [Github Wiki](https://github.com/google/google-authenticator/wiki)。目前 Google、Dropbox 和 Tumblr 等等服务都支持上述这种一次性密码方案，为用户提供多重保障。

Google Authenticator 也开发了 PAM 的模块，因此我们可以在登录 Linux 时使用 Google Authenticator。这一小节我们就来尝试一下。

在您的手机上，请安装 Google Authenticator 或者 Authy 等一次性密码生成器。笔者使用的是 Google Authenticator。

在 CentOS 中，请首先保证已安装了以下包，我们在安装过程中需要他们。

- `git`
- `autoconf`
- `automake`
- `cc`
- `gcc`
- `m4`
- `libtool`
- `pam-devel`

```
# yum install git autoconf automake cc gcc m4 libtool pam-devel
```

从 Github 下载 Google Authenticator 的源码：

```
# git clone https://github.com/google/google-authenticator/
```

进入 `./google-authenticator/libpam`，然后依次运行 `./bootstrap.sh`、`./configure`、`make && make install`。下面演示了笔者的安装过程。

<script type="text/javascript" src="https://asciinema.org/a/49xzouemr0kzjrmil3gztdep4.js" id="asciicast-49xzouemr0kzjrmil3gztdep4" async></script>

如果您在安装到最后出现了和笔者一样的提示，说明安装成功，该模块便安装进了 `/usr/local/lib/security`，文件名为 `pam_google_authenticator.so`。为了方便，您可以在 `/lib64/security/` 下创建一个这个文件的软连接，这样在 PAM 配置文件中就不需要写全路径了。

接下来我们来通过命令行来配置 Google Authenticator。直接输入 `google-authenticator`命令，可以打开一个交互的配置工具。在本例中，我们会对所有的提问回答“是”。该工具将在配置过程中输出一串密钥、一个二维码，以及若干备用的一次性密码。

<script type="text/javascript" src="https://asciinema.org/a/ebuhsyoxhylka4ypw7xs74mdf.js" id="asciicast-ebuhsyoxhylka4ypw7xs74mdf" async></script>

在我们之前下载好的手机 APP 中，您可以扫码，也可以手动输入刚才生成的密钥。本例中的密钥为 `DNRVSJIV5PUZJIVP`，我们现在将手动输入进去。

![手动输入密钥](http://i.imgur.com/mp2Rids.png)

点击“ADD”，此时便完成了一次性密码的配置。密码已经能够根据时间生成了。

![密钥配置完成，一次性密码开始生成](http://i.imgur.com/DkrtUDd.png)

然后，我们在 PAM 配置文件夹下的 `password-auth` 中添加 Google Authenticator 模块的策略。`password-auth` 被 `sshd` 所引用，因此可以用来控制 SSH 登录。

请在 `/etc/ssh/sshd_config` 配置文件中，保证 `ChallengeResponseAuthentication` 为 `yes`。然后修改 `password-auth` 文件中 `auth` 类别为如下所示。这里我们添加了第二行规则。

```
auth        required      pam_env.so
auth        sufficient    pam_google_authenticator.so
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
auth        required      pam_deny.so
```

笔者在这里特地使用了 `sufficient`，目的是为了测试。如果直接在这里使用 `required` 或者 `requisite`，而你在整个设置中出现了差错，很有可能导致进不去系统。`sufficient` 保证了我们即便有人为错误导致密码不正确也一样能够登录。

那我们现在该如何测试一次性密码是否生效了呢？很简单，只要在输入验证码之后直接进入系统，而不是又要求用户输入 `Password` 便是生效了。

现在让我们通过 SSH 登录，来检查一次性密码设置是否生效。我们将分别验证输入正确验证码和错误验证码的情况。

<script type="text/javascript" src="https://asciinema.org/a/8jluhi23ni7zj4pdtkkvtklx5.js" id="asciicast-8jluhi23ni7zj4pdtkkvtklx5" async></script>

在输入正确验证码时，我们直接登录进了系统；在输入错误的验证码时，我们被提示输入用户密码，然后才登录了系统。说明一次性密码的设置已经生效。此时您可以放心将 `sufficient` 改为 `required`，来校验每次登录了。

不过，由于我们使用的算法是基于时间的，因此您在使用这个模块时，请保证服务器时间的准确性，否则将导致无法登录。

有关 Google Authenticator 的更多信息，请参阅 `/usr/local/share/doc/google-authenticator/README.md`（CentOS），或该项目官方[Github](https://github.com/google/google-authenticator)。

---
[上一篇]({% post_url 2016-04-02-pam-tutorial-2-config-files %}) / [下一篇]({% post_url 2016-04-04-pam-tutorial-4-module-references %})
