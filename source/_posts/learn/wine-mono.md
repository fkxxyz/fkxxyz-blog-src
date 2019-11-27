---
title: 探索在 Archlinux 下使用 wine 时偶尔提示未找到 wine-mono 的完美解决方案
date: 2019-06-03 22:20:49
categories:
- 探究学习
tags:
- archlinux
- wine
typora-root-url: ../..
---

使用 archlinux 有一段日子了，发现有时候在使用 wine 的过程中，明明已经装了 wine-mono 这个包，但依然时常出现这个对话框，很恼火，是时候把这问题探究彻底了。

![no-mono](/img/no-mono.png)

<!--more-->

## 问题描述

我们知道，wine 的 mono 组件是.NET Framework的开源和跨平台实现，能够让 Wine 顺利运行很多 .NET应用程序。

出现上述对话框后，点击安装，虽然会自动从 wine 官网把 mono 组件下到 $WINEPREFIX 里面，也是可以用，但是下的很慢，耗费很多时间，而且用 .NET 程序用的少，有时候仅仅想测试一个 exe 程序，点取消的话，每次要 wine 一个程序的时候都会弹出这个，实在讨厌。

后来，我觉得最新版的 wine 没有 wine-stable 稳定，我自己从 aur 编译了 wine-stable 之后，开始次次出现以上对话框了，经过百度谷歌，搜到的全是解决别的问题，无奈之下只能自己动手丰衣足食。

## 问题探究

先看看 wine-mono 这个包包含什么。

```shell
pacman -Ql wine-mono
```

根据输出结果看出，原来只包含一个文件，还带版本号 /usr/share/wine/mono/wine-mono-4.9.3.msi

emmmm，什么？带版本号？我想起这问题偶然出现，什么时候出现呢，就是升级系统的时候会偶尔出现。那么 wine 是如何找到这个文件，以确认 mono 存在性呢？难道是扫描 /usr/share/wine/mono/ 整个目录？那可以找到的啊。

根据这些线索，我猜测，wine 是根据绝对路径和带版本号的文件名寻找 mono 的，一旦所需要的 mono 版本，和系统中存在的 mono 版本不一样，就会出现那找不到的对话框。

如何验证这个猜想呢，我想到 grep -rn 这个在所有子目录里面查找匹配的功能，开始行动。

先查看 wine-stable 包含哪些文件

```shell
pacman -Ql wine-stable
```

根据输出结果，大概知道 wine 的文件集中在以下几个目录

```
/usr/lib/wine
/usr/lib32/wine
/usr/share/wine
```

还有很多细小的目录，直接搜会很麻烦。我想到一个方案，把以前编译好的 wine-stable 安装包解压到一个目录里面，集中搜索。说干就干！

```shell
mkdir /tmp/wine
cd /tmp/wine
tar xf ~/.cache/yay/wine-stable/wine-stable-4.0.2-1-x86_64.pkg.tar.xz
ls
```

然后开始搜索

```shell
grep -rn wine-mono
```

几秒钟后，输出结果只有三行

```
匹配到二进制文件 usr/lib32/wine/appwiz.cpl.so
匹配到二进制文件 usr/lib/wine/appwiz.cpl.so
.BUILDINFO:1138:installed = wine-mono-4.9.2-1-any
```

最后一行保存的是软件包的信息，wine 启动的过程中肯定是不会用的。那.........重点研究下 usr/lib/wine/appwiz.cpl.so 这个文件好了

```shell
grep -a wine-mono usr/lib/wine/appwiz.cpl.so
```

输出结果

```
2.47wine_gecko-2.47-x86_64.msigeckoMSHTMLGeckoUrlGeckoCabDir4.7.5wine-mono-4.7.5.msimonoDotnetMonoUrlMonoCabDir%s does not exist and could not be created: %s
```

果然，包含了字符串 wine-mono-4.7.5.msi，也就是说，wine-stable 是依赖于 4.7.5 版本 wine-mono，而我系统里存在的是 wine-mono-4.9.3.msi，找不到很正常。

好的，现在做个实验，把 wine-mono-4.9.3.msi 复制为 wine-mono-4.7.5.msi，问题是不是解决了。

```shell
cd /usr/share/wine/mono
mv wine-mono-4.9.3.msi wine-mono-4.7.5.msi
rm -rf ~/.wine
winecfg
```

发现，确实没再弹出那个对话框了，猜想成立！

终于找到原因了，接下来想想怎么完美解决这个问题吧。

## 解决方案

我想到几个方案来解决这个问题：

