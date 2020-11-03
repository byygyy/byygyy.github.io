---
title: Linux之程序管理
date: 2020-03-24 23:00:00
tags: progress
categories: linux
---

Linux系统是一个多任务的开源操作系统，后台运行着很多的进程。今天我们来学习下进程相关的内容。
<!--more-->

#### 1.1 什么是进程
在linux系统中，触发任何一个事件时，系统都会将它定义为一个进程，并且给这个进程分配一个PID。

##### 1.1.1 进程与程序
简单地说，执行一个程序、脚本或命令，就可以触发一个事件并且取得一个PID。在我看来就是内核读取磁盘中的一个程序，并且加载到内存中，给他分配一个进程号PID。总结下来，进程就是一个运行中的程序。
```
运行bash后会生成一个子shell，查看其产生的进程
[root@aliyun-hk1 ~]# bash
[root@aliyun-hk1 ~]# ps -l
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  3819  3817  0  80   0 - 28895 do_wai pts/0    00:00:00 bash
4 S     0  3875  3819  0  80   0 - 26989 hrtime pts/0    00:00:00 sleep
0 S     0  3881  3819  0  80   0 - 26989 hrtime pts/0    00:00:00 sleep
4 S     0  3949  3819  0  80   0 - 28894 do_wai pts/0    00:00:00 bash
0 R     0  3960  3949  0  80   0 - 38312 -      pts/0    00:00:00 ps
```
##### 1.1.2 工作管理（job control )
将一个命令丢到后台去执行，即在命令末尾增加&，就会产生一个job，包括job number，进程id等，这里的1是job号，3991是进程号。最开始的job状态是Running，后来执行结束后就自动变成了Done。
```
[root@aliyun-hk1 ~]# jobs
[root@aliyun-hk1 ~]# sleep 20 &
[1] 3991
[root@aliyun-hk1 ~]# jobs
[1]+  Running                 sleep 20 &
[root@aliyun-hk1 ~]# jobs
[1]+  Done                    sleep 20
[root@aliyun-hk1 ~]#
```

使用CTRL+Z，将目前的工作丢到后台去，所以就会产生一个job number。
```
[root@aliyun-hk1 ~]# vim hello.js
[1]+  Stopped                 vim hello.js
[root@aliyun-hk1 ~]#
[root@aliyun-hk1 ~]# jobs
[1]+  Stopped                 vim hello.js
[root@aliyun-hk1 ~]#
```

使用jobs查看后前的后台工作状态
```
[root@aliyun-hk1 ~]# jobs
[1]+  Stopped                 vim hello.js
[root@aliyun-hk1 ~]#
```

使用fg将后台作业恢复到前台执行,继续执行刚才的文件编辑。
```
[root@aliyun-hk1 ~]#fg
```

使用bg将在后台而且已暂停执行的job成为运行状态。
```
[root@aliyun-hk1 ~]# jobs
[1]-  Stopped                 vim hello.js
[2]   Running                 nohup sleep 20000 &
[3]+  Stopped                 sleep 100000
[root@aliyun-hk1 ~]# bg 3
[3]+ sleep 100000 &
[root@aliyun-hk1 ~]# jobs
[1]+  Stopped                 vim hello.js
[2]   Running                 nohup sleep 20000 &
[3]-  Running                 sleep 100000 &
[root@aliyun-hk1 ~]#
```

管理后台运行中的job：kill
```
kill -signal job-number
kill -9 job-number 强制删除一个job
kill -15 以正常的程序方式终止一项工作
kill -2 代表由键盘执行CTRL+C一样的操作
```

##### 1.1.3 进程管理
学习和研究进程管理的意义其实很重大。在linux每个运行的程序，都会运行一个进程，这个程序是否还在运行需要看进程是否存在；如果linux系统的资源使用情况突然异常，需要快速找出这个可疑的进程；如果某个程序写得不好，导致有问题的进程存在，也需要找出他；如果有多个重要的用户级进程在系统运行，如果调整他们的优先级，也很重要。

