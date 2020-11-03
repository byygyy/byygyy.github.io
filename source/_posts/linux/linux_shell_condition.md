---
title: Linux之shell条件判断
date: 2020-03-16 21:57:37
tags: shell
categories: linux
---

之前讲过了linux之shell变量，光有变量还不够，对于任何脚本语言或者编程语言来说，变量、条件判断、循环等是必不可少的，shell编程也不例外。今天我就先来讲讲shell条件判断语句。
<!--more-->

#### 条件判断if-else语句
许多时候，我们的脚本并不是简单的一路执行下去，而需要做各种条件判断。我们学过C语言或者java中的条件判断，其实shell脚本条件判断跟他们类似。例如c语言，if后面紧跟一个表达式，这个表达式的计算结果如果为True，则执行后面的语句，如果表达式结算结果为False，则不执行后面的语句。**shell的if语句有点不一样，它根据if后面命令执行后的退出状态码判断，如果退出状态码为0，执行then后面的语句，否则不执行then后面的语句。**我们在讲“Linux之shell输入输出重定向”的时候，已经讲个各种退出返回码的含义。
```
pwd可以成功执行，退出码为0，所以then后面的命令执行了。
[root@aliyun-hk1 linux-shell-test]# cat ifelse.sh
#!/bin/bash

if pwd
then
   echo "hello world"
fi
[root@aliyun-hk1 linux-shell-test]# ./ifelse.sh
/apps/linux-shell-test
hello world
[root@aliyun-hk1 linux-shell-test]#

pwd后面的icantrun执行报错，退出返回码不为0，所以then后面的命令不执行
[root@aliyun-hk1 linux-shell-test]# cat ifelse2.sh
#!/bin/bash

if icantrun
then
   echo "hello world"
fi
[root@aliyun-hk1 linux-shell-test]# ./ifelse2.sh
./ifelse2.sh: line 3: icantrun: command not found
[root@aliyun-hk1 linux-shell-test]#

```

#### 条件判断if-then-else语句
if-then语句类似于c语言中的if语句，则if-then-else相当于c语言中的if-else语句。if-then-else语句中，如果if语句后面的命令退出返回码为0，then后面的命令会执行，这跟普通的if-then一样；如果if语句后面的命令退出返回码不为0，then语句后面的命令不执行，但是执行else后面的语句。
```
在用户配置文件中找不到lily，退出返回码不为0
[root@aliyun-hk1 linux-shell-test]# cat if-then-else.sh
#!/bin/bash

if cat /etc/passwd|grep lily
then
   echo "lily can found"
else
   echo "lily can't found"
fi
[root@aliyun-hk1 linux-shell-test]# ./if-then-else.sh
lily can't found
[root@aliyun-hk1 linux-shell-test]#
```

#### 条件判断if-then-elif-then语句
这个其实就是我们常说的if嵌套语句的简化写法，如果if后面的命令退出返回码不为0，不执行相邻的then语句，再如果elif后面的命令退出返回码为0，则执相邻的then语句。
```
[root@aliyun-hk1 linux-shell-test]# cat ifthenelifthen.sh
#/bin/bash

if iamnotcommand
then
  echo "first command return 0"
elif pwd
then
  echo "second command return 0"
fi
[root@aliyun-hk1 linux-shell-test]# chmod 755 ifthenelifthen.sh
[root@aliyun-hk1 linux-shell-test]# ./ifthenelifthen.sh
./ifthenelifthen.sh: line 3: iamnotcommand: command not found
/apps/linux-shell-test
second command return 0
[root@aliyun-hk1 linux-shell-test]#
```

#### 条件判断if-then-elif-then-多重嵌套语句
如果if后面的命令退出返回码不为0，不执行相邻的then语句，再如果elif后面的命令退出返回码为0，则执相邻的then语句，以此类推,只要第一次遇到命令退出返回码为0，执行相邻的then后，后面的命令都不执行了。
```
[root@aliyun-hk1 linux-shell-test]# cat ifthenelifthen.sh
#/bin/bash

if iamnotcommand
then
  echo "first command return 0"
elif 1pwd
then
  echo "second command return 0"
elif 2pwd
then
  echo "3st command retrun 0"
elif pwd
then
  echo "4st command return 0"
elif pwd
then
  echo "5st command return 0"
fi

[root@aliyun-hk1 linux-shell-test]# ./ifthenelifthen.sh
./ifthenelifthen.sh: line 3: iamnotcommand: command not found
./ifthenelifthen.sh: line 6: 1pwd: command not found
./ifthenelifthen.sh: line 9: 2pwd: command not found
/apps/linux-shell-test
4st command return 0
[root@aliyun-hk1 linux-shell-test]#
```
**注意then后面你可以使用不止一条命令，可以像在其他编程语言中一样，列出多条命令。**

