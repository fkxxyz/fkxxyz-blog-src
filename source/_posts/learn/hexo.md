---
title: 探究在 archlinux 上用 hexo 建立个人的博客
date: 2019-11-22 17:49:44
categories:
- 学习与探究
tags:
- archlinux
- hexo
---

## 探究 hexo 如何在 archlinux 上运行

archlinux 的官方仓库里面没有 hexo 这个包，而 aur 里有个，但是装了之后发现一些问题导致一头雾水，目录也很乱，不得不自己想办法探究探究原理，要明白彻底一点问题才能解决。

<!--more-->

首先看看[官方文档](https://hexo.io/zh-cn/docs/)得知 *Hexo*是一个用 node.js 实现的博客框架，然后看了 [node.js的archwiki](https://wiki.archlinux.org/index.php/Node.js_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 得知，npm 是 node.js 的包管理器 。

然后根据 wiki 上描述装上 nodejs 和 npm

```shell
pacman -S nodejs npm
```

然后习惯性查看他们的信息和目录结构

```shell
pacman -Qi nodejs
pacman -Qi npm
pacman -Ql nodejs
pacman -Ql npm
```

根据输出结果，发现一个庞大的目录 /usr/lib/node_modules，进去瞧瞧

```shell
cd /usr/lib/node_modules
ls
find
```

显而易见的是，安装的 node.js 模块都被放在 /usr/lib/node_modules 里，里面每个子目录对应一个 node.js 模块，一开始有三个模块 node-gyp、npm、semver。

再看看这个 /usr/lib/node_modules 目录都有那些 archlinux 的软件包包含

```shell
pacman -Qo .
```

更加确定了模块都是装在这个目录里，而且一个 archlinux 的软件包对应一个 node.js 模块，正如 python3.8 的模块都是装在 /usr/lib/python3.8/site-packages 里一样。

那么 npm 和 node.js 的关系，正如 pip 和 python 的关系一样。

类比一下可知，要装 hexo ，就得把 hexo 也打成 archlinux 的包，hexo 模块应该被放在 /usr/lib/node_modules/hexo 里。

首先看看 aur 里面有没有这样一个包

```shell
yay hexo
```

发现已经有了，包名是 nodejs-hexo，直接安装。装完后看看结果

```shell
pacman -Ql hexo
ls -l /usr/lib/node_modules
hexo
```

虽然能用，但是发现问题很大，/usr/lib/node_modules/hexo 的所有者是当前用户，而不是 root，可能打包的人的失误，也可能是使用的过程中需要修改这个目录？我觉得打包的人的失误可能性要大一些，毕竟这是模块目录怎么会被用户随便改。于是打算以后再解决这个问题，自己写 PKGBUILD，先能用了，建成了博客再说。

## 安装 hexo

暂时这么安装

```shell
yay -S nodejs-hexo
```

## 建站

根据 [hexo的官方建站文档](https://hexo.io/zh-cn/docs/setup) ，由于第一次建，没经验，先测试，建立到 /tmp 目录下

```shell
mkdir /tmp/a
cd /tmp/a
hexo init myfolder
```

等了很久之后，才结束，根据提示信息得知，hexo init 命令干了两件事：

1. 将 https://github.com/hexojs/hexo-starter.git 克隆到 /tmp/a/myfolder
2. 将 https://github.com/hexojs/hexo-theme-landscape.git 克隆到 /tmp/a/myfolder/themes/landscape

```shell
cd myfolder
ls
find
npm install
find
```

前后对比发现，npm install 这条命令可能是补充了所有 node.js 模块到 /tmp/a/myfolder/node_modules 里。

然后通过官网介绍，加上自己百度，大概得知了这里面大概各个目录的作用：

1. config.yml 网站相关的配置文件
2. package.json 一开始不知道干嘛的，不过看内容可以猜测出，这整个目录是个应用程序，需要依赖很多模块，而这些所要依赖的模块放在 node_modules 里。
3. node_modules 整个应用程序所有依赖的模块，由 npm 管理。
4. scaffold 模板，大概是新建文章的时候要用。
5. source 应该是个人写的所有东西都在这里面，平时写东西都在这写 md 格式的文章，图片也往这放。
6. themes 顾名思义主题。
7. public 可能是将 source 的东西翻译建成网站之后的结果，一系列 html 文件，最终要发布的结果。

大概了解后进入下一章。

## 配置

根据 [官方文档-配置](https://hexo.io/zh-cn/docs/configuration) ，编辑 _config.yml，里面的数据随便填写。通过这个页面，我得知：

1. 所有自定义配置围绕着这个文件修改。
2. 可以配合域名，设置网站主页地址
3. 原来 source、public 的目录位置都可以改，非常灵活。
4. 连配置文件本身都能用参数额外指定。

## 尝试各种命令

根据 [官方文档-命令](https://hexo.io/zh-cn/docs/commands) ，开始一条一条尝试里面的命令。

### init

上面已经试过。

### new

新建一篇文章，文章名为“第一篇文章”

```shell
hexo new 第一篇文章
```

看看效果

```shell
find | grep 第一篇文章
```

果然，在 source 里面找到了 第一篇文章.md 这个文件。

接下来实验各种参数

```shell
hexo new 第二篇文章 -s wwwwwwwwwwwww
hexo new 33333 -p aaa/bbb
```

然后检查所有的变化

```shell
grep -rn wwwwwwwwwwwww
find | grep wwwwwwwwwwwww
grep -rn 33333
find | grep aaa/bbb
grep -rn aaa/bbb
```

发现所有改动全在 source 这个目录里。

那么得出结论，hexo new 这条命令本质是在 source/_posts 里面创建相应的 md 文件，我完全可以自己手动创建这些文件，和 hexo new 命令达到同样的效果。

查看这些文件内容

```shell
cat source/_posts/第一篇文章.md
```

发现是不是和前面模板文件内容类似呢

```shell
cat scaffolds/post.md
```

### generate

直接执行试试

```shell
hexo generate
```

生成静态文件，顾名思义是把 sources 里面所有的东西，处理成了 html 文件放在了 public 目录里，检验猜想

```shell
find public
```

恩？有 index.html ，好奇用浏览器打开试一下

```shell
chromium public/index.html
```

哈啊，看到一个简陋的架子，也许是没把主题加上。继续往后看吧。

### server

直接执行试试

```shell
hexo server
```

然后浏览器里面输入网址 http://localhost:4000，

这下看到主题了，也能看到自己刚刚创建的 第一篇文章、第二篇文章、33333，还有最初始的 Hello World

联想到 github 提供的仓库可以建成网站，那我现在是不是就可以把 public 这个目录上传上去了呢，但是静态网站会不会加上主题呢？说试试就试试：

打开 github 网站登录自己的帐号 fkxxyz，根据要求创建一个仓库名是 fkxxyz.github.io 的仓库，然后找个地方开始动手

```shell
mkdir /tmp/b
cd /tmp/b
git clone https://github.com/fkxxyz/fkxxyz.github.io.git
cp -r /tmp/a/myfolder/public/* .
git add -A
git commit -m update
git push
```

一顿操作之后，打开 https://fkxxyz.github.io/ 果然，出现了网站。

那么我把自己的域名 www.fkxxyz.com 解析到这个网址，博客等于已经建成了。

### deploy

直接执行

```shell
hexo deploy
```

好像没什么用，查资料据之后，发现这条命令是代替上述一顿操作，能自动把 public 上传到 github 中自己的仓库，何乐而不为？直接看相应的官方介绍 [github-pages](https://hexo.io/zh-cn/docs/github-pages) 和 [one-command-deployment](https://hexo.io/docs/one-command-deployment)

```shell
cd /tmp/a/myfolder

## 安装模块
npm install hexo-deployer-git --save
```

修改 _config.yml ，将最后改成

```yaml
deploy:
  type: git
  repository: https://github.com/fkxxyz/fkxxyz.github.io.git
  branch: master
```

然后再执行

```shell
hexo deploy
```

这下有反应了，大功告成。

下篇文章准备写个教程总结，整理一下，今天探索到的一切。