**进程基本信息查看，可以使用ps命令**
```
[root@aliyun-hk1 ~]# ps -l  仅查看自己的bash相关进程
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  6931  6929  0  80   0 - 28895 do_wai pts/0    00:00:00 bash
0 R     0  7000  6931  0  80   0 - 38312 -      pts/0    00:00:00 ps
[root@aliyun-hk1 ~]#
```

```
[root@aliyun-hk1 ~]# ps aux 列出目前所有在内存中的进程
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.6  43532  3420 ?        Ss   Mar15   0:07 /usr/lib/systemd/systemd
root         4  0.0  0.0      0     0 ?        S<   Mar15   0:00 [kworker/0:0H]
root         5  0.0  0.0      0     0 ?        S    Mar15   0:01 [kworker/u2:0]
root         6  0.0  0.0      0     0 ?        S    Mar15   0:01 [ksoftirqd/0]
root         7  0.0  0.0      0     0 ?        S    Mar15   0:00 [migration/0]
```

```
[root@aliyun-hk1 ~]# ps axjf 查看进程并且包含进程树
    1   838   838   838 ?           -1 Ssl      0   1:39 /usr/bin/python2 -Es /usr/sbin/tuned -l -P
 6931  7004  7003  6931 pts/0     7003 S+       0   0:00          \_ grep --color=auto python
    1  1297  1296  1266 ?           -1 Sl       0   2:53 python proxyserver.py 11081
[root@aliyun-hk1 ~]#
```

**进程动态信息查看，可以使用top命令**
```
输入top后，默认系统就会列出所有进程的信息和整体资源的占用情况。
top - 23:14:35 up 11 days, 12:39,  1 user,  load average: 0.00, 0.01, 0.05
Tasks:  71 total,   1 running,  70 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.3 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :   498664 total,    36480 free,    95232 used,   366952 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   389812 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
  975 root      10 -10  127960  10768   6216 S  1.3  2.2 146:08.95 AliYunDun
    1 root      20   0   43532   3420   2104 S  0.0  0.7   0:07.60 systemd
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd
```

```
查看某个进程的负载，可以这样玩
[root@aliyun-hk1 linux-shell-test]# top -d 2 -p 975 
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.5 us,  0.0 sy,  0.0 ni, 99.5 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :   498664 total,    36232 free,    95160 used,   367272 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   389892 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
  975 root      10 -10  127960  10768   6216 S  1.0  2.2 146:13.63 AliYunDun
```
*top命令很强大，可以持续监测整个系统的工作状态，默认情况下5秒更新一次。这个命令今天暂时不展开讲了。*

**进程之间关系的查看**
```
[root@aliyun-hk1 ~]# pstree
systemd─┬─AliYunDun───23*[{AliYunDun}]
        ├─AliYunDunUpdate───3*[{AliYunDunUpdate}]
        ├─2*[agetty]
        ├─aliyun-service───2*[{aliyun-service}]
        ├─atd
        ├─auditd───{auditd}
        ├─chronyd
        ├─crond
        ├─dbus-daemon
        ├─dhclient
        ├─lvmetad
        ├─nginx───nginx
        ├─polkitd───6*[{polkitd}]
        ├─python───13*[{python}]
        ├─rsyslogd───2*[{rsyslogd}]
        ├─sshd───sshd───bash───pstree
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-udevd
        └─tuned───4*[{tuned}]
[root@aliyun-hk1 ~]#

```

**进程的管理和控制**
我们要控制某个进程，只要给他发信号就可以了，进程的唯一标识符就是进程ID。
通过进程PID来终止进程，语法为：kill -signal PID，例如: kill -9 12306
代号     | 含义
-------- | -----
1  | 启动被终止的进程
2 | 相当于键盘输入CTRL+C来中断一个进程
9  |强制中断某一个进程
15|等待进程完成后续操作后，结束该进程
17|相当于用户键入CTRL+Z暂停一个进程的执行

通过进程PID来终止进程，语法为：killall [command name]，例如：killall -9 httpd


