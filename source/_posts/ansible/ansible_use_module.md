---
title: ansible 模块使用和深入解析
date: 2020-02-12 00:18:18
tags: ansible-module
categories: ansible
---

今天在学习阿里云提供的免费ansible视频课程，特此做个笔记，虽然内容比较基础，但是挺重要的内容。
<!--more-->

### 1.ansible 主配置文件

/etc/ansible/ansible.cfg

remote_tmp 远程临时目录,默认为~/.ansible/tmp

local_tmp 本地临时目录，默认为~/.ansible/tmp

forks 并发执行的数量，默认为5

host_key_checking = False 是否验证对方的公钥，建议取消注释

log_path=/var/log/ansible.log 建议开启ansible的执行日志



### 2.ansible 常用命令

ansible --help

ansible-doc

ansible-doc -l 列出可用模块

ansible-doc ping 查看模块的文档

ansible-doc -s ping 大致了解模块的文档


### 3.ansible 简单方式使用
`ansible <host-pattern> [-m module_name] [-a args]`

* 查看某个组server
`ansible group1 --list`

* 在远程机器上执行ls
`ansible group -u user1 -k -m command -a 'ls /root' -b`
 

### 4.基于key验证的方式
`ssh-keygen`
`ssh-copy-id 192.168.30.101`

### 5.ansible 主机模式

```
ansible al -m ping

ansible group1 -m ping

ansible *roup1 -m ping

ansible 'group1:&group2' -m ping

ansible 'group1:!group2' -m ping
```


### 6.ansible执行过程
ansibe all -m ping -vvv 查看执行过程
加载自己的配置文件，例如ansible.cfg
加载自己的模块文件
生成临时的py文件，并复制到远程
给文件加+x权限
执行并返回结果
删除临时的py文件，sleep 0 退出

执行状态：
绿色：执行成功并不需要做改变
黄色：执行成功并做了修改

### 7.ansible常用模块
#### command使用，特殊符号不能解析。
ansible all -m command -a 'ifconfig'
ansible all -m command -a 'chdir=/data test.sh'


#### shell使用，管道和引用变量等都可以解析。
`ansible all -m shell -a 'chdir=/data test.sh'`

#### script使用，将ansible主控端的脚本，在目标机执行。
`ansible all -m script -a '/root/ansible/host.sh'`


#### copy使用，将主控制机的文件copy到目标机器。
`ansible all -m copy -a 'src=/root/ansible/file1 dest=/etc/cc/file1 backup=yes mode=0755'`
注意：如果在远程机器上执行copy，相当于在远端机器本机执行cp命令，remote_src: true。
对于asible 2.6，只支持copy单个文件，不允许递归copy目录及下面的文件,会抛出如下错误。
对于ansible 2.8 已经支持递归复制。
```
TASK [cp files below folder4 to bak1] *************************************************************
ok: [localhost] => (item=subfile1)
ok: [localhost] => (item=subfile2)
failed: [localhost] (item=subfolder1) => {"changed": false, "item": "subfolder1", "msg": "Remote co               py does not support recursive copy of directory: /apps/ansible-test/folder4/subfolder1"}
        to retry, use: --limit @/apps/ansible-test/test-cp.retry

PLAY RECAP ****************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=1

```


#### fetch使用，客户端文件抓取到服务端
`ansible all -m fetch -a 'src=/var/log/message des=/data/'`
`ansible all -m shell -a 'tar -zcvf log.tar.gz /var/log/*.log'`


#### file使用，设置文件属性或创建文件
* 新建文件
`ansible all -m file -a 'name=/data/file1.txt status=touch'`


* 查看文件
`ansible all -m shell -a 'ls -l /data'`


* 删除文件
`ansible all -m file -a 'name=/data/file1.txt status=absent'`


* 创建文件夹
`ansible all -m file -a 'name=/data/file1 status=directory'`


* 删除文件夹
`ansible all -m file -a 'name=/data/file1 status=absent'`


* 创建软连接
`ansible all -m file -a 'src=/etc/fstab dest=/data/fatab.link status=link'`


* 删除软链接
`ansible all -m file -a 'dest=/data/fatab.link status=absent'`


#### hostname模块使用
* 修改hostname
`ansible ip1 -m hostname -a 'name=node1'`


#### Cron 计划任务模块
`ansible all -m cron -a 'minute=* weekday=1,3,5 job="/usr/bin/wall warning" name=warning'`


* 禁用计划任务
`ansible all -m cron -a 'disabled=true job="/usr/bin/wall warning" name=warning'`


* 启用计划任务
`ansible all -m cron -a 'disabled=false job="/usr/bin/wall warning" name=warning'`


* 删除计划任务
`ansible all -m cron -a ' job="/usr/bin/wall warning" name=warning status=absent'`


#### yum 模块的使用
* 使用yum安装vsftpd
`ansible all -m yum -a 'name=vsftpd,httpd,'`


* 查看已经安装的包
`ansible all -m yum -a 'list=installed'`

* 将ansible core server上的yum包安装到远程主机
```
ansible all -m copy 'src=/data/ssss.rpm dest=/root/'
ansible all -m yum -a 'name=/root/ssss.rpm'
```

* 更新yum缓存
`ansible all -m yum -a 'name=dstat update_cache=yes'`

#### service 模块的使用
* 将服务设置为开机启动
`ansible all -m service -a 'name=vsftpd state=started enabled=yes'`

#### User 模块的使用
* 创建njs账号
`ansible all -m user -a 'name=njs shell=/sbin/nologin system=yes home=/var/njs groups=root,bin uid=8888 comment=nginx'`

* 删除njs账号
`ansible all -m user -a 'name=njs status=absent'`

#### Group 模块的使用
* 创建njs组
`ansible all -m group -a 'name=njs system=yes gid=8080'`

* 删除njs组
`ansible all -m group -a 'name=njs status=absend'`