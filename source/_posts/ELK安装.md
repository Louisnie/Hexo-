---
title: ELK安装部署
date: 2019-05-30 00:25:28
categories: 运维技术
tags: ELK

---
<blockquote class="blockquote-center">不断进步,直到羔羊变成雄狮!</blockquote>
<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=29539350&auto=0&height=66"></iframe></div>

## ELK Stack 简介

ELK 不是一款软件，而是 Elasticsearch、Logstash 和 Kibana 三种软件产品的首字母缩写。这三者都是开源软件，通常配合使用，而且又先后归于 Elastic.co 公司名下，所以被简称为 ELK Stack。根据 Google Trend 的信息显示，ELK Stack 已经成为目前最流行的集中式日志解决方案。

- Elasticsearch：分布式搜索和分析引擎，具有高可伸缩、高可靠和易管理等特点。基于 Apache Lucene 构建，能对大容量的数据进行接近实时的存储、搜索和分析操作。通常被用作某些应用的基础搜索引擎，使其具有复杂的搜索功能；
- Logstash：数据收集引擎。它支持动态的从各种数据源搜集数据，并对数据进行过滤、分析、丰富、统一格式等操作，然后存储到用户指定的位置；
- Kibana：数据分析和可视化平台。通常与 Elasticsearch 配合使用，对其中数据进行搜索、分析和以统计图表的方式展示；
- Filebeat：ELK 协议栈的新成员，一个轻量级开源日志文件数据搜集器，基于 Logstash-Forwarder 源代码开发，是对它的替代。在需要采集日志数据的 server 上安装 Filebeat，并指定日志目录或日志文件后，Filebeat 就能读取数据，迅速发送到 Logstash 进行解析，亦或直接发送到 Elasticsearch 进行集中式存储和分析。

## Elasticsearch安装:

### 部署环境

我当前系统为红帽7.4

```
[root@localhost ~]# cat /etc/redhat-release 

Red Hat Enterprise Linux Server release 7.4 (Maipo)  
```

因为Elasticsearch需要Java8以上的版本,所以需要检查Java环境,redhat7的java环境是ok的,不需要进行额外配置

```
[root@localhost ~]# java -version openjdk version "1.8.0_131"     //java8又称jdk1.8 OpenJDK Runtime Environment (build 1.8.0_131-b12) OpenJDK 64-Bit Server VM (build 25.131-b12, mixed mode) 
```

### 开始安装

下载Elasticsearch,我这里用得是6.6.2版本

```
[root@localhost local]#  wget  https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.6.2.tar.gz

[root@localhost local]# tar zxvf elasticsearch-6.6.2.tar.gz
```

创建目录,用来保存系统文件和日志

```
[root@localhost ~]# mkdir -p /usr/local/elk/data 

[root@localhost ~]# mkdir  /usr/local/elk/logs 
```

### 修改配置文件

我们需要去修改其配置文件/usr/local/elasticsearch-6.6.2/config下的elasticsearch.yml文件

```
[root@localhost config]# vim elasticsearch.yml
#设置集群名
cluster.name: my-cluster 

#设置节点名     
node.name: node-1  

#设置数据保存文件
path.data: /usr/local/elk/data

#设置日志保存文件
path.logs: /usr/local/elk/logs

#设置监听主机地址,允许任意主机均可访问
network.host: 0.0.0.0

#默认使用9200端口
http.port: 9200
```

然后去修改jvm.options配置文件

```
[root@localhost config]# vim jvm.options 

 -Xms512m   #设置最小堆内存为512M

-Xmx512m   #设置最大堆内存为512M 
```

因为elasticsearch不能使用root用户去打开,所以需要创建个elk用户,使用该用户去登陆

```
[root@localhost local]# useradd elk
[root@localhost local]# chown -R elk:elk elk
[root@localhost local]# chown -R elk:elk elasticsearch-6.6.2/
[root@localhost local]# su - elk
[elk@localhost ~]$
```

另外需要去修改系统配置文件

```
[root@localhost local]# vim /etc/security/limits.conf
#添加以下内容
soft    nofile          65536   

hard    nofile          65536
```

```
[root@localhost local]# vim /etc/sysctl.conf
#添加以下内容
vm.max_map_count=262144
```

使其生效

```
[root@localhost local]# sysctl -p   
vm.max_map_count = 262144
```

新开个连接,然后检测是否设置成功

