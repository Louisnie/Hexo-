---
title: w3af--命令行模式实验简述
date: 2018-10-23 23:31:30
categories: kali
tags: tools

---
<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=5181411&auto=1&height=66"></iframe></div>

## ﻿实验环境:

Kali:192.168.128.128
Metasploitable:192.168.128.129

## 安装W3af:[简介及安装w3af文档](https://blog.csdn.net/qq_39353923/article/details/82557316) 

## w3af用户接口:
1. ​	console命令行接口
1. ​	Gui图形界面化接口
1. ​	API接口

## 开始操作

```
root@kali:~/w3af-master# ./w3af_console 
w3af>>> 
w3af>>> help   #help命令列出当前命令提示符下的可用指令
|-----------------------------------------------------------------------------------|
| start         | Start the scan.  开始扫描                                                 |
| plugins       | Enable and configure plugins.   选择和配置插件                                 |
| exploit       | Exploit the vulnerability. 使用该模块进行攻击漏洞                                       |
| profiles      | List and use scan profiles. 列出可以用来扫描的文件                                      |
| cleanup       | Cleanup before starting a new scan.  在开始新扫描之前进行清理                             |
|-----------------------------------------------------------------------------------|
| help          | Display help. Issuing: help [command] , prints more specific help |
|               | about "command"                                                   |
| version       | Show w3af version information.    显示w3af版本信息                                |
| keys          | Display key shortcuts.  显示关键快捷方式。                                          |
|-----------------------------------------------------------------------------------|
| http-settings | Configure the HTTP settings of the framework.      配置框架的HTTP设置。               |
| misc-settings | Configure w3af misc settings.  配置w3af misc设置                                   |
| target        | Configure the target URL.  配置目标URL                                       |
|-----------------------------------------------------------------------------------|
| back          | Go to the previous menu.    返回前一目录                                      |
| exit          | Exit w3af.           退出                                             |
|-----------------------------------------------------------------------------------|
| kb            | Browse the vulnerabilities stored in the Knowledge Base     浏览存储在知识库中的漏洞      |
|-----------------------------------------------------------------------------------
```

```
w3af>>> plugins  #输入plugins,进入插件目录的内,
w3af/plugins>>> help #输入help显示当前可使用的命令
|-----------------------------------------------------------------------------------|
| list              | List available plugins.                                       |
|-----------------------------------------------------------------------------------|
| back              | Go to the previous menu.                                      |
| exit              | Exit w3af.                                                    |
|-----------------------------------------------------------------------------------|
| output            | View, configure and enable output plugins                     |
| grep              | View, configure and enable grep plugins                       |
| evasion           | View, configure and enable evasion plugins                    |
| audit             | View, configure and enable audit plugins                      |
| infrastructure    | View, configure and enable infrastructure plugins             |
| crawl             | View, configure and enable crawl plugins                      |
| auth              | View, configure and enable auth plugins                       |
| mangle            | View, configure and enable mangle plugins                     |
| bruteforce        | View, configure and enable bruteforce plugins                 |
|-----------------------------------------------------------------------------------|
w3af/plugins>>> list audit 或者audit #列出audit插件类中的小插件
```
也可以在图形界面化直观的看清其结构
 
![](1.png)
``` 
#使用audit模块中的xss脚本攻击,sql注入,本地文件调用这三个插件,可以选用一个,也可以选用多个
#那么再次列出时这些插件的Status将会变成Enabled,如果进行扫描,那么就会针对这些漏洞去扫描
w3af/plugins>>> audit xss sqli lfi 

#使用audit模块中的所有插件进行扫描
w3af/plugins>>> audit all

#输入crawl模块,按两次tab键,可以显示该模块下的插件
w3af/plugins>>> crawl 
genexus_xml wordpress_fingerprint dot_listing content_negotiation robots_txt archive_dot_org ria_enumerator wordnet user_dir sitemap_xml bing_spider dir_file_bruter phpinfo find_dvcs import_results urllist_txt google_spider url_fuzzer find_backdoors web_spider spider_man find_captchas oracle_discovery wsdl_finder wordpress_enumerate_users web_diff dwsync_xml pykto wordpress_fullpathdisclosure phishtank digit_sum open_api dot_ds_store ghdb all config desc 
w3af/plugins>>> crawl web_spider  #选择该模块下的web爬虫模块

```

