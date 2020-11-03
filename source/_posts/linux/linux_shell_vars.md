---
title: Linux之shell变量
date: 2020-03-14 23:00:37
tags: shell
categories: linux
---

之前讲过了linux之环境变量，其实环境变量属于最特殊的一种变量。今天我们来讲讲通用的shell变量，就像每种编程语言都有变量的概念一样，shell编程也有变量的概念。Linux变量之所以要叫shell变量，是因为shell是唯一的linux脚本运行环境，也可以简称linux变量。
<!--more-->

#### 变量的概念

想必大家上大学的时候，都学过了C语言、java语言等，最开始的章节都会学习到**变量的概念**，shell变量的概念类似。借用鸟哥的话，变量就是让一个特定的字符串代表不固定的内容。例如：最开始name=robin，过一会name=lily，那最后nama的值就是lily了。其实变量的真正用途，是给shell脚本中可变的内容提供一个修改的入口，在不同的环境中运行同一个shell脚本的时候，可以通过传参修改变量的值。
```
[root@aliyun-hk1 linux-shell-test]# name=robin
[root@aliyun-hk1 linux-shell-test]# echo $name
robin
[root@aliyun-hk1 linux-shell-test]# name=lily
[root@aliyun-hk1 linux-shell-test]# echo $name
lily
```

#### 变量的读取
先试下如何**读取变量的值**。echo $变量名 或 echo ${变量名}​
```
[root@aliyun-hk1 linux-shell-test]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@aliyun-hk1 linux-shell-test]# echo ${PATH}
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```

#### 变量的定义
由于Linux操作系统管理的需要，已经定义了好多默认的变量，例如之前已经讲过的环境变量PATH，**定义变量**名称的时候一定不要使用这些操作系统已经霸占的名称，否则你会有意想不到的惊喜。变量的名称可以是英文字母和数字，但是不能以数字开头，建议字母仅使用小写字母，与操作系统自带的变量区分开。变量与变量的内容以一个“=”号连接，等号两边不能直接接空格。变量的内容中若有空格，就要使用双引号“”或单引号‘’将变量内容包起来。
```
[root@aliyun-hk1 linux-shell-test]# name=robin h h li     
-bash: h: command not found                               
[root@aliyun-hk1 linux-shell-test]# name="robin h h li"   
[root@aliyun-hk1 linux-shell-test]# echo $name            
robin h h li                                              
[root@aliyun-hk1 linux-shell-test]# name='robin h h li2'  
[root@aliyun-hk1 linux-shell-test]# echo $name            
robin h h li2
```

但是，如果**双引号内部有特殊字符$**，可以保留原本的特性即解析变量。
```
[root@aliyun-hk1 linux-shell-test]# name="$name"
[root@aliyun-hk1 linux-shell-test]# echo $name
robin h h li2
[root@aliyun-hk1 linux-shell-test]# name="i am $name"
[root@aliyun-hk1 linux-shell-test]# echo $name
i am robin h h li2
```

如果**单引号内部有特殊字符$**，继续作为普通字符处理。
```
[root@aliyun-hk1 linux-shell-test]# name2='$name'
[root@aliyun-hk1 linux-shell-test]# echo $name2
$name
```

