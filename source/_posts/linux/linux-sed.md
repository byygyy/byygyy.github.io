---
title: Linux文本编辑命令-sed
date: 2020-03-04 23:08:08
tags: shell
categories: linux
---

**sed**用于过滤和转换文本的流编辑器。Sed是流编辑器。流编辑器用于对输入流（文件或来自管道的输入）执行基本的文本转换。尽管在某种程度上类似于允许脚本编辑（例如ed）的编辑器，但sed通过仅对输入进行一次传递来工作，因此效率更高。但是sed能够过滤管道中的文本，这使其与其他类型的编辑器特别有区别。grep,sed,awk合起来俗称linux shell三剑客。
<!--more-->

example1: 打印行，sed -n 'n1,n2 p'  file_name
```
[root@aliyun-hk1 linux-shell-test]# sed -n '1 p' sed.sh
hello world 2020#
[root@aliyun-hk1 linux-shell-test]# sed -n '1,3 p' sed.sh
hello world 2020#
hello china 2021#
hello xian 2022#
```
example2: 删除行并且内容不变，sed -n 'n1,n2 d'  file_name
```
[root@aliyun-hk1 linux-shell-test]# sed -n '1 d' sed.sh
[root@aliyun-hk1 linux-shell-test]# cat sed.sh
hello world 2020#
hello china 2021#
hello xian 2022#
[root@aliyun-hk1 linux-shell-test]# sed -n '1,3 d' sed.sh
[root@aliyun-hk1 linux-shell-test]# cat sed.sh
hello world 2020#
hello china 2021#
hello xian 2022#
```

example3: 插入行并且内容不变，sed  'ni top' file_name
```
[root@aliyun-hk1 linux-shell-test]# cat sed.sh          
hello world 2020#                                       
hello china 2021#                                       
hello xian 2022#                                                                                                
[root@aliyun-hk1 linux-shell-test]# sed '2i top' sed.sh 
hello world 2020#                                       
top                                                     
hello china 2021#                                       
hello xian 2022#                                                                                             
[root@aliyun-hk1 linux-shell-test]# sed '2a top' sed.sh 
hello world 2020#                                       
hello china 2021#                                       
top                                                     
hello xian 2022#
```

example4: 替换行且内容不变， sed 'nc newline' file_name
```
[root@aliyun-hk1 linux-shell-test]# cat sed.sh
hello world 2020#
hello china 2021#
hello xian 2022#
[root@aliyun-hk1 linux-shell-test]# sed '2c newline' sed.sh
hello world 2020#
newline
hello xian 2022#
[root@aliyun-hk1 linux-shell-test]# cat sed.sh
hello world 2020#
hello china 2021#
hello xian 2022#
```

example5: 替换每行第一个字符并且内容不变，-n ‘s/old/new/p’
```
[root@aliyun-hk1 linux-shell-test]# cat sed.sh
hello world 2020#
hello china 2021#
hello xian 2022#
[root@aliyun-hk1 linux-shell-test]# sed -n 's/hello/hallo/p' sed.sh
hallo world 2020#
hallo china 2021#
hallo xian 2022#
[root@aliyun-hk1 linux-shell-test]# cat sed.sh
hello world 2020#
hello china 2021#
hello xian 2022#
```

example6: 替换文件中的所有字符且内容不变，-n ‘s/old/new/pg
```
[root@aliyun-hk1 linux-shell-test]# cat sed.sh
hello world 2020#
hello china 2021#
hello xian 2022#
[root@aliyun-hk1 linux-shell-test]# sed -n 's/20/88/pg' sed.sh
hello world 8888#
hello china 8821#
hello xian 8822#
[root@aliyun-hk1 linux-shell-test]# cat sed.sh
hello world 2020#
hello china 2021#
hello xian 2022#
```

example7: 直接对源文件内容进行操作，删除第n行，sed -i
```
[root@aliyun-hk1 linux-shell-test]# cat sed.sh
hello world 9999#
hello china 9921#
hello xian 9922#
[root@aliyun-hk1 linux-shell-test]# sed -i '1 d' sed.sh
[root@aliyun-hk1 linux-shell-test]# cat sed.sh
hello china 9921#
hello xian 9922#
```

example8: 直接对文件内容操作，第n行前插入一行,sed -i​
```
[root@aliyun-hk1 linux-shell-test]# cat sed.sh
hello china 9921#
hello xian 9922#
[root@aliyun-hk1 linux-shell-test]# sed -i '1 i top' sed.sh
[root@aliyun-hk1 linux-shell-test]# cat sed.sh
top
hello china 9921#
hello xian 9922#
```

example9: 直接对源文件内容进行操作，替换字符，sed -i
```
[root@aliyun-hk1 linux-shell-test]# cat sed.sh
hello world 8888#
hello china 8821#
hello xian 8822#
[root@aliyun-hk1 linux-shell-test]# sed -i 's/8/9/g' sed.sh
[root@aliyun-hk1 linux-shell-test]# cat sed.sh
hello world 9999#
hello china 9921#
hello xian 9922#
```

example10: 更多命令请参考，man sed
```
或参阅 https://man.linuxde.net/sed
```