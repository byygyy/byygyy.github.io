---
title: Linux之管道命令
date: 2020-03-08 23:08:08
tags: shell
categories: linux
---

在linux世界，虽然你不一定熟悉**管道**的概念，但是你肯定已经用过了。当Bash命令执行的时候大部分时候都会有数据输出，如果这个输出的数据还要传递给其他的命令继续处理，就要用到管道(pipe)命令了，管道命令使用的是“|”这个符号。
<!--more-->

example1: 查询docker进程是否存在，我们将ps命令查询的结果通过管道传递给grep命令，再进行二次刷选，结果就出来了。**注意：这个管道命令“|”仅能处理经由前面一个命令传来的正确信息，也就是standard output的信息**，对于standard error并没有直接处理的能力。**管道后面必须是“命令”，而且这个命令必须可以接收standard input的数据才行**，例如less,more,grep等，至于例如ls,cp,mv就不会接收来自stdin的数据。
```
[root@aliyun-hk1 ~]# ps -ef|grep docker
root     10269 10243  0 00:02 pts/0    00:00:00 grep --color=auto docker
root     29006     1  0 Mar02 ?        00:06:53 /usr/bin/dockerd-current --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current --default-runtime=docker-runc --exec-opt native.cgroupdriver=systemd --userland-proxy-path=/usr/libexec/docker/docker-proxy-current --init-path=/usr/libexec/docker/docker-init-current --seccomp-profile=/etc/docker/seccomp.json --selinux-enabled --log-driver=journald --signature-verification=false --storage-driver overlay2
root     29011 29006  0 Mar02 ?        00:04:31 /usr/bin/docker-containerd-current -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --shim docker-containerd-shim --runtime docker-runc --runtime-args --systemd-cgroup=true
root     31808 29006  0 Mar02 ?        00:00:00 /usr/libexec/docker/docker-proxy-current -proto tcp -host-ip 0.0.0.0 -host-port 8081 -container-ip 172.17.0.2 -container-port 80
root     31826 29011  0 Mar02 ?        00:00:00 /usr/bin/docker-containerd-shim-current b380c65a2b75d32f53a56f17b1dd816449bfaf0048996b783a47daf023bb7870 /var/run/docker/libcontainerd/b380c65a2b75d32f53a56f17b1dd816449bfaf0048996b783a47daf023bb7870 /usr/libexec/docker/docker-runc-current
[root@aliyun-hk1 ~]#
```

example2：支持管道，也就是支持standard input的命令，这些命令用法后面章节细讲。
```
字符选取命令 cut,grep
字符显示命令 less,more,head,tail等文本处理命令
字符排序命令 sort,wc,uniq
双重定向命令 tee
字符转换命令 tr,col,join,paste,expand
```

example3：不支持管道，也就是不支持standard input的命令。
```
文件或目录操作命令 cp,mv,ls
```

example4：**不支持管道的命令，通过xargs传递数据给下一个命令**。xargs可以读入standard input的数据，并且以**空格符或断行字符**进行分辨，将stdin的数据分隔成为命令行参数，供其他命令使用。查询包含docker的文件，并且统计文件数量。
```
xargs [-0epn] command
[root@aliyun-hk1 /]# find . -name docker|xargs ls -alt|wc -l
158
```

example5：xargs能够处理管道或者stdin并将其转换成特定命令的命令参数。xargs也可以将单行或多行文本输入转换为其他格式，例如**多行变单行，单行变多行**。使用xargs是单行输出全部，使用xargs -nx每行输出x个，
```
[root@aliyun-hk1 linux-shell-test]# cat xargs.sh          
hello                                                     
world                                                     
ni                                                        
hao                                                       
[root@aliyun-hk1 linux-shell-test]# cat xargs.sh|xargs    
hello world ni hao                                        
[root@aliyun-hk1 linux-shell-test]# cat xargs.sh|xargs -n2
hello world                                               
ni hao                                                    
[root@aliyun-hk1 linux-shell-test]# cat xargs.sh|xargs -n3
hello world ni                                            
hao                                                       
[root@aliyun-hk1 linux-shell-test]# cat xargs.sh|xargs -n4
hello world ni hao                                        
[root@aliyun-hk1 linux-shell-test]#                       
​```

example6​：xargs使用非默认的分隔符，默认的分隔符为空格或​断行字符。
```
[root@aliyun-hk1 linux-shell-test]# cat xargs2.sh
helloXworldXnihaoXzhongguo
[root@aliyun-hk1 linux-shell-test]# cat xargs2.sh|xargs -dX -n2
hello world
nihao zhongguo
[root@aliyun-hk1 linux-shell-test]# cat xargs2.sh|xargs -dX
hello world nihao zhongguo
[root@aliyun-hk1 linux-shell-test]#
```

example7: xargs使用-I 和-i作为参数​，将管道的stdout转成stdin并逐个在后面命令中使用。
```
[root@aliyun-hk1 linux-shell-test]# cat xargs.sh
hello
world
ni
hao
[root@aliyun-hk1 linux-shell-test]# cat xargs.sh|xargs -I {} echo 'this line is {}'
this line is hello
this line is world
this line is ni
this line is hao
[root@aliyun-hk1 linux-shell-test]# cat xargs.sh|xargs -i echo 'this line is {}'
this line is hello
this line is world
this line is ni
this line is hao
```

example8: xargs逐行显示输出给后面命令的参数，尤其当你需要将文件查找的结果传递给sed的时候，并且sed使用了-i参数的时候。
```
[root@aliyun-hk1 linux-shell-test]# cat xargs.sh|xargs -t -i sed -i 's/hello/hallo/g' {}
sed -i s/hello/hallo/g xargs.sh
sed -i s/hello/hallo/g xargs1.sh
sed -i s/hello/hallo/g xargs2.sh
```