---
title: Linux之系统性能优化
date: 2020-03-29 23:00:00
tags: linux
categories: linux
---

如果你是操作系统管理员、中间件管理员、数据库管理员或者开发人员，肯定有机会遇到系统运行缓慢相关的问题。
<!--more-->


# 1.内存使用率
## 1.1 物理内存使用率高
这个其实是好多linux初学者最开始遇到的问题，刚申请的服务器，还没跑程序就发现物理内存使用率很高，少则80%，90%，有些甚至接近100%。其实，我们是用windows的使用经验就判断linux，当面对物理内存使用率高的时候，才会恐慌。其实，只要物理内存可用不为0，swap几乎还没开始使用，就是安全的。
```
Mem这一行代表物理内存的使用情况，看available就知道还有多少物理内存空闲。
[robin@instance-2 ~]$ free -m
              total        used        free      shared  buff/cache   available
Mem:           3529         327        1420          80        1780        2863
Swap:             0           0           0
[robin@instance-2 ~]$ free -g
              total        used        free      shared  buff/cache   available
Mem:              3           0           1           0           1           2
Swap:             0           0           0

```
## 1.2 虚拟内存使用高
大家都知道，虚拟内存是使用硬盘作为内存的一种方式，性能想比较物理内存，差很多。所以当swap开始大于零的时候，就要去考虑增大内存，或者去查找程序本身的错误了。
```
Swap这一行代表虚拟内存的使用情况，看used这个值，就知道已经使用了都少swap内存。
[robin@instance-2 ~]$ free -m
              total        used        free      shared  buff/cache   available
Mem:           3529         327        1420          80        1780        2863
Swap:             0           0           0
[robin@instance-2 ~]$ free -g
              total        used        free      shared  buff/cache   available
Mem:              3           0           1           0           1           2
Swap:             0           0           0

```
## 1.3 找出消耗内存的元凶
当某一天你发现，LInux操作系统的内存严重不足，已经开始大量使用 Swap，甚至都用了多少G的Swap内存。
### 1.3.1 使用top查看物理内存占用
```
使用top命令，并且CTRL+M就会按照内存的使用率给进程排序。
top - 00:10:46 up 8 days,  1:39,  2 users,  load average: 0.00, 0.01, 0.05
Tasks: 162 total,   1 running, 161 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.2 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  3756148 total,  2305048 free,  1037268 used,   413832 buff/cache
KiB Swap:  3932156 total,  3932156 free,        0 used.  2439208 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 4956 jenkins   20   0 4657416 783268  22980 S   0.3 20.9  29:19.30 java
 4796 mysql     20   0  978712  94268   7028 S   0.3  2.5  16:59.70 mysqld

```
### 1.3.2 循环查询虚拟内存占用
```
第一列为PID，第二列为这个进程占用的虚拟内存
[root@localhost ~]# for i in $( cd /proc;ls |grep "^[0-9]"|awk ' $0 >100') ;do awk '/Swap:/{a=a+$2}END{print '"$i"',a/1024"M"}' /proc/$i/smaps 2>/dev/null ; done | sort -k2nr | head -10
112 0M
1149 0M
1155 0M
1169 0M
1172 0M
1184 0M
1187 0M
1191 0M
1198 0M
1210 0M

```
# 2.CPU使用率
其实，总结来讲判断目前的CPU运算时间够不够用，主要看内存的使用率和系统负载，只要系统负载不高，一般都很安全。
## 2.1 CPU使用率高但负载低
这种情况说明，目前CPU资源较为紧张，但是不是性能的瓶颈，CPU基本满足要求。我们一般看负载，只要负载是CPU核心数的0.75倍，就很安全。第一行的load average: 0.00, 0.01, 0.05就代表最近5分钟、10分钟、15分钟的平均负载，第三行的%Cpu(s)代表整体的CPU使用率。
```
使用top的时候，默认情况下，看到第三行的CPU使用率，那是一个整体的CPU使用率。
top - 00:17:55 up 8 days,  1:46,  2 users,  load average: 0.00, 0.01, 0.05
Tasks: 161 total,   1 running, 160 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.2 us,  0.2 sy,  0.0 ni, 99.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  3756148 total,  2304972 free,  1037344 used,   413832 buff/cache
KiB Swap:  3932156 total,  3932156 free,        0 used.  2439132 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
25965 root      20   0  162024   2380   1600 R   1.7  0.1   0:01.42 top
 5302 robin     20   0  149972   2128   1600 S   0.8  0.1  12:36.84 ping

使用top的时候，如果还想查看单个CPU的使用情况，输入1就可以了。
top - 00:19:22 up 8 days,  1:48,  2 users,  load average: 0.17, 0.06, 0.06
Tasks: 162 total,   1 running, 161 sleeping,   0 stopped,   0 zombie
%Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us,  1.5 sy,  0.0 ni, 98.5 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  3756148 total,  2304932 free,  1037384 used,   413832 buff/cache
KiB Swap:  3932156 total,  3932156 free,        0 used.  2439092 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
25967 root      20   0  162024   2380   1600 R   1.5  0.1   0:00.03 top
    1 root      20   0  190976   3988   2620 S   0.0  0.1   1:11.02 systemd
    2 root      20   0       0      0      0 S   0.0  0.0   0:00.19 kthreadd
```
## 2.2 CPU使用率高负载也高
这种情况说明，目前CPU资源紧张，而且已经造成了系统缓慢，一般负载大于CPU核心数的0.75倍，我们就认为负载也高了。这个时候，就要查查CPU都是被谁占用了，而且可能还伴随着IO或者其他访问的问题。
# 3.系统负载高
系统负载的概念，在Linux系统管理方面非常实用，通过平均负载值就可以快速判断当前系统的安全状况。
## 3.1 使用uptime查看负载
```
其实，uptime不仅能查看系统开机多久了，而且也能查看系统在5min 10min 15min的平均负载情况。
[root@localhost ~]# uptime
 00:23:41 up 8 days,  1:52,  2 users,  load average: 0.00, 0.03, 0.05
[root@localhost ~]#
```
## 3.2 使用top查看负载
```
top是我认为的linux系统最好的性能优化命令或者工具，第一行load average: 0.00, 0.02, 0.05就是值最近5min 10min 15min的平均负载。
[root@localhost ~]# top
top - 00:24:38 up 8 days,  1:53,  2 users,  load average: 0.00, 0.02, 0.05
Tasks: 162 total,   1 running, 161 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.4 us,  2.9 sy,  0.0 ni, 95.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  3756148 total,  2304984 free,  1037212 used,   413952 buff/cache
KiB Swap:  3932156 total,  3932156 free,        0 used.  2439168 avail Mem
```
## 3.3 定位负载高的原因
先使用top命令查看整体的负载情况。
```
top - 00:13:53 up 12 min,  2 users,  load average: 0.00, 0.11, 0.13
Tasks: 170 total,   1 running, 169 sleeping,   0 stopped,   0 zombie
%Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  3756132 total,  1771392 free,  1026936 used,   957804 buff/cache
KiB Swap:  3932156 total,  3932156 free,        0 used.  2467352 avail Mem
```
第一行的load average后面的三个数值分别代表最近1分钟、5分钟、15分钟的负载数据。
第二行显示不同类型的task，其中zombie也就是传说的僵尸进程，如果存在可能会严重影响整体性能。
第三行到第6行，显示每个cpu的使用情况，us代表用户占用CPU百分比，sy代表内核占用CPU百分比，id代表空闲CPU百分比，wa代表IO等待占用的CPU百分比，wa如果超过了30要高度注意了，可能瓶颈在磁盘I/O

