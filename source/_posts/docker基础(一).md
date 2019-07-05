---
title: docker学习笔记(一)
date: 2019-05-14 20:57:28
categories: 运维技术
tags: docker
---
<blockquote class="blockquote-center">穷则思变,变则通,通则达!</blockquote>

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=325098&auto=1&height=66"></iframe></div>

### Docker简介:

Docker是一个开源的应用容器引擎,让开发者可以打包他们的应用以及依赖包到一个可移植的容器中,然后发布到任何流行的Linux机器上,也可以实现虚拟化.容器是完全使用沙盒机制,互相之间不会有任何接口(类似iPhone的APP)几乎没有性能开销,很容易的在机器和数据中心中运行.最重要的是,他们不依赖任何语言,框架或包装系统



#### 沙盒

沙盒也叫沙箱,英文sandbox.在计算机领域指的是一种虚拟技术,且多用于计算机安全技术.安全软件可以先让它在沙盒中运行,如果有恶意行为,则禁止程序的进一步运行,而这不会给系统造成任何危害



Docker是dotCloud公司开源的一个基于LCX的高级容器引擎,源代码托管在github上,基于go语言并遵从Apache2.0协议开源,其官网为:<https://www.docker.com

#### LCX

LCX是Linux Container的简写,Linux Container容器是一种内核虚拟化技术,可以提供轻量级的虚拟化,以便隔离进程和资源,而且不需要提供指令解释机制以及全虚拟化的其他复杂性



LCX主要是通过来自kernel的namespace实现每个用户实例之间的相互隔离,通过cgroup实现对资源的配置和度量.



#### **docker容器技术和虚拟机对比**

- 相同点:docker容器技术和虚拟机技术都是虚拟化技术
- 不同点:docker相当于vm虚拟机,但少了虚拟机操作系统这一层,所以docker效率要比vm强



### docker架构

