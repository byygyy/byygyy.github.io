---
title: Ansible 删除多个文件或目录
date: 2019-03-17 14:53:37
tags: ansible-module
categories: ansible
---

翻译和转载该网页内容 http://www.mydailytutorials.com/ansible-delete-multiple-files-directories-ansible/
# 背景
ansible 有多种方式删除一个文件或目录，删除一个目录中的所有文件，使用正则表达式删除文件等等。最安全的方式是使用ansible内置的file模块。当然你也可以使用shell 模块去实现。但它不是幂等的，因此重新执行会抛出错误。

<!--more-->

# 删除一个文件
```
- name: Ansible delete file example
  file:
    path: /etc/delete.conf
    state: absent
```
> 注意：当你知道一个文件名的时候，可以这样删除这个文件。


# 删除多个文件
```
- name: Ansible delete multiple file example
  file:
    path: "{{ item }}"
    state: absent
  with_items:
    - hello1.txt
    - hello2.txt
    - hello3.txt
```
> 注意：当你知道多个文件名的时候，可以这样删除这些文件。


# 删除一个目录或文件夹
```
- name: Ansible delete directory example
  file:
    path: removed_files
    state: absent
```
> 上面这个例子讲删除指定的目录，如果这个目录不存在，不会抛出错误。

# 使用shell 脚本删除多个文件
```
- name: Ansible delete file wildcard example
  shell: rm -rf hello*.txt
  ```
 > 上面这个例子讲删除指定的目录，如果这个目录不存在，不会抛出错误。

# 使用find和file模块结合linux shell模糊搜索删除文件
```
- hosts: all
  tasks:
  - name: Ansible delete file glob
    find:
      paths: /etc/Ansible
      patterns: *.txt
    register: files_to_delete

  - name: Ansible remove file glob
    file:
      path: "{{ item.path }}"
      state: absent
    with_items: "{{ files_to_delete.files }}"
```

# 使用find和file模块结合python的正则表达式删除文件
```
- hosts: all
  tasks:
  - name: Ansible delete file wildcard
    find:
      paths: /etc/wild_card/example
      patterns: "^he.*.txt"
      use:regex: true
    register: wildcard_files_to_delete

  - name: Ansible remove file wildcard
    file:
      path: "{{ item.path }}"
      state: absent
    with_items: "{{ wildcard_files_to_delete.files }}"
```

# 移除晚于某个日期的文件
```
- hosts: all
  tasks:
  - name: Ansible delete files older than 5 days example
    find:
      paths: /Users/dnpmacpro/Documents/Ansible
      age: 5d
    register: files_to_delete

  - name: Ansible remove files older than a date example
    file:
      path: "{{ item.path }}"
      state: absent
    with_items: "{{ files_to_delete.files }}"
```
