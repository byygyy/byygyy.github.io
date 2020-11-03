---
title: Linux之shell输出数据
date: 2020-03-21 23:00:00
tags: shell
categories: linux
---

讲完了如何处理用户的输入，接下来我在讲讲对应的shell输出数据。
<!--more-->

#### 1.1 理解输入和输出
至此你应该已经知道两种显示脚本输出的方式，在屏幕上显示输出或将输出重定向到文件。这两种方式，要么什么都显示出来，要么都不显示。但有时需要将一部分显示到频幕上，另一部分保存到文件。学习了如何处理输入输出后，就能帮助你将脚本输出放到正确的位置。

##### 1.1.1 标准文件描述符
linux系统将每个对象都当作文件处理，这包括输入和输出的进程。Linux系统本来支持9中文件描述符，出于特殊目的，bash shell只保留了前3个文件描述符(0,1,2)。
文件描述符      | 缩写  |描述 |对应位置
--------------  | ----- |-----|----
0  | STDIN      |标准输入|键盘
1  | STDOUT     |标准输出|屏幕
2  | STDERR     |标准错误|屏幕
*这三个特殊文件描述符会处理脚本的输入和输出。shell用他们将shell默认的输入和输出导向响应的位置。*

#### 1.2 交互式shell输出重新向
输出重定向，主要是将STDOUT和STDERR的数据重新指向到非默认的位置，下来来几个示例。使用2>将标准错误输出写入到一个文件，使用1>将标准输出写入到另外一个文件。

##### 1.2.1 将STDERR重定向到一个文件
这个时候如果有标准输出，则会显示在屏幕上。
```
[root@aliyun-hk1 linux-shell-test]# ls hello 2> hello.log
[root@aliyun-hk1 linux-shell-test]# cat hello.log
ls: cannot access hello: No such file or directory
[root@aliyun-hk1 linux-shell-test]# ls rs1.out hello  2> rss1.log
rs1.out
[root@aliyun-hk1 linux-shell-test]# cat rss1.log
ls: cannot access hello: No such file or directory
[root@aliyun-hk1 linux-shell-test]#​
```

##### 1.2.2 将STDERR和STDOUT的分别重定向到不同的文件
```
[root@aliyun-hk1 linux-shell-test]# ls rs1.out hello 2>hello.log 1>rs1.log
[root@aliyun-hk1 linux-shell-test]# cat hello.log
ls: cannot access hello: No such file or directory
[root@aliyun-hk1 linux-shell-test]# cat rs1.log
rs1.out
[root@aliyun-hk1 linux-shell-test]#
```

##### 1.2.3 将STDERR和STDOUT的重定向到同一个文件
```
[root@aliyun-hk1 linux-shell-test]# ls rs1.out hello  &> rss1.log
[root@aliyun-hk1 linux-shell-test]# cat rss1.log
ls: cannot access hello: No such file or directory
rs1.out
```

#### 1.3 shell脚本输出重定向
默认情况下，shell脚本运行的时候，Linux会将STDERR导向STDOUT，而且会显示在屏幕上。如果运行时重定向了STDERR，脚本中的所有导入STDERR的文本都会被重定向。
##### 1.3.1 临时重定向
运行shell脚本时通过传入参数的方式实现。
```
[root@aliyun-hk1 linux-shell-test]# cat shell-output.sh
#!/bin/bash

echo "this is a error" >&2
echo "this is a normail output"
[root@aliyun-hk1 linux-shell-test]# ./shell-output.sh
this is a error
this is a normail output
[root@aliyun-hk1 linux-shell-test]# ./shell-output.sh 2> test9
this is a normail output
[root@aliyun-hk1 linux-shell-test]# cat test9
this is a error
[root@aliyun-hk1 linux-shell-test]#
```
*脚本内部临时在需要的位置定义一个标准错误输出，外部调用这个shell脚本的时候，直接使用交互式重定向符号。*

##### 1.3.2 永久重定向
在shell脚本内部直接实现，可以使用exec命令告诉shell在脚本执行期间重定向某个特定文件描述符。
```
[root@aliyun-hk1 linux-shell-test]# cat shell-output.sh
#!/bin/bash

exec 1>test_stdout
exec 2>test_stderr
echo "this is a normail output"
ls testt
[root@aliyun-hk1 linux-shell-test]# ./shell-output.sh
[root@aliyun-hk1 linux-shell-test]# cat test_stdout
this is a normail output
[root@aliyun-hk1 linux-shell-test]# cat test_stderr
ls: cannot access testt: No such file or directory
[root@aliyun-hk1 linux-shell-test]#
```
*标准输出被重定向到了test_stdout,标准错误输出重定向到了test_stderr*

#### 1.4 shell脚本输入重定向
正常情况下，shell脚本会从键盘接收STDINT，通俗地说键盘输入内容会直接到标准输入。如果我们使用了输入重定向后，就会告诉shell不从键盘接收标准输入，而是从文件或者其他地方接收输入。这种操作，其实是Linux系统管理员的一项日常任务，例如从日志中读取数据并处理。
```
[root@aliyun-hk1 linux-shell-test]# cat shell-input.sh
#!/bin/bash
#read input from file testfile
exec 0< testfile
count=1

while read line
do
  echo "Line number $count is $line"
  count=$[ $count + 1]
done

[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh
Line number 1 is hello
Line number 2 is world
[root@aliyun-hk1 linux-shell-test]# cat testfile
hello
world
[root@aliyun-hk1 linux-shell-test]#
```

