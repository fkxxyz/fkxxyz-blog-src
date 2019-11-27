---
title: archlinux 下 nvidia 双显卡配置--大黄蜂方案
date: 2019-04-18 20:45:08
categories:
- 教程
tags:
- archlinux
- 双显卡切换
---

双显卡切换的问题是难倒很多新手的问题，我也是那么折腾过来的，[大黄蜂的wiki](https://wiki.archlinux.org/index.php/Bumblebee_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 也有详细的介绍，本文做一个速记总结。

<!--more-->

## 安装 nvidia 内核模块

### 安装

用包管理器安装英伟达显卡驱动（如果显卡较老加载不成功则可尝试nvidia-390xx，更老则可搜索aur里的驱动安装 nvidia-340xx ，注意后面加了 -390xx 之后，后面所有带 nvidia 的包名都得加 -390xx，其它以此类推）

```shell
pacman -S nvidia nvidia-utils

# 若是 390xx 的，则包名为 nvidia-390xx nvidia-390xx-utils
```

32位程序程序使用英伟达显卡驱动支持（记得需要[开启 multilib 仓库](https://wiki.archlinux.org/index.php/Official_repositories_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#multilib)）

```shell
pacman -S lib32-nvidia-utils

# 若是 390xx 的，则包名为 lib32-nvidia-390xx-utils
```

尝试加载驱动
```shell
modprobe nvidia nvidia_uvm nvidia_drm nvidia_modeset

# 如果这一步报错，则卸载刚刚装的所有，回到第一步，尝试其它驱动。
```

### 测试

查看驱动是否在运行（有输出代表成功运行）

```shell
lsmod | grep nvidia
nvidia-smi
```

查看显卡所有信息
```shell
nvidia-smi -q
```



## 安装 bbswitch

### 安装

安装显卡驱动开关

```shell
pacman -S bbswitch
```

### 启动

加载显卡驱动开关

```
modprobe bbswitch
```

### 测试

把开关打开

```shell
tee /proc/acpi/bbswitch <<< ON
```

把开关关闭
```shell
tee /proc/acpi/bbswitch <<< OFF
```

查看开关状态
```shell
cat /proc/acpi/bbswitch
```



## 安装大黄蜂

### 安装

安装配置双显卡切换器

```shell
pacman -S bumblebee
```

### 配置

将自己用户加入到 bumblebee 组（注销重新登录后生效）

```shell
sudo usermod -a -G bumblebee <用户名>
```

修改 /etc/bumblebee/bumblebee.conf :
```ini
Driver=nvidia
[driver-nvidia]
PMMethod=bbswitch
```

### 启动

启动服务

```shell
systemctl start bumblebeed
```

设置服务自动启动
```shell
systemctl enable bumblebeed
```

### 测试

测试英伟达显卡驱动（不加optirun为测试集显，终端输出了显卡型号，以后用optirun运行程序则表示使用英伟达显卡）

```shell
optirun glxspheres64
optirun glxspheres32
```
