---
title: 	JumpServer运维堡垒机安装及实战
date: 2019-07-02 01:45:28
categories: 运维技术
tags: 堡垒机

---
<blockquote class="blockquote-center">海纳百川，有容乃大；壁立千仞，无欲则刚!</blockquote>
<div class="aplayer" data-id="254265" data-server="netease" data-type="song" data-mode="single"></div>
## 需求分析

随着企业信息化进程不断深入，企业的IT系统变得日益复杂，不同背景的运维人员违规操作导致的安全问题变得日益突出起来，主要表现在：内部人员操作的安全隐患、第三方维护人员安全隐患、高权限账号滥用风险、系统共享账号安全隐患、违规行为无法控制的风险。

运维操作过程是导致安全事件频发的主要环节，所以对运维操作过程的安全管控就显得极为重要。而防火墙、防病毒、入侵检测系统等常规的安全产品可以解决一部分安全问题，但对于运维人员的违规操作却无能为力。如何转换运维安全管控模式，降低人为安全风险，满足企业要求，是当下所面临的迫切需求。

## 审计管理

审计管理其实很简单，就是把用户的所有操作都纪录下来，以备日后的审计或者事故后的追责。在纪录用户操作的过程中有一个问题要注意，就是这个纪录对于操作用户来讲是不可见的，什么意思？就是指，无论用户愿不愿意，他的操作都会被纪录下来，并且，他自己如果不想操作被纪录下来，或想删除已纪录的内容，这些都是他做不到的，这就要求操作日志对用户来讲是不可见和不可访问的，那么我们就可以通过堡垒机就可以很好的实现。

> 补充，跳板机和堡垒机得区别：
>
> 　　跳板机，只有跳转登录得功能。
>
> 如果跳板机提供了以下两条，叫做审计系统或堡垒机
>
> 1. 　　记录用户操作
> 2. 　　实现了权限管理

堡垒要想成功完全记到他的作用，只靠堡垒机本身是不够的， 还需要一系列安全上对用户进行限制的配合，堡垒机部署上后，同时要确保你的网络达到以下条件：

- 所有人包括运维、开发等任何需要访问业务系统的人员，只能通过堡垒机访问业务系统
  - 回收所有对业务系统的访问权限，做到除了堡垒机管理人员，没有人知道业务系统任何机器的登录密码
  - 网络上限制所有人员只能通过堡垒机的跳转才能访问业务系统 
- 确保除了堡垒机管理员之外，所有其它人对堡垒机本身无任何操作权限，只有一个登录跳转功能
- 确保用户的操作纪录不能被用户自己以任何方式获取到并篡改

## 堡垒机功能实现需求

**业务需求:**

1. 兼顾业务安全目标与用户体验，堡垒机部署后，不应使用户访问业务系统的访问变的复杂，否则工作将很难推进，因为没人喜欢改变现状，尤其是改变后生活变得更艰难
2. 保证堡垒机稳定安全运行， 没有100%的把握，不要上线任何新系统，即使有100%把握，也要做好最坏的打算，想好故障预案

**功能需求：**

1. 所有的用户操作日志要保留在数据库中

2. 每个用户登录堡垒机后，只需要选择具体要访问的设置，就连接上了，不需要再输入目标机器的访问密码

3. 允许用户对不同的目标设备有不同的访问权限，例:

   ​    对10.0.2.34 有mysql 用户的权限

   ​    对192.168.3.22 有root用户的权限

   ​    对172.33.24.55 没任何权限

5. 分组管理，即可以对设置进行分组，允许用户访问某组机器，但对组里的不同机器依然有不同的访问权限　　

## Jumpserver堡垒机

