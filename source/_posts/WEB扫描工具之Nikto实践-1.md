---
title: WEB扫描工具之Nikto实践
date: 2018-10-23 23:28:27
categories: kali
tags: tools

---
<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=5181411&auto=0&height=66"></iframe></div>

## 实验环境:
kali:192.168.136.128/24
				Metasploitable:192.168.136.129/24 
## **Nikto简介** 
Web扫描工具大部分都支持两种扫描模式:代理截断模式和主动扫描模式
Nikto:是一个Web服务器扫描程序，主要是去检查软件版本信息,搜索存在的安全隐患的文件,服务器配置漏洞,Web Application层面的安全隐患等,也能避免404误判（原因：很多服务器不遵循RFC标准，对于不存在的对象返回200响应码）；依据响应文件内容判断，不同扩展名的文件404响应内容不同；去时间信息后的内容取MD5值；不建议用-no404参数（-no404参数指去不校验404误判,它还可以捕获并打印收到的任何cookie.

Wiki百科对其功能的介绍

> Nikto is an Open Source (GPL) web server scanner which performs comprehensive tests against web servers for multiple items, including over 6700 potentially dangerous files/CGIs, checks for outdated versions of over 1250 servers, and version specific problems on over 270 servers. It also checks for server configuration items such as the presence of multiple index files, HTTP server options, and will attempt to identify installed web servers and software. Scan items and plugins are frequently updated and can be automatically updated.
> Nikto是一个开源（GPL）Web服务器扫描程序，可针对多个项目对Web服务器执行全面测试，包括超过6700个潜在危险文件/ CGI，检查超过1250台服务器的过期版本，以及超过270台服务器上的版本特定问题。它还会检查服务器配置项，例如是否存在多个索引文件，HTTP服务器选项，并将尝试识别已安装的Web服务器和软件。扫描项目和插件经常更新，可以自动更新。
 
## 开始操作
```
root@kali:~# nikto -update   #从CIRT.net网站更新nikto的数据库和插件

root@kali:~# nikto -list-plugins  #列出nikto内列出所有可用的插件

#扫描目标主机的Web层面的漏洞,格式为:nikto -host 目标服务器URL(可以是多个URL)
root@kali:~#  nikto -host http://192.168.136.129 

#也可以使用nikto -host 目标IP地址 -port 扫描端口,和上一条命令效果一致
root@kali:~# nikto -host 192.168.136.129 -port 80,443 

#使用ssl模式去扫描目标系统的信息
root@kali:~# nikto -host www.baidu.com -port 443 -ssl 

#扫描多个目标,将目标地址存放在某个文本文档中,
#目标地址格式为:http://主机名:端口或者IP地址:端口或者直接是IP地址
root@kali:~# nikto -host host.txt 

#使用nmap扫描目标网段的80端口,将开放80端口的主机IP筛选出后传送给nikto进行扫描web服务漏洞,
#参数-oG表示输出便于通过bash或者perl处理的格式,非xml
root@kali:~# nmap -p80 192.168.136.129/24 -oG - | nikto -host - 

#nikto支持代理功能
root@kali:~# nikto -host 192.168.1.1 -useproxy http://localhost:8087

Nikto互动功能:
Nikto包含几个可在活动扫描期间更改的选项，前提是它在提供POSIX支持的系统上运行，其中包括unix和其他一些操作系统。
在没有POSIX支持的系统上，将以静默方式禁用这些功能。

在主动扫描期间，按下面任何一个键将打开或关闭列出的功能或执行列出的操作。
请注意，这些区分大小写。

		SPACE - 报告当前扫描状态

		v - 打开/关闭详细模式

		d - 打开/关闭调试模式,极其详细信息

		e - 打开/关闭错误报告

		p - 打开/关闭进度报告

		r - 打开/关闭重定向显示

		c - 打开/关闭cookie显示

		o - 打开/关闭OK显示

		a - 打开/关闭验证显示

		q - 退出

		N - 下一个主持人

		P - 暂停,大写P
cookie简介[cookie wiki](https://zh.wikipedia.org/wiki/Cookie)
因为HTTP协议是无状态的，即服务器不知道用户上一次做了什么，
这严重阻碍了交互式Web应用程序的实现。在典型的网上购物场景中，用户浏览了几个页面，买了一盒饼干和两瓶饮料。
最后结帐时，由于HTTP的无状态性，不通过额外的手段，服务器并不知道用户到底买了什么，
所以Cookie就是用来绕开HTTP的无状态性的“额外手段”之一。
服务器可以设置或读取Cookies中包含信息，借此维护用户跟服务器会话中的状态。

在刚才的购物场景中，当用户选购了第一项商品，服务器在向用户发送网页的同时，还发送了一段Cookie，记录着那项商品的信息。
当用户访问另一个页面，浏览器会把Cookie发送给服务器，于是服务器知道他之前选购了什么。
用户继续选购饮料，服务器就在原来那段Cookie里追加新的商品信息。结帐时，服务器读取发送来的Cookie就行了。

Cookie另一个典型的应用是当登录一个网站时，网站往往会请求用户输入用户名和密码，并且用户可以勾选“下次自动登录”。
如果勾选了，那么下次访问同一网站时，用户会发现没输入用户名和密码就已经登录了。这正是因为前一次登录时，
服务器发送了包含登录凭据（用户名加密码的某种加密形式）的Cookie到用户的硬盘上。
第二次登录时，如果该Cookie尚未到期，浏览器会发送该Cookie，服务器验证凭据，于是不必输入用户名和密码就让用户登录了。

修改nikto的配置文件,写入cookie信息,即可扫描那些需要身份认证才可以访问的页面
root@kali:~# vim /etc/nikto.conf 	#编辑其配置文件
修改USERAGENT,防止扫描的时候被系统管理员发现(我目前设置为win10的浏览器)
USERAGENT=Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko (Evasions:@EVASIONS) (Test:@TESTID)
设置用户代理的方法:使用火狐浏览器登陆[User-Agent Switcher](https://addons.mozilla.org/zh-CN/firefox/addon/user-agent-switcher-revived/?src=search),
添加到Firefox,在右上角打开图标!
```

![](1.png)
![](2.png)

```
在STATIC-COOKIE这个命令下取消注释,
输入cookie信息,格式为"cookie name1"="value";"cookie name2"="value"
(可以设置多个cookie) 
STATIC-COOKIE="PHPSESSION"="9eb59920d99db2871254303ec47b3460";"security"="high"
(这是我的cookie,需要自行抓取cookie信息) 
然后保存退出,在终端开始用扫描(cookie扫描),将会获得更有效的扫描结果

# nikto加参数-evasion表示使用LibWhisker中对IDS的躲避技术,防止被发现,
root@kali:~# nikto -host http://192.168.136.129 
可使用以下几种类型: 
• 1 随机URL编码(非UTF-8方式) 
• 2 自选择路径(/./) 
• 3 过早结束的URL 
• 4 优先考虑长随机字符串
• 5 参数欺骗 
• 6 使用TAB作为命令的分隔符 
• 7 使用变化的URL 
• 8 使用Windows路径分隔符"\" 

#使用第一种,第六种,第七种方法,自行搭配
root@kali:~# nikto -host http://192.168.136.129 -evasion 167 
```
