---
title: Linux文本输出命令-echo
date: 2020-03-01 22:50:50
tags: shell
categories: linux
---

**echo** 在linux帮助文档的描述是显示一行文本，类似于python和java等编程语言中的print语句，实际上它的作用不仅仅如此。可以使用man echo查看详细的参数说明。
<!--more-->

**example1:** 显示一行文本，任何特殊字符都不会被转义
```
[root@aliyun-hk1 linux-shell-test]# echo hello\nworld
hellonworld
[root@aliyun-hk1 linux-shell-test]# echo 'hello\nworld'
hello\nworld
[root@aliyun-hk1 linux-shell-test]# echo hello world
hello world
[root@aliyun-hk1 linux-shell-test]#
```

**example2:** 显示一行文本，不要输出末尾的换行符
```
[root@aliyun-hk1 linux-shell-test]# echo -n hello world
hello world[root@aliyun-hk1 linux-shell-test]# echo hello world
hello world
```

**example3:** 显示一行文本，启用反斜杠后面的转义字符
```
[root@aliyun-hk1 linux-shell-test]# echo -e 'hello\nworld'
hello
world
[root@aliyun-hk1 linux-shell-test]# echo -e 'hello\tworld'
hello   world
```

**example4:** 显示一行文本，禁用反斜杠后面的转义字符，echo默认参数
```
[root@aliyun-hk1 linux-shell-test]# echo -E 'hello\nworld'
hello\nworld
[root@aliyun-hk1 linux-shell-test]# echo -E 'hello\tworld'
hello\tworld
```

**example5:** echo与cat的差异对比，echo只用于输出文本，cat用于输出文件内容或者从标准输入中输出
```
[root@aliyun-hk1 linux-shell-test]# echo hello
hello
[root@aliyun-hk1 linux-shell-test]# cat hello
cat: hello: No such file or directory
[root@aliyun-hk1 linux-shell-test]# echo /etc/hostname
/etc/hostname
[root@aliyun-hk1 linux-shell-test]# cat /etc/hostname
aliyun-hk1
[root@aliyun-hk1 linux-shell-test]# echo hello|cat
hello
[root@aliyun-hk1 linux-shell-test]#
```

**examle6:** echo在自动化构建中的作用，例如我们可以将DB中返回的数据格式化成ansible需要的数据，通过with_lines 传入某个task并循环使用。在某些情况下，从网络、DB等方式获取的标准输出，可以通过echo结合awk和grep等实现结果的格式化或数据清洗，然后用到后续的​脚本中。
```
[root@aliyun-hk1 linux-shell-test]# echo -en 'name phone addr\nrobin 13712345678 CN\ntom 13812345678 HK\n'
name phone addr
robin 13712345678 CN
tom 13812345678 HK
[root@aliyun-hk1 linux-shell-test]# echo -en 'name phone addr\nrobin 13712345678 CN\ntom 13812345678 HK\n'|awk  'NR>1 {print $1}'
robin
tom
- name: show the items from DB
      debug:
        msg: "{{ item }}"
      with_lines: "echo -en 'name phone addr\nrobin 13712345678 CN\ntom 13812345678 HK\n'|awk  'NR>1 {print $1}'
​
TASK [show the items from DB] ****************************************************************************************************************************************************************************************************************ok: [localhost] => (item=robin) => {
    "msg": "robin"
}
ok: [localhost] => (item=tom) => {
    "msg": "tom"
}
```

**example7:**  echo还可以将获取到并格式化好的数据写入到一个文件，等待后续使用​。
```
[root@aliyun-hk1 ansible-test]# echo -en 'name phone addr\nrobin 13712345678 CN\ntom 13812345678 HK\n'|awk  'NR>1 {print $1}' > DataFromDB1.txt
[root@aliyun-hk1 ansible-test]# cat DataFromDB1.txt
robin
tom
[root@aliyun-hk1 ansible-test]#
```