**变量的值若是需要执行其他命令**获取，可以使用\`命令\`或“$(命令)”，注意那个是键盘左上角的符号而不是单引号。
```
[root@aliyun-hk1 linux-shell-test]# version="$(uname -a)"
[root@aliyun-hk1 linux-shell-test]# echo $version
Linux aliyun-hk1 3.10.0-862.14.4.el7.x86_64 #1 SMP Wed Sep 26 15:12:11 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
[root@aliyun-hk1 linux-shell-test]# version2=`uname -a`
[root@aliyun-hk1 linux-shell-test]# echo $version
Linux aliyun-hk1 3.10.0-862.14.4.el7.x86_64 #1 SMP Wed Sep 26 15:12:11 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
[root@aliyun-hk1 linux-shell-test]#
```

**变量增加内容，可以使用"$变量名"或"${变量}"**,为了能跟普通变量的追加有效，最好统一使用引号，如下所示：
```
[root@aliyun-hk1 linux-shell-test]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@aliyun-hk1 linux-shell-test]# PATH=$PATH:/apps/
[root@aliyun-hk1 linux-shell-test]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/apps/
[root@aliyun-hk1 linux-shell-test]# PATH=${PATH}:/apps2/
[root@aliyun-hk1 linux-shell-test]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/apps/:/apps2/
[root@aliyun-hk1 linux-shell-test]# PATH="$PATH":/apps3/
[root@aliyun-hk1 linux-shell-test]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/apps/:/apps2/:/apps3/
[root@aliyun-hk1 linux-shell-test]# PATH="${PATH}":/apps4/
[root@aliyun-hk1 linux-shell-test]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/apps/:/apps2/:/apps3/:/apps4/
```

**取消变量**，使用“unset 变量名”然后通过变量名输出什么都没有了
```
[root@aliyun-hk1 linux-shell-test]# name=robin
[root@aliyun-hk1 linux-shell-test]# echo $name
robin
[root@aliyun-hk1 linux-shell-test]# unset name
[root@aliyun-hk1 linux-shell-test]# echo $name

```

**变量内容使用特殊字符，方法1**，使用“\”，将特殊符号（\[Enger\]、$、 \ 空格符、!等等）变为一般字符。其实就是让特殊符号的特殊作用失效。之前将变量别名的时候，也是同样的道理，使用“\rm”就会让他的别名失效。
```
[root@aliyun-hk1 kernel]# name=\robin
[root@aliyun-hk1 kernel]# echo $name
robin
[root@aliyun-hk1 kernel]# name=\\robin
[root@aliyun-hk1 kernel]# echo $name
\robin
[root@aliyun-hk1 kernel]# name=\$robin
[root@aliyun-hk1 kernel]# echo $name
$robin
[root@aliyun-hk1 linux-shell-test]# name=\'hello\ world\'
[root@aliyun-hk1 linux-shell-test]# echo $name
'hello world'
```

**变量内容使用特殊字符，方法2**，使用‘’号将变量内容包住,但是变量内容不能包含单引号本身。
```
[root@aliyun-hk1 linux-shell-test]# name='$/\ !$%robin'
[root@aliyun-hk1 linux-shell-test]# echo $name
$/\ !$%robin
```

**变量赋值太长，想换行继续输入**，可以使用\回车后继续输入，这里其实是将回车符转义让他失去本来的含义。注意前面已经讲过的，单引号会让特殊字符编变为普通字符，双引号会让特殊字符继续保持原有的作用。
```
[root@aliyun-hk1 linux-shell-test]# name='robin h h li li huan\
> huan very good'
[root@aliyun-hk1 linux-shell-test]# echo $name
robin h h li li huan\ huan very good
[root@aliyun-hk1 linux-shell-test]# name=hello\
> world
[root@aliyun-hk1 linux-shell-test]# echo $name
helloworld
[root@aliyun-hk1 linux-shell-test]# name="robin h h li li huan\
> huan very good"
[root@aliyun-hk1 linux-shell-test]# echo $name
robin h h li li huanhuan very good
```

#### 语系变量

**影响显示结果的语系变量**,目前的linux的发行版已经支持了多种语系了，其实不同语系对应不同的编码方式。locale -a查询目前发行版支持的语系，local查询本机已有的语系文件。这个变量呢，可千万不要随意更改，否则你会看到各种乱码。
```
[root@aliyun-hk1 ~]# locale -a
aa_DJ
aa_DJ.iso88591
aa_DJ.utf8
aa_ER
aa_ER@saaho
aa_ER.utf8
aa_ER.utf8@saaho
aa_ET
aa_ET.utf8
af_ZA
...省略若干行

[root@aliyun-hk1 ~]# locale
LANG=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
```

#### 变量的有效范围。
**变量的有效范围**，我们在前面已经讲过，可以使用export 变量名，让子进程或者子shell可以使用这个变量。普通变量被export后，就变成了用户级的环境变量了。在shell编程的时候，任何变量定义和使用的时候，我们都要搞清楚，他的有效范围。

**如果想让变量在子shell中也可以访问**，使用“export 变量名”,这个在shell编程中很重要！
```
[root@aliyun-hk1 linux-shell-test]# cat test_vars.sh
#!/bin/bash
echo $PATH
echo $name
[root@aliyun-hk1 linux-shell-test]# ./test_vars.sh
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

