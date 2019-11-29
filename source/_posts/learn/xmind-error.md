---
title: 在 Archlinux 下从 AUR 装的 xmind 启动报错的问题探究
cover: false
date: 2019-04-18 13:22:25
categories:
- 探究学习
tags:
- archlinux
- 思维导图
typora-root-url: ../..
---

电脑版的思维导图软件，我还是最钟爱 xmind，然而在 archlinux 下从 AUR 构建 xmind 直接启动之后报错。

<!--more-->

## 问题描述

1. 安装完之后，直接点击图标直接启动，弹出以下对话框。

   ![xmind-error](/img/xmind-error.png)

2. 在安装 XMind 时，输出的结果

   ```
   正在加载软件包...
   正在解析依赖关系...
   正在查找软件包冲突...
   
   软件包 (1) xmind-3.7.8+8update8-1
   
   全部安装大小：  125.02 MiB
   
   :: 进行安装吗？ [Y/n] 
   (1/1) 正在检查密钥环里的密钥                        [###########################] 100%
   (1/1) 正在检查软件包完整性                          [###########################] 100%
   (1/1) 正在加载软件包文件                            [###########################] 100%
   (1/1) 正在检查文件冲突                              [###########################] 100%
   (1/1) 正在检查可用存储空间                          [###########################] 100%
   :: 正在处理软件包的变化...
   (1/1) 正在安装 xmind                                [###########################] 100%
   If XMind crashed on start, trying delete ~/.xmind
   
   If you want to change gtk version or java version, please edit PKGBUILD and rebuild the package. Or edit /usr/share/xmind/XMind/XMind.ini, change number to your gtk version after "--launcher.GTK_version", and add/delete "--add-modules=java.se.ee" at the end of file if you use java 10/8.
   xmind 的可选依赖
       gtk2: gtk2 or gtk3 must install one [已安装]
       gtk3: gtk2 or gtk3 must install one [已安装]
       lame: needed for the feature audio notes [已安装]
   :: 正在运行事务后钩子函数...
   (1/6) Arming ConditionNeedsUpdate...
   (2/6) Updating fontconfig cache...
   (3/6) Updating 32-bit fontconfig cache...
   (4/6) Updating the desktop file MIME type cache...
   (5/6) Updating the MIME type database...
   (6/6) Updating X fontdir indices...
   ```

   


## 问题探究

### 常规思考

由打包者提示的信息，基本可以猜测，是 java 运行环境的问题，以及启动参数 --add-modules=java.se.ee 的问题。

那么尝试从命令行启动，看看有没有报错信息吧，先找到对应的命令

```shell
pacman -Ql xmind | grep desktop
# 输出 xmind /usr/share/applications/xmind.desktop

cat /usr/share/applications/xmind.desktop | grep Exec
# 输出 Exec=XMind %F
```

得知启动命令是 XMind，那么直接执行 XMind 试试之后，得到如下报错，同时弹出上面那错误对话框。

```
Unrecognized option: --add-modules=java.se.ee
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
```

提示很明显，不识别的参数 --add-modules=java.se.ee，结合上面安装的时候给的信息，去编辑 /usr/share/xmind/XMind/XMind.ini，发现里面有 --add-modules=java.se.ee 这一行，把这一行删掉。

然后再输入 XMind ，发现启动成功！



### 深入探究

抱着追根究低的态度继续思考，--add-modules=java.se.ee 有什么意义呢为什么会被加上？再细细读打包者给出的信息的最后一句话

```
and add/delete "--add-modules=java.se.ee" at the end of file if you use java 10/8.
```

很容易猜出，java 10 可能支持 --add-modules=java.se.ee，而 java 8 不支持。

看看当前的系统中的 java 版本吧

```
java -version
```

果然是 1.8 版本。

那么我们装个 java 10 试试。

```
sudo pacman -S java-runtime=10
```

