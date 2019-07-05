---
title: 	Nginx+ModSecurity实现WAF防护(转载)
date: 2019-07-05 22:36:28
categories: 运维技术
tags: 开源waf

---
<blockquote class="blockquote-center">生活的理想，就是为了理想的生活。</blockquote>
<div class="aplayer" data-id="25706282" data-server="netease" data-type="song" data-mode="single"></div>
## ModSecurity简介

ModSecurity是一个入侵侦测与防护引擎，它主要是用于Web应用程序，所以也被称为Web应用程序防火墙(WAF)。它可以作为Web服务器的模块或是单独的应用程序来运作。ModSecurity的功能是增强Web Application 的安全性和保护Web application以避免遭受来自已知与未知的攻击。

ModSecurity计划是从2002年开始，后来由Breach Security Inc.收购，但Breach Security Inc.允诺ModSecurity仍旧为Open Source，并开放源代码给大家使用。最新版的ModSecurity开始支持核心规则集(Core Rule Set)，CRS可用于定义旨在保护Web应用免受0day及其它安全攻击的规则。

ModSecurity还包含了其他一些特性，如并行文本匹配、Geo IP解析和信用卡号检测等，同时还支持内容注入、自动化的规则更新和脚本等内容。此外，它还提供了一个面向Lua语言的新的API，为开发者提供一个脚本平台以实现用于保护Web应用的复杂逻辑。

## 安装ModSecurity

安装依赖包:

```
yum install httpd-devel apr apr-util-devel apr-devel  pcre pcre-devel  libxml2 libxml2-devel zlib zlib-devel openssl openssl-devel
```

下载nginx和modsearity

```
[root@localhost ~]# cd /opt/
#下载modsecurity
[root@localhost opt]# wget -O modsecurity-2.9.1.tar.gz https://github.com/SpiderLabs/ModSecurity/releases/download/v2.9.1/modsecurity-2.9.1.tar.gz
#下载nginx
[root@localhost opt]# wget 'http://nginx.org/download/nginx-1.9.2.tar.gz'
```

## 编译安装ModSecurity

ginx加载ModSecurity模块有两种方式:一种是编译为Nginx静态模块，一种是通过ModSecurity-Nginx Connector加载动态模块。

### 方法一：编译为Nginx静态模块

- 编译为独立模块(modsecurity-2.9.1)

```
[root@localhost opt]# tar zxvf  modsecurity-2.9.1.tar.gz
[root@localhost opt]# cd modsecurity-2.9.1/
[root@localhost modsecurity-2.9.1]# ./autogen.sh
$ [root@localhost modsecurity-2.9.1]# ./configure --enable-standalone-module --disable-mlogc
[root@localhost modsecurity-2.9.1]# make
```

注:如果在运行./autogen.sh的时候系统报错,提示

```
[root@localhost modsecurity-2.9.1]# ./autogen.sh
#如果出现以下情况
./autogen.sh:行11: libtoolize: 未找到命令
./autogen.sh:行12: autoreconf: 未找到命令
./autogen.sh:行13: autoheader: 未找到命令
./autogen.sh:行14: automake: 未找到命令
./autogen.sh:行15: autoconf: 未找到命令
#表示系统未安装这些软件包,我们用yum安装即可
[root@localhost modsecurity-2.9.1]# yum install automake autoconf libtool 
```

- 编译安装Nginx并添加ModSecurity模块

```
[root@localhost opt]# tar xzvf nginx-1.9.2.tar.gz

[root@localhost opt]# cd nginx-1.9.2
[root@localhost nginx-1.9.2]# ./configure --add-module=/opt/modsecurity-2.9.1/nginx/modsecurity
[root@localhost nginx-1.9.2]# make && make install
```

### 方法二：编译通过ModSecurity-Nginx Connector加载的动态模块

- 编译LibModSecurity(modsecurity-3.0)

```
$ cd /root$ git clone https://github.com/SpiderLabs/ModSecurity
$ cd ModSecurity
$ git checkout -b v3/master origin/v3/master
$ sh build.sh$ git submodule init
$ git submodule update
$ ./configure
$ make
$ make install
```

LibModSecurity会安装在`/usr/local/modsecurity/lib`目录下。

