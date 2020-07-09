---
title: fcitx5 使用搜狗皮肤
cover: false
date: 2020-07-09 02:50:28
updated: 2020-07-09 02:50:28
categories:
- 原创开发
tags:
- fcitx
- fcitx5
- 输入法
- 主题
- 皮肤
btns:
  repo: https://github.com/fkxxyz/ssfconv
  feedback: https://github.com/fkxxyz/ssfconv/issues
typora-root-url: ../..
---

对，你没看错，这次是 fcitx5 是用搜狗皮肤！

（对输入法输入框截图有点困难，等有机会用虚拟机截图补上）

<!--more-->

上一次是用 python 实现了一个搜狗皮肤到 fcitx 皮肤的转换器 ssfconv，这次在其基础上扩展，支持了 fcitx5，更先进的 fcitx5 对皮肤的支持，已经没有 fcitx 的毛病，拉伸部分也能像搜狗拼音输入法一样任意调整坐标。

## 开始使用

### 下载此仓库

```shell
git clone https://github.com/fkxxyz/ssfconv.git
cd ssfconv
```

### 下载皮肤

先从[搜狗输入法的皮肤官网](https://pinyin.sogou.com/skins/)下载自己喜欢的皮肤，得到ssf格式的文件，例如 【雨欣】蒲公英的思念.ssf

### 转换皮肤

```shell
./ssfconv  -t fcitx5 【雨欣】蒲公英的思念.ssf  【雨欣】蒲公英的思念
```

### 复制到用户皮肤目录

```shell
mkdir -p ~/.local/share/fcitx5/themes/
cp -r 【雨欣】蒲公英的思念  ~/.local/share/fcitx5/themes/
```

### 使用该皮肤

打开 fcitx5 的配置，附加组件标签，经典用户界面，点配置，在主题的下拉列表里，选择这款皮肤。

或者你也可以直接修改配置文件 ~/.config/fcitx5/conf/classicui.conf，将 Theme 的值改成这个皮肤的名称即可。

用下面这条命令可以看到该皮肤的名称：

```shell
grep Name ~/.local/share/fcitx5/themes/【雨欣】蒲公英的思念/theme.conf
```

## 详细介绍

使用方法被封装得非常简单，像个转换器，可以在下面四种格式之间任意转换：

1. ssf格式（加密）
2. ssf格式（未加密，本质是zip）
3. 文件夹（解密或解压ssf格式得到）
4. fcitx格式（在文件夹的基础上多了fcitx_skin.conf，可用于fcitx）
5. fcitx5格式（在文件夹的基础上多了theme.conf，可用于fcitx5）

命令行参数

```
ssfconv <src> <dest> [-t type]
```

源文件和目标文件是必选参数，转换的目标类型 -t 是可选参数，type值是下面四个值之一：

```
fcitx			可直接用于fcitx的文件夹
fcitx5			可直接用于fcitx5的文件夹
dir				解包后的文件夹
encrypted		加密的ssf皮肤
zip				未加密的ssf皮肤（zip）
```

默认是转换为 fcitx 格式。

注意，源文件的格式可以是以上任意五个格式之一，不需要指定，程序已经可以智能识别格式。

## 已知缺陷

fcitx5 能够完美地像搜狗输入法一样调整，但是主题中所设置的字体是无效的，需要手动设置字体，经过我反复的实验，将字体设置为 "Sans 10" 似乎是大多数皮肤的最佳体验。