然后用 java 切换脚本（详见 [java的archwiki](https://wiki.archlinux.org/index.php/Java_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))），切换成 java 10

```
# 列出系统中所有 java 环境
archlinux-java status

# 切换 java 10 为默认环境
sudo archlinux-java set java-10-openjdk

# 查看切换结果
archlinux-java status
java -version
```

然后再试试启动 XMind，发现启动不了了，如下弹窗报错

![xmind-error1](/img/xmind-error1.png)

去看这个日志，发现一大堆，因为没学过太多 java，所以看不懂也并不想看。

直接把 --add-modules=java.se.ee 加回 /usr/share/xmind/XMind/XMind.ini 再试试启动，发现启动成功。

也就是验证了前面猜测是正确的：

1. 在 java 8 的环境下，不能有 --add-modules=java.se.ee
2. 在 java 10 的环境下，必须有 --add-modules=java.se.ee



## 修改 PKGBUILD 重新打包

我再打开这个包的 PKGBUILD 一看，发现有 JAVA_VERSION=10 语句，然后 package 函数里有 if [[ "$JAVA_VERSION" != "8" ]]; then 来决定是否往里写 --add-modules=java.se.ee

但是就算在这里改了 JAVA_VERSION 这个变量，那谁知道用户的机子里默认的是 java8 还是 java10 呢？这里 JAVA_VERSION 设置的是 10 ，而大多数用户默认肯定是 8，这么一来，岂不是对新手很不友好？

有没有更好的打包方案呢，有，我觉得可以这样做：

1. 将依赖行为 depends=('java-runtime>=8') 改成 depends=(“java-runtime=$JAVA_VERSION”)

2. 将启动脚本里面加上一行对应的 PATH 变量来显式指定 java 环境。

   即在最后一行 cp ${srcdir}/XMind              ${pkgdir}/usr/bin/ 后面加

   ```
   sed -i '/exec/iPATH=/usr/lib/jvm/java-'"$JAVA_VERSION"'-openjdk/bin:$PATH' ${pkgdir}/usr/bin/XMind
   ```

至此，观察到里面选择 gtk2 还是 gtk3 也是同样的道理。

最终给出我修改后完整的 PKGBUILD

```shell
# $Id: PKGBUILD 184754 2016-08-01 15:30:30Z felixonmars $
# Maintainer: RemiliaForever <remilia AT koumakan DOT cc>
# Contributor: Felix Yan <felixonmars@gmail.com>
# Contributor: Christoph Drexler <chrdr at gmx dot at>
# Contributor: Jelle van der Waa <jellevdwaa@gmail.com>

# GTK_VERSION 2/3
GTK_VERSION=3
# JAVA_VERSION 8/10
JAVA_VERSION=10

pkgname=xmind
pkgver=3.7.8+8update8
_filename=$pkgname-8-update8-linux
pkgrel=1
pkgdesc="Brainstorming and Mind Mapping Software"
arch=('i686' 'x86_64')
url="http://www.xmind.net"
license=('EPL' 'LGPL')
depends=("java-runtime=$JAVA_VERSION" "gtk$GTK_VERSION")
optdepends=('lame: needed for the feature audio notes')
install=xmind.install
source=("http://www.xmind.net/xmind/downloads/${_filename}.zip"
'XMind'
'xmind.desktop'
'xmind.xml'
'xmind.png'
'xmind_file.png')
sha512sums=('77c5c05801f3ad3c0bf5550fa20c406f64f3f5fa31321a53786ac1939053f5c4f0d0fb8ab1af0a9b574e3950342325b9c32cf2e9a11bf00a1d74d2be1df75768'
'SKIP'
'SKIP'
'SKIP'
'SKIP'
'SKIP')

package() {
    mkdir -p ${pkgdir}/usr/share/${pkgname}
    cp -r ${srcdir}/configuration   ${pkgdir}/usr/share/${pkgname}/
    cp -r ${srcdir}/features        ${pkgdir}/usr/share/${pkgname}/
    cp -r ${srcdir}/plugins         ${pkgdir}/usr/share/${pkgname}/
    cp ${srcdir}/*.xml              ${pkgdir}/usr/share/${pkgname}/
    mkdir -p ${pkgdir}/usr/share/licenses/${pkgname}
    cp ${srcdir}/{epl-v10,lgpl-3.0}.html    ${pkgdir}/usr/share/licenses/${pkgname}/
    cp ${srcdir}/xpla.txt                   ${pkgdir}/usr/share/licenses/${pkgname}/
    if [[ "$CARCH" == "i686" ]]; then
        cp -r ${srcdir}/XMind_i386  ${pkgdir}/usr/share/${pkgname}/XMind
    else
        cp -r ${srcdir}/XMind_amd64 ${pkgdir}/usr/share/${pkgname}/XMind
    fi
    mkdir -p ${pkgdir}/usr/share/fonts/${pkgname}
    cp -r ${srcdir}/fonts           ${pkgdir}/usr/share/fonts/${pkgname}/
    mkdir -p ${pkgdir}/usr/share/applications
    cp ${srcdir}/xmind.desktop      ${pkgdir}/usr/share/applications/
    mkdir -p ${pkgdir}/usr/share/mime/packages
    cp ${srcdir}/xmind.xml          ${pkgdir}/usr/share/mime/packages/
    mkdir -p ${pkgdir}/usr/share/pixmaps
    cp ${srcdir}/*.png              ${pkgdir}/usr/share/pixmaps/
    # fix configuration
    sed -i "s|^./configuration$|@user.home/.xmind/configuration|" ${pkgdir}/usr/share/${pkgname}/XMind/XMind.ini
    sed -i "s|^../workspace$|@user.home/.xmind/workspace|" ${pkgdir}/usr/share/${pkgname}/XMind/XMind.ini
    if [[ "$GTK_VERSION" != "2" ]]; then
        sed -i "s|^2$|3|" ${pkgdir}/usr/share/${pkgname}/XMind/XMind.ini
    fi
    if [[ "$JAVA_VERSION" != "8" ]]; then
        echo "--add-modules=java.se.ee" >> ${pkgdir}/usr/share/${pkgname}/XMind/XMind.ini
    fi
    mkdir -p ${pkgdir}/usr/bin
    cp ${srcdir}/XMind              ${pkgdir}/usr/bin/
    sed -i '/exec/iPATH=/usr/lib/jvm/java-'"$JAVA_VERSION"'-openjdk/bin:$PATH' ${pkgdir}/usr/bin/XMind
}
```

然后可以随意修改 GTK_VERSION 和 JAVA_VERSION 来达到目的，并且还不用额外修改任何东西，无论如何切换默认 java 版本，都不会再影响启动了。

重新打包测试，成功。

