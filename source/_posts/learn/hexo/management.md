---
title: 探索如何更方便的管理和部署 hexo 博客
cover: false
date: 2019-11-23 01:09:15
updated: 2019-11-23 01:09:15
categories:
- 探究学习
tags:
- archlinux
- hexo
---

在学会使用 hexo 的基本操作后，发现使用和部署的过程中，遇到几个问题，在此罗列出来逐一解决。

<!--more-->

## 遇到的不便之处

1. 由于使用很多主题的过程中，需要对 _config.yml、模块目录 node_modules 进行大量修改，当想要换主题时，得重新用新的 _config.yml ，如果模块不重装，则可能会有大量不需要的模块留下来。因此，要更换主题，不得不重新初始化建立博客目录，但是博客的源文件保存在 source 里面，重新 hexo init 初始化后，需要又需要保留原来的 source，同样，hexo deploy 部署用的是 .deploy_git 目录，需要备份。
2. 每次 hexo init 时会克隆 hexo-starter 和 hexo-theme-landscape 仓库，而 landscape 主题很大，国内克隆这个大仓库耗费时间，而这个仓库是默认主题，通常是不需要的。
3. 每次 hexo init 后都需要 npm install 来安装模块。
4. 在使用的过程中，如果 source 放到本机，要是本机故障或者突发情况导致 source 目录没了，那么损失会惨重。

## 解决思路

对于以上问题，想出一些一劳永逸的解决思路。

1. 对所有需要保存的重要的东西，托管到 github。
2. 提前保存好一份 hexo-starter 仓库到本地，并且 npm install 之后，将整个目录压缩打包保存到本地，我将它成为博客目录模板，要更换主题时，直接解压使用，大幅度提升效率。
3. 本地存一份保存重要的东西的仓库，包括 source 目录，将 source 目录用符号链接的方式链接到博客的主目录。
4. 将 .deploy_git 放到自己的本地 github 仓库目录 ~/github.com/fkxxyz/fkxxyz.github.io，同样用符号链接的方式链接到博客的主目录。
5. 将主题目录的改动也记录到此仓库，用符号链接的方式对应过去，这里来可以妙用 cp 的 -s 参数。
6. 以上一切可以使用 shell 脚本来自动化，快速快速更换主题，同时 shell 脚本本身也可以算作重要的东西托管到 github。

## 解决方案

### 建立重要的仓库

在 github 上建立一个仓库名为 fkxxyz-blog-src 的仓库，用来保存重要的信息，如配置好的初始 _config.yml 、source 目录、自动化脚本，其中， _config.yml 保存两份，一份是由默认的 _config.yml 模板修改成自己的信息得到，取名为 _config-fkxxyz.yml，一份是由 _config-fkxxyz.yml 修改成当前使用的主题相关的配置。 

```shell
# 克隆刚建好的空仓库
cd ~/github.com/fkxxyz
git clone https://github.com/fkxxyz/fkxxyz-blog-src.git
ls

# 保存重要信息到这个仓库
cd fkxxyz-blog-src
cp -r ~/myblog/source .
cp ~/myblog/_config.yml _config-fkxxyz.yml
cp ~/myblog/_config.yml .
touch README.md

# 提交上传仓库
git add -A
git commit -m 'first commit'
git push
```

### 制作好博客目录模板压缩包

此步骤可以使得以后用解压的方式代替 hexo init + npm install 二连大幅度提升效率。

```shell
# 克隆 hexo-starter 仓库到临时目录
mkdir /tmp/a
cd /tmp/a
git clone https://github.com/hexojs/hexo-starter.git

# 精简 hexo-starter 目录
cd hexo-starter
ls -a
rm -rf .*
rm -r source
rm -r themes/landscape
find

# 安装必要的模块
npm install
npm install hexo-deployer-git --save

# 打包压缩保存起来
cd ..
mv hexo-starter hexo-templete
tar cJf hexo-templete.tar.xz hexo-templete
mv hexo-templete.tar.xz ~/Documents

# 后续清理
cd
rm -rf /tmp/a
```

至此，hexo-templete.tar.xz 已经保存到 ~/Document 里了，后续随时解压使用。

### 编写一键脚本

首先设计这个脚本，这个脚本放在 fkxxyz-blog-src 仓库目录中运行，功能是解压指定的博客目录模板压缩包到特定位置，脚本名为 setup。

```shell
cd ~/github.com/fkxxyz/fkxxyz-blog-src
touch setup
chmod +x setup
```

然后编写 setup，由于以后随时会更新完善，实时内容详见 https://github.com/fkxxyz/fkxxyz-blog-src/blob/master/setup

为了方便起见再编写两个脚本 gen 和 push， gen 用于复制修改过主题到目标目录，并且一键 hexo clean、hexo generate；push 用于一键 hexo deploy 。

最后，不忘把此仓库复制到服务器。

```shell
git add -A
git commit -m update
git push
```

## 测试解决效果

现在，所有的一切工作都可以在 ~/github.com/fkxxyz/fkxxyz-blog-src 目录里进行了，首先切换到此目录。

```shell
cd ~/github.com/fkxxyz/fkxxyz-blog-src
```

下面开始逐一测试效果。

### 更换主题

```shell
./setup

# 然后下载 jacman 主题，放到 ~/hexo/themes/jacman
```

### 生成网站

```shell
./gen

# 然后根据提示打开浏览器，进入地址 http://localhost:4000 进行测试。
```

### 上传改动

```shell
./push

# 然后打开浏览器，输入自己域名 www.fkxxyz.com 查看效果。
```

### 写博客

一切写博客的操作都在当前目录的 source 里，手动复制模板来完成，写完之后，可以 ./gen 生成然后测试，然后 ./push 上传。



## 尾声

这下可算是大功告成了，以后再也不怕换主题了，也不会怕丢失什么了，以后终于能够把一切精里放在写博客上了，达到了一劳永逸的效果。

接下来，我打算把我以前写的 md 文档，一个一个格式化，放到此博客上。
