---
title: 搭建ELK实时分析nginx日志
date: 2019-06-04 21:59:28
categories: 运维技术
tags: ELK
---
<blockquote class="blockquote-center">路漫漫其修道远，吾将上下而求索。      </blockquote>
<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=36897723&auto=1&height=66"></iframe></div>
## 安装nginx

```
[root@localhost ~]# yum install nginx -y 

[root@localhost ~]# /usr/sbin/nginx -v 

nginx version: nginx/1.12.2 
```

检查elk是否均已启动:

elasticsearch

```
[root@localhost ~]# ps -ef | grep elasticsearch 
elk        2728      1  1 5月31 pts/0   01:07:32 /bin/java -Xms1g -Xmx1g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -XX:+AlwaysPreTouch -Xss1m -Des.distribution.flavor=default -Des.distribution.type=tar -cp /usr/local/elasticsearch-6.6.2/lib/* org.elasticsearch.bootstrap.Elasticsearch elk        2824   2728  0 5月31 pts/0   00:00:00 /usr/local/elasticsearch-6.6.2/modules/x-pack-ml/platform/linux-x86_64/bin/controller 
root      19874  13681  0 21:16 pts/1    00:00:00 grep --color=auto elasticsearch 
```

kibana

```
[root@localhost ~]# ps -ef | grep kibana 
root       2651      1  0 5月31 pts/0   00:55:12 /usr/local/kibana-6.6.2-linux-x86_64/bin/../node/bin/node --no-warnings --max-http-header-size=65536 /usr/local/kibana-6.6.2-linux-x86_64/bin/../src/cli root      19970  13681  0 21:17 pts/1    00:00:00 grep --color=auto kibana 
root      35609   2651  0 6月01 pts/0   00:00:06 /usr/local/kibana-6.6.2-linux-x86_64/node/bin/node --no-warnings --max-http-header-size=65536 /usr/local/kibana-6.6.2-linux-x86_64/src/legacy/core_plugins/interpreter/server/lib/route_expression/thread/babeled.js 
```

kibana

```
[root@localhost ~]# ps -ef | grep logstash 
root      17996      1  8 20:53 pts/3    00:02:04 /bin/java -Xms1g -Xmx1g -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djruby.compile.invokedynamic=true -Djruby.jit.threshold=0 -XX:+HeapDumpOnOutOfMemoryError -Djava.security.egd=file:/dev/urandom -cp /usr/local/logstash-6.6.2/logstash-core/lib/jars/animal-sniffer-annotations-1.14.jar:/usr/local/logstas-6.6.2/logstash-core/lib/jars/commons-codec-1.11.jar:/usr/local/logstash-6.6.2/logstash-3.10.0.jar:/usr/local/logstash-6.6.2/logstash-core/lib/jars/org.eclipse.osgi-3.7.1.jar:/usr/local/logstash-6.6.2/logstash-core/lib/jars/org.eclipse.text-3.5.101.jar:/usr/local/logstash-6.6.2/logstash-core/lib/jars/slf4j-api-1.7.25.jar org.logstash.Logstash -f /usr/local/logstash-6.6.2/config/logstash.conf 
root      20014  13681  0 21:18 pts/1    00:00:00 grep --color=auto logstash 
```

然后去修改logstash的配置文件,设置为nginx日志文件

## GROK插件

修改配置文件之前需要了解一些logstash的插件grok,我们这里使用logstash-filter-grok插件去匹配日志信息

grok插件:grok插件是logstash中非常强大的插件，其中内置了许多的正则表达式,用来正则匹配各种数据，但其性能和对资源的损耗也是让人为之诟病。

首先看一下nginx输入日志格式

```
[root@localhost ~]# cat /etc/nginx/nginx.conf 
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
```

在以下路径下存在一个httpd的正则匹配文件,与之nginx输入日志格式相符

```
[root@localhost patterns]# pwd 
/usr/local/logstash-6.6.2/vendor/bundle/jruby/2.3.0/gems/logstash-patterns-core-4.1.2/patterns 

[root@localhost patterns]# cat httpd 
HTTPDUSER %{EMAILADDRESS}|%{USER}
HTTPDERROR_DATE %{DAY} %{MONTH} %{MONTHDAY} %{TIME} %{YEAR}

# Log formats

HTTPD_COMMONLOG %{IPORHOST:clientip} %{HTTPDUSER:ident} %{HTTPDUSER:auth} \[%{HTTPDATE:timestamp}\] "(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})" %{NUMBER:response} (?:%{NUMBER:bytes}|-)
HTTPD_COMBINEDLOG %{HTTPD_COMMONLOG} %{QS:referrer} %{QS:agent}
```

## 修改配置文件

修改logstash的配置文件,使用来匹配HTTPD_COMMONLOG格式去匹配日志数据

另外也需要date插件来从字段中解析日期，然后用这个日期作为logstash中事件的时间戳（timestamp）。

```
[root@localhost ~]# vim /usr/local/logstash-6.6.2/config/logstash.conf 

#填写以下信息
input {
    file {
        path => "/var/log/nginx/access.log"   #填写nginx日志文件路径
        type => "nginxaccess"                  #设置类型,名字容易辨识就好
        start_position => "beginning"          #设置开始位置
    }
}

filter {
    grok {  
      match => { "message" => "%{HTTPD_COMBINEDLOG}" } #匹配HTTPD_COMBINEDLOG信息
    }
     date {
      match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"] #匹配tomestamp
      target => "@timestamp"     #覆盖@timestamp字段
      }
}

output {                                     #输出到elasticsearch中
    elasticsearch {
        hosts => ["127.0.0.1:9200"]
        index => "nginx-%{+YYYY.MM.dd}"         #设置索引格式
    }
}
```

然后运行logstash

```
[root@localhost logstash-6.6.2]# ./startup.sh  
```

## 访问kibana

在浏览器访问kibana界面,可以看到刚刚所添加的nginx日志索引

![](https://ae01.alicdn.com/kf/HTB1trPZbrus3KVjSZKb760qkFXaL.png)

然后去建一个pattern,成功匹配到刚刚创建的索引

![](https://ae01.alicdn.com/kf/HTB1CO6Obv1H3KVjSZFH762KppXaL.png)

选时间戳,创建pattern

![](https://ae01.alicdn.com/kf/HTB1PLDSbCWD3KVjSZSg763CxVXak.png)

然后在Discover模块选nginx-*

![](https://ae01.alicdn.com/kf/HTB1i2TQbv1G3KVjSZFk761K4XXap.png)

便可以查看到nginx的日志啦

![](https://ae01.alicdn.com/kf/HTB1i8jPbEKF3KVjSZFE760ExFXat.png)

查看客户端ip

![](https://ae01.alicdn.com/kf/HTB1NwfRbBGE3KVjSZFh763kaFXaT.png)

查看请求信息

![](https://ae01.alicdn.com/kf/HTB1tELPbEWF3KVjSZPh760clXXaG.png)