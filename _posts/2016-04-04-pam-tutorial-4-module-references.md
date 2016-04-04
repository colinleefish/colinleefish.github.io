---
layout: post
title: "PAM 教程：四、PAM 的配置文件综述"
tags: Linux, PAM, Authentication, Tutorial
unsplash_source_number: 4
excerpt: 本系列讲解了 Linux-PAM 的工作机制和配置方式。本文是该系列的第四篇：《常用的 Linux-PAM 模块一览》，介绍了比较常见的几个 Linux-PAM 模块，包括它们的功能、返回值和应用场景。
---

本系列讲解了 Linux-PAM 的工作机制和配置方式，并利用几个 Linux-PAM 模块做一些有趣的小实验。附录中介绍了一些常用的 Linux-PAM 模块。

本文的目标读者是期望了解 PAM 认证机制的 Linux 用户或者系统管理员。如果您是开发人员，希望编写一个使用 PAM 认证的应用程序，或者是为 PAM 写插件的开发人员，本文的内容可能并不能满足您的需求，请参阅[《Linux-PAM应用开发指南》](http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_ADG.html)（英文）和[《Linux-PAM 模块开发指南》](http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_MWG.html)（英文）。

本文是该系列的第四篇：《Linux-PAM 模块一览》。

### 目录

1. [PAM 简介]({% post_url 2016-03-30-pam-tutorial-1-intro %})

2. [Linux-PAM 的配置文件]({% post_url 2016-04-02-pam-tutorial-2-config-files %})

3. [Linux-PAM 小实验]({% post_url 2016-04-03-pam-tutorial-3-examples %})

4. [Linux-PAM 模块一览]({% post_url 2016-04-04-pam-tutorial-4-module-references %})（本文）

---


### 4. 常用的 Linux-PAM 模块一览

---
[上一篇]({% post_url 2016-04-03-pam-tutorial-3-examples %}) /
