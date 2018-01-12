---
layout: post
title: "使用 pyenv 在 macOS 上创建多版本且能隔离的 Python 环境"
tags: pyenv,macos
unsplash_id: gCTgP86MhNQ
excerpt: pyenv 可以方便地让我们在同一台机器上创建多版本，且相互隔离的 Python 环境。这样一来，用户便可以轻松尝试多个 Python 版本，以及轻松切换多个开发环境。
---

和众多 Linux 分发版本一样，macOS 中也自带了一份 Python 运行环境。打开命令行，直接键入 python 即可，触手可及。

但是，系统中自带的 Python 版本一般是 Python 2.7。如果希望尝试其他版本，这份系统自带的 Python 就无法满足需求了。很多人下意识的做法是去官方网站，寻求一份需要的版本进行安装，这种方式虽然正统，但也存在一定的弊端：不够灵活、会在系统中安装额外的软件和工具、切换不同 Python 版本时相对比较麻烦。

今天要介绍的 pyenv 正是为了解决这些问题而出现一款工具。pyenv 是一款 Python 版本管理器，可以让用户轻松的在多个 Python 版本之间进行切换，此外，pyenv 还可以集成 virtualenv 以项目为单位进行环境隔离。经由 pyenv 管理的 Python 环境（或者是项目）还可以继承给 Atom 或者 PyCharm，这样一来，开发人员只要打开了某个项目，也就打开了专门为这个项目创建的独立环境。

接下来我们将演示如何在 macOS 下配置 pyenv 环境。由于下面的步骤严重依赖国外网站，所以请确保命令行下配置了使用 http_proxy 和 https_proxy 环境变量的科学上网，否则可能导致安装失败。

### 安装

pyenv-installer 是 pyenv 附带的一份安装脚本。用户仅需一条命令，即可调用 pyenv-installer 脚本完成安装。

```
$ curl -L https://raw.githubusercontent.com/pyenv/pyenv-installer/master/bin/pyenv-installer | bash
```

这个脚本不仅会帮我们安装 pyenv，还会附带安装一些 pyenv 的插件，其中就包括 virtualenv。因此，在跑完这个脚本之后，virtualenv 也已经可以使用了。

我们还需要在 .bash_profile 中添加以下两行，使得每次打开命令行之后，pyenv 都可以自动配置在环境变量中。

```
# Enable pyenv
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

添加之后，保存 .bash_profile，再执行 source，即可生效。

```
$ source ~/.bash_profile
```

### 卸载

pyenv 会安装在用户家目录下的 .pyenv 文件夹中，不会安装在家目录外的系统文件夹中。因此，只要删除该文件夹，即是删除了 pyenv。

```
$ rm -rf ~/.pyenv
```

### Python 版本管理

在 pyenv 生效后（source或新开终端），便可以使用 pyenv 命令行工具进行 Python 版本管理。

##### 查看所有可下载的 Python 版本

```
$ pyenv install --list
Available versions:
  2.1.3
  2.2.3
  2.3.7
  2.4
  ...
  stackless-3.3-dev
  stackless-3.3.5
  stackless-3.4.1
  stackless-3.4.2
```

##### 安装某个 Python 版本

```
$ pyenv install 3.5.1
Downloading Python-3.5.1.tar.xz...-> https://www.python.org/ftp/python/3.5.1/Python-3.5.1.tar.xz
Installing Python-3.5.1...
Installed Python-3.5.1 to /Users/colinleefish/.pyenv/versions/3.5.1
```

这里的操作实际是从官方下载源码，然后现场编译出 Python 环境并放在 ~/.pyenv 中。注意：如果是首次尝试安装，多数都会因为各种原因失败。我故意没写解决办法，请自行上网寻求方法。[滑稽]

另外，经测试，在网络状况较好的环境中，下载该源码包的速度很理想。如果使用 http_proxy 等科学上网，反而会出现验证 HTTPS 超时的可能。因此，如果在网速快的情况下进行下载，建议暂时取消科学上网。

##### 查看已安装的 Python 版本和已创建的环境

```
$ pyenv versions
  system
  2.7.13
  2.7.13/envs/mkdocs
* 2.7.13/envs/tango_with_django
  ...
  tango_with_django
  theotherbarn
```
system 为系统自带的 Python 环境；标星号的为当前激活的全局 Python 环境。

##### 基于某个 Python 版本创建环境隔离的项目

```
$ pyenv virtualenv 3.4.6 my-awesome-project
Using base prefix '/Users/colinleefish/.pyenv/versions/3.4.6'
New python executable in /Users/colinleefish/.pyenv/versions/3.4.6/envs/my-awesome-project/bin/python3.4
Also creating executable in /Users/colinleefish/.pyenv/versions/3.4.6/envs/my-awesome-project/bin/python
Installing setuptools, pip, wheel...done.
```

在创建该环境的过程中，virtualenv 会很贴心地帮我们安装好 setuptools, pip 和 wheel，这样一来，我们马上就可以使用 pip 了。

##### 激活某个项目

```
$ pyenv activate my-awesome-project
pyenv-virtualenv: prompt changing will be removed from future release. configure `export PYENV_VIRTUALENV_DISABLE_PROMPT=1' to simulate the behavior.
(my-awesome-project)$
```

