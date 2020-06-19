---
title: 在 linux 发行版中，python 多版本共存并自由切换
cover: false
date: 2019-12-13 18:41:24
updated: 2019-12-13 18:41:24
categories:
- 教程
tags:
- python
- linux
---

最近有多个 python 版本共存的需求，在 archlinux 下只能 python3.8，而阿里云的 python 是 3.6，而且环境和库各不相同。包管理器也影响其存在。然后搜到了 pyenv 这个神器，这个项目直接解决了我所有遇到的 python 不同版本共存和切换的问题，不解释了，直接看[官方介绍](https://github.com/pyenv/pyenv#simple-python-version-management-pyenv)。

<!--more-->

## pyenv 的原理

### 选择Python版本

这里直接引用官方的[原文](https://github.com/pyenv/pyenv#choosing-the-python-version)

> 执行填充程序时，pyenv通过从以下来源按以下顺序读取来确定要使用的Python版本：
>
> 1. 在`PYENV_VERSION`环境变量（如果指定）。您可以使用该[`pyenv shell`](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md#pyenv-shell)命令在当前的Shell会话中设置此环境变量。
> 2. `.python-version`当前目录中的特定于应用程序的文件（如果存在）。您可以`.python-version`使用以下[`pyenv local`](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md#pyenv-local) 命令修改当前目录的 文件。
> 3. `.python-version`通过搜索每个父目录找到第一个文件（如果有），直到到达文件系统的根目录为止。
> 4. 全局`$(pyenv root)/version`文件。您可以使用[`pyenv global`](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md#pyenv-global)命令修改此文件。如果不存在全局版本文件，则pyenv假定您要使用“系统” Python。（换句话说，如果您没有pyenv，则可以运行任何版本 `PATH`。）

## pyenv 的使用速查

我把这一章放到最开头，作为速查命令

### 安装 python

列出所有可以安装的版本

```shell
pyenv install --list
# 或
pyenv install -l
```

只列出 python 的版本

```shell
pyenv install -l | grep '^ *[0-9]'
```

安装一个版本（例如 3.6.9）

```shell
pyenv install 3.6.9
# 如果卡在下载源码上，可以手动下载源码，放到 ~/.pyenv/cache 里。
# 如果源码已经被放在了 ~/.pyenv/cache/Python-3.6.9.tar.xz 那么就不会下载了，直接解压编译。
```

### 查询版本

#### 查看当前选择的 python 版本

该命令会提示当前环境如果执行 python 的话，会启动的 python 版本，以及如何选择的 python 版本

```shell
pyenv version
```

#### 查看所有可选择的 python 版本

```shell
pyenv versions
```

### 切换选择 python 版本

#### 以全局方式选择 python 版本

这种方式全局生效，在任意的 shell 调用 python 时，都会以设置的 python 版本启动。

查看全局 python 版本

```shell
pyenv global
```

设置全局 python 版本

```shell
pyenv global 3.6.9
```

#### 以目录模式选择 python 版本

此方式可以把某个目录设为特定版本的 python，设置时会在这个目录里写入 .python_version 文件

查看当前目录的 python 版本

```shell
pyenv local
```

设置当前目录的 python 版本

```shell
pyenv local 3.6.9
```

#### 以 shell 环境模式选择 python 版本

此方式可以把当前 shell 环境设置为特定版本的 python，设置时会改变 PYENV_VERSION 这个环境变量

查看当前 shell 的 python 版本

```sbell
pyenv shell
```

设置当前 shell 的 python 版本

```shell
pyenv shell 3.6.9
```



## pyenv 的安装

### 安装 pyenv

在 archlinux 发行版中，由于官方仓库自带 pyenv ，直接安装即可

```shell
sudo pacman -S pyenv
```

在其它不带 pyenv 的 linux 的发行版中，pyenv的github文档给出了详细的[安装过程](https://github.com/pyenv/pyenv#installation)，可以按照官方给出的[安装器](https://github.com/pyenv/pyenv-installer#installation--update--uninstallation)这样安装（如果没有 git 需要先装 git）

```
curl https://pyenv.run | bash
```

此命令克隆仓库到 ~/.pyenv 下，可执行文件在 ~/.pyenv/bin 可以看到里面只有一个 pyenv 符号链接指向 ../libexec/pyenv

### pyenv 的环境配置

官方给出了详细的[配置](https://github.com/pyenv/pyenv#basic-github-checkout)过程，那么就搬运官方给的原命令

1. 如果用的 bash ，则修改 ~/.bash_profile

   ```shell
   echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
   echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
   ```

2. 如果用的 zsh， 则修改 ~/.zshenv

   ```shell
   echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshenv
   echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshenv
   ```

然后重启 shell ，使配置生效

```shell
exec $SHELL
```

现在，可以执行 pyenv 了，试试吧

```shell
pyenv
```

若输出一大堆 pyenv 的帮助，则代表配置生效啦。

### python 环境配置

先执行

```shell
pyenv init
```

可以看到提示，将这个提示加入到他所说的文件中吧

```shell
# bash 下执行
pyenv init 2>> ~/.bashrc

# zsh 下执行
pyenv init 2>> ~/.zshrc
```

然后执行 exec $SHELL 生效

```shell
exec $SHELL
```

查看是否成功

```shell
echo $PATH
```

如果看到开头为 /home/用户名/.pyenv/shims 则配置算生效啦。

## 安装不同版本的 python

### 准备编译环境

这一步可谓是至关重要，由于忽略这个问题而直接编译 python，在 centos 上，默认很多所需的包没装，那么编译的时候就会没有把相应的功能编译进去，造成后续使用的时候出现一些问题。

按照[python官方的开发者文档安装依赖](https://devguide.python.org/setup/#linux)这一章，可以快速补上需要的依赖。

在 yum 包管理器管理的系统中：

```shell
sudo yum install yum-utils
sudo yum-builddep python3
```

在 dnf 包管理器管理的系统中：

```shell
sudo dnf install dnf-plugins-core  # install this to use 'dnf builddep'
sudo dnf builddep python3
```

在 debian 系的发行版中，看官方说明吧，等我要用到的时候再总结到这里。

> 在**Debian**，**Ubuntu**和其他`apt`基于系统的系统上，尝试通过使用`apt`命令获取正在使用的Python的依赖关系。
>
> 首先，请确保已在“来源”列表中启用了源软件包。您可以通过将源码包的位置（包括URL，发行名称和组件名称）添加到中来实现`/etc/apt/sources.list`。以Ubuntu Bionic为例：
>
> ```
> deb-src http://archive.ubuntu.com/ubuntu/ bionic main
> ```
>
> 对于其他发行版，例如Debian，请更改URL和名称以与特定发行版相对应。
>
> 然后，您应该更新软件包索引：
>
> ```
> $ sudo apt-get update
> ```
>
> 现在，您可以通过`apt`以下方式安装构建依赖项：
>
> ```
> $ sudo apt-get build-dep python3.6
> ```

### 下载 python 源码

虽然 pyenv 可以自动从 python 的官网下载源码，但是尝试过之后，发现一直卡住速度较慢。

可以从国内任意含有 gentoo 仓库的镜像站来下载一部分 python 源码。

> 3.8.0 版本
>
> https://mirrors.tuna.tsinghua.edu.cn/gentoo/distfiles/Python-3.8.0.tar.xz
>
> 3.7.5 版本
>
> https://mirrors.tuna.tsinghua.edu.cn/gentoo/distfiles/Python-3.7.5.tar.xz
>
> 3.6.9 版本
>
> https://mirrors.tuna.tsinghua.edu.cn/gentoo/distfiles/Python-3.6.9.tar.xz
>
> 3.5.9 版本
>
> https://mirrors.tuna.tsinghua.edu.cn/gentoo/distfiles/Python-3.5.9.tar.xz
>
> 2.7.17 版本
>
> https://mirrors.tuna.tsinghua.edu.cn/gentoo/distfiles/Python-2.7.17.tar.xz

不一定有所有的版本，但包含很多常用的版本了，大概是够用了。

将他们下载到 ~/.pyenv/cache 下，先创建这个目录吧

```shell
cd
cd .pyenv
mkdir cache
cd cache
```

然后开始下载，例如

```shell
wget https://mirrors.tuna.tsinghua.edu.cn/gentoo/distfiles/Python-3.6.9.tar.xz
```

### 用 pyenv 安装 python

如果源码已经被放在 ~/.pyenv/cache 里了，那么在执行安装就很快了。

列出所有可安装的版本

```shell
pyenv install --list
```

安装指定版本

```shell
pyenv install 3.6.9
# 如果上一步，源码已经被放在了 ~/.pyenv/cache/Python-3.6.9.tar.xz 那么这一步就不会下载了，直接解压编译。
```

等到输出如下信息时，代表安装成功了。

> Installed Python-3.6.9 to /home/用户名/.pyenv/versions/3.6.9

然后可以愉快的安装不同版本 python 和随意切换啦。