堡垒机的主要作用权限控制和用户行为审计，堡垒机就像一个城堡的大门，城堡里的所有建筑就是你不同的业务系统 ， 每个想进入城堡的人都必须经过城堡大门并经过大门守卫的授权，每个进入城堡的人必须且只能严格按守卫的分配进入指定的建筑，且每个建筑物还有自己的权限访问控制，不同级别的人可以到建筑物里不同楼层的访问级别也是不一样的。还有就是，每个进入城堡的人的所有行为和足迹都会被严格的监控和纪录下来，一旦发生犯罪事件，城堡管理人员就可以通过这些监控纪录来追踪责任人。 目前比较优秀的开源软件是jumpserver，认证、授权、审计、自动化、资产管理，适合中小型公司或服务器不多的情况。商业的堡垒机Citrix XenApp、齐治包括一些云机构提供的堡垒机这里不做记录。

### Jumpserver简介

官网地址:<http://www.jumpserver.org/>

> Jumpserver 是全球首款完全开源的堡垒机, 使用 GNU GPL v2.0 开源协议, 是符合 4A 的专业运维审计系统。
>
> Jumpserver 使用 Python / Django 进行开发, 遵循 Web 2.0 规范, 配备了业界领先的 Web Terminal 解决方案, 交互界面美观、用户体验好。
>
> Jumpserver 采纳分布式架构, 支持多机房跨区域部署, 中心节点提供 API, 各机房部署登录节点, 可横向扩展、无并发访问限制。
>
> Jumpserver 现已支持管理 SSH、 Telnet、 RDP、 VNC 协议资产。
>
> 改变世界, 从一点点开始。

### jumpserver堡垒机组件说明：

**1、Jumpserver：**

**现指** **Jumpserver 管理后台，是核心组件（Core）, 使用 Django Class Based View 风格开发，支持 Restful API。**

**2、Coco：**

**实现了** **SSH Server 和 Web Terminal Server 的组件，提供 SSH 和 WebSocket 接口, 使用 Paramiko 和 Flask 开发。**

**3、Luna：**

**现在是** **Web Terminal 前端，计划前端页面都由该项目提供，Jumpserver 只提供 API，不再负责后台渲染html等。**

## jumpserver必备功能

| Jumpserver提供的堡垒机必备功能 |                                |                    |
| ------------------------------ | ------------------------------ | ------------------ |
| 身份验证 Authentication        | 登录认证                       | 资源统一登录和认证 |
| LDAP认证                       |                                |                    |
| 支持OpenID，实现单点登录       |                                |                    |
| 多因子认证                     | MFA（Google Authenticator）    |                    |
| 账号管理 Account               | 集中账号管理                   | 管理用户管理       |
| 系统用户管理                   |                                |                    |
| 统一密码管理                   | 资产密码托管                   |                    |
| 自动生成密码                   |                                |                    |
| 密码自动推送                   |                                |                    |
| 密码过期设置                   |                                |                    |
| 批量密码变更(X-PACK)           | 定期批量修改密码               |                    |
| 生成随机密码                   |                                |                    |
| 多云环境的资产纳管(X-PACK)     | 对私有云、公有云资产统一纳管   |                    |
| 授权控制 Authorization         | 资产授权管理                   | 资产树             |
| 资产或资产组灵活授权           |                                |                    |
| 节点内资产自动继承授权         |                                |                    |
| RemoteApp(X-PACK)              | 实现更细粒度的应用级授权       |                    |
| 组织管理(X-PACK)               | 实现多租户管理，权限隔离       |                    |
| 多维度授权                     | 可对用户、用户组或系统角色授权 |                    |
| 指令限制                       | 限制特权指令使用，支持黑白名单 |                    |
| 统一文件传输                   | SFTP 文件上传/下载             |                    |
| 文件管理                       | Web SFTP 文件管理              |                    |
| 安全审计 Audit                 | 会话管理                       | 在线会话管理       |
| 历史会话管理                   |                                |                    |
| 录像管理                       | Linux 录像支持                 |                    |
| Windows 录像支持               |                                |                    |
| 指令审计                       | 指令记录                       |                    |
| 文件传输审计                   | 上传/下载记录审计              |                    |

## 开始安装:

### 安装实验环境:

jumpserver服务端:192.168.48.133,redhat7.4系统

上传安装包到服务端的/opt目录

```
链接：https://pan.baidu.com/s/1Ag4Uz7-SaQHddhAiKSiddA 
提取码：rrg6 
复制这段内容后打开百度网盘手机App，操作更方便哦
```

