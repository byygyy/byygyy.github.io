---
title: Linux之环境变量
date: 2020-03-07 23:08:08
tags: shell
categories: linux
---

我们只要使用linux shell，就避免不了**环境变量**这个话题，大家最开始接触环境变量估计都从接触PATH环境变量开始。
<!--more-->

**环境变量**按照生效范围可以分为**全局环境变量**和**局部环境变量**。全局环境变量对于shell会话和所有生成的子shell都是可见的；局部环境变量只对当前shell有效。Linux系统在你开始bash会话时就会设置一些全局环境变量，而且变量名都是大写，以区别于普通的环境变量。

使用printenv、env或export，可以查看这些默认的**全局环境变量**。
```
[root@aliyun-hk1 profile.d]# printenv
XDG_SESSION_ID=1541
HOSTNAME=aliyun-hk1
TERM=cygwin
SHELL=/bin/bash
HISTSIZE=1000
SSH_CLIENT=222.90.142.215 54197 22
SSH_TTY=/dev/pts/0
[root@aliyun-hk1 opstime.github.io]# env
XDG_SESSION_ID=16
HOSTNAME=aliyun-hk1
TERM=cygwin
SHELL=/bin/bash
HISTSIZE=1000
SSH_CLIENT=222.90.142.205 64344 22
OLDPWD=/apps/ansible-test
SSH_TTY=/dev/pts/1
[root@aliyun-hk1 opstime.github.io]# export
declare -x HISTCONTROL="ignoredups"
declare -x HISTSIZE="1000"
declare -x HOME="/root"
declare -x HOSTNAME="aliyun-hk1"
declare -x LANG="en_US.UTF-8"
declare -x LESSOPEN="||/usr/bin/lesspipe.sh %s"
declare -x LOGNAME="root"
```

显示**单个全局变量的值**，例如：HOME、PATH
```
[root@aliyun-hk1 profile.d]# env|grep HOME
HOME=/root
[root@aliyun-hk1 profile.d]# env|grep PATH
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@aliyun-hk1 profile.d]# echo $HOME
/root
[root@aliyun-hk1 profile.d]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@aliyun-hk1 profile.d]#
```

**全局环境变量**一般在以下几处定义或配置, 从优先级的高低分为/etc/profile、/etc/bashrc、/etc/profile.d/
```
[root@aliyun-hk1 profile.d]# cat /etc/profile
# /etc/profile
​
# System wide environment and startup programs, for login setup
# Functions and aliases go in /etc/bashrc
​
# It's NOT a good idea to change this file unless you know what you
# are doing. It's much better to create a custom.sh shell script in
# /etc/profile.d/ to make custom changes to your environment, as this
# will prevent the need for merging in future updates.
​
pathmunge () {
    case ":${PATH}:" in
        *:"$1":*)
            ;;
        *)
            if [ "$2" = "after" ] ; then
                PATH=$PATH:$1
            else
                PATH=$1:$PATH
            fi
    esac
}

[root@aliyun-hk1 profile.d]# cat /etc/bashrc
# /etc/bashrc
​
# System wide functions and aliases
# Environment stuff goes in /etc/profile
​
# It's NOT a good idea to change this file unless you know what you
# are doing. It's much better to create a custom.sh shell script in
# /etc/profile.d/ to make custom changes to your environment, as this
# will prevent the need for merging in future updates.
​
[root@aliyun-hk1 profile.d]# cd /etc/profile.d/
[root@aliyun-hk1 profile.d]# ll
total 64
-rw-r--r--  1 root root  771 Sep 27  2018 256term.csh
-rw-r--r--  1 root root  841 Sep 27  2018 256term.sh
-rw-r--r--. 1 root root  196 Mar 25  2017 colorgrep.csh
-rw-r--r--. 1 root root  201 Mar 25  2017 colorgrep.sh
-rw-r--r--. 1 root root 1741 Apr 11  2018 colorls.csh
-rw-r--r--. 1 root root 1606 Apr 11  2018 colorls.sh
-rw-r--r--. 1 root root   80 Apr 11  2018 csh.local
-rw-r--r--  1 root root 1706 Sep 27  2018 lang.csh
-rw-r--r--  1 root root 2703 Sep 27  2018 lang.sh
-rw-r--r--. 1 root root  123 Jul 31  2015 less.csh
-rw-r--r--. 1 root root  121 Jul 31  2015 less.sh
-rw-r--r--. 1 root root   81 Apr 11  2018 sh.local
-rw-r--r--  1 root root  105 Apr 11  2018 vim.csh
-rw-r--r--  1 root root  269 Apr 11  2018 vim.sh
-rw-r--r--. 1 root root  164 Jan 28  2014 which2.csh
-rw-r--r--. 1 root root  169 Jan 28  2014 which2.sh
```

**局部环境变量**可以分为**用户级环境变量**和**进程级环境变量**。用户级环境变量在当前用户的任何shell都有效，进程级环境变量只在运行进程的shell中有效。


**用户级环境变量**在​这个用户shell启动的时候会自动加载，按照顺序运行第一个被找到的文件，后面的直接忽略。这些文件都在当前用户的​家目录下以隐藏文件的形式存在，用户可以根据实际需要进行增删改​。如果修改配置文件后必须重新登陆该用户或者执行source filename 或者 . filename​才会生效。
```
~/.bash_profile 1​
~/.bashrc
~/.bash_login 2​
~/.profile 3
```

**进程级环境变量**在运行这个进程时临时定义，只在当前进程的shell及子shell中有效，想在子shell有效，需要使用export 变量名。​每个进程的shell及子shell也会自动继承全局环境变量。
```
[root@aliyun-hk1 ~]# bash
[root@aliyun-hk1 ~]# name=robin
[root@aliyun-hk1 ~]# echo $name
robin
[root@aliyun-hk1 ~]# export name
[root@aliyun-hk1 ~]# bash
[root@aliyun-hk1 ~]# echo $name
robin
[root@aliyun-hk1 ~]# ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
root      3142  3140  0 00:21 pts/0    00:00:00 -bash
root      3189  3142  0 00:22 pts/0    00:00:00 bash
root      3189  3142  0 00:22 pts/0    00:00:00 bash
root      3216  3189  0 00:22 pts/0    00:00:00 ps -f
[root@aliyun-hk1 ~]# exit
exit
[root@aliyun-hk1 ~]# exit
exit
[root@aliyun-hk1 ~]# echo $name
​
[root@aliyun-hk1 ~]#
```