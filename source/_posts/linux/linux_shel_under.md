---
title: Linux之理解shell
date: 2020-03-06 23:50:37
tags: shell
categories: linux
---

**Linux shell**是一个操作系统提供的、时刻都在运行的复杂交互式程序。它为用户提供了启动程序、管理文件系统中的文件以及运行在linux系统上进程的途径。对于linux OS 运维或linux shell编程而言，理解shell的概念​是重要的一课。

<!--more-->

当我们使用ssh客户端工具，通过密钥认证或者密码认证，登陆成功后，操作系统就为我们创建了一个shell进程​。
```
λ ssh root@lhh.pub
Last login: Fri Mar  6 20:59:55 2020 from 222.90.142.131
Welcome to Alibaba Cloud Elastic Compute Service !
Last login: Fri Mar  6 20:59:55 2020 from 222.90.142.131
Welcome to Alibaba Cloud Elastic Compute Service !
[root@aliyun-hk1 ~]# ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
root     26811 26808  0 23:35 pts/1    00:00:00 -bash
root     26829 26811  0 23:35 pts/1    00:00:00 ps -f
[root@aliyun-hk1 ~]# date
Fri Mar  6 23:35:17 CST 2020
```

所有Linux发行版本的默认shell都是bash shell，因为它的功能最强大，除此之外还有korn shell、ash shell、C shell等等。当前用户自动打开什么shell，决定于用户的shell配置。一般root用户默认会使用bash shell,查看所有用户的shell可以这样玩。
```
[root@aliyun-hk1 ~]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

bash shell程序位于/bin目录，可以发现它是个可执行程序。我们可能还听说过sh shell，它竟然是​一个指向bash的软连接，最起码在我们使用的CentOS是这样的。
```
[root@aliyun-hk1 ~]# ls -alt /bin/bash
-rwxr-xr-x. 1 root root 964544 Apr 11  2018 /bin/bash
[root@aliyun-hk1 ~]# ls -alt /bin/sh
lrwxrwxrwx. 1 root root 4 Nov 29  2018 /bin/sh -> bash
```


shell的父子关系，我们使用ssh客户端登陆成功后，启动的默认交互shell，是一个父shell，后续​创建的shell都是子shell。请看父shell的进程ID是26811，后面进程的父进程PPID都是上一级的​进程PID。进程就是正在运行的一个程序。bash shell 是一个程序，当它运行的时候，​就会成为一个进程。所以我们经常​看到shell和进程的概念交叉使用。
```
[root@aliyun-hk1 ~]# ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
root     26811 26808  0 Mar06 pts/1    00:00:00 -bash
root     28107 26811  0 00:00 pts/1    00:00:00 ps -f
[root@aliyun-hk1 ~]# bash
[root@aliyun-hk1 ~]# ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
root     26811 26808  0 Mar06 pts/1    00:00:00 -bash
root     28110 26811  0 00:00 pts/1    00:00:00 bash
root     28122 28110  0 00:00 pts/1    00:00:00 ps -f
[root@aliyun-hk1 ~]# bash
[root@aliyun-hk1 ~]# ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
root     26811 26808  0 Mar06 pts/1    00:00:00 -bash
root     28110 26811  0 00:00 pts/1    00:00:00 bash
root     28124 28110  0 00:00 pts/1    00:00:00 bash
root     28137 28124  0 00:00 pts/1    00:00:00 ps -f
[root@aliyun-hk1 ~]# pstree -p 26811
bash(26811)───bash(28110)───bash(28124)───pstree(28150)
[root@aliyun-hk1 ~]#
```

让子shell在后台运行，语法为shell command &
```
[root@aliyun-hk1 ~]# sleep 10000 &
[1] 28458
[root@aliyun-hk1 ~]# ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
root     26811 26808  0 Mar06 pts/1    00:00:00 -bash
root     28110 26811  0 00:00 pts/1    00:00:00 bash
root     28124 28110  0 00:00 pts/1    00:00:00 bash
root     28458 28124  0 00:06 pts/1    00:00:00 sleep 10000
root     28462 28124  0 00:07 pts/1    00:00:00 ps -f
[root@aliyun-hk1 ~]#
```

除了使用ps外，还可以使用jobs命令查看后台的作业信息。
```
[root@aliyun-hk1 ~]# jobs -l
[1]+ 28458 Running                 sleep 10000 &
[root@aliyun-hk1 ~]#
```

说到了进程，我们再大概说说线程、​协程。其实跟CPU通信的最小单位是线程，而不是进程​。每个进程使用独立的资源，而同一个进程内部的线程共享资源，同一个线程内的协程​共享资源。以下示例，29006、29011、31826、31827、31926是几个独立的进程，每个进程内都有若干个线程。这几个概念的水其实很深，今天就​先初步讲一讲。
```
[root@aliyun-hk1 ~]# pstree -p 29006
dockerd-current(29006)─┬─docker-containe(29011)─┬─docker-containe(31826)─┬─nginx(31872)───nginx(31926)
                       │                        │                        ├─{docker-containe}(31831)
                       │                        │                        ├─{docker-containe}(31832)
                       │                        │                        ├─{docker-containe}(31833)
                       │                        │                        ├─{docker-containe}(31834)
                       │                        │                        ├─{docker-containe}(31835)
                       │                        │                        ├─{docker-containe}(31836)
                       │                        │                        ├─{docker-containe}(31842)
                       │                        │                        └─{docker-containe}(31844)
                       │                        ├─{docker-containe}(29012)
                       │                        ├─{docker-containe}(29013)
                       │                        ├─{docker-containe}(29014)
                       │                        ├─{docker-containe}(29016)
                       │                        ├─{docker-containe}(29017)
                       │                        ├─{docker-containe}(29018)
                       │                        ├─{docker-containe}(29023)
                       │                        ├─{docker-containe}(29741)
                       │                        └─{docker-containe}(29742)
                       ├─docker-proxy-cu(31808)─┬─{docker-proxy-cu}(31809)
                       │                        ├─{docker-proxy-cu}(31810)
                       │                        ├─{docker-proxy-cu}(31811)
                       │                        └─{docker-proxy-cu}(31812)
                       ├─{dockerd-current}(29007)
                       ├─{dockerd-current}(29008)
                       ├─{dockerd-current}(29009)
                       ├─{dockerd-current}(29010)
                       ├─{dockerd-current}(29015)
                       ├─{dockerd-current}(29020)
                       ├─{dockerd-current}(29022)
                       ├─{dockerd-current}(29024)
                       ├─{dockerd-current}(29628)
                       └─{dockerd-current}(29629)
[root@aliyun-hk1 ~]#
```

shell的内建命令和外部命令，内建命令不会创建子shell和子进程，外部命令​运行时会创建子shell和子进程。例如ps就是一个外部命令，​pwd就是一个内部命令。当外部命令执行时，会创建一个子进程，这种操作也被称为衍生（forking）​。
```
[root@aliyun-hk1 ~]# ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
root     28932 28930  0 00:16 pts/2    00:00:00 -bash
root     28973 28932  0 00:16 pts/2    00:00:00 ps -f
[root@aliyun-hk1 ~]# cd /
[root@aliyun-hk1 /]# pwd
/
[root@aliyun-hk1 /]# ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
root     28932 28930  0 00:16 pts/2    00:00:00 -bash
root     28989 28932  0 00:17 pts/2    00:00:00 ps -f
```

history可以查看执行过的命令，仅限当前用户执行过的命令，无法跨用户查看history。​



命令别名查看，使用alias，这个东西其实很重要，可以让好多​操作简化。
```
[root@aliyun-hk1 /]# alias -p                                                       
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
[root@aliyun-hk1 /]#
```