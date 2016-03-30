---
layout: post
title: "PAM 笔记（一）：PAM 简介"
tags: Linux, PAM, Authentication
unsplash_source_number: 1
excerpt: 本系列讲解了 Linux-PAM 的工作机制和配置文件。本文是该系列的第一篇：《PAM 简介》。主要介绍了 “PAM 是什么”、“解决了什么问题”以及它在Linux系统中的存在形式。
---

本系列讲解了 Linux-PAM 的工作机制和配置文件，并利用几个 Linux-PAM 模块做一些有趣的小实验。附录中介绍了一些常用的 Linux-PAM 模块。

本文的目标读者是期望了解 PAM 认证机制的 Linux 用户或者系统管理员。如果您是开发人员，希望编写一个使用 PAM 认证的应用程序，或者是为 PAM 写插件的开发人员，本文的内容可能并不能满足您的需求，请参阅[《Linux-PAM应用开发指南》][1]（英文）和[《Linux-PAM 模块开发指南》][2]（英文）。

本文是该系列的第一篇：《PAM 简介》。

### 目录

1. [PAM 简介][3]

2. PAM 的配置文件综述

3. PAM 小实验

4. 附录：常用的 PAM 模块一览。

---


### 1. PAM 简介

PAM 的全称为“可插拔认证模块（Pluggable Authentication Modules）”。设计的初衷是将不同的底层认证机制集中到一个高层次的API中，从而省去开发人员自己去设计和实现各种繁杂的认证机制。如果没有 PAM 、认证功能作为应用程序的一部分一起编译时，一旦要修改某个认证方法，开发人员可能不得不重写程序，然后重新编译程序并安装；有了 PAM ，认证的工作都交给 PAM ，程序主体便可以不再关注认证问题了：“ PAM 说放行，那就放行。”

PAM 机制最初由 Sun 公司提出，并在其 Solaris 系统上实现。后来，各个版本的 UNIX 陆续增加了对它的支持。Linux-PAM 便是 PAM 在 Linux 上的实现，获得了几乎所有主流 Linux 发行版的支持。

Linux-PAM 的目标同 PAM 机制一致，都是为了给程序的开发人员提供一套执行认证操作的模块。一般情况下，PAM 的配置保存在操作系统内的文本文件中，可以是 `/etc/pam.conf` 这一个文件，也可以是 `/etc/pam.d/` 文件夹内的多个文件。PAM 模块一般存放在 `/lib/security/` 或 `/lib64/security/` 中，以动态库文件的形式存在（可参阅`dlopen(3)`），文件名格式一般为`pam_*.so`。


  [1]: http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_ADG.html
  [2]: http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_MWG.html
  [3]: http://colinleefish.github.com/2016/03/01/pam-notebook-1.html
