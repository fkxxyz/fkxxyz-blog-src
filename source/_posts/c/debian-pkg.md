---
title: 基于 debian 系的发行版常用包管理命令速查
cover: false
date: 2020-07-28 10:05:20
updated: 2020-07-28 10:05:20
categories:
- 教程
tags:
- debian
- linux
---



<!--more-->

安装一个包

```shell
apt install <pkg>
```

查看某个文件属于哪个包

```shell
dpkg -S <file>
```

查看一个包包含哪些文件

```shell
dpkg -L <pkg>
```

查询本地包的信息

```shell
apt-cache show live-build
```

查询远程数据库中哪个包里包含这个文件（需要安装 apt-file）

```shell
apt-file search <file>
```