**进程的执行顺序**
Linux是多用户、多任务的环境，由top输出的结果我们也可以发现，系统同时间有非常多的进程运行中，只是绝大部分的进程都在休眠（sleep)状态而已。操作系统内核，有默认的调度规则，会自动计算出一个“优先执行序”（priority,PRI)，这个PRI值越低代表越优先，用户无法干预这个PRI。
```
[robin@aliyun-hk1 linux-shell-test]$ ps -alt
F   UID   PID  PPID PRI  NI    VSZ   RSS WCHAN  STAT TTY        TIME COMMAND
4     0   560     1  20   0 110108   824 n_tty_ Ss+  ttyS0      0:00 /sbin/agetty --keep-baud 115200,38400,9600 ttyS0 vt220
4     0   561     1  20   0 110108   820 n_tty_ Ss+  tty1       0:00 /sbin/agetty --noclear tty1 linux
4     0  8614  8612  20   0 115580  2084 do_wai Ss   pts/0      0:00 -bash
4     0  8631  8614  20   0 191784  2340 do_wai S    pts/0      0:00 su - robin
4  1000  8632  8631  20   0 115448  2096 do_wai S    pts/0      0:00 -bash
0  1000  8675  8632  20   0 153248  1500 -      R+   pts/0      0:00 ps -alt
[robin@aliyun-hk1 linux-shell-test]$
```

可以通过Nice来调整，也就是上面的NI，一般他们的关系如下：
```
PRI(new) = PRI(old) + nice
```

root可以随意调整自己或他人进程的nice值，调整的范围是-20~19；
一般用户可以调整的范围仅为0-19（为了避免抢占系统资源），而且只能调大，或者初次使用时设置。
```
新启动一个程序，并设置nice为-5
[root@aliyun-hk1 ~]# nice -n -5 vi &
[1] 8699
[root@aliyun-hk1 ~]# ps -l
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  8614  8612  0  80   0 - 28895 do_wai pts/0    00:00:00 bash
4 T     0  8699  8614  0  75  -5 - 31055 do_sig pts/0    00:00:00 vi
0 R     0  8700  8614  0  80   0 - 38312 -      pts/0    00:00:00 ps

[1]+  Stopped                 nice -n -5 vi
[root@aliyun-hk1 ~]#
```

调整现有进程的优先级，nice值重新调整
```
[root@aliyun-hk1 ~]# renice 10 8699
8699 (process ID) old priority -5, new priority 10
[root@aliyun-hk1 ~]# ps -l
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  8614  8612  0  90  10 - 28895 do_wai pts/0    00:00:00 bash
4 T     0  8699  8614  0  90  10 - 31055 do_sig pts/0    00:00:00 vi
0 R     0  8718  8614  0  90  10 - 38312 -      pts/0    00:00:00 ps
[root@aliyun-hk1 ~]# renice 9 8699
8699 (process ID) old priority 10, new priority 9
[root@aliyun-hk1 ~]# ps -l
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  8614  8612  0  90  10 - 28895 do_wai pts/0    00:00:00 bash
4 T     0  8699  8614  0  89   9 - 31055 do_sig pts/0    00:00:00 vi
0 R     0  8720  8614  0  90  10 - 38312 -      pts/0    00:00:00 ps
[root@aliyun-hk1 ~]#
```
*因为一般用户只能调高nice，即降低优先级，所以我们一般通过调大nice降低非核心任务的优先级，保证核心业务运行的优先级。*

**系统资源查看**
除了系统进程管理外，我们还需要了解如何查询其他系统资源，例如：
```
[root@aliyun-hk1 ~]# free -m free 查看内存使用情况
              total        used        free      shared  buff/cache   available
Mem:            486          93          34           0         359         380
Swap:             0           0           0
```
*Linux系统管理内存的方式跟windows有很大差异，为了让用户的体验更好，系统性能更好，内核会尽可能使用现有的内存，所以如果内存几乎被用光，不要心慌。除非swap开始被大量使用， 才要开始警惕。*

```
[root@aliyun-hk1 ~]# uname -a 内核基本信息查看
Linux aliyun-hk1 3.10.0-1062.12.1.el7.x86_64 #1 SMP Tue Feb 4 23:02:59 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
[root@aliyun-hk1 ~]#
```
*内核基本信息主要包括，主机名、内核版本、创建日期、试用的硬件平台等。*

