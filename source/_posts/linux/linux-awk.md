---
title: Linux文本处理命令-awk
date: 2020-03-03 23:08:08
tags: shell
categories: linux
---

**awk**其名称得自于它的创始人 Alfred Aho 、Peter Weinberger 和 Brian Kernighan 姓氏的首个字母。在比较复杂的shell脚本中经常用到，awk grep sed三个合起来，被俗称为shell三剑客。它是linux系统很强大的文本处理命令，并且方便生成各种报表，语法借鉴了C语言。awk同样支持处理文件和标准输入的内容。
<!--more-->

**example1** 分列显示文本文件内容, $0所有列，$1第一列，$2第二列
```
[root@aliyun-hk1 linux-shell-test]# awk '{ print $0 }' awk.sh
hello robin
nihao huanhuan
byby joe
[root@aliyun-hk1 linux-shell-test]# awk '{ print $1 }' awk.sh
hello
nihao
byby
[root@aliyun-hk1 linux-shell-test]# awk '{ print $2 }' awk.sh
robin
huanhuan
joe
```

**example2** 分列显示标准输入文本
```
[root@aliyun-hk1 linux-shell-test]# ps -ef|grep python|awk '{ print $0 }'
root       803     1  0 Feb28 ?        00:00:43 /usr/bin/python -Es /usr/sbin/tuned -l -P
root      6820  6216  0 23:21 pts/0    00:00:00 grep --color=auto python
root     19785     1  0 Feb29 ?        00:00:50 python proxyserver.py 11081
[root@aliyun-hk1 linux-shell-test]# ps -ef|grep python|awk '{ print $2 }'
803
6828
19785
```

**example3**  使用自定义分隔符分列显示文本，-F '='
```
[root@aliyun-hk1 linux-shell-test]# awk -F '=' '{print $0 }' awk.sh
hello robin
nihao huanhuan
byby joe
name1=robin
name2=joe
age1=22
[root@aliyun-hk1 linux-shell-test]# awk -F '=' '{print $1 }' awk.sh
hello robin
nihao huanhuan
byby joe
name1
name2
age1
[root@aliyun-hk1 linux-shell-test]# awk -F '=' '{print $2 }' awk.sh



robin
joe
22
```

**example4** 分列后显示前N行,NR<=3
```
[root@aliyun-hk1 linux-shell-test]# awk  'NR<=3 { print $2 }' awk.sh
robin
huanhuan
joe
```

**example5** 读取shell中变量，用于awk命令参数
```
[root@aliyun-hk1 linux-shell-test]# awk -v var=$split_char -F '=' 'NR<=6 { print $1 var }' awk.sh
hello robin=
nihao huanhuan=
byby joe=
name1=
name2=
age1=
```

**example6** 拆分标准输入或者文件内容，并重新格式化
```
[root@aliyun-hk1 linux-shell-test]# echo -e 'line1=hello\nline2=world'|awk -F '=' '{ print $1": "$2 }'
line1: hello
line2: world
[root@aliyun-hk1 linux-shell-test]# awk -F '=' 'NR>3 { print $1": "$2 }' awk.sh
name1: robin
name2: joe
age1: 22
```

**example7** awk也算是一个编程语言，我们先把简单的、最常用的使用方式搞清除了，再来尝试高级的使用。
```
请打开后自行学习 https://awk.readthedocs.io/en/latest/index.html
```

