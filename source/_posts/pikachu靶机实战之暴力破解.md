---
title: pikachu靶机实战之暴力破解
date: 2019-01-02 22:38:28
categories: WEB安全
tags: 靶机实验

---
<blockquote class="blockquote-center">态度决定高度!</blockquote>
<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=5181411&auto=1&height=66"></iframe></div>

## 靶机简介
**Pikachu是一个带有漏洞的Web应用系统，在这里包含了常见的web安全漏洞。 是个适合新手练习的靶场**
Pikachu上的漏洞类型列表如下：
Burt Force(暴力破解漏洞)
XSS(跨站脚本漏洞)
CSRF(跨站请求伪造)
SQL-Inject(SQL注入漏洞)
RCE(远程命令/代码执行)
Files Inclusion(文件包含漏洞)
Unsafe file downloads(不安全的文件下载)
Unsafe file uploads(不安全的文件上传)
Over Permisson(越权漏洞)
../../../(目录遍历)
I can see your ABC(敏感信息泄露)
PHP反序列化漏洞
XXE(XML External Entity attack)
不安全的URL重定向
SSRF(Server-Side Request Forgery)
More...(找找看?..有彩蛋!)
管理工具里面提供了一个简易的xss管理后台,供你测试钓鱼和捞cookie~

## 安装和使用
Pikachu使用世界上最好的语言PHP进行开发-_-，数据库使用的是mysql，因此运行Pikachu你需要提前安装好"PHP+MYSQL+中间件（如apache,nginx等）"的基础环境，建议在你的测试环境直接使用 一些集成软件来搭建这些基础环境,比如XAMPP,WAMP等,作为一个搞安全的人,这些东西对你来说应该不是什么难事。接下来:
-->把下载下来的pikachu文件夹放到web服务器根目录下;
-->根据实际情况修改inc/config.inc.php里面的数据库连接配置;
-->访问 http://x.x.x.x/pikachu ,会有一个红色的热情提示"欢迎使用,pikachu还没有初始化，点击进行初始化安装!",点击即可完成安装。

## 暴力破解实验
## Burte Force（暴力破解）概述
> 暴力破解”是一攻击具手段，在web攻击中，一般会使用这种手段对应用系统的认证信息进行获取。 其过程就是使用大量的认证信息在认证接口进行尝试登录，直到得到正确的结果。 为了提高效率，暴力破解一般会使用带有字典的工具来进行自动化操作。
> 理论上来说，大多数系统都是可以被暴力破解的，只要攻击者有足够强大的计算能力和时间，所以断定一个系统是否存在暴力破解漏洞，其条件也不是绝对的。 我们说一个web应用系统存在暴力破解漏洞，一般是指该web应用系统没有采用或者采用了比较弱的认证安全策略，导致其被暴力破解的“可能性”变的比较高。 这里的认证安全策略, 包括：
> 1.是否要求用户设置复杂的密码；
> 2.是否每次认证都使用安全的验证码（想想你买火车票时输的验证码～）或者手机otp；
> 3.是否对尝试登录的行为进行判断和限制（如：连续5次错误登录，进行账号锁定或IP地址锁定等）；
> 4.是否采用了双因素认证；
> ...等等。