```
[root@aliyun-hk1 ~]# uptime 查看系统启动时间及负载
 00:03:22 up 12 days, 13:28,  1 user,  load average: 0.00, 0.01, 0.05
[root@aliyun-hk1 ~]#
```
*启动时间就是系统已经开启或重启多久了，负载就是只1，5，15分钟的平均负载。*

```
[root@aliyun-hk1 ~]# netstat -an 查看网络信息
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:11081           0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 172.31.200.164:11081    47.103.99.182:53826     ESTABLISHED

```
*一般我们最长去使用netstat去判断某个进程是否开启了xx监听端口。*


```
[root@aliyun-hk1 ~]# dmesg |more 分析内核产生的信息
[    0.000000]   0 base 000080000000 mask 3FFF80000000 uncachable
[    0.000000]   1 disabled
[    0.000000]   2 disabled
[    0.000000]   3 disabled
[    0.000000]   4 disabled
[    0.000000]   5 disabled
[    0.000000]   6 disabled
[    0.000000]   7 disabled
[    0.000000] PAT configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- UC
[    0.000000] found SMP MP-table at [mem 0x000f59f0-0x000f59ff] mapped at [ffffffffff2009f0]
[    0.000000] Base memory trampoline at [ffff9bf500099000] 99000 size 24576
[    0.000000] Using GB pages for direct mapping
[    0.000000] BRK [0x0b270000, 0x0b270fff] PGTABLE
[    0.000000] BRK [0x0b271000, 0x0b271fff] PGTABLE
```

**vmstat 检测系统资源变化，结合top分析系统瓶颈。**
他可以动态查看CPU/内存/磁盘/IO等数据。
```
[root@aliyun-hk1 ~]# vmstat 查看一次数据
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0  34064  67792 300812    0    0     1     2   31    4  0  0 99  0  0
[root@aliyun-hk1 ~]# vmstat 3 每3秒动态输出一次
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0  34064  67792 300812    0    0     1     2   31    4  0  0 99  0  0
 0  0      0  34064  67800 300812    0    0     0     5  267  875  0  0 99  0  0
^C
```

##### 1.1.4 特殊文件与程序
###### 1.1.4.1 具有SUID/SGID权限的命令执行状态
普通用户用户都有权限通过passwd修改自己的密码，但是密码文件的权限是root，那为什么普通用户有权限读写呢？因为普通用户有passwd程序的特殊权限，他在运行passwd程序时，会自动取得一个新的进程与PID，且具有owner的权限，即具有读写权限。
```
密码文件的默认权限644，也就是普通用户只可以去读。
[robin@aliyun-hk1 ~]$ ls -alt /etc/passwd
-rw-r--r-- 1 root root 1175 Mar 11 00:07 /etc/passwd
[robin@aliyun-hk1 ~]$

找到passwd程序，并看他的权限，他有SUID权限
[robin@aliyun-hk1 ~]$ whereis passwd
passwd: /usr/bin/passwd /etc/passwd /usr/share/man/man1/passwd.1.gz
[robin@aliyun-hk1 ~]$ ls -alt /usr/bin/passwd
-rwsr-xr-x 1 root root 27856 Aug  9  2019 /usr/bin/passwd
[robin@aliyun-hk1 ~]$
```

