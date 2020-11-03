---
title: ansible template用法
date: 2019-12-16 21:53:37
tags: 
  - ansible-template
categories: ansible
---

template作为ansible重要的企业实战解决方案。
<!--more-->

### 变量文件引入
`touch vars.yml`
````
---
- hosts: websrvs
  remote_user: root
  vars_files:
    - vars.yml
  
  tasks:
    - name: install package
      yum: name={{ vars1 }}

    - name: create file
      file: name=/data/{{ var2 }}.log state=touch
````

### template使用，变量优先级
````
-e
current playbook
hosts
````

### when条件的使用
在task后添加when子句可使用条件测试。
`when os_type == "linux"`