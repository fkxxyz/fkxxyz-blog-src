---
title: 记一次 excel 文档的密码破解的探索
cover: false
date: 2019-11-26 16:11:08
updated: 2019-11-26 16:11:08
categories:
- 探究学习
tags:
- 信息安全
- python
---

周末的时候，有个以前的同学说自己的 excel 密码忘记，所以找我帮忙看有没有办法破解开来。

由于之前从来没接触过这样的实战，对加密解密仅仅只是了解的概念，基本都是理论知识，于是开始了实战探索之路。

<!--more-->

## 探索之路

### 已有知识

在印象中，几年前下载过这样的破解 office 密码的软件，名字是 Advanced Office Password Recovery，还接触过很多压缩包密码破解，听说过 wifi 密码破解等等。

根据所学过的知识，我将密码破解分为三类：

1. **暴力破解：**这种方式是利用计算机高计算速度的特性，让程序按一定的规律或规则，一个一个试密码，直到试出来为止。
2. **逆向算法以及哈希碰撞：**一般的加密算法是不可能那么轻易可逆的，也没那么容易碰撞，一般只有顶尖的数学家密码学家牛人才能探索出这种算法，当今普遍采用的算法一般都是还没被探索出碰撞的。
3. **伪加密直接清除密码**：确切的说只对没有加密的文档管用，可以说是伪加密，仅仅是软件让你输入密码数对了才能访问到真正数据，而真正数据并未加密，严格说不算一种破解方法，局限性很大。

由于她用的是 excel 2013 的加密文档，然后去网上搜索 excel 2013 的解密，无一例外都是暴力破解的方式，和我想的一样，微软不可能傻到伪加密，也不可能用一些已经被破解了的不安全的算法，基本能确定只有暴力破解这一种方法。

### 软件尝试

这次破解密码，我首先尝试的是 Advanced Office Password Recovery，由于用的 archlinux 系统，直接使用 wine 来安装运行，启动成功。

先看看这款软件能否行的通吧，首先，我打开我电脑里已装好的 excel 2007，然后里面随便写点东西，然后加密保存，密码设为 qwertyuio，然后新建一个字典，随便输几行，并且把真实密码也放进去，打开这软件，加载字典，开始搜索，果然，秒出结果。证明了这软件还是行的通的。

### 字典构造

再搜集线索，这样可以缩小搜索范围，得知她设的是一句类似于“工作让我想死”的每个字的首字母组合，长度在 6～9 位，那么接下来让她想尽可能多的词语。比如“让人”、“令人”、“心烦”等等类似的词，首字母记下来。

然后再仔细想想其中的规律，”gz“肯定是开头两位，中间的词语不确定，有几个备选词语，结尾词语也不确定，同样几个备选，我设想这句话可以分为几个部分，每个部分都是一个词语，把每一部分写成一行，每一行里用逗号隔开词语的所有可能性，然后写了如下 txt 文档。

```
gz
,zs,zd
l,s,r
w,r
xf,xs
```

其中第二行逗号开头表示可缺省，那么一共有 1\*3\*3\*2\*2=36 种情况，虽然手动一个一个试是可以试完的，但是设想到万一都不对肯定要考虑别的词语，倒不如编个程序，根据上述 txt 文档来生成一个包含所有情况的字典，然后就可以放到破解软件里去跑，然后就可以随时加词语再生成字典再去跑。

说动手就动手，十分钟后，一个简易的 python 程序诞生。

```python
#!/bin/python

with open('a.txt') as f:
    r=f.read().strip()

lines=r.split('\n')
words=[]
for line in lines:
	words.append(line.split(','))

l=len(words)
def rec(s,i):
    if i==l:
        print(s)
        return
    
    for j in words[i]:
        rec(s+j,i+1)

rec('',0)
```

思路也很清晰，固定读取 a.txt，用一个递归函数，一行一行的遍历，达到最深（最后一行）时打印出来此情况并且回溯。

### 开始破解

把程序生成的字典放到软件里去跑，立马出结果，说密码未找到，说明 36 种情况都不是。

于是把思路和规则告诉她，让她加词语，加情况。通过实验得知，我的电脑每秒钟能试 400 次密码。