### 基于表单的暴力破解
我们使用burpsuite进行暴力破解,由于前端没有验证码等防范暴力破解的措施,我们直接输入用户名密码,发送到burpsuite的intruder模块,一般用户名为admin,administrator(Windows环境)或者root(Linux环境),我设置admin为用户名,对其密码进行爆破
![image](https://wx2.sinaimg.cn/large/0078beR7ly1fysptku9v3j30yx0erdgt.jpg)
![image](https://ws3.sinaimg.cn/large/0078beR7ly1fyspu685cvj30h707ewen.jpg)
![image](https://ws3.sinaimg.cn/large/0078beR7ly1fyspuumb95j30ki0c20tb.jpg)

### 不安全的验证码-on client常见问题
#### 验证码作用:
1,防止暴力破解
2,防止机器恶意注册

#### 验证码的认证流程:
客户端request登录页面,后台生成验证码
1,后台使用算法生成图片,并将图片response给客户端
2,同时将算法生成的值全局赋值存到session中.

#### 校验验证码:
1,客户端将认证信息和验证码一同提交
2,后台对提交的验证码和session里面的进行比较

客户端重新刷新页面,再次生出新的验证码
验证码算法中一般包含随机函数,所以每次刷新都会改变

#### 不安全的客户端验证码常见问题:
1,使用前端js实现验证码(纸老虎)
2,将验证码在cookie中泄露,容易被获取
3,将验证码在前端源代码中泄露,容易被获取

#### 开始试验:
首先尝试输入错误的用户名,密码+错误的验证码,点击登录页面返回验证码不正确
然后输入错误的用户名,密码+正确的验证码,点击登录页面返回用户名或者密码不正确
当输入错误的用户名,密码+空验证码,点击登录页面提示验证码不能为空,表示服务端对验证码的有效性做过校验,一切逻辑正常
当查看源代码的时候发现是前台生出的验证码
![image](https://ws2.sinaimg.cn/large/0078beR7ly1fyspzfud27j31g40jxdi7.jpg)
如果后台不对前台输入的验证码进行校验的话,那么通过burp代理(客户端和服务端中间人)即可绕过验证码
我们使用burp抓包看看是否对输入的验证码进行校验,结果是用户名或者密码不存在
![image](https://ws4.sinaimg.cn/large/0078beR7ly1fysq0e8w59j30yz0f9wft.jpg)
然后换个账号密码继续发包,判断服务器端是否对用户前端输入的验证码进行校验
![image](https://wx1.sinaimg.cn/large/0078beR7ly1fysq1dpfgtj30yb0ecgn2.jpg)
还是用户名密码不正确,但并未返回验证码不正确

我们都知道当用户输入账号密码和验证码之后,服务器端首先验证验证码是否正确,如果不正确直接返回验证码不正确,如果验证码正确,那么服务器端会接着验证用户名密码是否正确.我们刚刚的结果是用户名或者密码不存在,表示验证码验证那一关我们是完美的避过了,然后再爆破用户名密码即可
爆破出用户名为:
admin/123456
pikachu/000000
test/abc123



### 不安全的验证码-on server常见问题
#### 不安全的验证码-on server常见问题
1,验证码在后台不过期,导致可以长期被使用
2,验证码校验不严格,逻辑出现问题
3,验证码设计的太过简单和有规律,容易被猜解

针对于第一个验证码在后台不过期的漏洞,开始实验
首先尝试输入错误的用户名,密码+错误的验证码,点击登录burp抓返回包页面返回验证码不正确
然后输入错误的用户名,密码+正确的验证码,点击登录burp抓返回包页面返回用户名或者密码不正确
当输入错误的用户名,密码+空验证码,点击登录burp抓返回包页面提示验证码不能为空,表示服务端对验证码的有效性做过校验,一切逻辑正常

当刷新页面,客户端向服务器发出请求,生出新的验证码,同时后台会在session中将这个验证码存下来(存下来的目的是为了对用户输入的验证码进行验证),所以当输入错误的验证码或者空的验证码的时候都会提示验证码错误,只有正确的验证码才可以被服务器接受

但是如果这个验证码在后台不过期或者过期时间较长,足够我们去爆破用户名密码,那么漏洞就产生了.
1,首先先正常提交用户名密码验证码,然后发送到repeater模块中
2,关闭burp代理功能,刷新页面,会生出新的验证码,记住新的验证码
![image](https://wx4.sinaimg.cn/large/0078beR7ly1fysq86aafsj30mp0cfmxu.jpg)
3,在repeater模块中将新的验证码写入,重放发现其提示是用户名密码错误
![image](https://wx1.sinaimg.cn/large/0078beR7ly1fysq9fdt4xj30vb0cudgt.jpg)
4,将账户名密码替换,试试验证码还有没有效
![image](https://ws3.sinaimg.cn/large/0078beR7ly1fysqa9u7bxj30w50dq0ts.jpg)
5,因为无论怎么替换用户名和密码,验证码都正确,所以那么这一关我们是完美的避过了,然后再爆破用户名密码即可

#### 漏洞分析:
其漏洞根本在于服务器端未设定生出验证码的session的过期时间,那么按照PHP语言默认session的过期时间为24分钟,这个验证码24分钟内都是有效的,那么也足够黑客进行暴力破解啦

#### 修复方法:
法一,在php.ini配置文件中设置过期时间
法二,在代码中设定该验证码验证过一次之后,就将其session进行销毁(更有效)

## token防止暴力破解?
曾经网上有人说可以使用token防止暴力破解,其原理就是当用户打开页面时,后端生出一个token值,token会被存放到session中去,同时服务端会将token发送到前端的表单中,当用户输入账号密码点击确认的时候,客户端会将账号密码+token一起发送到服务器端,当刷新页面之后,token即就会变化

但是token会被显示在前端的表单中,黑客完全可以通过代码获取表单的token,然后配合暴力破解即可
![image](https://ws3.sinaimg.cn/large/0078beR7ly1fysqimk67fj310j0idmy3.jpg)