```
[root@localhost opt]# ll
总用量 24220
drwxr-xr-x.  5 root root      194 6月   8 2018 coco
drwxr-xr-x. 11 root root      253 6月   8 2018 jumpserver
-rw-r--r--.  1 root root  7910019 4月  10 2018 luna.tar.gz
-rw-r--r--.  1 root root 16872064 4月  10 2018 Python-3.6.1.tar.xz
drwxr-xr-x.  2 root root     8192 6月   8 2018 python-package 
```

关闭系统防火墙和selinux

```
[root@localhost ~]# systemctl stop firewalld 
[root@localhost ~]# setenforce 0 
```

查看当前系统语言环境:

```
[root@localhost ~]# cat /etc/locale.conf  
LANG="zh_CN.UTF-8" 
```

如果不是utf-8格式的话,那么需要去修改环境变量

```
[root@localhost ~]# localedef -c -f UTF-8 -i zh_CN zh_CN.UTF-8 

[root@localhost ~]# export LC_ALL=zh_CN.UTF-8 

[root@localhost ~]# echo 'LANG=zh_CN.UTF-8' > /etc/locale.conf 

[root@localhost ~]# exit 
```

再重新连接， 这样语言环境就改变了。



### 安装依赖包

注:在安装之前,可以开启yum缓存功能,把软件包下载下来,方便后期使用

```
[root@localhost ~]# vim /etc/yum.conf 

改：keepcache=0 
为：keepcache=1 
```

安装所需要的软件包

```
[root@localhost ~]# yum -y install wget sqlite-devel xz gcc automake zlib-devel openssl-devel epel-release git 
```

编译安装python3.6.1

```
[root@localhost ~]# cd /opt
[root@localhost ~]# tar xvf Python-3.6.1.tar.xz  && cd Python-3.6.1
[root@localhost ~]# ./configure  &&  make  -j 4 && make install
```

这里必须执行编译安装，否则在安装 Python 库依赖时会有麻烦...

然后我们创建个python3的虚拟环境

因为 CentOS 6/7 自带的是 Python2，而 Yum 等工具依赖原来的 Python，为了不扰乱原来的环境我们来使用 Python 虚拟环境

```
[root@localhost ~]# cd /opt
[root@localhost ~]# python3 -m venv py3  
[root@localhost ~]# source /opt/py3/bin/activate
(py3) [root@localhost ~]#        #切换成功的，前面有一个py3 标识
(py3) [root@localhost opt]# python -V
Python 3.6.1
```

因为jumpserver是基于python3的环境,所以就需要安装python3



### 开始安装

```
(py3) [root@localhost opt]# cd jumpserver/ (py3) 

[root@localhost jumpserver]# ls (py3) 

[root@localhost jumpserver]# cd requirements/ 
```

安装jumpserver所需要的数据包

```
(py3) [root@localhost requirements]# yum install -y `cat rpm_requirements.txt` 

或者 
(py3) [root@localhost requirements]# yum install -y  $(cat rpm_requirements.txt) 
```

如果有些软件安装不上的话,使用以下源

```
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/epel/7/$basearch
#mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch
failovermethod=priority
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
baseurl=https://mirrors.tuna.tsinghua.edu.cn/epel/7/$basearch/debug
#mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-debug-7&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1

[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
baseurl=https://mirrors.tuna.tsinghua.edu.cn/epel/7/SRPMS
#mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-source-7&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1
```

安装python依赖库,因为要从requirements.txt文件读取python依赖包,然后下载,但由于软件包太多,所以使用pip本地安装

```
(py3) [root@localhost python-package]# 

(py3) [root@localhost python-package]# pip install ./* 
```

安装redis,因为jumpserver中调用redis做cache和celery broke

```
(py3) [root@localhost python-package]# yum install redis -y 
```

启动redis

```
(py3) [root@localhost python-package]# systemctl enable redis;systemctl start redis 
```

安装数据库进行缓存数据

```
(py3) [root@localhost ~]# yum  install mariadb mariadb-devel mariadb-server   -y 
```

