---
title: 探索如何更方便的管理和部署 hexo 博客
p: learn-hexo
date: 2019-11-22 17:49:44
categories:
- 学习过程
tags:
- archlinux
- hexo
---

## 遇到的不便之处

在学会使用 hexo 的基本操作后，发现使用和部署的过程中，遇到几个问题，在此罗列出来逐一解决：

1. 由于使用很多主题的过程中，需要对 _config.yml、模块目录 node_modules 进行大量修改，当想要换主题时，得重新用新的 _config.yml ，如果模块不重装，则可能会有大量不需要的模块留下来。博客的源文件保存在 source 里面，重新 hexo init 初始化后，需要保留原来的 source。
2. 每次 hexo init 时会克隆 hexo-starter 和 hexo-theme-landscape 仓库，而 landscape 主题很大，国内克隆这个大仓库耗费时间，而这个仓库是默认主题，通常是不需要的。
3. 每次 hexo init 后都需要 npm install 来安装模块。
4. 在使用的过程中，如果 source 放到本机，要是本机故障或者突发情况导致 source 目录没了，那么损失会惨重。

## 解决思路

对于以上问题，想出一些一劳永逸的解决思路。

1. 对所有需要保存的重要的东西，托管到 github。
2. 提前保存好一份 hexo-starter 仓库到本地，并且 npm install 之后，将整个目录打包保存到本地，要更换主题时，直接解压使用，大幅度提升效率。
3. 本地存一份保存重要的东西的仓库，包括 source 目录，将 source 目录用符号链接的方式链接到博客的主目录。
4. 以上一切可以使用 shell 脚本来自动化，快速快速更换主题，同时 shell 脚本本身也可以算作重要的东西托管到 github。

## 解决方案

在 github 上建立一个仓库名为 fkxxyz-blog-src 的仓库，用来保存重要的信息，如配置好的初始 _config.yml 、source 目录、自动化脚本。

```shell
cd ~/github.com/fkxxyz
git clone https://github.com/fkxxyz/fkxxyz-blog-src.git
ls
cd fkxxyz-blog-src
cp -r ~/myblog/source .
cp ~/myblog/_config.yml .
touch README.md
git add -A
git commit -m 'first commit'
git push

```