1. 安装旧版本的 wine-mono 的包。
2. 更新 wine 到最新版本，确保所需的 mono 版本与官方仓库最新的 mono 的版本一致。
3. 对 appwiz.cpl.so 这个文件进行二进制编辑，修改里面的版本号字符串。
4. 把系统里的 wine-mono-4.9.3.msi 重命名 wine-mono-4.7.5.msi
5. 创建符号链接解决这个问题。

想出了这么多办法，逐一分析这些办法的利弊：

1. **安装旧版本的 wine-mono 包**：这需要降级软件包，需要用 downgrade，然后可以把 wine-mono 这个软件包设为忽略更新的包写到 pacman.conf 里，这样做的话，每次升级 wine-stable 都需要检查版本号手动装合适的 wine-mono，有些麻烦。
2. **更新 wine 到最新版本**：我大量实践过程中，感觉 wine-stable 确实要稳定一些，最新版的虽然有新特性但是免不了很多 bug。而且，就算更新到最新，也会偶尔出现版本不匹配的问题。
3. **对 appwiz.cpl.so 这个文件进行二进制编辑**：这个操作很骚，但是万一以后版本号长度不一样了，怎么办呢，而且每次更新都得改也很麻烦。
4. **重命名系统里的 wine-mono**：这个操作会影响到系统里包含的文件，而每次升级之后，旧的 wine-mono 文件就不会被删掉，然后装上了新的，白白占用空间，每次来处理，也很麻烦。
5. **创建符号链接**：创建符号链接，可谓是 linux 解决这类问题最妙的办法，直接将 wine-mono-4.9.3.msi 链接到 wine-mono-4.7.5.msi，非常方便，但是每次更新还是要过来处理，还是很麻烦。

可见创建符号链接是目前最低成本的解决方法，那，有没有完美的解决方法呢？

## 完美解决

通过以上思考，我最需要的就是，更新后不需要手动去创建符号链接，用脚本自动实现更新后的解决版本不一致的问题。

每次更新后，更新什么？更新 wine-stable 或 wine-mono 的时候。

如何在每次更新这两个包，触发调用脚本呢？利用包管理器的 [hook](https://wiki.archlinux.org/index.php/Pacman_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#Hooks) 功能。

看来完美的解决方案是存在的，下面来列出，需要解决的几个子问题：

1. 脚本如何读取 appwiz.cpl.so 这个文件来获取所需的版本号呢？
2. 脚本如何确定当前系统存在的 wine-mono 的版本号对应的文件呢？
3. 升级后旧版本留下的符号链接会多余存在很多垃圾要怎么办呢？

然后这些问题逐一得到解决：

1. 直接利用正则表达式匹配

   ```shell
   sed -n 's/.*\(wine-mono-[[:digit:].]\+.msi\).*/\1/p' /usr/lib/wine/appwiz.cpl.so
   ```

   输出结果为 wine-mono-4.7.5.msi

2. 直接用包管理器查询包含的文件，然后正则匹配到具体文件名

   ```shell
   pacman -Qlq wine-mono | grep -o 'wine-mono-\([[:digit:].]\+\).msi'
   ```

   输出结果为 wine-mono-4.9.3.msi

3. 每次更新后，先将 /usr/share/wine/mono 里的符号链接全删了，再建立即可。

将以上思路进行具体实施，写成 [hook](https://wiki.archlinux.org/index.php/Pacman_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#Hooks) 脚本，得到完美解决。具体 hook 的写法参见 [alpm-hooks文档](https://jlk.fjfi.cvut.cz/arch/manpages/man/alpm-hooks.5) 。



在 /etc/pacman.d/hooks 里面新建一个文件 wine-mono-version-fix.hook

里面写入

```ini
[Trigger]
Type = File
Operation = Install
Operation = Upgrade
Target = usr/lib/wine/appwiz.cpl.so
Target = usr/share/wine/mono/*

[Action]
Description = Fixing the version of wine-mono file.
When = PostTransaction
Exec = /usr/bin/sh -c 'find /usr/share/wine/mono -type l -exec unlink {} \; ; ln -sf "$(pacman -Qlq wine-mono | grep "wine-mono-\\([[:digit:].]\\+\\).msi")" "/usr/share/wine/mono/$(sed -n "s/.*\\(wine-mono-[[:digit:].]\\+.msi\\).*/\\1/p" /usr/lib/wine/appwiz.cpl.so)" 2>/dev/null ; true'
```

至此，完美解决，以后无论如何更新 wine 或 wine-mono，或者无论如何更换 wine 的版本，总是能找到对应的 wine-mono，也再也不会弹出那个对话框了。