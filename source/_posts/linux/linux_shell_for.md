---
title: Linux之shell循环
date: 2020-03-17 23:00:00
tags: shell
categories: linux
---

讲完了shell的变量和条件判断后，今天再来讲讲循环。就像c语言或者java一样，循环在所有的编程语言或脚本语言中必不可少，shell自然也是。
<!--more-->

#### 1.for循环
假设你学过c语言后者java，循环的概念我就不废话了。for循环假设列表中的值都是用空格分开的，因此列表中的值如果有空格，用双引号包起来。
##### 1.1 for 循环中list中都是常量
```
[root@aliyun-hk1 linux-shell-test]# cat for.sh
#!/bin/bash

for abc in hello world nihao zhongguo
do
   echo the next one is $abc
done
[root@aliyun-hk1 linux-shell-test]# chmod 755 for.sh
[root@aliyun-hk1 linux-shell-test]# ./for.sh
the next one is hello
the next one is world
the next one is nihao
the next one is zhongguo
[root@aliyun-hk1 linux-shell-test]#
```

##### 1.2 for 循环中list是一个字符串
```
[root@aliyun-hk1 linux-shell-test]# cat for.sh
#!/bin/bash

list1="ni hao hello world"
for abc in $list1
do
   echo the next one is $abc
done
[root@aliyun-hk1 linux-shell-test]# ./for.sh
the next one is ni
the next one is hao
the next one is hello
the next one is world
```

##### 1.3 for 循环中list是从文件读取的
```
[root@aliyun-hk1 linux-shell-test]# cat ./for.sh
#!/bin/bash

list1="ni hao hello world"
file="fortest.txt"
for abc in $(cat $file)
do
   echo the next one is $abc
done
[root@aliyun-hk1 linux-shell-test]# ./for.sh
the next one is hello
the next one is world
the next one is ni
the next one is hao
the next one is I
the next one is am
the next one is robin
```

##### 1.4 for循环中使用非默认的分隔符
因为有个特殊的环境变量IFS,它定义了bash shell用作字符分隔的一系列字符。默认的情况下，bash shell分隔符为分别为空格、制表符和换行符。
```
[root@aliyun-hk1 linux-shell-test]# cat for.sh
#!/bin/bash

file="fortest.txt"
IFS=$'\n'
for abc in $(cat $file)
do
   echo the next one is $abc
done
[root@aliyun-hk1 linux-shell-test]# ./for.sh
the next one is hello
the next one is world
the next one is ni
the next one is hao
the next one is I
the next one is am
the next one is robin h h li
[root@aliyun-hk1 linux-shell-test]#
```
** 在shell编程中一般为了后续继续使用默认分隔符，一般先临时备份默认分隔符，IFS.OLD=$IFS;IFS=$'\n';IFS=$IFS.OLD 保证后续脚本能在默认和非默认分隔符之间快速切换。**

##### 1.5 for循环使用c语言的分格
```
[root@aliyun-hk1 linux-shell-test]# cat forc.sh
#!/bin/bash

for (( i=1; i<=9; i++ ))
do
   echo the next one is $i
done
[root@aliyun-hk1 linux-shell-test]# ./forc.sh
the next one is 1
the next one is 2
the next one is 3
the next one is 4
the next one is 5
the next one is 6
the next one is 7
the next one is 8
the next one is 9
[root@aliyun-hk1 linux-shell-test]#
```

#### 2.while循环
while循环其实是if-else和for循环的混杂体。while命令允许定义一个要测试的命令，然后循环执行一组命令，只要定义的测试命令返回的是退出状态码0，这组命令就反复执行。while循环写不好，很容易造成死循环。
##### 2.1 使用一个测试命令的while循环
```
一个测试命令是最常见的方式。
[root@aliyun-hk1 linux-shell-test]# cat test-while.sh
#!/bin/bash
var1=5
while [ $var1 -gt 0 ]
do
    echo $var1
    var1=$[ $var1 -1 ]
done
[root@aliyun-hk1 linux-shell-test]# chmod 755 test-while.sh
[root@aliyun-hk1 linux-shell-test]# ./test-while.sh
5
4
3
2
1
[root@aliyun-hk1 linux-shell-test]#
```

##### 2.2 使用多个测试命令的while循环
```
多个测试命令的时候，只有最后一个测试命令退出码用来判单循环是否继续执行，前面的测试命令忽略。
[root@aliyun-hk1 linux-shell-test]# cat test-while.sh
#!/bin/bash
var1=5
while echo $var1
      [ $var1 -ge 0 ]
do
    echo $var1
    var1=$[ $var1 -1 ]
done
[root@aliyun-hk1 linux-shell-test]# ./test-while.sh
5
5
4
4
3
3
2
2
1
1
0
0
-1
[root@aliyun-hk1 linux-shell-test]#
```

#### 3.until循环
until循环其实碰巧跟while循环相反，只要测试命令的退出返回码非0，就一直执行后面的一组命令，直到退出返回码为0，循环才结束。
```
[root@aliyun-hk1 linux-shell-test]# cat test-until.sh
#!/bin/bash

var1=5
until [ $var1 -eq 0 ]
do
   echo $var1
   var1=$[ $var1 -1 ]
done
[root@aliyun-hk1 linux-shell-test]# ./test-until.sh
5
4
3
2
1
[root@aliyun-hk1 linux-shell-test]#
```

#### 4.循环读取多行内容
通过jenkins调用shell并执行ansible脚本的时候，经常需要传递参数到shell。常规的做法可以，把一组内容组装到一个变量中，并且使用换行符作为分隔符，然后传给shell之后，再进行拆分并使用。
```
[root@aliyun-hk1 linux-shell-test]#  var1=`echo -en 'name=robin\nage=21'`
[root@aliyun-hk1 linux-shell-test]# echo "$var1"
name=robin
age=21
[root@aliyun-hk1 linux-shell-test]# cat for.sh
#!/bin/bash

input1=$1
IFS=$'\n'
declare -A arr
for a1 in $input1
do
   #echo $a1
   left1=${a1%=*}
   right1=${a1#*=}
   arr["$left1"]=$right1
done
name=${arr[name]}
echo "name:$name"
age=${arr[age]}
echo "age:$age"
[root@aliyun-hk1 linux-shell-test]# ./for.sh "$var1"
name:robin
age:21
[root@aliyun-hk1 linux-shell-test]#
```
**这里的变量var1使用的时候，一定要用双引号包起来，因为变量里有特殊字符，换行符。我们现在需要特殊字符正常解析，当然要使用双引号。**