```
[root@localhost ~]# ulimit -Hn
65536
[root@localhost ~]# ulimit -Sn
65536
```

### 启动服务

接下来切换到elk用户去启动elasticsearch

```
[root@localhost ~]# su - elk 

上一次登录：五 5月 24 16:47:42 CST 2019pts/2 上 

[elk@localhost ~]$ cd /usr/local/elasticsearch-6.6.2/bin/ 

[elk@localhost bin]$ ./elasticsearch 
```

通过访问本地的9200端口检查是否安装成功

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g3iofamysjj30fa0fijrz.jpg)

但因为我们开启的elasticsearch是在终端中开启的,一旦终端关闭,那么该服务将关闭,所以我们需要让该程序在后台执行

```
nohup /usr/local/elasticsearch-6.6.2/bin/elasticsearch >> /usr/local/elasticsearch-6.6.2/output.log 2>&1 & 
```

### 后台运行

我们可以将其做出shell脚本让其在后台去执行

```
[elk@localhost elasticsearch-6.6.2]$ touch startup.sh
[elk@localhost elasticsearch-6.6.2]$ vim startup.sh
#将以下内容添加进去
#! /bin/bash
nohup /usr/local/elasticsearch-6.6.2/bin/elasticsearch >> /usr/local/elasticsearch-6.6.2/output.log 2>&1 &

#赋予权限
[elk@localhost elasticsearch-6.6.2]$ chmod a+x startup.sh 
#执行脚本 
[elk@localhost elasticsearch-6.6.2]$ ./startup.sh
```

### 关闭服务

```
#找到其进程号 

[elk@localhost elasticsearch-6.6.2]$ ps -ef | grep java  elk       18114      1  2 18:14 pts/1    00:00:54 /bin/java -Xms1g -Xmx1g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOcc 

#杀死进程 

[elk@localhost elasticsearch-6.6.2]$ kill -9 18114  
```



## Kibana安装

### 下载解压

```
[root@localhost local]#  wget  https://artifacts.elastic.co/downloads/kibana/kibana-6.6.2-linux-x86_64.tar.gz
[root@localhost local]# tar zxvf kibana-6.6.2-linux-x86_64.tar.gz 
```

注:kibana可以和elasticsearch不在同一台机器上,可以用来做成集群

### 修改配置文件

修改该目录中config文件中的kibana.yml配置文件

```
[root@localhost config]# vim kibana.yml 

#设置监听端口为5601

server.port: 5601

#设置可访问的主机地址

server.host: "0.0.0.0"

#设置elasticsearch主机地址

elasticsearch.hosts: ["http://localhost:9200"]

#如果elasticsearch设置了用户名密码,那么需要配置该两项,如果没配置,那就不用管

#elasticsearch.username: "user"

#elasticsearch.password: "pass"
```

### 后台启动服务

```
[root@localhost kibana-6.6.2-linux-x86_64]# vim startup.sh 
#添加以下内容
#! /bin/bash
nohup /usr/local/kibana-6.6.2-linux-x86_64/bin/kibana >> /usr/local/kibana-6.6.2-linux-x86_64/output.log 2>&1 &
```

### 访问服务

通过浏览器访问本地的5601端口去使用kibana服务

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g3iohc51l6j31b50n3445.jpg)

### 关闭服务

```
[root@localhost kibana-6.6.2-linux-x86_64]# ps -ef | grep kibana 
root       3080      1  1 22:16 pts/2    00:01:04 /usr/local/kibana-6.6.2-linux-x86_64/bin/../node/bin/node --no-warnings --max-http-header-size=65536 /usr/local/kibana-6.6.2-linux-x86_64/bin/../src/cli 
root       4245   2683  0 23:44 pts/2    00:00:00 grep --color=auto kibana  

[root@localhost kibana-6.6.2-linux-x86_64]# kill -9 3080 
```

## Logstash安装

### 下载解压缩

```
[root@localhost local]# wget https://artifacts.elastic.co/downloads/logstash/logstash-6.6.2.tar.gz 

[root@localhost logstash-6.6.2]# tar zxvf logstash-6.6.2.tar.gz  
```

修改配置文件

```
[root@localhost config]# vim jvm.options  

#修改如下 

-Xms512m     #设置最小内存  

-Xmx512m     #设置最大内存 
```

进入bin目录下运行程序,将日志信息输出到屏幕上

```
[root@localhost bin]# ./logstash -e 'input {stdin{}} output{stdout{}}' 
```

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g3ioifcd7lj31gv0i211m.jpg)