开启数据库

```
(py3) [root@localhost ~]# systemctl enable mariadb  ;  systemctl start mariadb 
```

创建数据库jumpserver并授权

```
MariaDB [(none)]> create database jumpserver default charset 'utf8';
Query OK, 1 row affected (0.00 sec)

#设置用户jumpserver@127.0.0.1对jumpserver数据库所有表都有权限,并设置密码为123456

MariaDB [(none)]> grant all on jumpserver.* to 'jumpserver'@'127.0.0.1' identified by '123456';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye 
```

修改配置文件

```
(py3) [root@localhost opt]# cd jumpserver/
(py3) [root@localhost jumpserver]# cp config_example.py config.py 
(py3) [root@localhost jumpserver]# vim config.py 
#python是以空格作为缩进的,所以需要注意格式
# SQLite setting:
    DB_ENGINE = 'sqlite3'
    DB_NAME = os.path.join(BASE_DIR, 'data', 'db.sqlite3')

    #MySQL or postgres setting like:
    DB_ENGINE = 'mysql'
    DB_HOST = '127.0.0.1'
    DB_PORT = 3306
    DB_USER = 'jumpserver'
    DB_PASSWORD = '123456'
    DB_NAME = 'jumpserver'
```

生成数据库表结构和初始化数据

```
(py3) [root@localhost jumpserver]# cd /opt/jumpserver/utils/
(py3) [root@localhost utils]# bash make_migrations.sh
```

启动服务

-d参数表示在后台启动

```
(py3) [root@localhost jumpserver]# ./jms start all -d 
```

访问主机的8080端口,默认用户名密码为admin/admin

![](https://pic2.superbed.cn/item/5d1a32da451253d178b903c4.png)



![](https://pic1.superbed.cn/item/5d1a3b56451253d178b9398e.png)

### 安装coco

安装ssh server和websocket server:coco

当点击web终端的时候会出现以下错误,因为我们没有部署luna和coco,所以无法使用web终端

![](https://pic.superbed.cn/item/5d1a3b68451253d178b93a1e.png)

![](https://pic.superbed.cn/item/5d1a3b79451253d178b93aa3.png)

安装coco的依赖包,为rpm和python数据包

```
(py3) [root@localhost jumpserver]# cd /opt/coco/requirements/ 

(py3) [root@localhost requirements]# yum -y  install $(cat rpm_requirements.txt) 

(py3) [root@localhost requirements]# pip install -r requirements.txt 
```

注:使用pip download -r requirements.txt可以直接把python包下到本地



修改配置文件

```
(py3) [root@localhost requirements]# cd /opt/coco/ 

(py3) [root@localhost coco]# cp conf_example.py conf.py 

(py3) [root@localhost coco]# chmod +x cocod 
```

运行服务

```
(py3) [root@localhost coco]# ./cocod start -d 
```



### 安装web terminal前端luna

Luna概述:Luna现在是web terminal前端,计划前端页面都由该项目提供,jumpserver只提供API,不再负责后台渲染HTML等

```
[root@localhost ~]# cd /opt/ 

[root@localhost opt]# tar zxvf luna.tar.gz 
```

也可以直接去在线下载

```
wget https://github.com/jumpserver/luna/releases/download/v1.0.0/luna.tar.gz 
```

配置nginx,整合各个组件

安装nginx

```
[root@localhost luna]# yum install nginx -y 
```

修改配置文件

```
[root@localhost ~]# vim /etc/nginx/conf.d/

server {
    listen 80;

    client_max_body_size 100m;  # 录像及文件上传大小限制

    location /luna/ {
        try_files $uri / /index.html;
        alias /opt/luna/;  # luna 路径, 如果修改安装目录, 此处需要修改
    }

    location /media/ {
        add_header Content-Encoding gzip;
        root /opt/jumpserver/data/;  # 录像位置, 如果修改安装目录, 此处需要修改
    }

    location /static/ {
        root /opt/jumpserver/data/;  # 静态资源, 如果修改安装目录, 此处需要修改
    }

    location /socket.io/ {
        proxy_pass       http://localhost:5000/socket.io/;
        proxy_buffering off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        access_log off;
    }

    location /coco/ {
        proxy_pass       http://localhost:5000/coco/;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        access_log off;
    }

    location /guacamole/ {
        proxy_pass       http://localhost:8081/;
        proxy_buffering off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $http_connection;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        access_log off;
    }

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

运行 Nginx

```
$ nginx -t   # 确保配置没有问题, 有问题请先解决
$ systemctl restart nginx
```

访问虚拟机地址,默认账号:admin,密码:admin

![](https://pic.superbed.cn/item/5d1a3b94451253d178b93b69.png)



![](https://pic.superbed.cn/item/5d1a3ba1451253d178b93bdc.png)

确定已安装成功之后到会话管理--终端管理,接受coco的注册,点接受

![](https://pic2.superbed.cn/item/5d1a3bb2451253d178b93c65.png)



![](https://pic3.superbed.cn/item/5d1a3bc9451253d178b93d0e.png)



测试连接:

```
py3) [root@localhost coco]# netstat -antup | grep 2222 tcp        0      0 0.0.0.0:2222            0.0.0.0:*               LISTEN      3343/python3  
```

本地使用ssh进行连接,账号admin,密码admin,端口2222

```
(py3) [root@localhost coco]# ssh -p 2222 admin@192.168.48.137

    Administrator, 欢迎使用Jumpserver开源跳板机系统  

    1) 输入 ID 直接登录 或 输入部分 IP,主机名,备注 进行搜索登录(如果唯一).
    2) 输入 / + IP, 主机名 or 备注 搜索. 如: /ip
    3) 输入 P/p 显示您有权限的主机.
    4) 输入 G/g 显示您有权限的主机组.
    5) 输入 G/g + 组ID 显示该组下主机. 如: g1
    6) 输入 H/h 帮助.
    0) 输入 Q/q 退出.

