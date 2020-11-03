---
title: Linux之文件与目录管理
date: 2020-03-11 23:23:23
tags: file
categories: linux
---

上一次我们讲了文件与目录的权限，今天来讲讲文件与目录的管理。
<!--more-->

绝对目录与相对目录的概念，绝对目录是指从linux根目录“/”开始，相对目录是从当前目录“.”开始。绝对目录任何时候都有效，相对目录的实际含义跟当前所在目录密切相关。
```
[root@aliyun-hk1 linux-shell-test]# cat /apps/linux-shell-test/sed.sh
hello
[root@aliyun-hk1 linux-shell-test]# cat ./sed.sh
hello
```

目录的相关操作，cd可以配合绝对目录或相对目录，让用户直接跳转到某个位置，pwd可以查看当前的绝对目录，mkdir可以新建一个目录，rmdir可以删除一个非空目录。
```
[root@aliyun-hk1 linux-shell-test]# cd /apps/linux-shell-test/dir1/
[root@aliyun-hk1 dir1]# ll
total 4
drw------- 2 robin root  4096 Mar 11 00:09 dir1
-rw------- 1 robin root     0 Mar 11 00:09 file1
-rw------- 1 robin robin    0 Mar 11 00:19 file2
[root@aliyun-hk1 dir1]# pwd
/apps/linux-shell-test/dir1
[root@aliyun-hk1 dir1]# mkdir subdir1
[root@aliyun-hk1 dir1]# pwd
/apps/linux-shell-test/dir1
[root@aliyun-hk1 dir1]# cd subdir1/
[root@aliyun-hk1 subdir1]# pwd
/apps/linux-shell-test/dir1/subdir1
[root@aliyun-hk1 subdir1]# cd ../
[root@aliyun-hk1 dir1]# pwd
/apps/linux-shell-test/dir1
[root@aliyun-hk1 dir1]# ll
total 8
drw------- 2 robin root  4096 Mar 11 00:09 dir1
-rw------- 1 robin root     0 Mar 11 00:09 file1
-rw------- 1 robin robin    0 Mar 11 00:19 file2
drwxr-xr-x 2 root  root  4096 Mar 12 22:42 subdir1
[root@aliyun-hk1 dir1]# rmdir subdir1/
[root@aliyun-hk1 dir1]# ll
total 4
drw------- 2 robin root  4096 Mar 11 00:09 dir1
-rw------- 1 robin root     0 Mar 11 00:09 file1
-rw------- 1 robin robin    0 Mar 11 00:19 file2
```

关于执行文件路径变量$PATH及用法，我们接触过linux的小伙伴，会觉得很神奇，输入一个命令后，马上会有一个返回。其实，你每次输入一个命令后，shell会自动去PATH定义的目录逐个查找正确的程序，查到后直接执行，如果有多个，则先查到的执行，后面的忽略。例如，下面的例子就是在/usr/bin/中找到了python2.7并且执行了。
```
[root@aliyun-hk1 dir1]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@aliyun-hk1 dir1]# python
Python 2.7.5 (default, Aug  7 2019, 00:51:29)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>>
[2]+  Stopped                 python
[root@aliyun-hk1 dir1]# whereis python
python: /usr/bin/python2.7 /usr/bin/python /usr/bin/python2.7-config /usr/lib/python2.7 /usr/lib64/python2.7 /etc/python /usr/include/python2.7 /usr/share/man/man1/python.1.gz
[root@aliyun-hk1 dir1]#
```

文件显示属性，可以使用ls，其实ll只是操作系统定义的一个别名，ll=ls -l，如果也要显示隐藏文件，使用ls -alt
```
[root@aliyun-hk1 dir1]# ll
total 4
drw------- 2 robin root  4096 Mar 11 00:09 dir1
-rw------- 1 robin root     0 Mar 11 00:09 file1
-rw------- 1 robin robin    0 Mar 11 00:19 file2
[root@aliyun-hk1 dir1]# ls
dir1  file1  file2
[root@aliyun-hk1 dir1]# alias
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
```

文件新建、复制、移动、删除
touch -a
cp -a -prd -u 
mv -f -u
rm -i -rf

目录的新建、复制、移动、删除，请自行通过帮助文档学习。
mkdir -p
rmdir
cp -r
cat -A

文件查看相关命令，请自行通过帮助文档学习。
more
less
head -n
tail -f
cat
tac
od

文件与目录默认权限与隐藏权限。
umask
SUID
SGID
SBIT

文件类型、位置和查找。
file
which
whereis
find