激活时，在上面实例第二行会有一个提醒，可以放心忽略。激活后，在命令行的开头部分会有括号标注的项目名称。

##### 使用该项目

```
(my-awesome-project)$ python --version
Python 3.4.6
(my-awesome-project)$ pip install requests
Collecting requests
  Using cached requests-2.18.4-py2.py3-none-any.whl
Collecting chardet<3.1.0,>=3.0.2 (from requests)
  Using cached chardet-3.0.4-py2.py3-none-any.whl
Collecting idna<2.7,>=2.5 (from requests)
  Using cached idna-2.6-py2.py3-none-any.whl
Collecting certifi>=2017.4.17 (from requests)
  Using cached certifi-2017.7.27.1-py2.py3-none-any.whl
Collecting urllib3<1.23,>=1.21.1 (from requests)
  Using cached urllib3-1.22-py2.py3-none-any.whl
Installing collected packages: chardet, idna, certifi, urllib3, requests
Successfully installed certifi-2017.7.27.1 chardet-3.0.4 idna-2.6 requests-2.18.4 urllib3-1.22
(my-awesome-project)$
```

我们在这个隔离的环境中安装了 requests，这个过程中还安装了一些 requests 的依赖包。那它们都被安装在了哪里呢？

```
(my-awesome-project)$ cd ~/.pyenv/versions/my-awesome-project/lib/python3.4/site-packages
(my-awesome-project)$ ls
__pycache__                   easy_install.py               pkg_resources                 urllib3
certifi                       idna                          requests                      urllib3-1.22.dist-info
certifi-2017.7.27.1.dist-info idna-2.6.dist-info            requests-2.18.4.dist-info     wheel
chardet                       pip                           setuptools                    wheel-0.30.0.dist-info
chardet-3.0.4.dist-info       pip-9.0.1.dist-info           setuptools-36.6.0.dist-info
```

这些依赖包，都被安装在了 ~/.pyenv 下一个独立的文件夹中，而且和系统的 Python 环境、其他 Python 环境做到了隔离。日后如果不再需要该项目，只需使用 pyenv 自带的命令删除，或删除这个项目文件夹即可。

如果您使用 Atom 等编辑器，并可以从命令行启动，这类编辑器会自动继承当前的项目环境，这样就可以在编辑器中运行该项目的代码了。

```
(my-awesome-project)$ atom
```

##### 取消激活某个项目

```
(my-awesome-project)$ pyenv deactivate
$
```

使用 pyenv deactivate 即可取消激活当前的环境。在此之后，命令行提示符前面的括号内容就消失了，Python 环境回归了正常的全局环境。

##### 设置全局 Python 环境

```
$ pyenv global my-awesome-project
(my-awesome-project)$ python --version
Python 3.4.6
```

使用 pyenv global 命令，可以为命令行设置全局生效的环境或项目。在关掉当前 Shell，再新开其他 Shell 时，一直都会使用当前环境。

如果希望恢复该设置，使用 pyenv global system。

```
(my-awesome-project)$ pyenv global system
$
```

此时，全局环境便又恢复为系统默认的环境了。

##### 查看当前 Python 项目或环境

```
(my-awesome-project)$ pyenv version
my-awesome-project (set by /Users/colinleefish/.pyenv/version)
```

### 在 PyCharm 中使用某个特定的项目环境

PyCharm 也是以独立的项目来组织代码的。每个项目中，均可以设置一个已有的 Python 环境。只要指定这个项目的 Python 解释器（interpretor，其实就是 Python 主程序）即可。

打开任意环境，在菜单栏中通过 “Run” - “Edit Configurations” 即可引导至项目环境设置界面。
![项目环境设置界面](https://i.imgur.com/IeKHEYQ.png)

在 “Python interpreter” 处，只要指定所需的 Python 环境，即是将整个环境，以及依赖包都应用在了这个 PyCharm 项目上。此时便可以进行开发了。

### 延伸阅读

pyenv 是受到 rbenv 启发而产生的项目，后者主要解决了 Ruby 环境下的多版本隔离问题。pyenv 使用了一种叫做 Shim 的命令行黑科技。技术细节在这里。更多文档，可以访问该项目的 [Github Wiki](https://github.com/pyenv/pyenv/wiki)。

### 后记（2018-01-12）

在一次给朋友的 Mac 笔记本配置 pyenv 的过程中发现：当时使用 brew 下载的那个版本有点问题，始终无法安装新的 Python 版本。后来，删掉了 brew 的 pyenv，使用 pyenv 自带的安装工具（pyenv-installer）重新下载了之后，故障得以解决。