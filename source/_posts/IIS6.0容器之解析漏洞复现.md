---
title: IIS6.0容器之解析漏洞复现
date: 2018-10-30 00:15:28
categories: WEB安全
tags: 手工挖掘漏洞

---
<blockquote class="blockquote-center">态度决定高度!</blockquote>
<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=5181411&auto=0&height=66"></iframe></div>


# **漏洞简介**
解析漏洞是指web服务器因对HTTP请求处理不当导致将非可执行的脚本,文件等当作可执行的脚本去执行.该漏洞一般配合web容器(iis,nginx,apache,tomcat等)的文件上传功能去使用,以获取服务器的权限.

**IIS5.X/6.X解析漏洞**
对于IIS服务器5版本和6版本存在两个解析漏洞,分别为目录解析和文件解析

# **目录解析**
## 简介:
在网站下建立文件夹的名称中以.asp或.asa等作为后缀的文件夹,其目录内任何扩展名的文件都被IIS当作asp可执行文件去解析并执行.
举个栗子:/xx.asp/xx.jpg为xx.asp目录下存在xx.jpg文件,但将会被IIS解析成asp文件去执行,与原文件的后缀无关.

## 实验:
我们这里使用墨者学院提供的实验环境去复现该漏洞执行过程.([墨者学院解析漏洞链接](https://www.mozhe.cn/bug/detail/Umc0Sm5NMnkzbHM0cFl2UlVRenA1UT09bW96aGUmozhe))

我们在界面先上传一个普通文件,通过F12控制台查看消息头,得知目标服务器为Microsoft-IIS/6.0,也有需要上传的地方,我们可以试试目录解析漏洞.
![mark](http://phem9sn6g.bkt.clouddn.com/blog/181030/B4d608dKBf.png)
我们先随意上传一个文件,观其url发现是asp脚本构造的页面,然后在本地制作一个asp的一句话木马保存到一个文件中,然后打开burpsuite的代理功能去进行抓包修改
我们在发送的POST请求中发现刚刚发送的asp.txt被保存的第二个upload文件下,为了让其执行,所以我们在第二个upload后面加入/webshell.asp文件,这样就能将asp.txt这个一句话木马放入webshell.asp中,便可以利用解析漏洞直接将asp.txt当作asp脚本去执行
![mark](http://phem9sn6g.bkt.clouddn.com/blog/181030/GlEfEIEbb0.png)
在burp中转发浏览器显示成功上传,并列出上传的地址
![mark](http://phem9sn6g.bkt.clouddn.com/blog/181030/1EkHk9cJ5E.png)
成功的将asp一句话木马上传到目标服务器中,这样我们可以使用中国菜刀去远程连接
![mark](http://phem9sn6g.bkt.clouddn.com/blog/181030/IClbb69ACC.png)
![mark](http://phem9sn6g.bkt.clouddn.com/blog/181030/5690409fdK.png)
# **文件解析/后缀解析**
## 简介:
在IIS6.0下,分号后面的内容不被解析,举个栗子,xx.asp;.jpg将会当作xx.asp去解析执行.
IIS6.0 默认的可执行文件除了.asp，还包含这三种：.asa .cdx .cer.  例如：test.asa 、 test.cdx 、 test.cer

## 实验:
继续使用刚刚的环境,我们将刚刚的asp木马文件名修改为webshell.asp;.txt,因为该网站不允许上传以asp作为后缀的文件名,所以我们使用.txt后缀,但分号后面的内容将会被IIS过滤不去解析,所以这就是个asp脚本.
![mark](http://phem9sn6g.bkt.clouddn.com/blog/181030/CHDgG0BA30.png)
我们将文件直接上传到upload目录下,然后使用菜刀连接,也是可以连接上的.
https://i.loli.net/2018/10/30/5bd7346bd84d0.png
当然顺便也能找到所需要的key值
![mark](http://phem9sn6g.bkt.clouddn.com/blog/181030/b9K2md7c7K.png)