```
$ ls /usr/local/modsecurity/lib
libmodsecurity.a  libmodsecurity.la  libmodsecurity.so  libmodsecurity.so.3  libmodsecurity.so.3.0.0
```

- 编译安装Nginx并添加ModSecurity-Nginx Connector模块

使用ModSecurity-Nginx模块来连接LibModSecurity

```
$ cd /opt
$ git clone https://github.com/SpiderLabs/ModSecurity-nginx.git modsecurity-nginx
$ tar xzvf nginx-1.9.2.tar.gz
$ cd nginx-1.9.2$ ./configure --add-module=/root/modsecurity-nginx
$ make$ make && make install
```

## 添加OWASP规则

### OWASP CRS

OWASP ModSecurity核心规则集（CRS）是一组用于ModSecurity或兼容的Web应用程序防火墙的通用攻击检测规则。CRS旨在保护Web应用程序免受各种攻击，包括OWASP十大攻击，并且只需最少的虚假警报。ModSecurity之所以强大就在于OWASP提供的规则，我们可以根据自己的需求选择不同的规则，也可以通过ModSecurity手工创建安全过滤器、定义攻击并实现主动的安全输入验证。

ModSecurity核心规则集(CRS)提供以下类别的保护来防止攻击。

- HTTP Protection(HTTP防御)

HTTP协议和本地定义使用的detects violations策略。

- Real-time Blacklist Lookups(实时黑名单查询)

利用第三方IP名单。

- HTTP Denial of Service Protections(HTTP的拒绝服务保护)

防御HTTP的洪水攻击和HTTP Dos攻击。

- Common Web Attacks Protection(常见的Web攻击防护)

检测常见的Web应用程序的安全攻击。

- Automation Detection(自动化检测)

检测机器人，爬虫，扫描仪和其他表面恶意活动。

- Integration with AV Scanning for File Uploads(文件上传防病毒扫描)

检测通过Web应用程序上传的恶意文件。

- Tracking Sensitive Data(跟踪敏感数据)

信用卡通道的使用，并阻止泄漏。

- Trojan Protection(木马防护)

检测访问木马。

- Identification of Application Defects(应用程序缺陷的鉴定)

检测应用程序的错误配置警报。

- Error Detection and Hiding(错误检测和隐藏)

检测伪装服务器发送错误消息。

### 下载OWASP规则并生成配置文件

```
[root@localhost opt]# git clone https://github.com/SpiderLabs/owasp-modsecurity-crs.git
[root@localhost opt]# cp -rf owasp-modsecurity-crs /usr/local/nginx/conf/
[root@localhost opt]# cd /usr/local/nginx/conf/owasp-modsecurity-crs/
[root@localhost owasp-modsecurity-crs]# cp crs-setup.conf.example crs-setup.conf
```

### 配置OWASP规则

编辑crs-setup.conf文件

```
$ sed -ie 's/SecDefaultAction "phase:1,log,auditlog,pass"/#SecDefaultAction "phase:1,log,auditlog,pass"/g' crs-setup.conf
$ sed -ie 's/SecDefaultAction "phase:2,log,auditlog,pass"/#SecDefaultAction "phase:2,log,auditlog,pass"/g' crs-setup.conf
$ sed -ie 's/#.*SecDefaultAction "phase:1,log,auditlog,deny,status:403"/SecDefaultAction "phase:1,log,auditlog,deny,status:403"/g' crs-setup.conf
$ sed -ie 's/# SecDefaultAction "phase:2,log,auditlog,deny,status:403"/SecDefaultAction "phase:2,log,auditlog,deny,status:403"/g' crs-setup.conf

```

默认ModSecurity不会阻挡恶意连接，只会记录在Log里。修改SecDefaultAction选项，默认开启阻挡。

### 启用ModSecurity模块和CRS规则

复制ModSecurity源码目录下的modsecurity.conf-recommended和unicode.mapping到Nginx的conf目录下，并将modsecurity.conf-recommended重新命名为modsecurity.conf。

modsecurity.conf-recommended是ModSecurity工作的主配置文件。默认情况下，它带有.recommended扩展名。要初始化ModSecurity，我们就要重命名此文件。

```
[root@localhost owasp-modsecurity-crs]# cd /opt/modsecurity-2.9.1/
[root@localhost modsecurity-2.9.1]# cp modsecurity.conf-recommended /usr/local/nginx/conf/modsecurity.conf
[root@localhost modsecurity-2.9.1]# cp unicode.mapping /usr/local/nginx/conf/
```

