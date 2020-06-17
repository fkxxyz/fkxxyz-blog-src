---
title: fcitx 真的也可以这么美 —— 对 fcitx 使用搜狗皮肤的改进
cover: false
date: 2020-05-25 12:46:51
categories:
- 原创开发
tags:
- fcitx
- 输入法
- 皮肤
typora-root-url: ../..
---

fcitx输入法框架能够自定义皮肤，然后有个很nb的作者开发了个搜狗皮肤转换成fcitx皮肤的，这是原项目地址https://github.com/VOID001/ssf2fcitx

然后我亲自试了几个我喜欢的皮肤，居然真的可以转换，跟搜狗差不多了，不过一段时间后，发现一些bug：设置了皮肤之后，输入法菜单隔空而且透明，字都看不清。部分皮肤文字位置很奇怪。于是，我看了他的源码，发现逻辑还挺简单，然后看了下fcitx的自定义皮肤的各种格式，打算亲自研究研究这是怎么回事。

最终打算参考这个项目，自己用python写个，不仅解决了zip压缩格式，还实现了图片自动裁剪（不愧是胶水语言）

![](/img/ssfskin-5.png)

<!--more-->

最终两个函数实现，取名为转换器ssfconv，放到 github 托管 https://github.com/fkxxyz/ssfconv

在原作者的基础上进行了下面几方面改进：

1. 部分皮肤文字位置重新计算，摆放更合理
2. 将菜单的背景也设置成皮肤的主题色，文字大小和颜色均计算到合理
3. 字体单位改成像素，和搜狗一致，完美还原
4. 调整了翻页指示器的位置，自动生成指示器的图像

直接上图吧（我自己都没想到linux的输入法也可以这么美）

![](/img/ssfskin-1.png)

![](/img/ssfskin-2.png)

![](/img/ssfskin-3.png)

![](/img/ssfskin-4.png)

![](/img/ssfskin-6.png)

![](/img/ssfskin-7.png)

## 开始使用

下面直接举例吧。

### 下载此仓库

```shell
git clone https://github.com/fkxxyz/ssfconv.git
cd ssfconv
```

### 下载皮肤

先从[搜狗输入法的皮肤官网](https://pinyin.sogou.com/skins/)下载自己喜欢的皮肤，得到ssf格式的文件，例如 【雨欣】蒲公英的思念.ssf

### 转换皮肤

```shell
./ssfconv  【雨欣】蒲公英的思念.ssf  【雨欣】蒲公英的思念
```

### 复制到用户皮肤目录

```shell
mkdir -p ~/.config/fcitx/skin/
cp  【雨欣】蒲公英的思念  ~/.config/fcitx/skin/
```

### 使用该皮肤

右键输入法托盘图表，选中皮肤，这款皮肤是不是出现在列表里了呢，尽情享用吧。

## 详细介绍

使用方法被封装得非常简单，像个转换器，可以在下面四种格式之间任意转换：

1. ssf格式（加密）
2. ssf格式（未加密，本质是zip）
3. 文件夹（解密或解压ssf格式得到）
4. fcitx格式（在文件夹的基础上多了fcitx_skin.conf，可用于fcitx）

命令行参数

```shell
ssfconv <src> <dest> [-t type]
```

源文件和目标文件是必选参数，转换的目标类型 -t 是可选参数，type值是下面四个值之一：

```
fcitx			可直接用于fcitx的文件夹
dir				解包后的文件夹
encrypted		加密的ssf皮肤
zip				未加密的ssf皮肤（zip）
```

默认是转换为 fcitx 格式。

注意，源文件的格式可以是以上任意四个格式之一，不需要指定，程序已经可以智能识别格式。

## 已知缺陷

因为fcitx的限制，输入框里只能对文字的外边距进行设置，无法像搜狗拼音输入法一样任意调整坐标，导致部分皮肤只能在图片拉升和文件位置靠右来二选一的取舍。不过大多数皮肤都能挺不错的转换，只有少数皮肤实在是没办法了，只好用图片拉升代替（原作者是将文字调整到靠右，留了很多空白）。

