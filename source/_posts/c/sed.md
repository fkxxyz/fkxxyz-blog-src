---
title: sed 命令从入门到精通
cover: false
date: 2019-11-29 16:03:38
categories:
- 教程
tags:
- linux
- shell
- sed
---

作为 Linux 三剑客之一的 sed，shell 编程中掌握它是很有必要的。

本文用实验的方式循序渐进地学习 sed，可谓最科学的学习方式，一点一点遍历 sed 所有知识，实现真正的精通，同时还能作为速查。

快来动手跟着一起做吧！

<!--more-->

## 入门实践

边实践边学习是真正掌握技能的唯一途径，下面从易到难来实验 sed 的各个命令。

### 实验准备

为了更好的实验，在临时目录建立一个文本文件以便测试。

```shell
cd /tmp
mkdir a
cd a
```

创建一个文本文件 a.txt ，并写入十行

```shell
for a in $(seq 1 1 10); do echo line$a; done > a.txt
cat a.txt
```

以下是 a.txt 的内容

```
line1
line2
line3
line4
line5
line6
line7
line8
line9
line10
```



### 开始实验

#### a 命令 —— 添加新行

在第 4 行的下面添加一行 newline

```shell
sed '4anewline' a.txt
# 4 表示第 4 行
# a 表示添加
# newline 表示要添加的行

# 为了方便代码阅读，也可以加个'\'或空格，写成如下等价形式
sed '4a\newline' a.txt
sed '4a newline' a.txt
```

输出结果

```
line1
line2
line3
line4
newline
line5
line6
line7
line8
line9
line10
```



也可以一次性添加多行

```shell
sed '4anewline1\nnewline2' a.txt
```

输出结果

```
line1
line2
line3
line4
newline1
newline2
line5
line6
line7
line8
line9
line10
```



#### 指定地址范围

##### , —— 范围分隔符

从第 4 行到第 8 行的所有行的下面分别添加一行 newline

```shell
sed '4,8anewline' a.txt
# 4,8 表示从第4行到第8行，用逗号隔开地址
```

输出结果

```
line1
line2
line3
line4
newline
line5
newline
line6
newline
line7
newline
line8
newline
line9
line10
```



##### + —— 到接下来几行

从第 4 行到其后两行的每一行分别添加一行 newline

```shell
sed '4,+2anewline' a.txt
# 4,+2 表示从第4行到第4+2行，等同于 4,6
```

输出结果

```
line1
line2
line3
line4
newline
line5
newline
line6
newline
line7
line8
line9
line10
```



##### ~ —— 倍数匹配

从第 5 行到第一次遇到的 4 的倍数行的每一行分别添加一行 newline

```shell
sed '5,~4anewline' a.txt
# 5,~4 相当于第 5 行到从第 5 行开始，遇到行号为 4 的倍数的行为止，等价于 5,8
```

输出结果

```
line1
line2
line3
line4
line5
newline
line6
newline
line7
newline
line8
newline
line9
line10
```



##### ~ —— 步进

从第 5 行起，所有行号为 2 的倍数的行后面添加一行 newline

```shell
sed '5~2anewline' a.txt
# 5~2 表示从第 5 行起，所有行号为 2 的倍数的行
```

输出结果

```
line1
line2
line3
line4
line5
newline
line6
line7
newline
line8
line9
newline
line10
```



##### $ —— 最后一行

从第 4 行到最后一行的每一行分别添加一行 newline

```shell
sed '4,$anewline' a.txt
# $ 表示最后一行
# 4,$ 表示从第 4 行到最后一行
```

输出结果

```
line1
line2
line3
line4
newline
line5
newline
line6
newline
line7
newline
line8
newline
line9
newline
line10
newline
```



##### ! —— 范围取反

从第 4 行到第 8 行之外的所有行后面添加一行 newline

```shell
sed '4,8!anewline' a.txt
# ! 表示地址取反
# 4,8! 表示从第 4 行到第 8 行之外的所有行
```

输出结果

```
line1
newline
line2
newline
line3
newline
line4
line5
line6
line7
line8
line9
newline
line10
newline
```



##### // —— 正则表达式范围

在匹配到 line4 的行后面添加一行 newline

```shell
sed '/line4/anewline' a.txt
# /line4/ 表示匹配到 line4 的行

# 等价于
sed '\?line4?anewline' a.txt
# 其中 ? 可以替换成任意字符
```

输出结果

```
line1
line2
line3
line4
newline
line5
line6
line7
line8
line9
line10
```



在匹配到 line4 到第 6 行的行后面添加一行 newline

