---
title: Ansible常见错误解析
date: 2020-02-23 18:00:00
tags: ansible-error
categories: ansible
---

# 背景
由于工作中经常用到ansible,所以整理了常用的ansible错误及原因分析，方便自己也方便别人参考。

<!--more-->

# 1.shell 模块常见错误
## 1.1 使用shell遇到"msg": "non-zero return code"
### ansible 脚本如下：
```
- name: Check the weblogic without wc
  shell: "ps -ef|grep weblogic|grep -v grep"
  register: check_weblogic0
  ignore_errors: true
```

### ansible 返回错误:
```
TASK [Check the weblogic without wc] *********************************************************************************************************************************************************************************************************fatal: [robin.org.cn]: FAILED! => {"changed": true, "cmd": "ps -ef|grep weblogic|grep -v grep", "delta": "0:00:00.036565", "end": "2020-02-23 18:08:03.100106", "msg": "non-zero return code", "rc": 1, "start": "2020-02-23 18:08:03.063541", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring

ok: [robin.org.cn] => {
    "msg": {
        "changed": true,
        "cmd": "ps -ef|grep weblogic|grep -v grep",
        "delta": "0:00:00.036565",
        "end": "2020-02-23 18:08:03.100106",
        "failed": true,
        "msg": "non-zero return code",
        "rc": 1,
        "start": "2020-02-23 18:08:03.063541",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "",
        "stdout_lines": []
    }
}
```

### 原因分析：
当使用shell模块并且返回为空的时候，ansible就会认为shell脚本出错了，rc就返回1。

### 解决方案：
在shell命令末尾增加cat，将返回的内容通过管道传递给cat，使用cat返回的rc始终为0. 最好的解决方式，无论你要获取整个返回内容或者返回行数。
```
- name: Check the weblogic without wc but use cat
  shell: "ps -ef|grep weblogic|grep -v grep|cat"
  register: check_weblogic1
  ignore_errors: true

- name: print the check_weblogic1
  debug:
    msg: "{{ check_weblogic1 }}"
```

在shell命令末尾增加wc -l，计算返回的行数，保证shell返回始终不为空。
```
- name: Check the weblogic with wc
  shell: "ps -ef|grep weblogic|grep -v grep|wc -l"
  register: check_weblogic2
  ignore_errors: true

- name: print the check_weblogic2
  debug:
    msg: "{{ check_weblogic2.stdout|int }}"
```

在脚本最后面增加ignore_errors: true，最不推荐的方式，除非暂时没找到根本原因，应急。
```
- name: Check the weblogic without wc
  shell: "ps -ef|grep weblogic|grep -v grep"
  register: check_weblogic0
  ignore_errors: true
```

## 1.2 使用shell遇到"/bin/sh: pip: command not found"错误

```
- name: pip insall web.py
  shell: "pip install requests"
  async: 200
  poll: 0
  register: pip_install_status
```

```
TASK [YUM - check on fire and forget task] ***************************************************************************************************************************************************************************************************fatal: [gcp2.lhh.pub]: FAILED! => {"ansible_job_id": "943105447304.15557", "attempts": 1, "changed": true, "cmd": "pip install requests", "delta": "0:00:00.003629", "end": "2020-02-24 15:05:40.012909", "finished": 1, "msg": "non-zero return code", "rc": 127, "start": "2020-02-24 15:05:40.009280", "stderr": "/bin/sh: pip: command not found", "stderr_lines": ["/bin/sh: pip: command not found"], "stdout": "", "stdout_lines": []}
```
### 原因分析
使用ansible的shell模块的时候，默认不会完全使用当前shell的环境变量PATH,只是导入最基本的几个目录,请看我使用echo $PATH打印的结果。
```

TASK [echo teh result2] **********************************************************************************************************************************************************************************************************************ok: [gcp.lhh.pub] => {
    "msg": {
        "ansible_job_id": "479995747105.21088",
        "attempts": 1,
        "changed": true,
        "cmd": "echo $PATH",
        "delta": "0:00:00.005452",
        "end": "2020-02-24 15:16:30.108684",
        "failed": false,
        "finished": 1,
        "rc": 0,
        "start": "2020-02-24 15:16:30.103232",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "/sbin:/bin:/usr/sbin:/usr/bin",
        "stdout_lines": [
            "/sbin:/bin:/usr/sbin:/usr/bin"
        ]
    }
}
```
### 解决方案
如果要使用的命令在/sbin:/bin:/usr/sbin:/usr/bin没有，则使用命令的绝对路径或者事先导入常用的所有环境变量。
```
 - name: import all the path and run pip insall web.py
   shell: "export PATH=$PATH:/usr/bin/:/usr/local/bin/;pip install web.py"
   async: 20
   poll: 10
   register: rst1
```


# 2.copy模块常见错误
## 2.1 使用copy模块，遇到Remote copy does not support recursive copy of directory
`ansible all -m copy -a 'src=/root/ansible/file1 dest=/etc/cc/file1 remote_src=yes backup=yes mode=0755'`

```
TASK [cp files below folder4 to bak1] *************************************************************
ok: [localhost] => (item=subfile1)
ok: [localhost] => (item=subfile2)
failed: [localhost] (item=subfolder1) => {"changed": false, "item": "subfolder1", "msg": "Remote copy does not support recursive copy of directory: /apps/ansible-test/folder4/subfolder1"}
        to retry, use: --limit @/apps/ansible-test/test-cp.retry

PLAY RECAP ****************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=1

```

### 原因分析：
如果在远程机器上执行copy，相当于在远端机器本机执行cp命令，remote_src: true。对于asible 2.6，只支持copy单个文件，不允许递归copy。对于ansible 2.8 已经支持递归复制。详见官方说明：https://docs.ansible.com/ansible/latest/modules/copy_module.html

### 解决方案：
使用ansible 2.8 或者 使用linux shell cp -rf实现递归复制。
`ansible all -m shell -a 'cp -rf /root/ansible/* /etc/cc/file1'`