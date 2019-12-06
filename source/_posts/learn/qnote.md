---
title: 关于手机和电脑之间文字快速互通
cover: false
date: 2019-12-06 21:53:30
categories:
- 探究学习
tags:
- 前端
- php
typora-root-url: ../..
---

有时候在手机上搜到什么好代码好网址，想立刻转到电脑上用，发现有时候不怎么方便。有这几个方案：

1. 用通信软件，手机电脑同时启动QQ或微信，发送到电脑上。
2. 用百度网盘或者github等服务互通。
3. 用局域网软件，如feem、ftp等等。
4. 用数据线，adb工具，访问手机储存。

发现都有一定的不方便之处，要启动这么庞大的QQ，要同一局域网，要数据线。是时候解决这个问题了。

<!--more-->

## 思路

由于我有个租的阿里云服务器，如果有这样一个轻量集软件，甚至不需要软件，只用浏览器，用一个网页就能实现电脑和手机互通，岂不是会非常迅速，对工作效率也会提升很多。

先找找有没有这方面的软件，闭源的商业软件要么有广告要么要登录很慢很复杂，不喜欢这样的。开源的软件也暂时找不到很方便的，希望有推荐。

应对自己这样的需求，我打算自己实现一个。

怎样的方案实现呢？想出几个备选方案：

1. 用 python 网络编程，tcp协议，服务端放服务器，客户端在任意终端。

   麻烦之处在于在tcp协议之上得自己设计自己的协议，而且手机用python并不方便，显然实现起来比较麻烦成本比较高，不做这种重复造轮子的事情。

2. 看来浏览器是个好东西，哪个终端设备都有浏览器，那么，用前端技术实现是不是最好的选择呢。

   看起来是，用php编个后端在服务器上运行，打开浏览器就能记住笔记，然后后端把笔记记录到服务器中的某个文件里，任何浏览器访问的时候，后端读取这个文件前端处理显示出来。

看来php这个方案是最方便的，就选它了。虽然前端开发经验比较少，仅限于知道http的get和post的请求，前端的html5的语法，css的基础知识，还有js基本怎么作用，后端大概怎么处理请求返回html页面的逻辑。

## 预备的php知识

由于没有php开发经验，只是知道php是个什么东西，只会php的hello world。没关系，先想想我需要什么，百度搜就是了。

1. 需要用php读写文件

   百度一搜，搜到了 file_get_contents、fopen、fwrite、fclose 几个函数的语法，这不和众多编程语言一模一样的语法和参数嘛，这问题解决了。

2. 变量的用法

   编程免不了使用变量，哪怕再简单的程序。查了下，发现 php 的数据类型有字符串、整数、浮点数、逻辑、数组、对象，还是挺少的。这里主要用字符串。语法也很简单，不需要声明变量，每个变量前面都要加 $ ，无论是初始化赋值还是使用。

3. 处理post请求

   这个是重点，思路是前端的文本框用post发送到后端，后端得能收到内容并且存到文件。查了下，发现这个特殊变量 $_POST ，有点类似于python的字典用法，只需要指定前端传过来的键值，那么我就用 $_POST["content"]。

4. 编码解码

   信息传递的过程中，由于我域名没有备案，用不了https，是http方式明文传输，中途被过滤关键字什么可能会降低传输速度，我想加密或者加一层编码，哪怕只用base64状况会好很多，那就选base64吧。

   查到了这些函数：

   1. base64_encode 和 base64_decode
   2. urlencode 和 urldecode
   3. rawurlencode 和 rawurldecode

5. 语法

   关于语法，在实验的过程中踩了很多坑，服务器总是返回 500 的错误码，检查发现都是漏分号。那么和 C++ 以及 java 的编程习惯一样，每个语句最后加分号。

差不多就需要这么点东西了，前端如何处理编码解码呢，html实现不了，恐怕需要js，那么还得搜集一点js的函数和知识。

## 预备的前端知识

在前端的js和html，有需要实现编码解码，需要解码之后改变文本框的内容，post之前也得编码。

1. 多行文本框，随着窗口变化改变尺寸

   用 textarea 标签，设置它及其 form 的样式 width

   ```css
   form {
       width:90%;
   }
   textarea {
       width:90%;
       overflow:auto;
       word-break:break-all;
   }
   ```

2. post 请求

   设置 form 的 method 属性为 post

3. post 之后页面自动跳转回去

   搜索查到这样的办法，在 head 标签里面加个 meta，表示一秒钟之后刷新到 /

   ```html
   <meta http-equiv="refresh" content="1;url=/" />
   ```

4. 编码解码

   查到了这些函数：
   1. encodeURI 和 decodeURI
   2. encodeURIComponent 和 decodeURIComponent
   3. window.btoa 和 window.atob

   在查询百度和反复用浏览器的控制台实验，终于弄清了他们的区别。

   1. 改变文本框内容

      1. 改变 textarea 的内容

         document.getElementById(textarea的ID).value = 内容

      2. 改变 html 结构的内容

         document.getElementById(标签的ID).innerText = 内容

5. 适应手机端

   搜索轻易查到在 head 标签里面加个 meta

   ```html
   <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=0.5, maximum-scale=2.0, user-scalable=yes" />
   ```

   

##　逻辑设计

### 前端

