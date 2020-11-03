---
title: Linux之shell输入输出重定向
date: 2020-03-15 16:25:25
tags: shell
categories: linux
---

之前我们讲解shell的变量、环境变量以及各种命令的时候，命令执行结果输出每次都是显示到屏幕上给用户看，其实还有更多的方式。今天我们就专门讲讲输入和输出重定向。
<!--more-->

#### 输出重定向
##### 将命令的标准输出写入文件或黑洞/dev/null
可以使用>或1>将一个命令的输出**覆盖写入到一个文件**，如果这个文件不存在，系统会自动创建，如果文件已存在，内容会被完全覆盖。
```
[root@aliyun-hk1 linux-shell-test]# echo 'hello world' > rs.out
[root@aliyun-hk1 linux-shell-test]# cat rs.out
hello world
[root@aliyun-hk1 linux-shell-test]# echo 'hello world' 1> rs.out
[root@aliyun-hk1 linux-shell-test]# cat rs.out
hello world
[root@aliyun-hk1 linux-shell-test]#
```

可以使用>>或1>>将一个命令的输出**追加写入到一个文件**,如果这个文件不存在，系统会自动创建，如果文件已存在，内容会被追加过去。
```
[root@aliyun-hk1 linux-shell-test]# echo 'hello china' >> rs.out
[root@aliyun-hk1 linux-shell-test]# cat rs.out
hello world
hello china
[root@aliyun-hk1 linux-shell-test]# echo 'hello china' 1>> rs.out
[root@aliyun-hk1 linux-shell-test]# cat rs.out
hello world
hello china
hello china
[root@aliyun-hk1 linux-shell-test]#
```

##### 将命令的标准错误输出写入文件或黑洞/dev/null
使用 2> 和 2>> 将命令的标准错误输出覆盖写入或追加写入到一个文件,如果命令执行无错误，则输出空字符到文件。
```
[root@aliyun-hk1 linux-shell-test]# cat he1.out 2>> rs3.out
[root@aliyun-hk1 linux-shell-test]# cat rs3.out
cat: he1.out: No such file or directory
[root@aliyun-hk1 linux-shell-test]#

[root@aliyun-hk1 linux-shell-test]# cat rs.out 2>> rs4.out
hello world
hello china
[root@aliyun-hk1 linux-shell-test]# cat rs4.out
[root@aliyun-hk1 linux-shell-test]#
[root@aliyun-hk1 linux-shell-test]# echo 'hello' 2> rs.out
hello
[root@aliyun-hk1 linux-shell-test]# cat rs.out
```

#### 输入重定向
##### 将文件的内容重定向到命令
可以使用<或0<将一个文件的内容输入到一个命令,此例是将一个文件的内容传递给wc，然后计算文件中字符的行数。
```
[root@aliyun-hk1 linux-shell-test]# cat rs.out
hello world
hello china
[root@aliyun-hk1 linux-shell-test]# wc -l < rs.out
2
[root@aliyun-hk1 linux-shell-test]# wc -l 0< rs.out
2
```

可以使用<<实现内联输入重定向，不需要通过文件，你只要指定结束符，逐个手动输入数据就可以了。
```
[root@aliyun-hk1 linux-shell-test]# wc -l << "eof"
> hell
> world
> hi
> hao
> eof
4
```

#### 输入输出总结
```
标准输入(stdin): 代码为0，使用<或<<
标准输出(stdout): 代码为1，使用>或>>
标准错误输出(stdout):代码为2,使用2>或2>>
全部输出到一个文件，使用2>&1
```

#### shell输出回传码
用户每次在shell中执行一个命令，shell都会回传一个状态码，0或1，可以通过echo $?获取，0代表执行成功，非0代表执行失败。
```
[root@aliyun-hk1 linux-shell-test]# echo hello
hello
[root@aliyun-hk1 linux-shell-test]# echo $?
0
[root@aliyun-hk1 linux-shell-test]# echo hello world
hello world
[root@aliyun-hk1 linux-shell-test]# echo $?
0
[root@aliyun-hk1 linux-shell-test]# name=robin li
-bash: li: command not found
[root@aliyun-hk1 linux-shell-test]# echo $?
127
[root@aliyun-hk1 linux-shell-test]#
```

#### Linux常见的退出状态码
```
0 命令成功退出
1 一般性未知错误
2 不适合的shell命令
126 命令不可执行
127 没找到命令
128 无效的退出参数
130 通过CTRL+C终止的命令
255 正常范围之外的退出状态码
```


