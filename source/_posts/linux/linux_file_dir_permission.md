---
title: Linux之文件与目录权限
date: 2020-03-10 23:08:09
tags: file
categories: linux
---

Linux最优秀的地方之一，就在于它的多用户、多任务环境。为了方便每个用户的使用和数据安全，Linux有严格而又强大的文件和目录管理机制。
<!--more-->

Linux系统文件和目录权限，**按照身份**可以分为，owner、group、others，例如这个文件夹里的文件，owner是root，group也是root，others是指其他人。
```
[root@aliyun-hk1 httpd]# ll
total 12
drwxr-xr-x 2 root root 4096 Feb 27 01:06 conf
drwxr-xr-x 2 root root 4096 Mar  8 22:10 conf.d
drwxr-xr-x 2 root root 4096 Mar  8 22:10 conf.modules.d
lrwxrwxrwx 1 root root   19 Feb 27 01:06 logs -> ../../var/log/httpd
lrwxrwxrwx 1 root root   29 Feb 27 01:06 modules -> ../../usr/lib64/httpd/modules
lrwxrwxrwx 1 root root   10 Feb 27 01:06 run -> /run/httpd
```

Linux系统文件和目录权限，**按照操作类型**可以分为，read、write、execute，例如文件夹conf的权限，从左往右看，d代表conf是一个目录，rwx代表ower的权限为读写执行，r-x代表组成员的权限为读执行权限，r-x代表其他人的权限也为读执行权限。例如log最左边是l代表它是一个软连接，如果最左边是-代表它是一个文件。
```
[root@aliyun-hk1 httpd]# ll
total 12
drwxr-xr-x 2 root root 4096 Feb 27 01:06 conf
drwxr-xr-x 2 root root 4096 Mar  8 22:10 conf.d
drwxr-xr-x 2 root root 4096 Mar  8 22:10 conf.modules.d
lrwxrwxrwx 1 root root   19 Feb 27 01:06 logs -> ../../var/log/httpd
lrwxrwxrwx 1 root root   29 Feb 27 01:06 modules -> ../../usr/lib64/httpd/modules
lrwxrwxrwx 1 root root   10 Feb 27 01:06 run -> /run/httpd
```

Linux系统，文件的读权限r代表用户可以查看文件里的内容，可以执行类似cat的命令；文件的写权限w代表用户可以修改文件内容，可以执行类似sed的命令，文件的执行权限x代表用户可以把文件内容当作shell脚本一样执行，可以执行bash file_name命令。这个比较好理解，就不用举例了。


Linux系统，目录的权限一般都是r-x组合，用户可以查看目录内容或者进入到目录，rwx组合用户可以对目录进行任何操作。当目录具有单独的r、w、x权限时，没有实际的用途，例如r权限代表用户具有读取目录结构列表的权限，w权限代表用户有修改目录结构的权限，x权限代表用户能否进入该目录成为工作目录而已。


Linux文件和目录权限修改，可以使用chown修改文件和目录ower和group的身份，使用chmod可以修改文件和目录的读写执行权限。
```
[root@aliyun-hk1 linux-shell-test]# ll
total 44
-rwxr-xr-x 1 root  root    0 Mar  3 23:32 0
-rw-r--r-- 1 root  root    0 Mar 11 00:36 1
drwxr-xr-x 2 root  root 4096 Mar 11 00:36 2
[root@aliyun-hk1 linux-shell-test]# chmod 755 1
[root@aliyun-hk1 linux-shell-test]# ll
total 44
-rwxr-xr-x 1 root  root    0 Mar  3 23:32 0
-rwxr-xr-x 1 root  root    0 Mar 11 00:36 1
drwxr-xr-x 2 root  root 4096 Mar 11 00:36 2
[root@aliyun-hk1 linux-shell-test]# chown robin:robin 2
[root@aliyun-hk1 linux-shell-test]# ll
total 44
-rwxr-xr-x 1 root  root     0 Mar  3 23:32 0
-rwxr-xr-x 1 root  root     0 Mar 11 00:36 1
drwxr-xr-x 2 robin robin 4096 Mar 11 00:36 2
[root@aliyun-hk1 linux-shell-test]#
```

Linux系统，在每个shell环境都有一个默认权限概念，使用umask可以获取当前shell的默认权限。umask决定，你直接使用mkdir新建目录或者使用touch新建文件时的默认权限。例如root用户的默认umask为0022，则新建文件的默认权限为644，(-rw-rw-rw-)-(-----w--w-)=(-rw-r--r--)，新建目录的默认权限为755，(-rwxrwxrwx)-(-----w--w-)=(-rwxr-xr-x) 这种用字符计算的方式是我跟高手鸟哥学习的，保证有效。之前用数字相减的方法有漏洞，不要使用！
```
[root@aliyun-hk1 linux-shell-test]# umask
0022
[root@aliyun-hk1 linux-shell-test]# touch 1
[root@aliyun-hk1 linux-shell-test]# ll
total 40
-rwxr-xr-x 1 root  root    0 Mar  3 23:32 0
-rw-r--r-- 1 root  root    0 Mar 11 00:36 1
[root@aliyun-hk1 linux-shell-test]# mkdir 2
[root@aliyun-hk1 linux-shell-test]# ll
total 44
-rwxr-xr-x 1 root  root    0 Mar  3 23:32 0
-rw-r--r-- 1 root  root    0 Mar 11 00:36 1
drwxr-xr-x 2 root  root 4096 Mar 11 00:36 2
```