![](http://ww1.sinaimg.cn/large/0078beR7ly1g313juc6o9j30j10dg41f.jpg)

#### 工作流程:

服务器A上运行docker engine服务,在docker engine上启动多个container,从外网docker hub上把image操作系统镜像下载下来,放到container容器中运行,这样一个容器的实例就运行起来了.最后通过docker client对docker容器虚拟化平台进行控制



#### image和container的区别:

image可以理解为一个系统镜像,container是image在运行时的一个状态.如果拿虚拟机作一个比喻的话,image是关机状态下的磁盘文件,container是虚拟机运行时的磁盘文件,包括内存数据	



#### docker hub

是docker官方的镜像存储站点,其中提供了很多常用的镜像供用户下载,如ubuntu,centos等系统镜像,通过docker hub,用户也可以发布自己的docker镜像,为此用户需要注册一个账号,在网站上创建一个docker仓库



#### docker核心技术:

namespace-----实现container的进程,网络,消息,文件系统和主机名的隔离

cgroup------实现对资源的配额和度量



#### docker的特性

- - 文件系统隔离:每个进程容器运行在一个完全独立的根文件系统中
  
  - 资源各类:系统资源,像CUP和内存等可以分配到不同的容器中,使用cgroup
  
  - 日志记录:docker将会收集和记录每个进程容器的标准流(stdout/stderr/stdin),用于实时检索或批量检索
  
  - 变更管理:容器文件系统的变更可以提交到新的镜像中,并可重复使用已创建更多的容器,无需使用模板或手工配置
  
  - 交互式shell:docker可以分配一个虚拟终端并关联到任何容器的标准输入上,例如运行一个一次性交互的shell
  
    

### Docker的优缺点:

#### 优点：

1. 一些优势和 VM 一样，但不是所有都一样。

    VM 小，比 VM 快，Docker 容器的尺寸减小相比整个虚拟机大大简化了分布到云和从云分发时间和开销。Docker 启劢一个容器实例时间徆短，一两秒就可以启劢一个实例。

   对于在笔记本电脑，数据中心的虚拟机，以及任何的云上，运行相同的没有变化的应用程序，IT 的发布速度更快。

   Docker 是一个开放的平台，构建，发布和运行分布式应用程序。

   Docker 使应用程序能够快速从组件组装和避免开发和生产环境之间的摩擦。

2. 您可以在部署在公司局域网戒云戒虚拟机上使用它。

3. 开发人员并不关心具体哪个 Linux 操作系统

   使用 Docker，开发人员可以根据所有依赖关系构建相应的软件，针对他们所选择的操作系统。

   然后，在部署时一切是完全一样的，因为一切都在 DockerImage 的容器在其上运行。

   开发人员负责并且能够确保所有的相关性得到满足。

4. Google，微软，亚马逊，IBM 等都支持 Docker。

5. Docker 支持 Unix/Linux 操作系统，也支持 Windows 戒 Mac



#### 缺点：

1.Docker 用于应用程序时是最有用的，但并丌包含数据。日志，跟踪和数据库等通常应放在 Docker

容器外。一个容器的镜像通常都徆小，丌适合存大量数据，存储可以通过外部挂载的方式使用。比如使用：NFS，ipsan，MFS 等, -v 映射磁盘分区

2.一句话：docker 叧用于计算，存储交给别人。

3.oracle 不适合使用 docker 来运行，太大了，存储的数据太多。



### 安装docker

在红帽7/centos7上安装

下载docker引擎的rpm安装包

```
wget https://get.docker.com/rpm/1.7.1/centos-7/RPMS/x86_64/docker-engine-1.7.1-1.el7.centos.x86_64.rpm        
```

 

安装docker

```
 rpm -ivh docker-engine-1.7.1-1.el7.centos.x86_64.rpm        
```

开启docker

```
systemctl  start docker   
```

设置开机自启

systemctl  enable docker  

查看docker信息

```
docker info   
```

查看所存在的镜像

```
docker images   
```

从 Docker Hub 仓库下载一个 Ubuntu 12.04 操作系统的镜像

```
docker pull ubuntu:12.04     
```

  这条命令实际上相当于  docker pull registry.hub.docker.com/ubuntu:12.04 命令，即从注册服务器 registry.hub.docker.com 中的 ubuntu 仓库来下载标记为 12.04 的镜像。如果不指定版本的话,那么就去下载最新版的ubuntu



用镜像创建一个容器  **-t** :指定要创建的目标镜像名

```
docker run -t -i ubuntu:12.04 /bin/bash       
root@afcdf3ef30bc:/# cat /etc/issue   //查看版本,发现已经进入了ubuntu Ubuntu 12.04.5 LTS \n \l  

root@afcdf3ef30bc:/# exit     //退出镜像 exit 
```

查看所存在的镜像

```
[root@localhost ~]# docker images     

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE 

ubuntu              12.04               62b726df5062        23 months ago       103.6 MB 
```

docker inspect +IMAGE ID可以查看当前镜像的具体信息

```
[root@localhost ~]# docker inspect 62b726df5062     
```

通过docker rmi ubuntu:版本/id删除镜像

```
[root@localhost ~]# docker rmi ubuntu:12.04    
```

 

修改镜像的tag(标签)

```
[root@localhost ~]# docker tag  ubuntu:12.04  ubuntu:newtag  
```

docker下载kali系统

```
[root@localhost ~]# docker pull kalilinux/kali-linux-docker   
```



在Docker Hub搜寻镜像

```
[root@localhost ~]# docker search httpd  
```

  

**查看docker详细信息**

```
[root@localhost ~]# docker info 
```

**从docker hub搜索所需要的镜像,如果 OFFICIAL 为[ok] ，说明可以放心使用。** 

```
[root@localhost ~]# docker search centos 
```

**从docker hub拉取(下载)镜像 pull:拉**

```
[root@localhost ~]# docker pull centos 
```

### pull镜像出错解决方法

#### 错误一

```
[root@xuegod63 ~]# docker pull docker.io/centos

Using default tag: latest

Trying to pull repository docker.io/library/centos ...  

latest: Pulling from docker.io/library/centosGet 

https://registry-1.docker.io/v2/library/centos/manifests/sha256:822de5245dc5b659df56dd

32795b08ae42db4cc901f3462fc509e91e97132dc0: net/http: TLS handshake timeout
```



##### 法一:换国内源

**修改/etc/docker/daemon.json**

```
[root@xuegod63 ~]# vim /etc/docker/daemon.json #改成以下内容

改： {}

为：

{

"registry-mirrors": ["https://e9yneuy4.mirror.aliyuncs.com"]

}

[root@xuegod63 ~]# systemctl daemon-reload**

[root@xuegod63 ~]# systemctl restart docker**

[root@xuegod63 ~]# docker pull docker.io/centos #再下载，就可以了。
```



##### 法二：把之前下载好的 image 镜像导入 image：

**把 docker.io-centos.tar 镜像上传到 linux 上**

**参数： -i " docker.io-centos.tar " 指定载入的镜像归档。**

```
[root@xuegod63 ~]# docker load -i /root/docker.io-centos.tar
```



##### 法三：直接下载其他站点的镜像

```
[root@xuegod63 ~]# docker pull hub.c.163.com/library/tomcat:latest
[root@xuegod63 ~]# docker images

REPOSITORY TAG IMAGE ID CREATED SIZE

hub.c.163.com/library/tomcat** **latest 72d2be374029 4 months ago 292.4 MB
```

**查看 images 列表**

```
[root@xuegod63 ~]# docker images #列出本地所有镜像。其中 [name] 对镜像名称进行关键

词查询。
[root@xuegod63 ~]# docker images

REPOSITORY TAG IMAGE ID CREATED SIZE

docker.io/centos latest 8caf41e7a3ea 31 minutes ago 205.3 MB
```



#### 错误二:

如果报以下错误:表示没有开启网络转发功能的话就会报错,默认已自动打开

![](http://ww1.sinaimg.cn/large/0078beR7ly1g2zo3zq2o5j30i701da9w.jpg)

那么输入以下内容

```
[root@localhost ~]# echo 1 >/proc/sys/net/ipv4/ip_forward 
```

或者修改一下文件

```
[root@xuegod63 ~]# vim /etc/sysctl.conf 

#插入以下内容 

net.ipv4.ip_forward = 1 

[root@xuegod63 ~]# sysctl -p #生效

 net.ipv4.ip_forward = 1

 [root@xuegod63 ~]# cat /proc/sys/net/ipv4/ip_forward 

1 
```



### docker实践:

#### 实例1:在实例中执行bash命令

运行一个container并加载centos,运行起来之后,在实例中执行/bin/bash命令

- run:运行
- -i:以交互模式运行容器,通常与-t同时使用
- -t:为容器重新分配一个伪输入终端,通常与-i同时使用

格式:docker run -it 镜像名:tags  /bin/bash

```
[root@localhost ~]# docker run -it centos /bin/bash 
[root@47a67a198aa8 /]# exit exit 
```

#### 实例2:模拟后台运行服务

例2:在container中运行一个长久运行的进程,不断向stdin输出helloworld,模拟一个后台运行服务

docker常用参数:

- - -d:在后台运行容器,并返回容器ID
  - -c:后面跟待完成的命令,bash指的是使用bash去执行命令

```
[root@localhost ~]# docker run -d centos:latest bash  -c "while true;do echo hello,world;sleep 1;done" 2822f36cb76defc5923ea509d32eaa3fae165830064484746c97aabbc0339839  
```

返回值为容器的ID

也可以使用docker ps查看所有运行的容器,可以发现其容器ID

```
[root@localhost docker]# docker ps CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES 2822f36cb76d        centos:latest       "bash -c 'while true   11 minutes ago      Up 11 minutes                           goofy_hodgkin    
```

​    

列所有的容器(包含沉睡/退出状态的容器)

```
docker  ps  -a  
```

从容器中取日志,查看输出内容

语法:docker logs 容器id

```
[root@localhost ~]# dockerCONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES 2822f36cb76d        centos:latest       "bash -c 'while true   11 minutes ago      Up 11 minutes                           goofy_hodgkin        logs 2822f36cb76defc5923ea509d32eaa3fae165830064484746c97aabbc0339839 
```

![](http://ww1.sinaimg.cn/large/0078beR7ly1g2zo3l5gfpj31800jqdgj.jpg)



#### 实例3:kill掉一个容器

**首先列出所有的容器**

```
[root@localhost ~]# docker ps -a  
```

**杀死容器docker kill  容器ID**

```
[root@localhost ~]# docker kill c4a213627f1b c4a213627f1b 
```

**关闭容器**

```
[root@localhost ~]# docker stop 2822f36cb76d 2822f36cb76d 
```

**开启容器**

```
[root@localhost ~]# docker start 2822f36cb76d 2822f36cb76d 
```

**删除容器(不能删除正在运行的容器)**

```
[root@localhost ~]# docker rm 2822f36cb76d 
```

**强制删除容器(可以删正在运行的容器)**

```
[root@localhost ~]# docker rm -f 2822f36cb76d 
```

**删除镜像(image)**

```
[root@localhost ~]# docker rmi 9ab5ff067039 
Deleted: 9ab5ff0670399b1ac4dad4c2aaf61e59489155325d3fb5082ecae75d2b3e5fc8
```



### docker镜像制作方法:

- 法一:使用docker commit 容器ID(或者镜像名),保存container的当前状态到image后,然后生成对应的image

  ```
  [root@localhost ~]# docker commit 95e796801e15433631bb6cee9e4f101503396db5732f578c4a842ceeb6d82862 centos1:lastest 
  #docker commit 容器ID  生成的容器名:标签
  #也可以直接docker commit 容器ID
  ```

- 法二:在Docker file文件下使用docker build自动化制作镜像,Dockerfile有点像源码编译时./configure后产生的Makefile

  ```
  [root@localhost ~]# cd /
  [root@localhost /]# mkdir docker-build
  [root@localhost /]# cd docker-build/
  [root@localhost docker-build]# touch Dockerfile
  [root@localhost docker-build]# vim Dockerfile 
  #填写以下内容
  FROM docker.io/centos:latest   #FROM基于哪个镜像
  MAINTAINER <mk@xuegod.cn> #MAINTAINER镜像创建者
  RUN yum -y install httpd   #RUN安装软件
  ADD start.sh /usr/local/bin/start.sh  #把start.sh启动脚本安装到镜像的/usr/local/bin/start.sh目录下
  
  #把index.html启动脚本安装到镜像的/var/www/html/index.html里
  ADD index.html /var/www/html/index.html 
  
  #container启动时执行的命令或启动服务,但是一个Dockerfile中只能有一条CMD命令,多条则
  CMD echo hello,world 
  
  ```

  或者写入另一个Dockerfile文件

  ```
  # vim dockefile1
  FROM ubuntu
  MAINTAINER xxx
  RUN echo hello1 > test1.txt
  RUN echo hello2 > /test2.txt
  EXPOSE 80
  EXPOSE 81
  CMD ["/bin/bash"]
  ```

  

3、创建 start.sh 脚本启劢 httpd 服务和 apache 默认首页 index.html 文件 

```
[root@localhost docker-build]#  echo "/usr/sbin/httpd -DFOREGROUND" > start.sh 

注: /usr/sbin/httpd -DFOREGROUND 相当于执行了 systemctl start httpd 

[root@localhost docker-build]#  chmod a+x start.sh 
```

创建 index.html 

```
[root@localhost docker-build]#  echo "docker image build test" > index.html 
```

4、使用命令 build 来创建新的 image 

语法：docker build -t 父镜像名：镜像的 tag Dockerfile 文件所在路径 

-t :表示 tage，镜像名 

例：使用命令 docker build 来创建新的 image,并命名为 docker.io/centos:httpd 

```
[root@localhost docker-build]#  docker build -t docker.io/centos:httpd ./ 
```

注： ./ 表示当前目彔。另外你的当前目彔下要包含 Dockerfile 

### Docker Image的发布

docker镜像=应用/程序+库

方法一:save image to tarball,保存镜像到tar包

语法:

```
docker save -o 导出的镜像名.tar  本地镜像名:镜像标签
```

然后导入时使用

```
docker load -i 导出的镜像名.tar
```

方法二:push到docker hub上  

1.注册账号:https://hub.docker.com/

2.登陆docker hub

```
docker login -u 用户名 -p 密码 -e 邮箱地址
```

3.上传镜像

```
docker image 镜像名:标签
```

4.下载镜像

```
docker pull 用户名/镜像名
```

### docker端口映射

#### 端口映射

-d:设置容器在在后台一直运行

-p:设置端口映射,格式为本地端口:docker容器端口

-c:执行系统命令,这个文件是我本地写好的自动化脚本用来打开HTTP服务

```
[root@localhost ~]# docker run -d -p 80:80 centos:httpd /bin/bash -c /usr/local/bin/start.sh
```

![](http://ww1.sinaimg.cn/large/0078beR7ly1g36zg8qh9yj30h605qmxa.jpg)

注:当前使用的docker实例运行的网络模式相当于VMware中的NAT模式



#### 查看正在运行的容器

```
[root@localhost ~]# docker ps
```

#### 访问容器实例

docker exec -it  <docker ID|name> /bin/bash

```
docker exec -it  centos:httpd /bin/bash
```

