---
title: Linux文本查找命令-grep
date: 2020-03-02 22:23:23
tags: shell
categories: linux
---

**grep**搜索命名的输入文件（如果没有命名文件，则搜索标准输入）以查找包含与给定PATTERN匹配的行。默认情况下，grep打印匹配的整行。
<!--more-->

**example1**： 搜索一个文件，最普通模式
```
[root@aliyun-hk1 linux-shell-test]# grep hello grep1.sh
hello
```

**example2**：递归搜索一个目录下的所有文件
```
[root@aliyun-hk1 linux-shell-test]# grep python2 /etc/
grep: /etc/: Is a directory
[root@aliyun-hk1 linux-shell-test]# grep -r python2 /etc/
/etc/rpm/macros.python:  CFLAGS="%{optflags}" %{__python} %{py_setup} %{?py_setup_args} build --executable="%{__python2} %{py_shbang_opts}" %{?*}
/etc/rpm/macros.python:  if (string.starts(package, "python2-")) then
/etc/rpm/macros.python:    print("Provides: python2-"
```

**example3**：从一个标准输入搜索关键字，从管道搜索
```
[root@aliyun-hk1 linux-shell-test]# ps -ef|grep python
root       803     1  0 Feb28 ?        00:00:34 /usr/bin/python -Es /usr/sbin/tuned -l -P
root     25027 24873  0 21:34 pts/0    00:00:00 grep --color=auto python
[root@aliyun-hk1 linux-shell-test]#
```

**example4**:  搜索时忽略大小写，使用-i
```
[root@aliyun-hk1 linux-shell-test]# grep Hello grep1.sh
[root@aliyun-hk1 linux-shell-test]# grep -i Hello grep1.sh
hello
[root@aliyun-hk1 linux-shell-test]# ps -ef|grep -i Python
root       803     1  0 Feb28 ?        00:00:34 /usr/bin/python -Es /usr/sbin/tuned -l -P
root     25054 24873  0 21:40 pts/0    00:00:00 grep --color=auto -i Python
[root@aliyun-hk1 linux-shell-test]#
```

**example5**: 搜索时排除某些关键字，使用-v
```
[root@aliyun-hk1 linux-shell-test]# ps -ef|grep -i Python
root       803     1  0 Feb28 ?        00:00:34 /usr/bin/python -Es /usr/sbin/tuned -l -P
root     25063 24873  0 21:42 pts/0    00:00:00 grep --color=auto -i Python
[root@aliyun-hk1 linux-shell-test]# ps -ef|grep -i Python|grep -v grep
root       803     1  0 Feb28 ?        00:00:34 /usr/bin/python -Es /usr/sbin/tuned -l -P
[root@aliyun-hk1 linux-shell-test]#
```

**example6**:  搜索时使用正则表达式，-e
```
grep –e "正则表达式" 文件名
```

**example7**: 搜索结果只返回匹配的文件名，-l
```
[root@aliyun-hk1 linux-shell-test]# grep -il Hello grep1.sh
grep1.sh
```

**example8**: 搜索结果只返回不匹配的文件名，-L
```
[root@aliyun-hk1 linux-shell-test]# grep -irL python grep1.sh
grep1.sh
[root@aliyun-hk1 linux-shell-test]# grep -irL Hello grep1.sh
[root@aliyun-hk1 linux-shell-test]#
```

**example9**: 结合find命令搜索log或者config文件
```
[root@aliyun-hk1 linux-shell-test]# find / -name *.log|xargs grep -il error
/var/log/yum.log
/var/log/cloud-init.log
/var/log/nginx/access.log
/var/log/nginx/error.log
/usr/local/share/aliyun-assist/1.0.1.259/log/aliyun_assist_update.log
/usr/local/share/aliyun-assist/1.0.1.259/log/aliyun_assist_main.log
/usr/share/doc/libjpeg-turbo-1.2.90/change.log
```

**example10**: 搜索gz或tar.gz文件，使用zgrep​
```
[root@aliyun-hk1 linux-shell-test]# zgrep -ai hello grep1.tar.gz                              
echo1.sh                                                                                      
ustar   root                            root                                                  
hello                                                                                         
hello                                                                                         
                                                                                              
grep1.sh                                                                                      
ustar   root                            root                                                  
hello world                                                                                   
hEllo word                                                                                    
HEllo world                                                                                   
hellO sir                                                                                     
[root@aliyun-hk1 linux-shell-test]# zgrep -i hello grep1.sh.gz                                
hello world                                                                                   
hEllo word                                                                                    
HEllo world                                                                                   
hellO sir                                                                                     
[root@aliyun-hk1 linux-shell-test]#                                                           
```

**example11**: 更多参数请查看文档，man grep