###### 1.1.4.2 进程数据的查看
其实进程都是加载到内存中运行的，而内存中的数据又都是写入到/proc/*这个目录的。我们当然可以查看/proc/下面的数据。我们发现目前主机上运行的进程，都是以PID编号命名，存到这里的。
```
[robin@aliyun-hk1 ~]$ ls -ltr /proc/
total 0
lrwxrwxrwx  1 root    root                  0 Mar 15 10:34 self -> 10312
-r--r--r--  1 root    root                  0 Mar 15 10:34 swaps
dr-xr-xr-x  9 root    root                  0 Mar 15 10:34 342
dr-xr-xr-x  9 root    root                  0 Mar 15 10:34 362
dr-xr-xr-x  9 root    root                  0 Mar 15 10:34 363
dr-xr-xr-x  9 root    root                  0 Mar 15 10:34 468
dr-xr-xr-x  9 polkitd polkitd               0 Mar 15 10:34 492
dr-xr-xr-x  9 root    root                  0 Mar 15 10:34 496
dr-xr-xr-x  9 dbus    dbus                  0 Mar 15 10:34 503
dr-xr-xr-x  9 chrony  chrony                0 Mar 15 10:34 510

[robin@aliyun-hk1 ~]$ ps -ef|grep 342
root       342     1  0 Mar15 ?        00:00:03 /usr/lib/systemd/systemd-journald
robin    10314 10261  0 22:33 pts/1    00:00:00 grep --color=auto 342
[robin@aliyun-hk1 ~]$
```

这个以PID编号命名的为文件夹里，有什么内容呢？这里面有好多内容，我们可以先了解下cmdline和environ，前者存储的是这个进程被启动的命令串，后者存储的是进程的环境变量。
```
[robin@aliyun-hk1 10319]$ ps -ef|grep sleep
robin    10319 10261  0 22:34 pts/1    00:00:00 sleep 100000
robin    10346 10261  0 22:39 pts/1    00:00:00 grep --color=auto sleep
[robin@aliyun-hk1 10319]$ cd /proc/10319/
[robin@aliyun-hk1 10319]$ ll
total 0
dr-xr-xr-x 2 robin robin 0 Mar 28 22:35 attr
-rw-r--r-- 1 robin robin 0 Mar 28 22:35 autogroup
-r-------- 1 robin robin 0 Mar 28 22:35 auxv
-r--r--r-- 1 robin robin 0 Mar 28 22:34 cgroup
--w------- 1 robin robin 0 Mar 28 22:35 clear_refs
-r--r--r-- 1 robin robin 0 Mar 28 22:34 cmdline
-rw-r--r-- 1 robin robin 0 Mar 28 22:35 comm
-rw-r--r-- 1 robin robin 0 Mar 28 22:35 coredump_filter
-r--r--r-- 1 robin robin 0 Mar 28 22:35 cpuset
lrwxrwxrwx 1 robin robin 0 Mar 28 22:34 cwd -> /proc/342
-r-------- 1 robin robin 0 Mar 28 22:35 environ
.....

[robin@aliyun-hk1 10319]$ cat cmdline
sleep100000[robin@aliyun-hk1 10319]$ cat environ
XDG_SESSION_ID=2314HOSTNAME=aliyun-hk1TERM=xtermSHELL=/bin/bashHISTSIZE=1000SSH_CLIENT=
......
```

Linux系统，常见/prod/下文件与对应的内容。
文件名     | 文件内容
-------- | -----
/proc/cmdline | 加载kernel时所执行的相关参数
/proc/cpuinfo | 本机CPU相关的信息，例如频率、类型、数量
/proc/devices | 记录了系统各个主要设备的代号
/proc/filesystems| 目前系统上已加载的文件系统
/proc/loadavg|top uptime命令中负载的三个平均值就记录在这里
/proc/meminfo|查阅内存信息，相当于free的作用
/proc/mounts|	查阅系统已经挂载的数据，相当于mount
/proc/swaps|记录swap内容使用的文件系统
/proc/version|记录内核的版本

###### 1.1.4.3 查询程序已打开的文件
大家都应该遇到过，"too many open files..." 之类的错误吧? 这个其实跟打开文件数量超过系统限制有关系。
**使用fuser找出正在使用该文件的程序**
```
不加参数，默认打印PID号
[robin@aliyun-hk1 ~]$ fuser nohup.out
/home/robin/nohup.out: 10319
[robin@aliyun-hk1 ~]$ ps -ef|grep 10319
robin    10319 10261  0 22:34 pts/1    00:00:00 sleep 100000
robin    10380 10261  0 22:58 pts/1    00:00:00 grep --color=auto 10319
[robin@aliyun-hk1 ~]$

加参数可以获取更多的进程信息
[robin@aliyun-hk1 ~]$ fuser -uv nohup.out
                     USER        PID ACCESS COMMAND
/home/robin/nohup.out:
                     robin     10319 F.... (robin)sleep
[robin@aliyun-hk1 ~]$
```

**使用lsof找出被进程打开的文件**
```
找出robin用户下，被程序打开的所有文件
[root@aliyun-hk1 ~]# lsof -u robin|wc -l
108
[root@aliyun-hk1 ~]#
```