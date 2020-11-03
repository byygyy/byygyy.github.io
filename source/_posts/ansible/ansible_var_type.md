---
title: Ansible变量类型
date: 2020-02-11 22:09:09
tags: ansible-variables
categories: ansible
---

参考官方文档：https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html
# 背景
很少有人注意到ansible 变量类型的问题，最近工作的时候偶然遇到when条件判断不生效的问题，才注意到了变量数据类型的问题。
查阅了好多资料后才知道，原来ansible定义变量的时候，还像python一样有动态数据类型的概念。

<!--more-->

# 直接上示例：
## 变量的定义
age1是一个int类型的变量，例如：age1: 21
age2是一个string类型的变量,例如：age2: '21'
married和married2是一个布尔类型的变量，例如：married: True 或 married2: true
married3是一个string类型的变量，例如：married3: 'true'

## 变量类型转换
age1|string 可以变int转换为string，然后进行比较运算
age2|int 可以把string转为为int，然后进行比较运算
married|string 可以把布尔类型转换为string，然后进行比较运算。
married2|string 可以把布尔类型转换为string，然后进行比较运算。

```
---

- hosts: "{{ hosts_group }}"
  remote_user: root
  vars:
    name1: robin
    hosts_group: "localhost"
    date1: 2020-02-02
    age1: 21
    age2: '21'
    married: True
    married2: true
    married3: 'true'


  tasks:
    - name: def age1 as int
      debug:
        msg: "age1 is {{ age1 }} as int"
      when: age1 == 21

    - name: def age2 as string
      debug:
        msg: "age2 is {{ age2 }} as string"
      when: age2 == '21'

    - name: def age1 as int to string
      debug:
        msg: "age1 is {{ age1 }}"
      when: age1|string == '21'

    - name: def age2 as string to int
      debug:
        msg: "age2 is {{ age2 }}"
      when: age2|int == 21 and age2|int >= 18

    - name: change to string and compare age1 and age2
      debug:
        msg: "{{ age1}} {{ age2 }}"
      when: age1|string + age2|string == '2121'

    - name: change to int and compare age1 and age2
      debug:
        msg: "{{ age1}} {{ age2 }}"
      when: age1|int - age2|int == 0

    - name: show the married or not
      debug:
        msg: "married is {{ married }}"
      when: married|string == 'True'

    - name: show the married2 or not
      debug:
        msg: "married2 is {{ married2 }}"
      when: married2|string == 'True'

    - name: show the married3 or not
      debug:
        msg: "married3 is {{ married3 }}"
      when: married3|string == 'true'


```

## 运行该示例
`ansible-playbook test-var.yml`

```
PLAY [localhost] ****************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [def age1 as int] **********************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "age1 is 21 as int"
}

TASK [def age2 as string] *******************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "age2 is 21 as string"
}

TASK [def age1 as int to string] ************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "age1 is 21"
}

TASK [def age2 as string to int] ************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "age2 is 21"
}

TASK [change to string and compare age1 and age2] *******************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "21 21"
}

TASK [change to int and compare age1 and age2] **********************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "21 21"
}

TASK [show the married or not] **************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "married is True"
}

TASK [show the married2 or not] *************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "married2 is True"
}

TASK [show the married3 or not] *************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "married3 is true"
}


PLAY RECAP **********************************************************************************************************************************************************************************************************************************
localhost                  : ok=7    changed=0    unreachable=0    failed=0

```