```shell
sed '/line4/,6anewline' a.txt
# /line4/,6 表示匹配到 line4 到第 6 行的行
```

输出结果

```
line1
line2
line3
line4
newline
line5
line6
line7
line8
line9
line10
```



#### i 命令 —— 插入新行

在第 4 行前插入一行 newline

```shell
sed '4inewline' a.txt
```

输出结果

```
line1
line2
line3
newline
line4
line5
line6
line7
line8
line9
line10
```



#### d 命令 —— 删除

删除 4 到 8 行

```shell
sed '4,8d' a.txt
# d 表示删除
```

输出结果

```
line1
line2
line3
line9
line10
```



#### c 命令 —— 行替换

将 4 到 8 行替换成 newline

```shell
sed '4,8cnewline' a.txt
# c 表示替换
```

输出结果

```
line1
line2
line3
newline
line9
line10
```



#### p 命令 —— 打印

打印一遍 4 到 8 行

```shell
sed '4,8p' a.txt
# p 表示打印
```

输出结果

```
line1
line2
line3
line4
line4
line5
line5
line6
line6
line7
line7
line8
line8
line9
line10
```



只打印 4 到 8 行

```shell
sed -n '4,8p' a.txt
# -n 表示不打印原文
```

输出结果

```
line4
line5
line6
line7
line8
```



从匹配到 “line4” 的行打印到匹配到 “ne6” 的行

```shell
sed -n '/line4/,/e6/p' a.txt
# 本例配合地址可灵活运用
```

输出结果

```
line4
line5
line6
```



#### = 命令 —— 打印行号

从匹配到 “line4” 的行打印到匹配到 “ne6” 的行号

```shell
sed -n '/line4/,/e6/=' a.txt
# = 表示打印那一行的行号
```

输出结果

```
4
5
6
```



#### s 命令 —— 正则替换

将第 4 行到第 8 行的 ne 替换成 on

```shell
sed '4,8s/ne/on/' a.txt
# s 表示替换
#     格式为： s/正则表达式/要替换的内容/
```

输出结果

```
line1
line2
line3
lion4
lion5
lion6
lion7
lion8
line9
line10
```



s 命令最后跟个 g 表示替换该行所有匹配

```shell
echo 'abcdabcd\nbcdebcde' | sed 's/b/x/g'
```

输出结果为

```
axcdaxcd
xcdexcde
```

而不加 g 的 s 只能替换每一行匹配到的第一个

```shell
echo 'abcdabcd\nbcdebcde' | sed 's/b/x/'
```

输出结果为

```
axcdabcd
xcdebcde
```



正则表达式的实例

```shell
sed '4,8s/\(line\)\([[:digit:]]\+\)/\2 - \1/' a.txt
# 4,8 表示从第 4 行到第 8 行
# \(line\)\([[:digit:]]\+\) 是正则表达式
# \1 表示正则表达式中第一个括号匹配的结果
# \2 表示正则表达式中第二个括号匹配的结果

# 可以加 -r 参数开启扩展正则表达式，写成等价形式
sed -r '4,8s/(line)([[:digit:]]+)/\2 - \1/' a.txt
```

输出结果为

```
line1
line2
line3
4 - line
5 - line
6 - line
7 - line
8 - line
line9
line10
```



#### y 命令 —— 字符映射替换

将第 4 行到第 5 行按照字符映射 {e,i,l,n} -> {o,c,e,h} 进行替换

```shell
sed '4,5y/eiln/oceh/' a.txt
```

输出结果

```
line1
line2
line3
echo4
echo5
line6
line7
line8
line9
line10
```



#### r 命令 —— 行替换为文件内容

```shell
# 先写入一个 b.txt 文件作为测试文件
echo 'aaa\nbbb' > b.txt

sed '4,5rb.txt' a.txt
# r 表示以后面作为文件名读取文件内容进行替换
```

输出结果

```
line1
line2
line3
line4
aaa
bbb
line5
aaa
bbb
line6
line7
line8
line9
line10
```



#### {} —— 脚本块

在大括号内用分号隔开命令，可以实现执行多个命令

打印 3 到 4 行和 7 到 8 行

```shell
sed -n '{3,4p;7,8p}' a.txt

# 最外层大括号可省略
sed -n '3,4p;7,8p' a.txt
```

输出结果

```
line3
line4
line7
line8
```



如果每个命令的地址相同，那么也可以类似于像分配律一样的语法

```shell
sed -n '4{p;=}' a.txt

# 等价于
sed -n '{4p;4=}' a.txt
```

输出结果

```
line4
4
```



连 s 命令都能包含，体会到替换前和替换后的不同

