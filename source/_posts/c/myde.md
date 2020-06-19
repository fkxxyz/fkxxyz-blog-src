---
title: 猜猜我最终选择了什么桌面环境
cover: false
date: 2019-12-18 23:15:51
updated: 2019-12-18 23:15:51
categories:
- 教程
tags:
- archlinux
- 桌面环境
typora-root-url: ../..
---

接触archlinux也有两年多了，桌面环境到底应该选哪个，我也纠结过这个问题，而且桌面环境各有各的优点。

gnome和kde虽然完善但过于庞大，性能不好时常卡顿。deepin的桌面环境虽然漂亮但是bug多时常也假死，lxde、lxqt这些轻量桌面环境虽然小巧但是界面美观性堪忧，xfce4美观比不上庞大桌面环境，性能不如lxde都不占优，用平铺式的如i3、dwm等也不太容易适应，还有fvwm？那配置复杂度了根本没时间搞那玩意好嘛。桌面环境的选择简直难上加难啊，哈哈。

<!--more-->

![](/img/myde.png)

## 我的探索历程

### 体验各个桌面环境

由于包的数量众多，要实验各个软件，就会留下很多包，尤其是桌面环境涉及到的包更多依赖更复杂，很难找到这些包名，卸载的话会漏很多，强迫症的我不想留一些不必要的软件，一开始比较蠢，把每次装了什么都记录下来，然后之后要卸载的时候，一条一条的 pacman -Rsc 。由于有着重复的工作都能用编程代替的思想，就开始思考，为何不自己编个脚本来代替这个重复的过程呢？我能不能把我需要的软件记录到一起，然后脚本自动为我卸载不必要的包呢？

于是，一个伟大的白名单列表机制管理软件包的脚本诞生了。[详见spacman](/d/spacman/)

只需要手动记录一下我所有需要的顶层包到一个文本文档，然后脚本读取文本文档，和系统里的所有包进行对比，按照一系列依赖计算，算出多余了哪些包，一下子卸载得没有残留。

这下，可以放心的实验各种桌面环境了，我列表定好，然后随便装什么包，不用关心装了多少东西，然后一条 spacman -a 命令直接把系统打回原样，这感觉就和虚拟机的快照一样。

秒切换桌面环境的梦实现了，实验效率能达到十分钟之内能体验五个桌面环境，而且迅速卸载无残留。

接下来开始先后体验了这些桌面环境。

#### xfce4

这是我一开始的主力桌面环境，用了很久，启动速度有时候感觉慢了一点，虽然比windows快，美观也感觉差一些，虽然比lxde好些，基本功能挺全。

#### lxde

追求轻量一般选择这个，lxqt也是类似，速度很快，就是功能少了点，美观也都没考虑，openbox能换换主题就不错了。

#### kde

然后开始再次实验这个kde在archlinux下运行速度咋样。这经历不提了，我i3的cpu和机械硬盘，简直伤不起，进个桌面活活用了一分钟才稳定下来，不知道后台在干嘛，硬盘疯狂转，我的天哪，我好怕硬盘坏掉。。。。。赶紧注销执行 spacman -a，拜拜。

#### gnome

这个桌面总的来说还行，但是不方便的一点就是通知栏显示QQ的问题，还有应用列表也没有很好的分类，或许可以配置但是没深究，流畅度还可以，没有像kde卡得那么夸张。不过还是不喜欢这么庞大的东西，不过我觉得对于很多人来说这个桌面环境是个不错的选择。

#### mate

这是我后来才听说的桌面，各种组件也挺简约，但是面板自定制程度不高，还是不喜欢，也就没用多久。

### 组件拆开的探索

试了好几个桌面，发现还是xfce4最合我的口味，但是速度不理想而且美观程度一般。那我能不能把桌面组件全部拆开呢，自己来选择用什么面板用什么窗口管理器用什么文件管理器呢？