#### 1.5 shell脚本非标准重定向
Linux系统本来支持9中文件描述符，出于特殊目的，bash shell只保留了前3个文件描述符(0,1,2)。我们这里来讲讲如何定义并使用3-8的文件描述符，这个需求有时候可能很迫切，例如程序调试期间。
##### 1.5.1 创建输出文件描述符
使用exec命令来给输出分配非标准的描述符，然后在后面重定向某些输出到这个文件描述符即可。
```
[root@aliyun-hk1 linux-shell-test]# cat shell-output1.sh
#!/bin/bash

exec 3>test3out

echo "this is a normail output"
echo "this is should been to the file" >&3
[root@aliyun-hk1 linux-shell-test]# ./shell-output1.sh
this is a normail output
[root@aliyun-hk1 linux-shell-test]# cat test3out
this is should been to the file
[root@aliyun-hk1 linux-shell-test]#
```
##### 1.5.2 重定向文件描述符
可能由于业务需求，需要将非标准的文件描述符重定向到标准文件描述符。
```
[root@aliyun-hk1 linux-shell-test]# cat shell-output2.sh
#!/bin/bash                                             
                                                        
exec 3>&1                                               
exec 1>test4out                                         
                                                        
echo "this is should been to the file" >&3              
echo "this is a normail output"                         
                                                        
exec 1>&3                                               
echo "this is from 1"                                   
[root@aliyun-hk1 linux-shell-test]# ./shell-output2.sh  
this is should been to the file                         
this is from 1                                          
[root@aliyun-hk1 linux-shell-test]# cat test4out        
this is a normail output                                
[root@aliyun-hk1 linux-shell-test]#                     
```
*第一个exec将文件描述符3重定向到了文件描述符1，但是文件描述符1的默认输出位置是屏幕。第二个exec将文件描述符1重定向到文件，所以文件描述符1的内容就不会在显示屏上打印了，但是文件描述符3仍然指向文件描述符1原来的位置，也就是显示器。第三个exec又将文件描述符1重定向到文件描述符3，即也是输出到屏幕。每种文件描述符本身可以重定向，重定向以后一直有效，除非再次重定向。*

重定向文件描述符，除了在shell脚本内部可以实现，在shell脚本运行的时候，也可以通过参数方式修改。
```
[root@aliyun-hk1 linux-shell-test]# cat shell-output3.sh
#!/bin/bash

echo "this is should been to the file" >&3
echo "this is should been to the file2" >&2
echo "this is a normail output"

[root@aliyun-hk1 linux-shell-test]# ./shell-output3.sh 3>&1 2>&1
this is should been to the file
this is should been to the file2
this is a normail output
[root@aliyun-hk1 linux-shell-test]# ./shell-output3.sh 3>&1>>shellout10 2>&1>shellout10
this is should been to the file
[root@aliyun-hk1 linux-shell-test]# cat shellout10
this is a normail output
e file2
[root@aliyun-hk1 linux-shell-test]#
```
*这样的参数，只是修改了文件描述符3和文件描述符2的输出方式，文件描述符1的输出方式继续保持。*

##### 1.5.3 创建输入文件描述符
我们可以用和重定向输出文件描述符同样的办法，重新向输入文件描述符。重定向之前，可以先将STDIN文件描述符保存到另外一个自定义的文件描述符，然后在读完文件后再将STDIN恢复到它原来的位置。

```
[root@aliyun-hk1 linux-shell-test]# cat shell-input.sh
#!/bin/bash
#read input from file testfile

exec 6<&0
exec 0<testfile

count=1
while read line
do
  echo "line $count is $line"
  count=$[ $count + 1 ]
done

exec 0<&6
[root@aliyun-hk1 linux-shell-test]# cat testfile
hello
world
[root@aliyun-hk1 linux-shell-test]# ./shell-input.sh
line 1 is hello
line 2 is world
[root@aliyun-hk1 linux-shell-test]#

```
#### 1.6 阻止命令输出
有时候可能不想显示脚本的任何输出，可以将STDERR重定向到一个叫做/dev/null的文件。
```
[root@aliyun-hk1 linux-shell-test]# ls -alt > /dev/null
[root@aliyun-hk1 linux-shell-test]# cat /dev/null
[root@aliyun-hk1 linux-shell-test]#
```

有时候想快速清空一个很大的日志文件，可以在输出重定向中，将/dev/null以覆盖的方式输出到某个文件，由于/dev/null是空的，所以文件会被清空。
```
[root@aliyun-hk1 linux-shell-test]# cat testfile
hello
world
[root@aliyun-hk1 linux-shell-test]# cat /dev/null > testfile
[root@aliyun-hk1 linux-shell-test]# cat testfile
[root@aliyun-hk1 linux-shell-test]#
```
更加简单快速清空文件的办法。
```
[root@aliyun-hk1 linux-shell-test]# cat testfile
hello
world
nihao
zhongguo
[root@aliyun-hk1 linux-shell-test]# >testfile
[root@aliyun-hk1 linux-shell-test]# cat testfile
[root@aliyun-hk1 linux-shell-test]#

```

#### 1.7 使用tee命令同时多出输出
有时候你想在显示器显示log，也想让文件保存下，你不用将输出重定向两次，只要使用特殊的命令tee即可。
```
默认是覆盖写入文件
[root@aliyun-hk1 linux-shell-test]# date| tee testfile
Mon Mar 23 23:42:53 CST 2020
[root@aliyun-hk1 linux-shell-test]# cat testfile
Mon Mar 23 23:42:53 CST 2020
[root@aliyun-hk1 linux-shell-test]#

使用非默认的追加写入
[root@aliyun-hk1 linux-shell-test]# date| tee -a testfile
Mon Mar 23 23:43:51 CST 2020
[root@aliyun-hk1 linux-shell-test]# cat testfile
Mon Mar 23 23:42:53 CST 2020
Mon Mar 23 23:43:51 CST 2020
[root@aliyun-hk1 linux-shell-test]#
```