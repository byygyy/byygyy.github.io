---
title: Linux之shell用户输入
date: 2020-03-19 23:00:00
tags: shell
categories: linux
---

讲完了shell的变量、条件判断和循环后，今天再来讲以下如何处理用户的输入。
<!--more-->

#### 1.1命令行参数
我们编写shell脚本的时候，可能有些是需要用户传递参数进去的，命令行参数是最常用的方式。$0是可以获取程序名，$1获取第一个参数，$2获取第二个参数，以此类推，不同的参数用空格分隔，如果参数本身包含空格，必须使用单引号或双引号包起来。直接上示例:

##### 1.1.1参数本身没有空格的话，加不加引号无所谓
```
[root@aliyun-hk1 linux-shell-test]# cat shell-input.sh
#!/bin/bash

echo $(basename $0)
echo $0
echo $1
echo $2
[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh 1 2 3
shell-input.sh
./shell-input.sh
1
2
```

##### 1.1.2参数本身有空格，不加引号的话，会被当做多个变量处理，用引号包起来才对
```
[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh "hello world" "nihao zhongguo"
shell-input.sh
./shell-input.sh
hello world
nihao zhongguo
[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh hello world nihao zhongguo
shell-input.sh
./shell-input.sh
hello
world
```

##### 1.1.3检查参数是否为空
```
[root@aliyun-hk1 linux-shell-test]# cat shell-input.sh
#!/bin/bash

if [ -n "$1" ]
then
  echo $(basename $0)
  echo $0
  echo $1
fi
[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh
[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh 10
shell-input.sh
./shell-input.sh
10
[root@aliyun-hk1 linux-shell-test]#
```
*-z判断变量的值,如果为空返回0，否则返回1；-n判断变量的值，如果为空返回1，否则返回0，使用[]的时候，变量用双引号包起来*

##### 1.1.4检查参数的个数
```
先打印变量的个数，如果变量个数大于0，输出它。
[root@aliyun-hk1 linux-shell-test]# cat shell-input.sh
#!/bin/bash

echo "$#"

if [ "$#" -gt 0 ]
then
  echo $(basename $0)
  echo $0
  echo $1
fi
[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh
0
[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh 10 11
2
shell-input.sh
./shell-input.sh
10
[root@aliyun-hk1 linux-shell-test]#
```

##### 1.1.5 抓取所有的参数值
抓取所有参数，并且当作一个整体处理，使用$*
```
[root@aliyun-hk1 linux-shell-test]# cat shell-input.sh
#!/bin/bash

echo "$*"
count1=0

for a in "$*"
do
  echo "$a"
  count1=$[ $count1 +1 ]
  echo "$count1"
done

[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh 10 20 hello
10 20 hello
10 20 hello
1
```

抓取所有参数，并且把每个参数分开处理，使用$@
```
[root@aliyun-hk1 linux-shell-test]# cat shell-input.sh
#!/bin/bash

echo "$@"
count1=0

for a in "$@"
do
  count1=$[ $count1 +1 ]
  echo "$count1"
  echo "$a"
done

[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh 10 20 hello
10 20 hello
1
10
2
20
3
hello
```

##### 1.1.6 移动参数变量
使用shift是遍历参数的一个简单好用的办法，默认情况下它会将每个参数变量向左移动一个位置，移动后变量$3变为$2，$2变为$1,$1被丢弃，$0不变。
```
[root@aliyun-hk1 linux-shell-test]# cat shell-input.sh
#!/bin/bash

count1=1
while [ -n "$1" ]
do
  echo "parameter #$count1 is $1"
  count1=$[ $count1 + 1 ]
  shift
done
[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh 10 20 30
parameter #1 is 10
parameter #2 is 20
parameter #3 is 30
[root@aliyun-hk1 linux-shell-test]#
```

#### 1.2获取用户输入
shell脚本运行时，可能有时候需要用户临时输入参数，继续后面的操作。接下来，我们就来讲讲获取用户输入的多种方式。
##### 1.2.1 基本的读取
使用read name读取输入到name，使用read -s password读取输入到password并且不shell上隐藏输入。
```
[root@aliyun-hk1 linux-shell-test]# cat shell-input.sh
#!/bin/bash

echo -n "please input your name:"
read name
echo -n "please input your password:"
read -s password
echo ""
echo "your user name is $name, password is $password"
[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh
please input your name:robin
please input your password:
your user name is robin, password is nihao
[root@aliyun-hk1 linux-shell-test]#

```

##### 1.2.2 设置等待用户超时时间
```
[root@aliyun-hk1 linux-shell-test]# cat shell-input.sh
#!/bin/bash

if read -t 5 -p "please input name" name
then
   echo "hello $name"
else
   echo
   echo "sorry fo timeout"
fi
[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh
please input name
sorry fo timeout
[root@aliyun-hk1 linux-shell-test]#

```

##### 1.2.3 从文件读取参数
最后再讲讲如何从文件读入参数，并直接来使用。
```
[root@aliyun-hk1 linux-shell-test]# cat shell-input.sh
#!/bin/bash

count=1
cat test | while read line
do
  echo "Line $count: $line"
  count=$[ $count + 1 ]
done

echo ”finish read the file"
[root@aliyun-hk1 linux-shell-test]# vim test
[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh
Line 1: 10
Line 2: 20
Line 3: hello
finish read the file
[root@aliyun-hk1 linux-shell-test]#

```


#### 1.3 参数标准化
在linux世界里，对于传递给shell程序的选项，已经有了某种程序的保准含义。我们在自己编写shell脚本时，也可以参考该标准。
```
-a 显示所有对象
-c 生成一个计数器
-d 一定一个目录
-e 扩展一个对象
-f 指定读入数据的文件
-h 显示命令的帮助信息
-i 忽略文本的大小写
-l 显示输出的长格式版本
-n 使用非交互模式
-o 将所有输出重定向到指定的文件
-q 以安静模式运行
-r 递归的处理目录和文件
-s 以安静模式运行
-v 生成详细输出
-x 排除某个对象
-y 对所有问题回答yes
```