然后加了几个词语继续试。

120种情况：

```
gz
,zs,zd,ztm
l,s,r
w,r
xs,xf,fs,ns,ty
```

400种情况：

```
gz
,z,t,y,u,I,o,p,g,h,j,k,l,v,b,n,m
l,s,r
w,r
xs,xf,fs,
```

153900种情况（约6分钟）

```
gz
,z,t,y,u,I,o,p,g,h,j,k,l,v,b,n,m,
,d,s,h,x
z,t,y,u,I,o,p,g,h,j,k,l,v,b,n,m,l,s,r
z,t,y,u,I,o,p,g,h,j,k,l,v,b,n,m,w,r
xs,xf,fs,xyqs,xqs
```

均未果。

再后来，她觉得，大周末的也挺麻烦的，而且反正也不是什么重要的东西，文档也可以慢慢补回来，就决定放弃了。

尽管没能解决问题，也不重要了，给她带来的陪伴价值是很珍贵的，同时也让我有了一些破解经验。



## 后续探索

在闲暇之余，我开始去网上搜相关的文章博客论坛等等，看看别人的破解经验，在浏览了 20+ 篇的文章之后，发现了这么一个软件：[hashcat](https://www.hashcat.net)，自称是世界上最快的最先进的密码恢复工具，而且还开源免费，支持包括 MS Office 2013 在内的上百种类型的 hash 破解。

看了介绍就觉得是神器，我必须试试了。

### 尝试用 hashcat 破解 linux 用户登录密码

在 CSDN上 [hashcat的学习和使用记录](https://blog.csdn.net/qq_37865996/article/details/83863075) 这篇博客的引导下，我开始了第一次尝试，尝试实验破解 linux 的用户登录密码。

#### 建立测试用户

首先，建立一个新用户 qwer，密码为 123456

```shell
sudo useradd -m qwer
sudo passwd qwer
# 然后输入两次 123456
```

尝试登录

```shell
su - qwer
# 然后输入密码 123456
```

登录成功。

#### 提取哈希

我知道，用户的密码是哈希加密之后，放在了 /etc/shadow 里，这个文件对普通用户是没有读取权限的，只有 root 用户能访问，直接查看这文件，取得哈希。

```shell
sudo cat /etc/shadow | grep qwer
```

输出结果为

```
qwer:$6$teTb/7A1JUDBJ3Ea$oU8XOGYd.nN96vi7x0yzFdrhQVk5IcK4AHn/gKPcBFHuXtFtFsF64628pPQBI0yEeJH47E5jqLdgTZkYUR7Rs1:18225:0:99999:7:::
```

保留 hashcat 所需要的值，去掉不需要的之后

```
$6$teTb/7A1JUDBJ3Ea$oU8XOGYd.nN96vi7x0yzFdrhQVk5IcK4AHn/gKPcBFHuXtFtFsF64628pPQBI0yEeJH47E5jqLdgTZkYUR7Rs1
```

把这串值写到一个文本文件里，建个工作目录放进去吧

```shell
cd
mkdir hashcat
cd hashcat
echo '$6$teTb/7A1JUDBJ3Ea$oU8XOGYd.nN96vi7x0yzFdrhQVk5IcK4AHn/gKPcBFHuXtFtFsF64628pPQBI0yEeJH47E5jqLdgTZkYUR7Rs1' > hash.txt
```

这样，哈希文件就准备好了。

#### 构造字典

总得要构造个字典才能实验，虽然能直接指定掩码暴力的方式，但我更钟爱字典。

继续写个简易的 python 程序来构造一个字典吧

```python
#!/bin/bash
import itertools
it = itertools.product('1234567890', repeat=6)
for a in it:
	print(''.join(a))
```

然后运行这个程序

```shell
chmod +x gendict
./gendict > dict.txt
```

然后看看正确密码在哪一行

```shell
cat dict.txt | grep -n 123456
```

得知在第 987655 行，要搜索这么多次，正好可以看看 hashcat 有多快。

#### 开始破解

直接执行

```shell 
hashcat -m 1800 -a 0 -o output.txt hash.txt dict.txt
```

各参数解释

1. **-m 1800**  指定 hash 的类型的 id ，这里是破解 linux 密码所以 id 是 1800，通过 hashcat --help 可以看到解密不同类型 hash 的 id。
2. **-a 0**  指定攻击模式， 0代表字典，3代表掩码暴力等等，这里直接指定字典。
3. **-o output.txt**   指定破解出来之后，输出结果保存的文件。
4. **hash.txt**  指定要破解的 hash 文件。
5. **dict.txt**  指定字典。

ok，按回车之后，提示失败，输出结果

```
hashcat (v5.1.0) starting...

clGetPlatformIDs(): CL_PLATFORM_NOT_FOUND_KHR

Started: Wed Nov 25 22:20:45 2019
Stopped: Wed Nov 25 22:20:45 2019
```

这是为什么呢，百度了一下 clGetPlatformIDs 这个函数，查到应该是个 opencl 相关的函数，这个函数顾名思义，这是在寻找平台吗，也就是 hashcat 默认是要用 GPU 来破解计算的，哈哈，难怪堪称世界第一。那好嘛，我的 nvidia 显卡肯定是被大黄蜂禁用的状态，你会 not found 很正常咯，我直接启用。

```
optirun hashcat -m 1800 -a 0 -o output.txt hash.txt dict.txt
```

这下好像成功了，开始破解了，虽然看到一些警告。

最后一行是

```
[s]tatus [p]ause [b]ypass [c]heckpoint [q]uit =>
```

可以看出是几个选项，可以输首字母来操控，输入个 s 提示了以下信息

```
Session..........: hashcat
Status...........: Running
Hash.Type........: sha512crypt $6$, SHA512 (Unix)
Hash.Target......: $6$teTb/7A1JUDBJ3Ea$oU8XOGYd.nN96vi7x0yzFdrhQVk5IcK...UR7Rs1
Time.Started.....: Wed Nov 25 22:42:10 2019 (36 secs)
Time.Estimated...: Wed Nov 25 23:10:08 2019 (27 mins, 22 secs)
Guess.Base.......: File (dict.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:      596 H/s (10.19ms) @ Accel:32 Loops:16 Thr:32 Vec:1
Recovered........: 0/1 (0.00%) Digests, 0/1 (0.00%) Salts
Progress.........: 20480/1000000 (2.05%)
Rejected.........: 0/20480 (0.00%)
Restore.Point....: 20480/1000000 (2.05%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:1968-1984
Candidates.#1....: 080620 -> 088583
Hardware.Mon.#1..: Temp: 64c
```

很容易读懂这些信息，我最关心的是速度，速度是 596 次每秒，那么....正确密码在 987655 行，算一算需要 27 分钟才能找到密码。

我只是想尝试一下而已，不想费那么久时间还烧显卡。直接输入 q 退出。然后手动打开字典改一改，把 123456 放到大概一万行的位置吧，然后再执行破解。

很快，执行结束了，出结果了。

```
Session..........: hashcat
Status...........: Cracked
Hash.Type........: sha512crypt $6$, SHA512 (Unix)
Hash.Target......: $6$teTb/7A1JUDBJ3Ea$oU8XOGYd.nN96vi7x0yzFdrhQVk5IcK...UR7Rs1
Time.Started.....: Wed Nov 25 22:50:57 2019 (21 secs)
Time.Estimated...: Wed Nov 25 22:51:18 2019 (0 secs)
Guess.Base.......: File (dict.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:      602 H/s (10.22ms) @ Accel:32 Loops:16 Thr:32 Vec:1
Recovered........: 1/1 (100.00%) Digests, 1/1 (100.00%) Salts
Progress.........: 12288/1000000 (1.23%)
Rejected.........: 0/12288 (0.00%)
Restore.Point....: 10240/1000000 (1.02%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:4992-5000
Candidates.#1....: 090860 -> 098823
Hardware.Mon.#1..: Temp: 67c

Started: Wed Nov 25 22:50:37 2019
Stopped: Wed Nov 25 22:51:19 2019
```

在第二行 Status...........: Cracked ，可以得知已经破解了。然后 ls 一下发现果然有个 output.txt，执行

```
cat output.txt
```

看看结果

```
$6$teTb/7A1JUDBJ3Ea$oU8XOGYd.nN96vi7x0yzFdrhQVk5IcK4AHn/gKPcBFHuXtFtFsF64628pPQBI0yEeJH47E5jqLdgTZkYUR7Rs1:123456
```

好的，这次尝试很顺利，123456 被破解出来了。

### 尝试用 hashcat 破解 excel 密码

已经熟悉了 hashcat 的基本使用了，该用它试试满足我的需求了。

得知提取 office 文档的 hash 用的是开源的 office2john，可以直接下载

```
wget https://github.com/magnumripper/JohnTheRipper/raw/bleeding-jumbo/run/office2john.py
```

然后执行它，指定要破解的 xlsx 文件。

```shell
python office2john.py 2.xlsx > 2.hash
```

然后看看 2.hash 的内容，把开头的 2.xlsx 和冒号去掉。

用 hashcat --help 查询得知 office 2013 的加密方式 id 是 9600，那我可以用原来 python 生成的字典开始了。

```shell
optirun hashcat -m 9600 -a 0 -o output.txt 2.hash dict.txt
```

很快，又跑起来了。看到，速度，只有？158 H/s ？？还没有 Advanced Office Password Recovery？ 说好的世界第一呢？

### 严重失误的发现

抱着好奇的态度，我再次打开 Advanced Office Password Recovery，然后打开 2.xlsx ，加载字典，然后开始，发现速度只有 16 H/s ？？

那么我前面那个实验得出的 400 次每秒是怎么回事，难道是我刚刚把电脑烧烫了，速度就慢了？那也不该慢这么多，我重复着之前的动作，发现，我之前一直加载的根本不是 2.xlsx ，而是用的我自己最开始实验的时候，用 excel 2007 加密的文档！我再次打开我那用来实验加密的 excel 2007 的文档，发现速度又是 400次每秒了。

也就是说，我后来帮她破解的时候，根本没有选对文件，我一直破解的是我自己加密的文档，这是一个严重的失误，导致我后续的暴力破解全都是无效的破解。

然后后续继续实验，作出对比总结

1. 用 Advanced Office Password Recovery ：
   1. 破解 office 2007 速度为 400 H/s
   2. 破解 office 2013 速度为 16 H/s
2. 用 hashcat ：
   1. 破解 office 2007 速度为 2221 H/s
   2. 破解 office 2013 速度为 158 H/s

用 GPU 计算确实速度快了不少，虽然我这是几年前的垃圾卡。

对于失误，我后来又重新用 hashcat 跑了一遍，之前生成的字典，那个15万次的用了半小时。然后发现 hashcat 有个细节很贴心

```
Watchdog: Temperature abort trigger set to 90c
```

温度超过 90 度时自动停止，休息一会，然后可以运行 hashcat --restore 来接着上次的破解，能很好的保护机器，后续探索发现会话保存在 ~/.hashcat/session 里面，也可以用命令行参数来指定会话的位置，看来，hashcat 是一个很成熟考虑周全的软件了，有时间的话，我会总结出它所有的功能用法，列出来以便以后使用。

发现失误后，用 hashcat 加载以前的字典，跑了半小时还是没跑出来，不过这次经历让我收获很多。

## 总结

hashcat 能破解上百种类型的 hash，而且支持 GPU 运算，对高性能的显卡来说速度会更快。那么破解 hashcat 支持的 hash 类型，我把步骤程序化，以后遇到要破解的问题都按这个思路了。

1. **提取要破解的 hash**

   每种文档或者密码，提取 hash 的方式不同，以后遇到什么样的密码，可以现查资料。

2. **尽可能搜集多的关于密码的情报**

   这一步主要是为了生成字典，尽可能了解多的信息，来缩小搜索范围。

3. **生成字典**

   生成字典可以自己写 python 程序来实现，可以妙用迭代器来快速生成，也可以利用 crunch 等软件来生成字典。

4. **用 hashcat 开始破解**

   破解的过程中可能出现温度过高的情况，那么可以用计划任务的方式尽情发挥想象力破解。