比如输入个hello,world然后回车,那么就会把结果输出到屏幕上

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g3ioj9dve6j30n00740th.jpg)

### 使用配置文件启动

编辑主配置文件

```
[root@localhost logstash-6.6.2]# cd config/
[root@localhost config]# mv logstash-sample.conf  logstash.conf
[root@localhost config]# vim logstash.conf 
#删除其文件内容,添加以下内容
input {
    # 从文件读取日志信息
      file {
          path => "/var/log/messages"
          type => "system"
          start_position => "beginning"
           }
}

filter {
}

output {
      # 标准输出
      stdout {}
}
```

使用主配置文件去启动程序

```
[root@localhost bin]# ./logstash -f ../config/logstash.conf  
```

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g3iokg8fdjj31hc0gaguy.jpg)

### 使用脚本启动

```
[root@localhost logstash-6.6.2]# touch startup.sh
[root@localhost logstash-6.6.2]# vim startup.sh
#内容如下
#!/bin/bash
nohup /usr/local/logstash-6.6.2/bin/logstash -f /usr/local/logstash-6.6.2/config/logstash.conf >> /usr/local/logstash-6.6.2/output.log 2>&1 &

[root@localhost logstash-6.6.2]# chmod a+x startup.sh 
[root@localhost logstash-6.6.2]# ./startup.sh
```

### logstash插件

logstash是通过插件对其功能进行加强

插件分类:

- inputs 输入
- codecs 解码
- filters 过滤
- outputs 输出

在Gemfile文件里记录了logstash的插件

```
[root@localhost logstash-6.6.2]# cat Gemfile 
```

如果需要其他插件的话,那么需要去其github上的库下载插件,地址为:<https://github.com/logstash-plugins>

使用filter插件logstash-filter-mutate

```
[root@localhost config]# vim logstash2.conf 
#创建一个新的配置文件用来过滤 
input {
    stdin {
    }
}

filter {
   mutate {
        split => ["message", "|"]
    }
}

output {
    stdout {
    }
}
```

当输入sss|sssni|akok223|23即会按照|分隔符进行分隔

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g3ioldkha1j30qn0b70u3.jpg)

其数据处理流程:input-->解码-->filter-->解码-->output

## ELK联动

我们需要使用logstash-output-elasticsearch插件将logstash日志信息收集到elasticsearch当中

### 检查插件

因为在logstash当中就存在elasticsearch的插件,那么就可以直接使用的

```
[root@localhost logstash-6.6.2]# cat Gemfile |grep elasticsearch 

gem "logstash-filter-elasticsearch" 

gem "logstash-input-elasticsearch" 

gem "logstash-output-elasticsearch" 
```

### 修改配置文件

那么我们去写个配置文件,通过配置文件去将elasticsearch和logstash结合起来

```
[root@localhost config]# vim logstash3.conf 

#填写以下内容 
input {
    # 从文件读取日志信息
    file {
        path => "/var/log/messages"
        type => "system"
        start_position => "beginning"
    }
}

filter {
}

output {
    elasticsearch {
        hosts => ["127.0.0.1:9200"]
        index => "msg-%{+YYYY.MM.dd}"
    }
}
```

### 同步系统时间

因为系统时间不准确,所以更新一下系统时间,与阿里的ntp服务器进行同步

ntp1.aliyun.com ~ ntp5.aliyun.com这几个ntp服务器都可以使用的

```
[root@localhost config]# ntpdate ntp1.aliyun.com  #进行同步 

29 May 15:04:13 ntpdate[20946]: adjust time server 120.25.115.20 offset -0.004502 sec  

[root@localhost config]# date -R  #查看当前时区时间 Wed, 29 May 2019 15:04:56 +0800 
```

### 启动服务

然后去启动logstash服务

```
[root@localhost config]# ../bin/logstash -f logstash3.conf  
```

### ELK联动

去访问本地的5601端口,打开kibana

可以在kibana上看到增加了一个索引

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g3iom7zhfej319b0gpn06.jpg)

然后去创建模式,设置索引模式为msg-*,即是以msg-开头的索引都进行匹配

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g3iomyon0wj31hc0pn45g.jpg)

设置按系统时间来进行过滤

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g3iont5fwfj311w0jqq5z.jpg)

然后在Discover面板选择msg-*模块就可以看到当前的数据

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g3ioohrod1j31hc0ozgvg.jpg)