[root@aliyun-hk1 linux-shell-test]# name=robin
[root@aliyun-hk1 linux-shell-test]# ./test_vars.sh
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

[root@aliyun-hk1 linux-shell-test]# export name
[root@aliyun-hk1 linux-shell-test]# ./test_vars.sh
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
robin
[root@aliyun-hk1 linux-shell-test]#
```

**如果让export过的变量变成普通变量**，使用declare +x 变量名称
```
[root@aliyun-hk1 ~]# name=robin
[root@aliyun-hk1 ~]# export name
[root@aliyun-hk1 ~]# export|grep name
declare -x name="robin"
[root@aliyun-hk1 ~]# declare +x name
[root@aliyun-hk1 ~]# export|grep name
[root@aliyun-hk1 ~]#
```

#### 变量的输入、数组与声明

**变量通过键盘读入**，read或read -s,一个是普通读入方式，一个是加密读入，并且只在当前shell有效。如果想在子shell中使用，需要使用export 变量名。
```
[root@aliyun-hk1 ~]# read name
robin
[root@aliyun-hk1 ~]# echo $name
robin
[root@aliyun-hk1 ~]# read -s age
[root@aliyun-hk1 ~]# echo $age
12
```

**申明变量的类型**，变量定义的时候，我没有提到变量类型的概念，因为shell默认变量类型为字符串，通过declare/typeset可以申明变量的类型。
```
-i 定义一个整数类型变量，而且经过测试进行数字加减计算并赋值时，被赋值的变量也必须为整数型，即不支持自动类型转换。
[root@aliyun-hk1 ~]# declare -i age1=20
[root@aliyun-hk1 ~]# declare -i age2=18
[root@aliyun-hk1 ~]# declare -i count
[root@aliyun-hk1 ~]# count=$age1-$age2
[root@aliyun-hk1 ~]# echo $count
2
[root@aliyun-hk1 ~]# count2=$age1-$age2
[root@aliyun-hk1 ~]# echo $count2
20-18
[root@aliyun-hk1 ~]#

-x 将后面的变量变成环境变量，用法跟export一样。
[root@aliyun-hk1 ~]# declare -x date='2020-03-15'
[root@aliyun-hk1 ~]# export|grep date
declare -x date="2020-03-15"
[root@aliyun-hk1 ~]# bash
[root@aliyun-hk1 ~]# echo $date
2020-03-15

-r 将后面的变量变为只读变量
[root@aliyun-hk1 ~]# declare -r size=big
[root@aliyun-hk1 ~]# echo $size
big
[root@aliyun-hk1 ~]# size=small
bash: size: readonly variable

-a 定义一个数组变量
[root@aliyun-hk1 ~]# declare -a vars
[root@aliyun-hk1 ~]# echo $vars

[root@aliyun-hk1 ~]# vars[0]=robin
[root@aliyun-hk1 ~]# echo ${vars[0]}
robin
[root@aliyun-hk1 ~]# vars[1]=lily
[root@aliyun-hk1 ~]# echo ${vars[1]}
lily
[root@aliyun-hk1 ~]# echo $vars
robin
```

#### 字典类型变量
其实大部分资料都把这种叫做shell的第二种数组，但是我觉得叫它字典最合适了，因为其他编程语言一般都这样叫的。
```
定义字典时，可以直接赋值，定义以后也可以继续给字典增加内容，而且只能通过${dic_name[key]}的方式访问。
[root@aliyun-hk1 opstime.github.io]# declare -A arr_string3[name]=robin
[root@aliyun-hk1 opstime.github.io]# echo ${arr_string3[name]}
robin
[root@aliyun-hk1 opstime.github.io]# arr_string3[age]=18
[root@aliyun-hk1 opstime.github.io]# echo ${arr_string3[age]}
18
[root@aliyun-hk1 opstime.github.io]#
```