将SecRuleEngine设置为On，默认值为DetectOnly即为观察模式，建议大家在安装时先默认使用这个模式，规则测试完成后在设置为On，避免出现对网站、服务器某些不可知的影响。

```
[root@localhost modsecurity-2.9.1]# vim /usr/local/nginx/conf/modsecurity.conf
									SecRuleEngine On
```

ModSecurity中几个常用配置说明：

> 1.SecRuleEngine：是否接受来自ModSecurity-CRS目录下的所有规则的安全规则引擎。因此，我们可以根据需求设置不同的规则。要设置不同的规则有以下几种。SecRuleEngine On：将在服务器上激活ModSecurity防火墙，它会检测并阻止该服务器上的任何恶意攻击。SecRuleEngine Detection Only：如果设置这个规则它只会检测到所有的攻击，并根据攻击产生错误，但它不会在服务器上阻止任何东西。SecRuleEngine Off:这将在服务器上上停用ModSecurity的防火墙。
>
> 2.SecRequestBodyAccess：它会告诉ModSecurity是否会检查请求，它起着非常重要的作用。它只有两个参数ON或OFF。
>
> 3.SecResponseBodyAccess：如果此参数设置为ON，然后ModeSecurity可以分析服务器响应，并做适当处理。它也有只有两个参数ON和Off，我们可以根据求要进行设置。
>
> 4.SecDataDir：定义ModSecurity的工作目录，该目录将作为ModSecurity的临时目录使用。

在`owasp-modsecurity-crs/rules`下有很多定义好的规则，将需要启用的规则用Include指令添加进来就可以了。

- 3.x版本CRS

  ```
  $ cd /usr/local/nginx/conf/owasp-modsecurity-crs
  # 生成例外排除请求的配置文件
  $ cp rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
  $ cp rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf.example rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
  $ cp rules/*.data /usr/local/nginx/conf
  ```

  为了保持modsecurity.conf简洁，这里新建一个modsec_includes.conf文件,内容为需要启用的规则。

  ```
  vim /usr/local/nginx/conf/modsec_includes.conf
  
  include modsecurity.conf
  include owasp-modsecurity-crs/crs-setup.conf
  include owasp-modsecurity-crs/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
  include owasp-modsecurity-crs/rules/REQUEST-901-INITIALIZATION.conf
  Include owasp-modsecurity-crs/rules/REQUEST-903.9002-WORDPRESS-EXCLUSION-RULES.conf
  include owasp-modsecurity-crs/rules/REQUEST-905-COMMON-EXCEPTIONS.conf
  include owasp-modsecurity-crs/rules/REQUEST-910-IP-REPUTATION.conf
  include owasp-modsecurity-crs/rules/REQUEST-911-METHOD-ENFORCEMENT.conf
  include owasp-modsecurity-crs/rules/REQUEST-912-DOS-PROTECTION.conf
  include owasp-modsecurity-crs/rules/REQUEST-913-SCANNER-DETECTION.conf
  include owasp-modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf
  include owasp-modsecurity-crs/rules/REQUEST-921-PROTOCOL-ATTACK.conf
  include owasp-modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf
  include owasp-modsecurity-crs/rules/REQUEST-931-APPLICATION-ATTACK-RFI.conf
  include owasp-modsecurity-crs/rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf
  include owasp-modsecurity-crs/rules/REQUEST-933-APPLICATION-ATTACK-PHP.conf
  include owasp-modsecurity-crs/rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf
  include owasp-modsecurity-crs/rules/REQUEST-942-APPLICATION-ATTACK-SQLI.conf
  include owasp-modsecurity-crs/rules/REQUEST-943-APPLICATION-ATTACK-SESSION-FIXATION.conf
  include owasp-modsecurity-crs/rules/REQUEST-949-BLOCKING-EVALUATION.conf
  include owasp-modsecurity-crs/rules/RESPONSE-950-DATA-LEAKAGES.conf
  include owasp-modsecurity-crs/rules/RESPONSE-951-DATA-LEAKAGES-SQL.conf
  include owasp-modsecurity-crs/rules/RESPONSE-952-DATA-LEAKAGES-JAVA.conf
  include owasp-modsecurity-crs/rules/RESPONSE-953-DATA-LEAKAGES-PHP.conf
  include owasp-modsecurity-crs/rules/RESPONSE-954-DATA-LEAKAGES-IIS.conf
  include owasp-modsecurity-crs/rules/RESPONSE-959-BLOCKING-EVALUATION.conf
  include owasp-modsecurity-crs/rules/RESPONSE-980-CORRELATION.conf
  include owasp-modsecurity-crs/rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
  ```