1. 在打开页面之后，立刻执行js代码，把后端传输过来的编码过的字符串解码，然后把html内容设置成解码后的内容
2. 在 post 请求之前，利用 input 标签的 onclick 属性提前执行一段js代码，把内容编码然后再 post

### 后端

需要两个页面，一个是读取和编辑页面的 php，一个是处理 post 请求的页面 php，分别取名为 index.php 和 submit.php

1. **index.php**

   先读取文件，再编码文件，再显示到 testarea 的值里。

2. **submit.php**

   先解码得到的内容，再把内容写入文件。

## 代码实现

在经过一段时间的实验和调试后，index.php 和 submit.php 的代码基本完工，以下是简易的初始版本，后续可以随时改进。项目地址在 https://github.com/fkxxyz/qnote

### index.php

```html
<!DOCTYPE HTML>
<html>
<head>
	<title>fkxxyz</title>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=0.5, maximum-scale=2.0, user-scalable=yes" />
	<style type="text/css">
		form {
			width:90%;
		}
		textarea {
			width:90%;
			overflow:auto;
			word-break:break-all;
		}
		input {
			width:120px;
			height:60px;
			font-size:20px;
		}
	</style>
</head>
<body>
	<form class="form" name="editform" method="post" action="submit.php">
	<input type="submit" onclick="encode_texta()" value="提交" />
	<input type="button" onclick="clear_texta()" value="清空">
	<p>
		<textarea id="texta" rows="20" name="content"><?php
			$file="/srv/ftp/note.txt";
			echo base64_encode(rawurlencode(base64_decode(file_get_contents($file))));
		?></textarea>
	</p>
	</form>
	<script>
		function t_encode(s){
			return window.btoa(encodeURIComponent(s));
		}
		function t_decode(s){
			return decodeURIComponent(window.atob(s));
		}
		function clear_texta(){
			document.getElementById("texta").value = "";
		}
		function encode_texta(){
			texta = document.getElementById("texta");
			texta.value = t_encode(texta.value);
		}
		function decode_texta(){
			texta = document.getElementById("texta");
			texta.value = t_decode(texta.value);
		}
		decode_texta();
	</script>
</body>
</html>
```

### submit.php

```html
<!DOCTYPE HTML>
<html>
<head>
	<title>提交</title>
	<meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=0.5, maximum-scale=2.0, user-scalable=yes" />
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<meta http-equiv="refresh" content="1;url=/" />
</head>

</head>
<body>
	<?php
		$file="/srv/ftp/note.txt";
		$f = fopen($file, "w") or die("服务器出错！无法打开文件。");
		fwrite($f, base64_encode(rawurldecode(base64_decode($_POST["content"]))));
		fclose($f);
		echo "<h1>提交成功</h1>";
		echo "<div id=\"textd\">";
		echo $_POST["content"];
		echo "</div>"
		?>
	</body>
	<script>
		function t_decode(s){
			return decodeURIComponent(window.atob(s));
		}
		function decode_textd(){
			textd = document.getElementById("textd");
			textd.innerText = t_decode(textd.innerText);
		}
		decode_textd();
	</script>
</html>
```

## 电脑端更快速的处理

在服务器端，我没有直接把内容直接放到文件里，而是加了层 base64 编码放进去。

因为考虑到，在 linux 端，我服务器开了 ftp，我可以不用打开浏览器也能快速连接到服务器获取内容，由于传输也没有加密，为了不以明文传输所以也 base64 一下。

以下命令快速获取服务器的笔记内容

```shell
curl -s ftp://地址/note.txt | base64 -d
```

把它保存为脚本，一执行，就能在控制台里面看到内容，美滋滋。



同样的思路，能不能不打开浏览器也能 post 呢，一查果真容易实现，curl 就能发送 post 请求，连 pyhon 都免了！

以下命令快速记录笔记上传到服务器，返回错误代码

```shell
curl -d "content=$(cat | base64)" http://地址/submit.php
```

写成脚本吧，获取状态码，返回是否成功

```shell
#!/bin/bash

code="$(curl -d "content=$(cat | base64)" -o /dev/null -w %{http_code} -s http://地址/submit.php)"
echo
if [ "$code" == "200" ]; then
    echo "提交成功"
else
    echo "出错，http 状态码： $code"
fi
```

至此，大功告成。

再也不用启动 QQ微信什么的了，也不需要局域网软件了。只要能联网，任意地方都能快速笔记互通了。

## 手机端

手机端就更简单了，用的按卓手机，加上 chromium 浏览器，把我的服务器地址添加到桌面快捷方式，桌面上随时点开粘贴，然后提交。

## 后续可能的改进

1. 由于传输过程只是 base64 ，未加密，中途被拦截篡改或者泄露。

   这个问题目前不需要解决，因为我个人不是什么重要机构信息没那么大的价值，也没有人恶意盯着陷害我，等以后有了安全需求再说。

2. 谁都可以打开浏览器通过 http 协议访问我的服务器，只要知道我的服务器地址。

   这个问题目前也不需要解决，因为我服务器私人用，也不会有谁故意捣乱。等到有这个需求再说，可以加个验证什么的。
   
3. 有时候会有图片传输的需求，甚至文件传输的需求，一般都 ftp 或者在线私人网盘了，暂时也不需要。

4. 美观。

