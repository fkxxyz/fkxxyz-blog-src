---
title: 用 python 实现 xmind 和 mindjet 格式互转
date: 2019-09-28 10:34:51
categories:
- 原创开发
tags:
- python
- 思维导图
---

一直想找一款跨平台的免费又好用的思维导图软件，可是哪有两全其美的事呢，个人感觉安卓版的 mindjet 相对好用一些，windows 和 linux 版的 xmind 相对好用一些，但是 xmind 和 mindjet 的格式肯定是不兼容的，而探索发现，他们的文档解压之后都是以 xml 方式储存的，压缩也是简单的 zip 压缩，也没有任何加密，于是，故事开始了。

<!--more-->

## 简介

经过大概八小时的开发后，这样一个转换器成功诞生。这是一款用 python3 实现的简单的 xmind 与 mindjet 格式之间的互转工具，只保留树状思维导图以及折叠功能，另外还可以额外可以转化成 txt，用缩进来表示树状图。

项目已放到 github 开源，以便保存和后续随时修改。

https://github.com/fkxxyz/mmconv

## 用法

### 命令格式

```
mmconv.py -t [格式] [源文件] [目标文件]
```

其中 -t [格式] 是指要转换的目标文件格式，可缺省，默认为 txt，可选 mmap,xmind。

源文件的格式不用指定，会自己识别，详见 --help

### 用法示例

```shell
# 将 a.xmind 转换成 txt 格式
mmconv.py a.xmind a.txt

# 将 a.xmind 转换成 mmap 格式
mmconv.py -t mmap a.xmind a.mmap

# 将 a.txt 转换成 xmind 格式
mmconv.py -t xmind a.txt a.xmind
```