```shell
sed -n '4{p;s/ne/on/;p;=}' a.txt
```

输出结果

```
line4
lion4
4
```



大括号可以嵌套

```shell
sed -n '{4{p;=};8{=;p}}' a.txt
```

输出结果

```
line4
4
8
line8
```



如果嵌套时，地址不同，则会取交集

1. ```shell
   sed -n '3,7{4,5p}' a.txt
   ```

   输出结果

   ```
   line4
   line5
   ```

2. ```shell
   sed -n '4,7{3,6p}' a.txt
   ```

   输出结果

   ```
   line4
   line5
   line6
   ```



#### h,H,g,G,x —— 空间交换相关

首先需要弄懂两个概念：模式空间、保持空间。

**模式空间：**sed 命令默认是一行一行处理的，每读取到一行，会放到模式空间里，然后执行脚本来处理模式空间的内容，处理完后会清空模式空间，并且读取下一行继续处理，如此循环直到行尾。模式空间相当于专门用于 sed 处理的字符串缓冲区。

**保持空间：**默认的 sed 是不会操作保持空间的，保持空间是专门为用户提供的空间，这几个命令可以操作保持空间，来实现更复杂的功能。

> **h：**把模式空间内容覆盖到保持空间中
> **H：**把模式空间内容追加到保持空间中
> **g：**把保持空间内容覆盖到模式空间中
> **G：**把保持空间内容追加到模式空间中
> **x：**交换模式空间与保持空间的内容



将第 4 行和第 6 行复制到第 8 行后面。

```shell
sed '{4h;6H;8G}' a.txt

# 4h 表示将第 4 行放入保持空间
# 6H 表示将第 6 行加入保持空间
# 8G 表示将保持空间内容追加到模式空间
```

输出结果

```
line1
line2
line3
line4
line5
line6
line7
line8
line4
line6
line9
line10
```

#### n,N,P,D —— 模式空间操作

有了模式空间的概念后，可以重新理解一下前面 p、d、n 等命令的本质。

> **p：**打印当前模式空间所有内容，追加到默认输出之后。
>
> **P：**打印当前模式空间开端至\n的内容，并追加到默认输出之前。Sed并不对每行末尾\n进行处理，但是对N命令追加的行间\n进行处理，因为此时sed将两行看做一行。
>
> **n：**命令简单来说就是提前读取下一行，覆盖模型空间前一行，然后执行后续命令。然后再读取新行，对新读取的内容重头执行sed。
>
> **N：**命令简单来说就是追加下一行到模式空间，同时将两行看做一行，但是两行之间依然含有\n换行符，然后执行后续命令。然后再读取新行，对新读取的内容重头执行sed。此时，新读取的行会覆盖之前的行（之前的两行已经合并为一行）。
>
> **d：**命令是删除当前模式空间内容（不再传至标准输出）， 并放弃之后的命令，并对新读取的内容，重头执行sed。
>
> **D：**命令是删除当前模式空间开端至\n的内容（不在传至标准输出）， 放弃之后的命令，但是对剩余模式空间重新执行sed。



删除匹配 line4 行的下一行

```shell
sed '/line4/{n;d}' a.txt
# 操作到第 4 行时，提前读取第 5 行并删掉。
```

输出结果

```
line1
line2
line3
line4
line6
line7
line8
line9
line10
```





打印偶数行

```shell
sed -n '{n;p}' a.txt
# n 命令表示操作每一行时，读取下一行，覆盖掉已经读取的这一行
# p 表示打印出来
# 每次循环时，模式空间中，奇数行全部被偶数行覆盖

# 打印偶数行也可以用步进地址实现
sed -n '2~2p' a.txt
```

输出结果

```
line2
line4
line6
line8
line10
```



删除匹配到 line4 行和下一行

```shell
sed '/line4/{N;d}' a.txt
# 操作到第 4 行时，提前读取第 5 行追加到模式空间，然后两行一起删掉。
```

输出结果

```
line1
line2
line3
line6
line7
line8
line9
line10
```



打印奇数行

```shell
sed –n '$!N;P' a.txt
# 除了最后一行之外所有行，都提前读取下一行追加到模式空间，然后打印追加前的第一行，最后一行单独输出

# 打印奇数行也可以用步进地址实现
sed -n '1~2p' a.txt
```

输出结果

```
line1
line3
line5
line7
line9
```



每次提前读取三行，输出第一行

```shell
sed -n '{N;N;P}' a.txt
```

输出结果

```
line1
line4
line7
```



#### b —— 标签跳转

**b：**跳转到用冒号开头表示的标签

