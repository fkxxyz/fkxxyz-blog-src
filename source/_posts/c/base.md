---
title: archlinux 的基本配置
cover: false
date: 2019-04-15 00:04:54
categories:
- 教程
tags:
- archlinux
---



本文速记一些刚装完 archlinux 之后所需的必要配置，以便以后速查。

<!--more-->

## 基本配置

### 设置键盘布局

列出所有可用的键盘布局

```shell
ls /usr/share/kbd/keymaps/**/*.map.gz
```

设置想要的键盘布局（默认 us，只需指定文件名即可，无需拓展名）
loadkeys us
设置键盘布局
写入文件 /etc/vconsole.conf

```
KEYMAP=us
```



### 设置时区

设置为上海时区

```shell
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```



### 设置系统时间

（以下两个二选一）
将硬件时间设置为系统的本地时间（与windows默认相同）

```
hwclock -s -l
```

将硬件时间设置为系统的UTC时间（与mac系统默认相同）

```
hwclock -s -u
```

启用 ntp 服务，获取网络时间并设置为当前系统时间

```
timedatectl set-ntp true
```

生成时间偏差（/etc/adjtime）

```
hwclock -w
```



### 设置本地语言

修改 /etc/locale.gen，去除en_US.UTF-8和zh_CN.UTF-8前面的井号

```
locale-gen
echo LANG=en_US.UTF-8 >/etc/locale.conf
```



### 修改主机名

```
hostnamectl set-hostname ???
```
在 /etc/hosts 里添加（设置网络主机名）

```
127.0.0.1          localhost
::1                localhost
127.0.1.1          ???.localdomain ???
```



### 用户管理

设置 root 用户的密码

```
passwd root
```
创建新用户

```
useradd -m ???
```



### sudo

将 /etc/sudoers 中 %wheel 前面的 去掉

将某用户设成管理员（能够用sudo）

```
usermod -G wheel ???
```



### 配置软件源

参见：

1. [Arch Linux 软件仓库镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/archlinux/)  
2. [ArchlinuxCN 镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/archlinuxcn/)  


更新软件数据库

```
pacman -Syy
```
更新系统

```
pacman -Syu
```

开启别的仓库只需要取消注释 /etc/pacman.conf 相应的项，参见 [archwiki-官方仓库](https://wiki.archlinux.org/index.php/Official_repositories_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))



## 高级配置

### 禁用 beep 响铃

在 tty 下敲命令会时不时的发出 beep 声音，超级大声很烦，必须禁掉。

暂时生效
```
rmmod pcspkr
```
永久生效
```
echo blacklist pcspkr>>/etc/modprobe.d/nobeep.conf
```

### 禁用 nouveau 驱动

此驱动bug过多，可能导致死机，花屏，卡顿等未知故障。
在 /etc/modprobe.d/no-nouveau.conf 中写入：

```
blacklist nouveau
options nouveau modeset=0
```

### 禁用蓝牙

如果蓝牙没怎么用过，禁了会省电些

```
find /lib/modules/`uname -r`/kernel -name bluetooth|xargs find|grep \.xz$|awk -F'/' '{print $NF}'|awk -F'.' '{print "blacklist " $1}' >>/etc/modprobe.d/no-bluetooth.conf
```

### 解开 rf 锁

用查看 wifi 的 rf锁

```
rfkill list
```
解除 rf 锁
```
rfkill unlick all
```