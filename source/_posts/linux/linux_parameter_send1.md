---
title: Linux之shell参数传递
date: 2019-03-11 15:15:37
tags: shell
categories: linux
---

# 背景
最近编写ansible脚本，需要自动调用之前写好的shell脚本，由于shell脚本中包含了read命令接收用户的手动输入，为了实现ansible脚本的自动运行，不得不将手动输入转换为自动输入。以下为常见的两种场景：

<!--more-->

# read命令接收输入
```
#!/bin/sh
#file name start.sh
echo "please input name"
read name
echo "please intput pssword"
read password

echo $name
echo 'line end'
echo $password
```

## 第一种方式
```
echo -e "robin\npassword"|./start.sh
```
第一种方式，其实就是使用管道将一个或多个输入传递给待执行的脚本，如果传入多个变量，一定要注意这个\n，经检验read每次读入一个变量值，遇到\n则截断。

## 第二种方式
```
echo -e "robin\npassword" >parm
./start.sh < parm
```
第二种方式，其实就是将一个或多个变量值存入一个参数文件，每个变量以\n结尾，再将该参数文件传递给shell脚本文件。

## 第三种方式
```
使用expect
```
由于需要yum install expect 暂未研究。

# $1 $2 .. 接收传入的参数
```
#!/bin/sh
#file name start.sh
name=$1
password=$2

echo -e  $name\n
echo $password
```
```
./start.sh robin password
```
这种接收参数的方式，也是最简单的好用的方式，shell文件按照顺序接收传入的参数值。

# 使用getopts接收传入的参数
```
#!/bin/sh

while getopts ":a:b:c:" opt
do
   case $opt in
        a)
        echo "参数a的值$OPTARG"
        ;;
        b) echo "参数b的值$OPTARG"
        ;;
        c) echo "参数c的值$OPTARG"
        ;;
        ?) echo "未知参数"
           exit 1;;
   esac
done
```

```
./start_4.sh -a 1 -b 2
```