用标签跳转的方式实现：将除了匹配 line4 的行其它行的 ne 换成 on

```shell
sed '/line4/bend;s/ne/on/;:end' a.txt
# 匹配到 line4 时，会执行 b 命令，跳转到最后的 :end 标签，也就相当于跳过了替换命令
# 如果标签在前可能造成死循环，需要注意指定行

# 当标签在最后时，可以省略，而且当 b 命令指定的标签不存在时，会自动跳转到最后
# 所以可简写成
sed '/line4/b;s/ne/on/' a.txt

# 当然，可以用前面的地址范围取反来实现
sed '/line4/!s/ne/on/' a.txt
```

输出结果

```
lion1
lion2
lion3
line4
lion5
lion6
lion7
lion8
lion9
lion10
```



#### t —— 条件标签跳转

**t：**如果前一条命令执行成功，则跳转到用冒号开头表示的标签。

用条件标签跳转的方式实现：将除了匹配 line4 的行其它行的 ne 换成 on，4换成 F

```shell
sed 's/4/F/;tend;s/ne/on/;:end' a.txt

# 当标签在最后时，可以省略，而且当 b 命令指定的标签不存在时，会自动跳转到最后
# 所以可简写成
sed 's/4/F/;t;s/ne/on/' a.txt

# 当然，可以用前面的地址范围取反来实现
sed '/4/!s/ne/on/;/4/s/4/F/' a.txt
```

输出结果

```
lion1
lion2
lion3
lineF
lion5
lion6
lion7
lion8
lion9
lion10
```



## 速查