再用vmstat命令查看更加全面的性能指标。
```
[root@localhost ~]# vmstat 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 1771872   2132 955968    0    0    68    84   81  119  2  0 98  0  0
```
r表示运行队列(就是说多少个进程真的分配到CPU)，一般不能超过CPU数量。
b表示堵塞的进程数量，只要大于0了，就会影响性能。
swpd 表示虚拟内存已使用的大小，如果大于0，表示你的机器物理内存不足了。
free表示物理内存还有多少空闲。
cache直接用来记忆我们打开的文件，给文件做缓冲。
si代表每秒从磁盘读入虚拟内存的大小，只要大于0就说明内存不足了。
so代表每秒虚拟内存写入磁盘的大小，只要大于0就说明内存不足了。
bi/bo代表块设备每秒接收和发送的块数据。
in 代表每秒CPU的中断次数，包括时间中断。
cs 每秒上下文切换次数，例如我们调用系统函数，就要进行上下文切换，线程的切换，也要进程上下文切换，这个值要越小越好，太大了，要考虑调低线程或者进程的数目。系统调用也是，每次调用系统函数，我们的代码就会进入内核空间，导致上下文切换，这个是很耗资源，也要尽量避免频繁调用系统函数。上下文切换次数过多表示你的CPU大部分浪费在上下文切换，导致CPU干正经事的时间少了，CPU没有充分利用，是不可取的。
us 用户CPU时间，我曾经在一个做加密解密很频繁的服务器上，可以看到us接近100,r运行队列达到80(机器在做压力测试，性能表现不佳)。
sy 系统CPU时间，如果太高，表示系统调用时间长，例如是IO操作频繁。
id  空闲 CPU时间。
wt 等待IO CPU时间，一般来说，id + us + sy+ wa = 100。