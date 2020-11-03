---
title: ANSIBLE 文件和目录操作
date: 2019-06-22 15:53:37
tags: ansible-module
categories: ansible
---

ansible file 模块参考： refer to https://docs.ansible.com/ansible/latest/modules/file_module.html?highlight=file

ansible shell模块参数：https://docs.ansible.com/ansible/latest/modules/shell_module.html?highlight=shell

ansible 删除多个文件和目录：https://www.cnblogs.com/lihuanhuan/p/10612100.html

<!--more-->

## 当前页面的脚本，在ansible 2.6.4测试通过

* 1.创建目录，删除目录（包括目录下的所有文件）
```
- name: Create a directory if it does not exist
  file:
    path: /appvol/some_directory
    state: directory
    mode: '0755'
```
```
- name: Remove a directory if it exist
  file:
    path: /appvol/some_directory
    state: absent
```

* 2.创建文件，删除文件（单个文件删除）
```
- name: Create a file if it does not exist
  file:
    path: /appvol/some_directory/hello.txt
    state: touch
    mode: '0755'
```

```
- name: Remove a file if it exist
  file:
    path: /appvol/some_directory/hello.txt
    state: absent
``` 

* 3.删除某个目录下的所有文件，或者符合条件的文件名
```
- name: list the files of dir some_directory
  shell: ls
  args:
    chdir: /appvol/some_directory
  register: files_list
```
```  
- name: Remove a directory if it does not exist
  file:
    path: /appvol/some_directory/{{ item }}
    state: absent
  with_items:
    - "{{ files_list.stdout_lines }}"
```

该篇博客基于原创，并且经过了严格测试，如果帮到了您，可以给我赞赏哟。