#### 使用test判断一个条件是否成立
我们能不能像其他编程语言一样，判断一个条件是否成立，成立的话执行一个语句，不成立的话执行另外语句？答案是肯定的。我们可以使用if语句结合test命令来实现。test command，如果这个commad成立，则这条test语句返回0，否则返回非0。test命令可以用来做数值比较、字符串比较、文件比较。
```
[root@aliyun-hk1 linux-shell-test]# cat iftestthen.sh
这里就通过test判断两个变量的值是否相等。
#!/bin/bash

name1=robin
name2=robin

if test $name1 = $name2
then
   echo "name1 same as name2"
fi
[root@aliyun-hk1 linux-shell-test]# ./iftestthen.sh
name1 same as name2
[root@aliyun-hk1 linux-shell-test]#
```

#### 使用[]判断一个条件是否成立
如果[]内的条件成立，则退出返回码为0，执行then后面的命令。
```
[root@aliyun-hk1 linux-shell-test]# cat ifelse.sh
#!/bin/bash
age=18
if [ $age -gt 18 ]
then
   echo "age is more than 18"
elif [ $age -eq 18 ]
then
   echo "age is 18"
fi
[root@aliyun-hk1 linux-shell-test]# ./ifelse.sh
age is 18
[root@aliyun-hk1 linux-shell-test]#
```
**这里有人提出疑问了，定义一个整形变量不是要使用declare -i吗? 为什么这里不使用了？ 经过测试发现，目前的shell版本已经可以不用申明变量类型了。当然你加了declare也不会报错。**

#### 讲if-else放在一行执行
```
[root@aliyun-hk1 linux-shell-test]# num1=1; num2=2; if [ $num1 = $num2 ]; then echo "num1 is same as num2";fi;
[root@aliyun-hk1 linux-shell-test]# num1=1; num2=2; if [ $num1 -eq $num2 ]; then echo "num1 is same as num2";fi;
[root@aliyun-hk1 linux-shell-test]# num1=1; num2=2; if [ $num1 -ne $num2 ]; then echo "num1 is same as num2";fi;
num1 is same as num2
[root@aliyun-hk1 linux-shell-test]# num1=1; num2=2; if [ $num1 -le $num2 ]; then echo "num1 is same as num2";fi;
num1 is same as num2
[root@aliyun-hk1 linux-shell-test]# num1=1; num2=2; if [ $num1 -ge $num2 ]; then echo "num1 is same as num2";fi;
[root@aliyun-hk1 linux-shell-test]# num1=1; num2=2; if [ $num1 -gt $num2 ]; then echo "num1 is same as num2";fi;
[root@aliyun-hk1 linux-shell-test]#
```

#### test或[]数值比较用到的参数 
```
a -eq b  a是否与b相等
a -ge b  a是否大于等于b
a -gt b  a是否大于b
a -le b  a是否小于等于b
a -lt b  a是否小于b
a -ne b  a是否不等于b
```
**在shell脚本中如果数值比较如果使用了字符串用的参数，它会进行强制类型转换，这里要注意。**


#### test或[]字符串比较用到的参数
```
str1 = str2  str1和str2是否相同
str1 != str2 str1和str2是否不同
str1 < str2  str1是否比str2小
str1 > str2  str1是否比str2大
-n str1      str1长度是否非0
-z str1      str1长度是否为0
```
**在shell脚本中使用的时候一定要记得给<或<转义，字符串不可以使用数值专用的参数，否则会报错。**


#### test或[]文件比较用到的参数
```
-d file 检查file是否存在并且式一个目录
-e file 检查file是否存在
-f file 检查file是否存在并且式一个文件
-r file 检查file是否存在并可读
-s file 检查file是否存在并非空
-w file 检查file是否存在并且可写
-x file 检查file是否存在并可执行
-O file 检查file是否存在并属于当前用户
-G file 检查file是否存在并且默认组与当前用户相同
file1 -nt file2 检查file1是否比file2新
file1 -ot file2 检查file1是否比file2旧
```

#### shell编程if语句的高级用法
例如if (( $var1 ** > 90 )) 双括号允许你使用高级的数学表达式。
例如if [[ $USER == r* ]] 双方括号允许你使用高级的字符串表达式。
**shell 脚本语言对符号的使用很多，这一点让用习惯了python、java、c的同学有点不习惯。所以，我们先挑选熟悉常用的，高级复杂的需要了再学。**


#### 下次一我们将会讲解shell for循环的使用。
敬请期待。