*以下内容使用谷歌翻译自 [sed(1) - Linux man page](https://linux.die.net/man/1/sed)* 以及 sed --help，可供参考。

### 名称

sed - 流编辑器，用于过滤和转换文本

### 语法

sed [可选参数]... {脚本} [输入文件]...

如果未提供-e，-expression，-f或--file选项，则将第一个非可选参数用作要解释的sed脚本。 其余所有参数均为输入文件的名称； 如果未指定输入文件，那么将读取标准输入。

### 描述

Sed是流编辑器。 流编辑器用于对输入流（文件或来自管道的输入）执行基本的文本转换。 尽管在某种程度上类似于允许脚本编辑（例如ed）的编辑器，但sed通过仅对输入进行一次传递来工作，因此效率更高。 但是sed能够过滤管道中的文本，这使其与其他类型的编辑器特别有区别。

#### -n, --quiet, --silent

禁止自动打印模式空间

#### -e 脚本, --expression=脚本

添加“脚本”到程序的运行列表

#### -f 脚本文件, --file=脚本文件

添加“脚本文件”到程序的运行列表

#### --follow-symlinks

直接修改文件时跟随符号链接； 硬链接仍然会断开。

#### -i[扩展名], --in-place[=扩展名]

直接修改文件（如果指定扩展名则备份文件）

#### -c, --copy

在 -i 模式下直接修改文件且备份时，使用复制操作而不是重命名。 尽管这样做可以避免断开链接（符号链接或硬链接），但最终的编辑操作不是原子的。 这很少是理想的模式。 --follow-symlinks通常就足够了，并且更快，更安全。

#### -l N, --line-length=N

指定“l”命令的换行期望长度

#### --posix

关闭所有 GNU 扩展

#### -E, -r, --regexp-extended

在脚本中使用扩展正则表达式（为保证可移植性使用 POSIX -E）。

#### -s, --separate

将输入文件视为各个独立的文件而不是单个长的连续输入流

#### -u, --unbuffered

从输入文件读取最少的数据，更频繁的刷新输出

#### --help

打印帮助并退出

#### --version

输出版本信息并退出

### 命令语法

这只是sed命令的简要提要，以提示那些已经知道sed的人。 必须参考其他文档（例如texinfo文档）以获取更完整的描述。

#### : 标签

**b** 和 **t** 命令的标签

#### # 注释

注释会一直延伸到下一个换行符（或**-e **脚本片段的末尾）。

#### }

关闭 {}块 的括号的右半括号

#### =

打印当前行号

#### a 文本

在模式空间后追加文本，该文本的每个嵌入换行符前都有一个反斜杠。

#### i 文本

在模式空间前插入文本，该文本的每个嵌入换行符前都有一个反斜杠。

#### q [退出代码]

立即退出 sed 脚本而不处理任何其他输入，除非如果未禁用自动打印，将打印当前图案空间。 退出代码参数是GNU扩展。

#### Q [退出代码]

立即退出 sed 脚本，而不处理更多输入。 这是一个GNU扩展。

#### r 文件名

从文件中读取的文本追加到模式空间中。

#### R 文件名

从文件中读取的文本追加到模式空间中。 每次调用该命令都会从文件中读取一行。这是一个GNU扩展。

#### {

启动 {}块 的括号的左半括号

#### b [标签]

跳转到标签； 如果省略了标签参数，则跳转到脚本结尾。

#### t [标签]

如果上一条读取的输入行被 s/// 命令替换成功，则跳转至标签 ; 如果省略标签，则跳转到脚本结尾。

#### T [标签]

如果上一条读取的输入行没有被 s/// 命令替换成功，则跳转至标签 ; 如果省略标签，则跳转到脚本结尾。这是一个GNU扩展。

#### c 文本

将所选行替换为文本，该文本的每个嵌入换行符前都有一个反斜杠。

#### d

删除模式空间。 开始下一个循环。

#### D

删除模式空间中的第一个嵌入的行。 从下一个循环开始，但是如果模式空间中仍有数据，则跳过从输入中读取的操作。

#### h H

复制/追加模式空间到保持空间。

#### g G

复制/追加保持空间到模式空间。

#### x

交换保持空间和模式空间的内容。

#### l

以“视觉清晰”的形式列出当前行。

#### l 宽度

以“视觉清晰”的形式列出当前行，并以根据指定宽度的字符将其断开。 这是一个GNU扩展。

#### n N

将输入的下一行读取/追加到模式空间。

#### p

打印当前模式空间

#### P

打印当前模式空间直到第一个嵌入式换行符。

#### s/正则表达式/替代串/

尝试将正则表达式与模式空间进行匹配。 如果成功，则用替代串/替换该部分。替代串可以包含特殊字符 “&” 来表示匹配的模式空间部分，而特殊转义 \1 到 \9 则表示正则表达式中的相应匹配子表达式。

#### w 文件名

将当前模式空间写入文件。

#### W 文件名

将当前模式空间的第一行写入文件。 这是一个GNU扩展。

#### y/源串/目标串/

将模式空间中出现在源串中的字符翻译成目标串中的相应字符。

### 地址

sed命令可以不带地址，在这种情况下，将对所有输入行执行该命令。 具有一个地址，在这种情况下，该命令仅对与该地址匹配的输入行执行； 或使用两个地址，在这种情况下，将对所有输入行执行命令，这些输入行与从第一个地址开始一直延伸到第二个地址的所有行包括在内。 有关地址范围的三点注意事项：语法为“地址1,地址2”（即地址用逗号分隔）； 即使地址2选择了较早的行，也将始终接受与地址匹配的行； 如果地址2是正则表达式，则不会针对地址1匹配的行进行测试。

在地址（或地址范围）之后，在命令之前，! 可以插入，它指定仅当地址（或地址范围）不匹配时才执行命令。

#### 数字

仅匹配指定的数字行。

#### 首行~步进

匹配从首行开始的每个相隔步进的行。 例如，“ sed -n 1~2p”将打印输入流中的所有奇数行，并且地址2~5将与第二行开始的每第五行匹配。首行可以为零； 在这种情况下，sed 的操作就好像等于步进。 （这是一个扩展。）

#### $

表示最后一行。

#### /正则表达式/

匹配正则表达式的行。

#### \c正则表达式c

匹配正则表达式的行。c 可以是任何字符。

### GNU sed 还支持一些特殊的2地址形式

#### 0,地址2

从“匹配的第一个地址”状态开始，直到找到地址2。 这类似于“1,地址2”，不同之处在于，如果地址2与输入的第一行匹配，则“0,地址2”形式将在其范围的末尾，而“1,地址2”形式仍为在其范围的开始。仅当地址2为正则表达式时有效。

#### 地址1,+N

将匹配地址1和地址1之后的 N 行。

#### 地址1,~N

将匹配地址1和地址1之后的行，直到输入行号是 N 的倍数的下一行。

### 正则表达式

本来应该支持POSIX.2 BRE，但是由于性能问题，它们并不完全支持。 正则表达式中的 \n 序列与换行符匹配，对于 \a，\t 和其他序列也是如此。

## 参考文献

1. [sed(1) - Linux man page](https://linux.die.net/man/1/sed)
2. [菜鸟教程 —— Linux sed 命令](https://www.runoob.com/linux/linux-comm-sed.html)
3. [肖邦linux —— sed入门详解教程](https://www.cnblogs.com/liwei0526vip/p/5644163.html)
4. [DataCareer —— Sed命令n，N，d，D，p，P，h，H，g，G，x解析](https://www.cnblogs.com/irockcode/p/8018575.html)