答案是可以的！linux正适合这样高度定制，尤其是 archlinux，而且我还发现了这个 [打造自己的桌面环境的官方wiki](https://wiki.archlinux.org/index.php/Desktop_environment_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E8%87%AA%E5%B7%B1%E6%89%93%E9%80%A0%E6%A1%8C%E9%9D%A2%E7%8E%AF%E5%A2%83) 。

可以配合 startx 来读取 ~/.xinitrc 文件配置，来实现自定制桌面环境，参见 [archwiki-xinit](https://wiki.archlinux.org/index.php/Xinit_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))，当然这需要一定的 shell 脚本编写能力和实验探索精神。

思维打开之后，开始一个一个组件的研究，鉴于窗口管理器是一个桌面环境的基本组件，那先从窗口管理器开始试吧。

#### 选择窗口管理器

xfwm是xfce自带，然后又试了openbox，然后了解到compiz，一开始对compiz不熟，但是用过之后，发现这窗口管理器太漂亮了，而且自定制程度简直秒杀所有窗口管理器。

#### 选择面板

然后找面板，一共预选出三个面板：

1. xfce4-panel

   论面板，感觉 xfce4 的还是老大，电源管理的托盘、声音调整的图标，应用列表菜单，都很对胃口。

2. lxpanel

   lxpanel 虽然也有 xfce4面板的基本功能，但是美观度差了点。

3. mate-panel

   mate的面板定制程度不高，连顺序都不能自由调整，不嫌弃的话可以用。

所以面板决定用 xfce4-panel 了。

#### 尝试 startx

首先卸载所有桌面环境，然后安装 openbox，确保 xorg-server 和 xorg-xinit 也装上了才能 startx。

```shell
pacman -S xorg-xinit
pacman -S openbox
```

然后编辑文件 ~/.xinitrc

写入几行

```shell
openbox
```

然后执行 startx

```shell
startx
```

可以发现进入了 openbox 的界面，启动也很快。由于缺个终端，可以在 tty下装个终端，然后可以从 openbox 的右键打开，然后输入 xfce4-panel ，居然真的启动了面板，这样就很不错。

总结出，拆开组件的装，是完全可行的，那么把 xfce4-panel 也写入 .xinitrc 呢？

```shell
xfce4-panel &
openbox
```

后面加了个 & 符号是为了让它后台运行接着执行下一条命令，不至于卡住这等着面板被结束。

那怎么注销呢，只能用结束 openbox的命令代替注销了

```shell
killall openbox
```

然后，killall openbox ，再次 startx ，发现面板和窗口管理器都启动成功了。

上述要是哪里卡住，可以按 ctrl + alt + 1~6 来回到 tty，然后管理进程 killall 结束掉一些进程。

#### 会话管理器

再后来的探索，发现，会话管理器也是一个桌面环境的组件之一，它的功能大概有管理启动项，掌控用户登录注销和开关机功能等。

启动项分为两类，一类是系统启动项，一类是用户启动项

##### 系统启动项

启动项都是放在 /etc/xdg/autostart 里面的，可以看看哪些软件把自己加入了启动项

```
pacman -Qo /etc/xdg/autostart
```

##### 用户启动项

用户启动项放在 ~/.config/autostart 里面，通常由各个软件加入进去。

而我刚刚的 startx，只启动了 openbox 和 xfce4-panel

那么我可以不要会话管理器了，把一切启动项都交给 .xinitrc 管理怎么样？是不是好办法。

在 .xinitrc 里面写入（需要安装 exo）

```shell
if [ -d /etc/xdg/autostart ] ; then
 for f in /etc/xdg/autostart/*.desktop ; do
  exo-open "$f" &
 done
 unset f
fi
```

然后 killall openbox 再 startx ，发现启动项也全启动了，同理，用户启动项也可以这样干

```shell
if [ -d ~/.config/autostart ] ; then
 for f in ~/.config/autostart/*.desktop ; do
  [ -x "$f" ] && exo-open "$f" &
 done
 unset f
fi
```

加了一层判断，用 [ -x "$f" ] 来判断文件是否可执行，可执行才启动它，然后我可以去 ~/.config/autostart 用标记文件是否可执行的方式来管理用户启动项了。

例如开机启动 conky，创建一个 conky.desktop 到  ~/.config/autostart 里面，然后写入 

```ini
[Desktop Entry]
Exec="conky"
Type=Application
```

然后将其设为可执行

```shell
chmod +x ~/.config/autostart/conky.desktop
```

然后再 killall openbox在 startx，可见 conky也启动了，类似的，所有自己想自定义让什么开机启动的，都可以这么设置

#### 完整的编写 .xinitrc

对于自定制桌面环境组件来说，自己编写 .xinitrc 至关重要，在这里总结出它的功能，然后完整的编写

1. 管理会话，startx会启动此脚本，此脚本运行完意味着 startx 的结束，会回到 tty。
2. 启动面板、窗口管理器等组件
3. 管理系统启动项、用户启动项

由于每次 killall openbox 很麻烦，而且万一 openbox 出bug崩溃岂不是所有进程都会强制退出，要是没保存什么文档岂不是悲剧，想个什么办法才能维持 startx 的运行呢？

可以想到办法，用 sleep 代替

```shell
sleep 3650000d
```

这条语句的意思是等待一万年，放到 .xinitrc 的最后，怎么样，不怕 openbox 崩溃了吧。但是引发另一个问题，如何注销？killall sleep 吗？万一我某些别的脚本正在 sleep 怎么办，岂不是一起结束了？改名吧。最终方案是把 sleep 这个二进制文件复制到 /tmp/ 里面的某个地方改成别的名字，然后启动它，这样进程名就不是 sleep了，可以指定 pid 杀死进程，把 pid 也保存到 /tmp/目录吧，最终代码如下

```shell
# deamon process
daemon_dir=/tmp/xinitrc_deamon_$USER
mkdir -p $daemon_dir
cp -f /usr/bin/sleep $daemon_dir/xinitrc_deamon_$USER
$daemon_dir/xinitrc_deamon_$USER 3650000d &
main_pid=$!
echo $main_pid > $daemon_dir/pid

#### 各个启动项（注意每个启动项后面都要加 & ，防止停住）
...

wait $main_pid
```

#### 输入法问题

装 fcitx 的中文输入法（如搜狗拼音）要配置环境变量，网上很多都是说配置到 ~/.xprofile 里面，而自己组件拆开是默认不会读取这个文件的。所以，在 .xinitrc 开头加上读取，兼容这一特点。

```shell
[ -f ~/.xprofile ] && . ~/.xprofile
```

用 startx 启动的时候，为了让中文输入法正常工作，需要五个环境变量，这是我的 .xprofile

```shell
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"

export LANG="zh_CN.UTF-8"
export LC_CTYPE="zh_CN.UTF-8"
```

当然 LANG 和 LC_CTYPE 这两个环境变量可能在桌面管理器的配置里面配置好了，但是 startx 的时候是没配置的，所以这两个再在这里面写一遍，兼容所有。

## 各个组件的选择

下面列出我几个桌面环境组件目前的选择

### 窗口管理器

窗口管理器我选择 compiz，原因很简单，超级美观，堪比 vista，而且所有颜色自定义，所有动画自定义，所有快捷键功能自定义，设置多的研究不过来，找不到比它更爽是桌面管理器了。

![](/img/ccsm-1.png)
![](/img/ccsm-2.png)
![](/img/ccsm-3.png)
![](/img/emerald-1.png)
![](/img/emerald-2.png)
![](/img/emerald-3.png)
![](/img/emerald-4.png)


安装方法也很简单，在 AUR 里面安装compiz-core、ccsm、emerald 即可

```shell
yay -S ccsm compiz-core emerald
```

装完后，启动也很简单，直接 compiz 就可以，将它写入 .xinitrc，替换掉 openbox

定制：

1. 运行 ccsm，就可以打开 compiz 的设置，可以进行快捷键、窗口动画等等设置。

   还有更多设置扩展，觉得定制不过瘾的，都可以选择性的安装如下

   ```shell
   yay -S compiz-fusion-plugins-main
   yay -S compiz-fusion-plugins-extra
   yay -S compiz-fusion-plugins-experimental
   ```

2. 运行 emerald-theme-manager 就可以进行装饰器的主题定制，还可以装一些已经有的主题

   ```shell
   yay -S emerald-themes
   ```

   我基于其中一个主题定制成了高仿 wista 和win7的主题，也上传到了aur包名是 emerald-theme-qaz-blue-vista

   ```shell
   yay -S emerald-theme-qaz-blue-vista
   ```

以上的定制，只是对窗口管理器的定制，边框等等，至于窗口内元素颜色等等，可以用 gtk主题去设置（后面有讲到）。

当然，最后必须说明的是，在 archlinux 里选择 compiz 还是有一些缺点的：

1. 由于没进入官方仓库，每次更新安装都得从 aur 编译，编译需要时间，还有就是，当arch官方仓库的一些底层库（如 protobuf）版本大更新的时候，compiz 就会出现问题，会找不到库，需要重新从 aur 编译。
2. 有时候开机会小概率的出现装饰器启动不了 bug，也就是窗口没有边框都关闭之类的没有，我也不知道是怎么回事，出现这个bug的时候，可以打开终端运行 nohup emerald --replace >/dev/null & 暂时解决。

### 面板

面板我选 xfce4-panel，因为以下组件很容易配合 xfce4-panel 来添加托盘图标运行

```shell
pacman -S xfce4-pulseaudio-plugin  # pulseaudio 音量管理插件（仅支持 xfce4 面板）
pacman -S xfce4-power-manager   # xfce4 的电源管理器
pacman -S xfce4-whiskermenu-plugin   # 美化的 xfce4 菜单
```

另外，还需要解决两个问题

1. xfce4-panel 配合 compiz 会有些bug，工作区切换会有问题，不过有大神修改了 xfce4-panel的源码修复这些 bug，上传到了 aur，可以直接用，替换掉 xfce4-panel

   ```shell
   yay -S xfce4-panel-compiz
   ```

   但是有一次，xfce4-panel 来了一次大更新，而 aur 上的 xfce4-panel-compiz 迟迟没有更新，我没办法就用了一段时间的 lxpanel

2. 无会话时 whiskermenu 菜单的注销按钮和锁定屏幕的按钮是不管用的。

   我研究过这个问题，单击这俩按钮时，会分别执行 /usr/bin/xflock4 和 /usr/bin/xfce4-session-logout ，那么，这两个可执行程序，我可以自己写 shell 脚本代替。为了方便的解决这个问题，我打包上传到了 aur，符号链接的方式到 /etc/xdg/xfce4/whiskermenu/，方便配合包管理器解决这个问题。
   
   ```shell
yay -S xfce4-whiskermenu-plugin-button
   ```
   
   然后自己编写 /etc/xdg/xfce4/whiskermenu/xfce4-session-logout 和 /etc/xdg/xfce4/whiskermenu/xflock4 ，就可以定义这两个按钮的行为了。

### 会话管理器

前面说过，自己编写的 .xinitrc 脚本来定制启动项，另外，根据 archwiki 的推荐，[oblogout](https://wiki.archlinux.org/index.php/Oblogout) 是不错的选择

```shell
pacman -S oblogout
```

然后执行 oblogout 即可弹出几个按钮，可以实现注销关机等等。里面每个按钮都可以自定义行为，修改 /etc/oblogout.conf 以下是我的配置

```ini
[settings]
usehal = false

[looks]
opacity = 70
bgcolor = black
buttontheme = oxygen
buttons = logout, restart, shutdown, suspend, lock, cancel

[shortcuts]
cancel = Escape
shutdown = S
restart = R
suspend = U
logout = L
hibernate = H
lock = K

[commands]
shutdown = systemctl poweroff
restart = systemctl reboot
suspend = systemctl suspend
hibernate = systemctl hibernate
logout = xlogout
lock = dm-tool switch-to-greeter
#safesuspend = safesuspend
```

其中呢，logout = xlogout 注销是我自己配合 .xinitrc 写的，在 /usr/local/bin/xlogout 里面，就一行代码

```shell
#!/bin/bash
daemon_dir=/tmp/xinitrc_deamon_$USER
kill "$(cat $daemon_dir/pid)"
```

还有 lock = dm-tool switch-to-greeter 这一行，dm-tool 是 lightdm 的组件，如果没用 lightdm，那这条命令是无效的可以删掉。

如果要用 lightdm 或者其它桌面管理器，那么可以从 aur 装这个包，把自己编写的 .xinitrc 作为会话管理器，来让桌面管理器操控

```shell
yay -S xinit-xsession
```

### 终端

我一直使用的是不用配置的 xfce4 终端，可以根据自己喜好选择别的终端。

```shell
pacman -S xfce4-terminal
```

### polkit 身份认证组件

这个组件几乎每个桌面环境都带个，也就是当执行 systemctl 修改系统配置的时候或者其它地方申请权限的时候，会弹出一个对话框申请权限，让你输入密码，大概有这些。

```ini
# polkit 身份认证组件
polkit-gnome  # gnome 默认简易的身份认证
#lxqt-policykit  # lxqt 的身份认证
#mate-polkit  # mate 的身份认证
#deepin-polkit-agent  # deepin 的身份认证
```

在都试了的情况下，对于颜色简约等等，我选择的是 polkit-gnome

```
yay -S polkit-gnome
```

这个服务也需要开机自启，将以下行写入 .xinitrc

```shell
# polkit
/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1 &
```

### 通知服务

通知服务也是一个比较重要的组件之一，当某些软件想发送通知时，右上角会弹出一个漂浮的窗口来提醒用户，也可以 shell 执行来主动发送通知，参见 [archwiki用bash发送通知](https://wiki.archlinux.org/index.php/Desktop_notifications#Bash)

```shell
notify-send 'Hello world!' 'This is an example notification.' --icon=dialog-information
```

![](/img/nodify.png)

它需要一个后台的通知服务程序才能生效，有以下通知服务程序（摘取自[我的软件列表](https://github.com/fkxxyz/archlinux-config/blob/master/spacman/spacman.conf)）

```shell
# 通知服务
#lxqt-notificationd # lxqt 的简易通知服务
xfce4-notifyd  # xfce4 通知服务
#mate-notification-daemon # mate 的通知服务
#notification-daemon # gnome 最原始的通知服务
#notify-osd # unity 的通知服务
#statnot  # 使用python2实现的简易通知脚本
```

也是颜色字体等等各有各的特点，我最终选择的是 xfce4-notifyd。

```shell
yay -S xfce4-notifyd
```

当然，要让他开机自启，将以下行写入 .xinitrc

```shell
/usr/lib/xfce4/notifyd/xfce4-notifyd &
```

### 文件管理器与桌面

上面没有提到过桌面背景，用 compiz 的时候桌面背景是黑的怎么办呢。得装上一个文件管理器，把文件管理器设为自动启动才可以，有以下文件管理器。

```ini
# 文件管理器与桌面
#thunar
	#xfdesktop  # xfce 桌面
	#thunar-volman  # 可移动设备管理
	#thunar-archive-plugin  # 压缩解压插件
	#thunar-media-tags-plugin # 媒体文件标签插件
	#tumbler  # 访问文件的缩略图支持
#pcmanfm    # lxde 的文件管理器和桌面
feh   # 命令行图片查看器（可用来显示壁纸）
#pcmanfm-qt    # lxqt 的文件管理器
#deepin-file-manager  # deepin 的文件管理器
#nemo  # Cinnamon 的文件管理器
#nautilus  # gnome 的文件管理器
#caja  # mate 的文件管理器
```

这个可以根据自己的喜好选择，我曾经用过一段时间的 thunar ，后来换成了 feh 来显示壁纸，不需要文件管理器了，因为对我来说，用shell命令行管理文件比图形界面个管理器要高效的多。

用文件管理器，也需要将它加入 .xinitrc 开机启动。

下面以 pcmanfm 为例

```shell
pcmanfm --desktop &
```

其它文件管理器如何显示桌面，看其 --help 即可

我用的是 feh 显示壁纸

```shell
feh --bg-scale <壁纸路径>
```

用 feh 的好处是，没有后台进程。设置了壁纸了之后，会在家目录生成一个脚本文件

cat ~/.fehbg

```shell
#!/bin/sh
feh --no-fehbg --bg-scale '/usr/share/wallpapers/deepin/Scenery_in_Plateau_by_Arto_Marttinen.jpg'
```

所以很方便的将 feh加入到 .xinitrc

```shell
# wallpaper
~/.fehbg &
```

壁纸的安装也有很多包提供，也可以自己到网上找图片。

```shell
# 壁纸
mate-backgrounds
gnome-backgrounds
deepin-community-wallpapers
archlinux-wallpaper
deepin-wallpapers
xfce4-artwork
deepin-wallpapers-plasma
```

另外，配合 feh 我还遍了两个脚本，可以实现一键切换并设置到下一个或上一个壁纸，自定义壁纸路径，自行体会吧

cat ~/.config/wallpaper-path.conf

```
/usr/share/backgrounds
/usr/share/wallpapers
```

cat wallpaper-next

```shell
#!/bin/bash

path_conf=~/.config/wallpaper-path.conf

p_now="$(tail -1 ~/.fehbg | cut -d"'" -f2)"
p_all="$(find $(cat "$path_conf") -path \*.jpg -type f)"
p_next="$(echo "$p_all" | sed -n '/'${p_now//\//\\/}'/{n;p}')"
[ "${#p_next}" == "0" ] && p_next="$(echo "$p_all" | head -1)"

feh --bg-scale "$p_next"
```

cat wallpaper-prev

```shell
#!/bin/bash

path_conf=~/.config/wallpaper-path.conf

p_now="$(tail -1 ~/.fehbg | cut -d"'" -f2)"
p_all="$(find $(cat "$path_conf") -path \*.jpg -type f)"
p_next="$(echo "$p_all" | sed -n '/'${p_now//\//\\/}'/{x;p};h')"
[ "${#p_next}" == "0" ] && p_next="$(echo "$p_all" | tail -1)"

feh --bg-scale "$p_next"
```

将这两个脚本设为 compiz 的快捷键，直接按键就能切换壁纸。

### 设置

我发现还缺少个设置程序，我目前用的 xfce4-setting ，而且正好 deepin-wine 的QQ是依赖这个组件的。

```shell
pacman -S xfce4-setting
```

也可以选择 lxde 的一些设置程序

```conf
#lxinput  # lxde 鼠标键盘偏好设置
#lxrandr  # lxde 显示器设置
#lxappearance  # lxde 自定义外观和体验
```

包括 gtk 主题也是在这里设置，可以设置样式、图标等等，这些有区别于 compiz 的窗口外观设置。

![](/img/gtk-style.png)

这些主题在这些软件包里面包含，可以都装上，然后自己慢慢选择。

```ini
# 主题和图标
gnome-icon-theme
lxde-icon-theme
mate-themes
mate-icon-theme
gtk-engine-murrine
papirus-icon-theme
arc-gtk-theme
arc-icon-theme
deepin-icon-theme
deepin-gtk-theme
adapta-gtk-theme
numix-icon-theme-git
numix-circle-icon-theme-git
vertex-themes
```

为了方便的预览主题，可以从 aur 安装 awf-git 这个包，切换主题可以实时预览各种控件元素的样式。

### 网络连接管理器

管理网络一般我用的是命令行版的 networkmanager 的 nmcli 命令。图形界面的话，network-manager-applet 就是 nmcli 的前端，它也是 gnome 默认的网络连接管理器，暂时找不到替代

```shell
pacman -S network-manager-applet
```

设置网络连接和连接 wifi 都很方便，tty 里面也可以用 nmcli 管理网络

### 其它软件

其它还有很多很多组件，如计算器、截图工具、图片查看器、归档管理器、浏览器、音乐播放器、文本编辑器、pdf阅读器、录音工具、录屏工具、计时器等等，大多数桌面环境都是自带这些，都可以去尝试，如果喜欢别的哪个桌面环境的某个组件，都可以直接装到自己这里，出了 kde 的组件之外，别的桌面环境的组件依赖都很少，喜欢的话可以直接拿过来用。

可以继续参见[我的试过的所有软件列表](https://github.com/fkxxyz/archlinux-config/blob/master/spacman/spacman.conf)，看看我装过的所有软件，同时也希望大家推荐给我一些更好用的软件。



## 我的完整 .xinitrc 配置

贴上我完整的 .xinitrc 配置，当你看到这篇博客的时候可能很旧了，最新的配置在[我的github的xorg-xinit设置](https://github.com/fkxxyz/archlinux-config/blob/master/xorg-xinit/.xinitrc)

```shell

[ -f ~/.xprofile ] && . ~/.xprofile

# deamon process
daemon_dir=/tmp/xinitrc_deamon_$USER
mkdir -p $daemon_dir
cp -f /usr/bin/sleep $daemon_dir/xinitrc_deamon_$USER
$daemon_dir/xinitrc_deamon_$USER 3650000d &
main_pid=$!
echo $main_pid > $daemon_dir/pid

export > $daemon_dir/xrun
echo 'exec "$@"' >> $daemon_dir/xrun
chmod +x $daemon_dir/xrun

# wallpaper
~/.fehbg &

# window manager
compiz &

# nodify
/usr/lib/xfce4/notifyd/xfce4-notifyd &

# polkit
/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1 &

if [ -d /etc/xdg/autostart ] ; then
 for f in /etc/xdg/autostart/*.desktop ; do
  exo-open "$f" &
 done
 unset f
fi

# panel
xfce4-panel &

if [ -d ~/.config/autostart ] ; then
 for f in ~/.config/autostart/*.desktop ; do
  [ -x "$f" ] && exo-open "$f" &
 done
 unset f
fi

wait $main_pid
```

