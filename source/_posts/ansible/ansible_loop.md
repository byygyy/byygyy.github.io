---
title: asible之-玩转各种循环
date: 2020-04-19 22:20:30
tags: ansible
categories: ansible
---

使用过ansible的同学都知道，在某些场景下，你不得不去使用循环语句，今天我来总结下ansible循环的各种玩法，并结合实例去理解。
<!--more-->

# 1.with_items的使用
这个应该是大家最常见的ansible循环语句，主要作用是对列表元素进行循环，输入的参数类型为列表。
第一种方式：
```
- name: get the instance name
  shell: "echo {{ item }}"
  with_items:
     - instance1
     - instance2
```
*将实例名传入shell语句中使用，这里传入的是yaml格式的列表数据。*

第二种方式：
```
- name: get the instance name 2st way
  shell: "echo {{ item }}"
  with_items: ["instance1","instance2"]
```
*将实例名传入shell语句中使用，这里传入的是json格式的列表数据。*

第三种方式：
```
vars:
   instances: ["instance1","instance2"]
   
- name: get the instance name 3st way
  shell: "echo {{ item }}"
  with_items: "{{ instances }}"
  register: instances3   
```
`ansible-playbook test-loop.yml -e "hosts_group=vagrant1 instances=[\"instance3\",\"instance4\"]"`
*将实例名传入shell语句中使用，这里传入的是一个列表变量，这个列表变量的值可以被覆盖。*

# 2.with_list的使用
当列表没有嵌套式，with_item 与 with_list 效果完全一样。但是当有列表嵌套时，with_list不会展开每个子list的元素，而with_item会展开每个子list的元组，至于list的定义和传参，这里就不重复讲解了。
使用with_list遍历嵌套list的示例：
```
- debug:
    msg: "{{ item }}"
  with_list:
    - [instance1,instance3]
    - [instance2,instance4]

or

- debug:
    msg: "{{ item }}"
  with_list:
    - 
      - instance1
      - instance3
    - 
      - instance2
      - instance4


ok: [vagrant1] => (item=[u'instance1', u'instance3']) => {
    "msg": [
        "instance1", 
        "instance3"
    ]
}
ok: [vagrant1] => (item=[u'instance2', u'instance4']) => {
    "msg": [
        "instance2", 
        "instance4"
    ]
}
```

使用with_items遍历嵌套list的示例：
```
- debug:
    msg: "{{ item }}"
  with_items:
    - [instance1,instance3]
    - [instance2,instance4]

TASK [debug] ********************************************************************************************************************************************************************************
ok: [vagrant1] => (item=instance1) => {
    "msg": "instance1"
}
ok: [vagrant1] => (item=instance3) => {
    "msg": "instance3"
}
ok: [vagrant1] => (item=instance2) => {
    "msg": "instance2"
}
ok: [vagrant1] => (item=instance4) => {
    "msg": "instance4"
}
```

# 3.with_lines的使用
有时候，可能想对某个shell命令执行的结果逐行遍历，最佳选择就是with_lines了。
例如，使用shell构造一个多行的文本，用于遍历。
```
- name: list a shell command result
  debug:
    msg: "{{ item }}"
  with_lines: "echo -en 'hello\nworld'"

TASK [list a shell command result] **********************************************************************************************************************************************************
ok: [vagrant1] => (item=hello) => {
    "msg": "hello"
}
ok: [vagrant1] => (item=world) => {
    "msg": "world"
}
```

例如：ls / 查询跟目录的文件夹，并进行遍历。
```
- name: list a shell command result2
  debug:
    msg: "{{ item }}"
  with_lines: "ls /"
  
TASK [list a shell command result2] *********************************************************************************************************************************
ok: [vagrant1] => (item=appvol) => {
    "msg": "appvol"
}
ok: [vagrant1] => (item=bin) => {
    "msg": "bin"
}
ok: [vagrant1] => (item=boot) => {
    "msg": "boot"
}
省略若干结果
```

# 4.with_dict的使用
有时候我们需要遍历字典的每个项目，可以使用with_dict.
```
- name: list a dict 1st
  debug:
    msg: "{{ item.key }} {{item.value}}"
  with_dict: {'name':robin,'age':22}

TASK [list a dict 1st] **********************************************************************************************************************************************************************
ok: [vagrant1] => (item={'value': 22, 'key': u'age'}) => {
    "msg": "age 22"
}
ok: [vagrant1] => (item={'value': u'robin', 'key': u'name'}) => {
    "msg": "name robin"
}
```

# 5.with_nested的使用
列表的循环嵌套可以使用with_nested, 虽然使用不是太多，但是总有用到的时候。看个简单的示例：
```
- name: use with_nested exampl
      debug:
        msg: "{{ item }}"
      with_nested:
        - [hello,world]
        - [nihao,shijie]
 
 ok: [vagrant1] => (item=[u'hello', u'nihao']) => {
    "msg": [
        "hello", 
        "nihao"
    ]
}
ok: [vagrant1] => (item=[u'hello', u'shijie']) => {
    "msg": [
        "hello", 
        "shijie"
    ]
}
ok: [vagrant1] => (item=[u'world', u'nihao']) => {
    "msg": [
        "world", 
        "nihao"
    ]
}
ok: [vagrant1] => (item=[u'world', u'shijie']) => {
    "msg": [
        "world", 
        "shijie"
    ]
}
```

# 6.with_together的使用
每次取列表的一个元素进行遍历，相当于按列遍历。
```
 - name: item.0 returns from the 'a' list, item.1 returns from the '1' list
   debug:
     msg: "{{ item.0 }} and {{ item.2 }}"
   with_together:
    - ['a', 'b', 'c', 'd']
    - [1, 2, 3, 4]
    - ['A','B','C','D']

TASK [item.0 returns from the 'a' list, item.1 returns from the '1' list] *******************************************************************************************************************
ok: [vagrant1] => (item=[u'a', 1, u'A']) => {
    "msg": "a and A"
}
ok: [vagrant1] => (item=[u'b', 2, u'B']) => {
    "msg": "b and B"
}
ok: [vagrant1] => (item=[u'c', 3, u'C']) => {
    "msg": "c and C"
}
ok: [vagrant1] => (item=[u'd', 4, u'D']) => {
    "msg": "d and D"
}
```

# 7.循环控制-设置变量名称
循环的时候，默认引用元组使用的是item，这个默认值在2.1以后可以修改了。这个功能在复杂循环中也许会很有用。

```
- name: list many dict
  debug:
    msg: "{{ user.name }}"
  with_items:
    - {'name':robin}
    - {'name':lily}
    - {'name':gil}
  loop_control:
    loop_var: user

TASK [list many dict] ***********************************************************************************************************************************************************************
ok: [vagrant1] => (item={u'name': u'robin'}) => {
    "msg": "robin"
}
ok: [vagrant1] => (item={u'name': u'lily'}) => {
    "msg": "lily"
}
ok: [vagrant1] => (item={u'name': u'gil'}) => {
    "msg": "gil"
}
```