Opt>
```

## JumpServer实战

### 添加站点

1.登陆进系统-->系统设置-->设置当前站点URL为服务器地址---->提交

![](https://pic.superbed.cn/item/5d1a3c0b451253d178b93f08.png)

### 设置邮箱

![](https://pic3.superbed.cn/item/5d1a3c21451253d178b93fdd.png)

注:使用该功能必须确定自己的邮箱已开启了smtp和pop3服务

服务器地址:

网易邮箱:

```
pop服务器:pop.163.com

smtp:smtp.163.com

imap:imap.163.com
```

配置完成之后,需要手动重启服务,不然后期创建用户,收不到邮箱

```
(py3) [root@localhost jumpserver]# ./jms restart all -d 
```

配置邮件服务后，点击页面的"测试连接"按钮，如果配置正确，Jumpserver 会发送一条测试邮件到您的 SMTP 账号邮箱里面：

![](https://pic.superbed.cn/item/5d1a3c32451253d178b9406f.png)

注意： 在使用jumpserver过程中，有一步是系统用户推送，要推送成功，client（后端服务器）要满足以下条件： 

1）后端服务器需要有python、sudo环境才能使用推送用户，批量命令等功能 

2）后端服务器如果开启了selinux，请安装libselinux-python。一般情况服务器上都关闭了selinux



### 用户管理

1)添加用户组

用户名即jumpserver登陆账号,用户组是用来资产授权,当某个资产对一个用户组授权后,这个用户组下面的所有用户都可以使用这个资产了.角色用于区分一个用户是管理员还是普通用户.

点击用户管理-->用户组-->添加用户组

![](https://pic.superbed.cn/item/5d1a3c4b451253d178b9413e.png)

创建用户,并将其添加到刚刚创建的jumpserver组中

![](https://ae01.alicdn.com/kf/HTB1Jy2ceMmH3KVjSZKz7622OXXat.png)

密码会自动产生,并通过邮件发送到用户邮箱中

然后登陆账号,首次登陆需要填写信息

![](https://pic3.superbed.cn/item/5d1a3c6f451253d178b9426c.png)

第二步需要ssh公钥,所以本地生成一个公钥

```
[root@localhost ~]# ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:8bIwbzG55sTSvCCGCgI/ajtdxeOp6UXJfBDBcG6jgyo root@localhost.localdomain
The key's randomart image is:
+---[RSA 2048]----+
|     .o+.        |
|      o..        |
|      .=.        |
|    . =+++       |
|.  . o==S..      |
|.... .oX.*       |
|E.= + =.@        |
|+= + +.B .       |
|+.o ..  o        |
+----[SHA256]-----+
```

粘贴公钥填入个人信息中

```
[root@localhost ~]# cd .ssh/
[root@localhost .ssh]# ls
id_rsa  id_rsa.pub  known_hosts
[root@localhost .ssh]# cat id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQClEDeVNMYP61JPrCXUQHQYC1ddIpPqroNrQn3SXZXLfef2g0c5xmwlHqgiHEjBGLR+0TIjkXDFY0Z9JU/TebGyrBbo9bTM0tkxVfWVAR/Ayba7yN98Sr43evev1yHIsg31eyPa2wE6TRmpz6jCHTMOodw4+TMkfiXdPDyw2Ny+6zOXGtK8Kz2Sie1SCgSWrNaGp364aZInjB7J2H5fCXLwW6SQqmcwer29q2djNw0ILc4acYpDe+pOMm2CGbnFvTaB6T9A1hfdpmQ74TnfI4frukM2vUqKwms6/At+TVqvWl+RX9jU83y7pOSoWZeRKKoJct4frpcPghAYJI1+qhtz root@localhost.localdomain
```





### 创建资产

创建Linux资产

编辑资产树

节点名不能重名,右击节点可以添加,删除和重命名节点,以及进行资产相关的操作.

![](https://pic3.superbed.cn/item/5d1a3c80451253d178b942f7.png)

### 创建管理用户

jumpserver里各个用户的说明

![](https://pic.superbed.cn/item/5d1a3c8f451253d178b94400.jpg)

管理用户是服务器的 root，或拥有 NOPASSWD: ALL sudo 权限的用户，Jumpserver 使用该用户来推送系统用户、获取资产硬件信息等。

![](https://pic.superbed.cn/item/5d1a3ca2451253d178b944c4.png)

### 创建系统用户

![](https://pic.superbed.cn/item/5d1a3cb4451253d178b94555.png)

系统用户是 Jumpserver 跳转登录资产时使用的用户，可以理解为登录资产用户， Jumpserver使用系统用户登录资产。

系统用户的 Sudo 栏填写允许当前系统用户免sudo密码执行的程序路径，如默认的/sbin/ifconfig，意思是当前系统用户可以直接执行 ifconfig 命令或 sudo ifconfig 而不需要输入当前系统用户的密码，执行其他的命令任然需要密码，以此来达到权限控制的目的。

\# 此处的权限应该根据使用用户的需求汇总后定制，原则上给予最小权限即可。

系统用户创建时，如果选择了自动推送 Jumpserver 会使用 Ansible 自动推送系统用户到资产中，如果资产(交换机、Windows )不支持 Ansible, 请手动填写账号密码。

Linux 系统协议项务必选择 ssh 。如果用户在系统中已存在，请去掉自动生成密钥、自动推送勾选。



### 创建资产

点击页面左侧的“资产管理”菜单下的“资产列表”按钮，查看当前所有的资产列表。

点击页面左上角的“创建资产”按钮，进入资产创建页面，填写资产信息。

IP 地址和管理用户要确保正确，确保所选的管理用户的用户名和密码能"牢靠"地登录指定的 IP 主机上。资产的系统平台也务必正确填写。公网 IP 信息只用于展示，可不填，Jumpserver 连接资产使用的是 IP 信息。![](https://ae01.alicdn.com/kf/HTB1tu6beUCF3KVjSZJn762nHFXaf.png)

再次更新之后就变成了可连接的了

![](https://pic.superbed.cn/item/5d1a3cda451253d178b9467f.png)

也可以去测试资产是否可以连接

![](https://pic.superbed.cn/item/5d1a3cf1451253d178b9472e.png)

![](https://pic.superbed.cn/item/5d1a3d02451253d178b947b5.png)

如果资产不能正常连接，请检查管理用户的用户名和密钥是否正确以及该管理用户是否能使用 SSH 从 Jumpserver 主机正确登录到资产主机上。



### 网域列表

网域功能是为了解决部分环境无法直接连接而新增的功能，原理是通过网关服务器进行跳转登录。

这个功能，一般情况不用到。



### 资产授权

节点，对应的是资产，代表该节点下的所有资产。

用户组，对应的是用户，代表该用户组下所有的用户。

系统用户，及所选的用户组下的用户能通过该系统用户使用所选节点下的资产。

节点，用户组，系统用户是一对一的关系，所以当拥有 Linux、Windows 不同类型资产时，应该分别给 Linux 资产和 Windows 资产创建授权规则。

![](https://pic1.superbed.cn/item/5d1a3d12451253d178b94838.png)

在授权成功后,jumpserver会自动推送一个帐号，自动在资产服务器上创建系统用户

![](https://pic.superbed.cn/item/5d1a3d23451253d178b948d4.png)



![](https://pic3.superbed.cn/item/5d1a3d36451253d178b9496b.png)

其原理就是在/etc/sudoers设置该用户的权限,sudo相关的规则也会被自动推送过来

```
test1 ALL=(ALL) NOPASSWD: /sbin,/bin 
```

### 用户使用资产

登录 Jumpserver

创建授权规则的时候，选择了用户组，所以这里需要登录所选用户组下面的用户才能看见相应的资产。

使用无痕浏览器，再打开一个窗口，进行登录：

使用刚刚创建的用户haha去登陆,连接资产主机

![](https://pic3.superbed.cn/item/5d1a3d49451253d178b94a01.png)

也可以通过xshell去连接

```
[root@localhost ~]# ssh -p 2222 haha@192.168.48.139
The authenticity of host '[192.168.48.139]:2222 ([192.168.48.139]:2222)' can't be established.
RSA key fingerprint is SHA256:51aZmkQvw20kIozk9n3Sg0aGUJ6ZSJMQyJInC3HQ08w.
RSA key fingerprint is MD5:82:e3:ef:bf:8e:5b:db:bd:e2:56:67:4e:08:e1:d1:b0.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[192.168.48.139]:2222' (RSA) to the list of known hosts.


    haha, 欢迎使用Jumpserver开源跳板机系统  

    1) 输入 ID 直接登录 或 输入部分 IP,主机名,备注 进行搜索登录(如果唯一).
    2) 输入 / + IP, 主机名 or 备注 搜索. 如: /ip
    3) 输入 P/p 显示您有权限的主机.
    4) 输入 G/g 显示您有权限的主机组.
    5) 输入 G/g + 组ID 显示该组下主机. 如: g1
    6) 输入 H/h 帮助.
    0) 输入 Q/q 退出.

Opt> 

```

 

在xshell字符终端下连接jumpserver管理服务器

输入ip或者ID直接可以连接到主机

```
Opt> 192.168.48.139

Connecting to test1@资产主机 0.4
Last login: Thu Jun 13 00:27:36 2019 from 192.168.48.139
[test1@localhost ~]$ whoami
test1
[test1@localhost ~]$ exit
登出

Opt> 
```

输入p(不区分大小写)查看你有权限的主机

```
Opt> p

 ID Hostname        IP              LoginAs        Comment                              
  1 资产主机            192.168.48.139  [检查服务器运行状态的用户] 

总共: 1 匹配: 1

```

输入g(不区分大小写)查看你有权限的组

```
Opt> g

   ID Name            Assets     Comment                                                    1 jumpserver服务器   1                                     

总共: 1
```

### 查看历史回话

![](https://pic1.superbed.cn/item/5d1a3d5f451253d178b94ab7.png)

### 查看历史命令

![](https://ae01.alicdn.com/kf/HTB1DVLbeMaH3KVjSZFj763FWpXaN.png)