---
title: NetworkManager 网络配置速查
date: 2019-04-15 03:06:10
categories:
- 教程
tags:
- archlinux
- linux
---

一般普通用户上网的方式无非就三种：有线以太网、无线网络、宽带PPPOE拨号。而 NetworkManager 这个软件将以上三种方式集成一体，而且配置方便。

<!--more-->

NetworkManager 是 gnome 桌面环境自带的网络管理服务，在 tty 的命令行中也可以运行，刚装完的系统，想要网络，只需要装这一个包就够！

因此写这篇文章作为一个速查备忘和新手入门，更高级的用法可以自己查看 nmcli 自带的 --help 。

## 服务管理

启动网络管理器服务

```shell
systemctl start NetworkManager
```

将此服务设为自动启动
```shell
systemctl enable NetworkManager
```



## 全局管理

查看网络概况

```shell
nmcli
nmcli -overview
```

打开关闭总网络开关
```shell
nmcli n on/off
nmcli network on/off
```



## 设备管理

查看网络设备状态

```shell
nmcli d
nmcli device
nmcli device status
```

查看网络设备详细信息
```shell
nmcli d sh
nmcli device show
nmcli device show eth0
```

连接/断开设备
```shell
mncli d conn/dis eth0
mncli device connect/disconnect eth0
```

删除软件设备
```shell
nmcli d del 
```

监控某个设备的连接过程
```shell
nmcli d mon eth0
nmcli d monitor eth0
```

设置某个设备自动连接
```shell
nmcli d set eth0 auto yes/no
```

设置某个设备是否本程序管理
```shell
nmcli d set eth0 man yes/no
```

配置某个设备（暂时的，重启失效）
```shell
nmcli d mod eth0 ??
```

让某个设备重新应用
```shell
mncli d re eth0
```

查询wifi
```shell
nmcli d wifi
nmcli d wifi list
nmcli d wifi list ifname wlp3s0
```

刷新wifi
```shell
nmcli d wifi rescan
nmcli d wifi rescan ifname wlp3s0
```

连接wifi
```shell
nmcli d wifi <SSID> password <password>
nmcli d wifi <SSID> password <password> ifname wlp3s0
```



## 配置管理

查看所有配置
所有配置文件保存在 /etc/NetworkManager/system-connections
```shell
nmcli c
nmcli c show
```

删除某个配置
```shell
nmcli c del <name>
```

复制某个配置
```shell
nmcli c clone <name> <new_name>
```

连接/断开某个配置
```shell
nmcli c up/down <name>
```

重新载入配置文件
```shell
nmcli c reload
```

导入/导出配置文件
```shell
nmcli ??
```

监视某个配置
```shell
nmcli c mon <name>
```

修改某个配置的某一行
```shell
nmcli c mod <name> <配置>.<属性> <值>
```

启动编辑某个配置
```shell
nmcli c edit <name>
```

创建配置
```shell
nmcli c add con-name <name> type ??? ...
```

创建pppoe拨号配置
```shell
nmcli c add con-name <name> type pppoe ifname <设备> username <username> password <password>
```
