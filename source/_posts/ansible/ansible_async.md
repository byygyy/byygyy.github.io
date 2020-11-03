---
title: ansible异步任务
date: 2019-11-22 00:24:30
tags: ansible-module
categories: ansible
---

### 转载于简书博客 https://www.jianshu.com/p/3962bf94ae70

ansible方便在于能批量下发，并返回结果和呈现。简单、高效。
但有的任务执行起来却不那么直接，可能会花比较长的时间，甚至可能会比ssh的超时时间还要长。这种情况任务是不是没法执行了？
ansible考虑到了这种情况，官方文档介绍了这个问题的解决方法，就是让下发的任务执行的连接变为异步：任务下发之后，长连接不再保持，而是每隔一段时间轮询结果，直到任务结束。

<!--more-->

这是官网相关的介绍
他们在playbook的任务中加入两个参数：async和poll。

* async参数值代表了这个任务执行时间的上限值。即任务执行所用时间如果超出这个时间，则认为任务失败。此参数若未设置，则为同步执行。

* poll参数值代表了任务异步执行时轮询的时间间隔。
官方给出例子：

```
  ----
    hosts: all
    remote_user: root
    tasks:
      - name: simulate long running op (15 sec), wait for up to 45 sec, poll every 5 sec
        command: /bin/sleep 15
        async: 45
        poll: 5
```

这时候已经不怕任务超时了。可以执行一个45s的任务，当然也可以根据需要自己设置。另外，如果poll为0，就相当于一个不关心结果的任务。

如果还想要更方便地看轮询结果，ansible还提供了这个模块async_status。

```
  ---
    # Requires ansible 1.8+
    - name: 'YUM - fire and forget task'
      yum: name=docker-io state=installed
      async: 1000
      poll: 0
      register: yum_sleeper

    - name: 'YUM - check on fire and forget task'
      async_status: jid={{ yum_sleeper.ansible_job_id }}
      register: job_result
      until: job_result.finished
      retries: 30
```

第一个job执行异步任务，并且注册了一个名字叫yum_sleeper，用于提供给第二个job作为轮询对象，并且poll设为0，它自己不再轮询。
第二个job使用async_status模块，进行轮询并返回轮询结果。准备检查30次。结果如下：

```
PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [cloudlab001]

TASK [YUM - fire and forget task] **********************************************
ok: [cloudlab001]

TASK [YUM - check on fire and forget task] *************************************
FAILED - RETRYING: TASK: YUM - check on fire and forget task (29 retries left).
FAILED - RETRYING: TASK: YUM - check on fire and forget task (28 retries left).
FAILED - RETRYING: TASK: YUM - check on fire and forget task (27 retries left).
FAILED - RETRYING: TASK: YUM - check on fire and forget task (26 retries left).
FAILED - RETRYING: TASK: YUM - check on fire and forget task (25 retries left).
FAILED - RETRYING: TASK: YUM - check on fire and forget task (24 retries left).
changed: [cloudlab001]

PLAY RECAP *********************************************************************
cloudlab001                : ok=3    changed=1    unreachable=0    failed=0
```