注：考虑到可能对主机性能上的损耗，可以根据实际需求加入对应的漏洞的防护规则即可。

## 配置Nginx支持Modsecurity

### 启用Modsecurity

- 使用静态模块加载的配置方法

在需要启用Modsecurity的主机的location下面加入下面两行即可：

```
[root@localhost modsecurity-2.9.1]# vim /usr/local/nginx/conf/nginx.conf
 server {
        listen       80;
        server_name  localhost;

        location / {
        	#加入下面两行内容
            ModSecurityEnabled on;    
            ModSecurityConfig modsec_includes.conf;
            root   html;
            index  index.html index.htm;
        }

```

- 使用动态模块加载的配置方法

  在需要启用Modsecurity的主机的location下面加入下面两行即可：

  ```
  [root@localhost modsecurity-2.9.1]# vim /usr/local/nginx/conf/nginx.conf
  server {
          listen       80;
          server_name  localhost;
  
          location / {
          	#加入下面两行内容
              modsecurity on;
  			modsecurity_rules_file modsec_includes.conf;
              root   html;
              index  index.html index.htm;
          }
  
  
  
  ```

  ### 验证Nginx配置文件

  ```
  [root@localhost modsecurity-2.9.1]# /usr/local/nginx/sbin/nginx -t
  nginx: [emerg] ModSecurityConfig in /usr/local/nginx/conf/nginx.conf:45: Cannot open config file: /usr/local/nginx/conf/owasp-modsecurity-crs/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
  nginx: configuration file /usr/local/nginx/conf/nginx.conf test failed
  ```

  发现没有REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf文件,然后去其目录发现存在着REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf,所以将其修改名称即可,再次测试,发现RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf也是没有的,使用相同的方法修改

  ```
  [root@localhost modsecurity-2.9.1]# cd /usr/local/nginx/conf/owasp-modsecurity-crs/rules/
  
  [root@localhost rules]# cp REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
  
  [root@localhost rules]# cp RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf.example RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
  
  ```

  最后测试成功

  ```
  [root@localhost rules]# /usr/local/nginx/sbin/nginx -t
  nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
  nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
  ```

  - 启动Nginx

    ```
    [root@localhost rules]# /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
    ```

    ### 测试Modsecurity

    正常访问时ok的

    ![](https://ae01.alicdn.com/kf/HTB1CZqkXoT1gK0jSZFr763NCXXaF.png)

那么我们尝试构造一些注入参数进去试试

![](https://ae01.alicdn.com/kf/HTB1IZSkXoY1gK0jSZFM761WcVXa6.png)

![](https://ae01.alicdn.com/kf/HTB1x9CjXfb2gK0jSZK9761EgFXa4.png)

其日志保存在`/var/log/modsec_audit.log`

```
[root@localhost log]# tail -n 20  modsec_audit.log 
Connection: keep-alive

--9621e831-H--
Message: Warning. Pattern match "^[\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/local/nginx/conf/owasp-modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "682"] [id "920350"] [msg "Host header is a numeric IP address"] [data "192.168.204.129"] [severity "WARNING"] [ver "OWASP_CRS/3.1.0"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "OWASP_CRS/PROTOCOL_VIOLATION/IP_HOST"] [tag "WASCTC/WASC-21"] [tag "OWASP_TOP_10/A7"] [tag "PCI/6.5.10"]
Message: Warning. detected XSS using libinjection. [file "/usr/local/nginx/conf/owasp-modsecurity-crs/rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf"] [line "58"] [id "941100"] [msg "XSS Attack Detected via libinjection"] [data "Matched Data: XSS data found within ARGS:search: <script>alert(/xss/)</scrity>"] [severity "CRITICAL"] [ver "OWASP_CRS/3.1.0"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-xss"] [tag "OWASP_CRS/WEB_ATTACK/XSS"] [tag "WASCTC/WASC-8"] [tag "WASCTC/WASC-22"] [tag "OWASP_TOP_10/A3"] [tag "OWASP_AppSensor/IE1"] [tag "CAPEC-242"]
Message: Warning. Pattern match "(?i)[<\xef\xbc\x9c]script[^>\xef\xbc\x9e]*[>\xef\xbc\x9e][\\s\\S]*?" at ARGS:search. [file "/usr/local/nginx/conf/owasp-modsecurity-crs/rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf"] [line "88"] [id "941110"] [msg "XSS Filter - Category 1: Script Tag Vector"] [data "Matched Data: <script> found within ARGS:search: <script>alert(/xss/)</scrity>"] [severity "CRITICAL"] [ver "OWASP_CRS/3.1.0"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-xss"] [tag "OWASP_CRS/WEB_ATTACK/XSS"] [tag "WASCTC/WASC-8"] [tag "WASCTC/WASC-22"] [tag "OWASP_TOP_10/A3"] [tag "OWASP_AppSensor/IE1"] [tag "CAPEC-242"]
Message: Warning. Pattern match "(?i)<[^\\w<>]*(?:[^<>\"'\\s]*:)?[^\\w<>]*(?:\\W*?s\\W*?c\\W*?r\\W*?i\\W*?p\\W*?t|\\W*?f\\W*?o\\W*?r\\W*?m|\\W*?s\\W*?t\\W*?y\\W*?l\\W*?e|\\W*?s\\W*?v\\W*?g|\\W*?m\\W*?a\\W*?r\\W*?q\\W*?u\\W*?e\\W*?e|(?:\\W*?l\\W*?i\\W*?n\\W*?k|\\W*?o\\W*?b\\W*?j\\W*?e\ ..." at ARGS:search. [file "/usr/local/nginx/conf/owasp-modsecurity-crs/rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf"] [line "207"] [id "941160"] [msg "NoScript XSS InjectionChecker: HTML Injection"] [data "Matched Data: <script found within ARGS:search: <script>alert(/xss/)</scrity>"] [severity "CRITICAL"] [ver "OWASP_CRS/3.1.0"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-xss"] [tag "OWASP_CRS/WEB_ATTACK/XSS"] [tag "WASCTC/WASC-8"] [tag "WASCTC/WASC-22"] [tag "OWASP_TOP_10/A3"] [tag "OWASP_AppSensor/IE1"] [tag "CAPEC-242"]
Message: Access denied with code 403 (phase 2). Operator GE matched 5 at TX:anomaly_score. [file "/usr/local/nginx/conf/owasp-modsecurity-crs/rules/REQUEST-949-BLOCKING-EVALUATION.conf"] [line "91"] [id "949110"] [msg "Inbound Anomaly Score Exceeded (Total Score: 18)"] [severity "CRITICAL"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-generic"]
Message: Warning. Operator GE matched 5 at TX:inbound_anomaly_score. [file "/usr/local/nginx/conf/owasp-modsecurity-crs/rules/RESPONSE-980-CORRELATION.conf"] [line "86"] [id "980130"] [msg "Inbound Anomaly Score Exceeded (Total Inbound Score: 18 - SQLI=0,XSS=15,RFI=0,LFI=0,RCE=0,PHPI=0,HTTP=0,SESS=0): individual paranoia level scores: 18, 0, 0, 0"] [tag "event-correlation"]
Message: Audit log: Failed to lock global mutex: Permission denied
Action: Intercepted (phase 2)
Apache-Handler: IIS
Stopwatch: 1562335360000217 222381 (- - -)
Stopwatch2: 1562335360000217 222381; combined=3851, p1=333, p2=3340, p3=0, p4=0, p5=178, sr=8, sw=0, l=0, gc=0
Producer: ModSecurity for nginx (STABLE)/2.9.1 (http://www.modsecurity.org/); OWASP_CRS/3.1.0.
Server: ModSecurity Standalone
Engine-Mode: "ENABLED"

--9621e831-Z--

```

可以查看到其匹配了REQUEST-920-PROTOCOL-ENFORCEMENT.conf和REQUEST-941-APPLICATION-ATTACK-XSS.conf文件中的规则.

## 参考文档:

https://www.hi-linux.com/posts/45920.html