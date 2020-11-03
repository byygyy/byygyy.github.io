---
title: ansible 加入外部变量文件
date: 2019-12-21 22:13:30
tags: 
  - include_vars
  - delegate_to
  - ansible-module
categories: ansible
---

对于较大的项目部署或者构建，也许需要动态导入配置文件，并使用该配置文件中的变量，我们可以这样做。

<!--more-->

#### 将文件下载到ansible master端
```yml
- name: donwload vars file to ansible master /tmp/
  get_url:
    validate_certs: no
    url: https://github.com/raw/org_name/project/master/env1.yml
    dest: /tmp/env1.yml
    mode: 0755
    force: yes
    headers:
      Authorization: token xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  delegate_to: localhost
```

#### 导入已下载好的变量文件
```yml
- name: Include only files matching env1.yml (2.2)
  include_vars:
    dir: /tmp/
    files_matching: env1.yml
```

#### 接着你就可以在playbook task中使用导入的变量了
```yml
- name: echo a var
  debug:
    msg: "var1 value is {{ var1 }}"
```
