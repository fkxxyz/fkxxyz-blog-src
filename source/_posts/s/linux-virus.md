---
title: 我的阿里云服务器中毒了，把这有趣的事情记录下来。
cover: false
date: 2019-12-09 18:24:19
updated: 2019-12-09 18:24:19
categories:
- 记录
tags:
- 信息安全
typora-root-url: ../..
---

自从用了 linux，由于 linux 用户量小又开源代码审查人多的特点，从而从来没为 linux 的安全担心过，这次算是第一次遭遇到病毒，值得记录下来。

<!--more-->

## 病毒的发现

### 发现

偶然情况进了一下阿里云控制台，看了一眼监控，眼前的景象惊呆了。

![](/img/ali-cpu.png)

连续，一个月，CPU占用100%？我阿里云平时基本空闲，也只有我访问，也没什么复杂的计算，都是挂挂反向代理和做普通网站用，怎么也不可能出现这种现象。所以到这里基本确定，是有什么问题了，但是还没联想到病毒。

### 怀疑

直接登录阿里云，用 top 命令看了下，确实有个进程占 100% 的CPU，进程名是 .dhpcd 我还给看错了，第一反应看成 dhcpd ，以为是 dhcp 客户端出了什么故障，但前面为什么有个 点？而且 dhcp 客户端也没必要一直开着啊，就获取地址的时候用一下，难道是出了bug，陷入死循环？

又仔细看了下 top，用户名为 user ？？？？？更奇怪了，这个用户是我以前临时建立的，用来给一朋友登录玩 gcc 的，平时从来没用过。

### 定位

用 ps 看下这用户的所有进程信息

```shell
ps -ef | grep user
```

![](/img/vir-proc.png)

在家目录的隐藏文件。直接去看看这个文件。

```shell
ls -al
```

![](/img/vir-file.png)

这文件还挺大，大到 3 兆，看看它的类型。

```shell
file .dhpcd
```

![](/img/vir-type.png)

这是编译好的二进制文件，排除了是我那朋友捣鬼。

把这文件上传到 [virscan](https://www.virscan.org/) 扫描一下吧。几分钟后看到[扫描结果](http://r.virscan.org/language/zh-cn/report/f17b54910531cf6e2d98a963acadab48)。

![](/img/vir-scan-result.png)

![](/img/vir-scan-result-1.png)
![](/img/vir-scan-result-2.png)
![](/img/vir-scan-result-3.png)
![](/img/vir-scan-result-4.png)
![](/img/vir-scan-result-5.png)

可以看到，出现 BitCoinMiner 的字样，让我几乎确定是比特币挖矿病毒。而且现象也符合，狂烧 CPU，就是在挖矿嘛。

## 处理

先 kill 掉它，然后设置它的权限为 000。把 user 这个用户也给删了。

```shell
chmod 000 .dhpcd
```

![](/img/vir-000.png)

删除它吗？不删了，放那吧，等哪天想研究了再研究研究，把这小可爱留着做纪念。

但是是时候考虑安全问题了，为什么会中毒。再仔细想想，病毒只是感染了这一个用户，我这个用户的密码设置的 123456，所以很随意的就进来了，虽然没有 root 权限，没加入到 sudo 组，但是这种挖矿会占很大的资源，它可能已经挖了几个月了，而我今天才察觉到。



以后设置密码，无论是再简单的用户，再也不设这么简单的密码了，我密码最后头随便加个符号，都要好很多。