```
w3af>>> profiles  #进入profiles模块,这个模块用于自定义组合插件,当然w3af自定义了一些组合插件
w3af/profiles>>> help
|-----------------------------------------------------------------------------------|
| use        | Use a profile.                                                       |
| list       | List available profiles.                                             |
| save_as    | Save the current configuration to a profile.                         |
|-----------------------------------------------------------------------------------|
| back       | Go to the previous menu.                                             |
| exit       | Exit w3af.                                                           |
|------------------------------------------------------------------------------
w3af/profiles>>> list  #列出在profiles模块下的插件

w3af/profiles>>> help
|-----------------------------------------------------------------------------------|
| use        | Use a profile.                                                       |
| list       | List available profiles.                                             |
| save_as    | Save the current configuration to a profile. #保存当前配置到一个文件内                        |
|-----------------------------------------------------------------------------------|
| back       | Go to the previous menu.                                             |
| exit       | Exit w3af.                                                           |
|-----------------------------------------------------------------------------------|


w3af/profiles>>> save_as test   #使用save_as后面自定义一个文件名,表示将刚刚的配置存放在test文件内
Profile saved.

#使用下列命令将test文件独立出来,以便供其他人使用
w3af/profiles>>> save_as test self-contained

w3af/profiles>>> use test      #使用自定义的test文件中的配置进行扫描
The plugins configured by the scan profile have been enabled, and their options configured.
Please set the target URL(s) and start the scan.

w3af/profiles>>> back #返回上一级


w3af>>> http-settings  #进入http-settings,设置全局参数
w3af/config:http-settings>>> help
|-----------------------------------------------------------------------------------|
| view   | List the available options and their values.                             |
| set    | Set a parameter value.          #设置参数值                                         |
| save   | Save the configured settings.          #保存配置                                  |
|-----------------------------------------------------------------------------------|
| back   | Go to the previous menu.       #返回上一目录                                          |
| exit   | Exit w3af.                                                               |
|-----------------------------------------------------------------------------------|
w3af/config:http-settings>>> view  #列出可用的操作和其值
|---------------------------------------------------------------------------------

#设置随机用户代理浏览器,默认位w3af的代理,容易被管理员查看日志发现
w3af/config:http-settings>>> set rand_user_agent True 

w3af/config:http-settings>>> back
The configuration has been saved.
w3af>>> misc-settings  #进入misc-setting全局设置选项中
w3af/config:misc-settings>>> view  #查看需要配置的参数
w3af/config:misc-settings>>> set fuzz_url_filenames True  #设置对URL中的文件名进行模糊测试(Fuzz)

w3af/config:misc-settings>>> back
The configuration has been saved.
w3af>>> target  #进入target模块,设置目标信息
w3af/config:target>>> help  #查看在此模块下可以使用的命令
|-----------------------------------------------------------------------------------|
| view   | List the available options and their values.                             |
| set    | Set a parameter value.                                                   |
| save   | Save the configured settings.                                            |
|-----------------------------------------------------------------------------------|
| back   | Go to the previous menu.                                                 |
| exit   | Exit w3af.                                                               |
|-----------------------------------------------------------------------------
w3af/config:target>>> view  #列出可用的操作
|----------------------------------------------------------------------------------|
| Setting        | Value | Modified | Description                                      |
|----------------------------------------------------------------------------------|
| target_framework | unknown |       | Target programming framework                     |
|                |      |       | (unknown/php/asp/asp.net/java/jsp/cfm/ruby/perl) |
| target         |      |       | A comma separated list of URLs                   |
| target_os      | unknown |       | Target operating system (unknown/unix/windows)   |
|-----------------------------------------------------------------------------

#设置目标URL
w3af/config:target>>> set target http://192.168.128.129
w3af/config:target>>> set target_os unix #设置目标系统为unix
w3af/config:target>>> back  #返回上一级目录
w3af>>>start      #开始扫描
```

也可以使用w3af中集成的脚本去进行扫描
```
root@kali:~/w3af-master# cd scripts/
root@kali:~/w3af-master/scripts# ls
allowed_methods.w3af             login_brute_form_GET.w3af
all.w3af                         login_brute_password_only.w3af
auth_detailed.w3af               mangle_request.w3af
bing_spider.w3af                 mangle_response.w3af
blind_sqli_detection.w3af        os_commanding-lnx-vdaemon.w3af
cookie_fuzzing.w3af              os_commanding-lnx-w3afAgent.w3af
cross_domain.w3af                os_commanding_shell.w3af
csrf.w3af                        os_commanding.w3af
dav_shell.w3af                   php_sca-payload.w3af
detect_transparent_proxy.w3af    profile-fast_scan.w3af
digit_sum.w3af                   remote_file_include_local_ws.w3af
dvwa.w3af                        remote_file_include_proxy.w3af
eval_shell.w3af                  remote_file_include_shell.w3af
eval.w3af                        remote_file_include_shell-xss.w3af
exploit_all.w3af                 remote_file_include_w3af_site.w3af
exploit_fast.w3af                spider_man.w3af
filename_xss.w3af                sqli.w3af
file_upload_shell.w3af           sqlmap_exploit_int.w3af
frontpage_version.w3af           targets_from_file.w3af
header_fuzzing.w3af              web_spider-ignore_regex.w3af
html_output.w3af                 web_spider-only_forward.w3af
list_all_plugins.w3af            web_spider.w3af
local_file_include-payload.w3af  xss_simple.w3af
local_file_include.w3af          xss_stored.w3af

#参数-s表示指定具体的脚本去进行扫描,但需要首先去进入该脚本进行配置目标信息,然后调用w3af的console接口去扫描是否存在sql注入
root@kali:~/w3af-master# ./w3af_console -s scripts/sqli.w3af
```


