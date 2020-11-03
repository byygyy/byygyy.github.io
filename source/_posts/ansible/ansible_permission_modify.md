---
title: ansible 提升权限的多种方式
date: 2019-11-22 00:20:30
tags: ansible
categories: ansible
---
ansible-playbook 可以方便快速的批量执行部署和运维任务，对于不同的场景和服务器，需要使用不同的权限提升方式。

最佳实现：为了提高playbook的兼容性，跟功能没有直接关系的权限提升脚本，不要出现在palybook正文中，可以在ansible-playbook运行的时候，通过-e传入

<!--more-->
场景一：我们有服务器的root密码，而且允许root直接登陆。
````
ansible-playbook -i 'aliyun.lihuanhuan.net,' ./ansible_playbook_test/site.yml -e "ansible_ssh_user='root' ansible_ssh_pass='password'"
````
````
#切换到app_user，并执行java程序
- name: run app by java_user
  shell: java -jar hello.jar
  become: yes
  become_method: su
  become_user: app_user
````

场景二：我们有服务器的root密码，但是只允许普通用户user1使用su切换到root。
````
ansible-playbook -i 'aliyun.lihuanhuan.net,' ./ansible_playbook_test/site.yml -e "ansible_ssh_user='user1' ansible_ssh_pass='user1_password' ansible_become='yes' ansible_become_method='su' ansible_become_user='root' ansible_become_pass='root_password' " -vvv
````
````
#切换到app_user，并执行java程序
- name: run app by java_user
  shell: java -jar hello.jar
  become: yes
  become_method: su
  become_user: app_user
````

场景三：我们只有服务器的app_user密码，而且只允许普通用户user1使用su切换到app_user。
````
ansible-playbook -i 'aliyun.lihuanhuan.net,' ./ansible_playbook_test/site.yml -e "ansible_ssh_user='user1' ansible_ssh_pass='user1_password' ansible_become='yes' ansible_become_method='su' ansible_become_user='app_user' ansible_become_pass='app_user_password' " -vvv
````
````
#切换到app_user，并执行java程序
- name: run app by java_user
  shell: java -jar hello.jar
  become: yes
  become_method: su
  become_user: app_user
````

场景四：我们只有user1和password，但是允许使用特定的实用程序切换到root，例如：dzdo su -

````
ansible-playbook -i 'aliyun.lihuanhuan.net,' ./ansible_playbook_test/site.yml -e "ansible_ssh_user='user1' ansible_ssh_pass='user1_password' ansible_become_exe='dzdo su -' ansible_become='yes' ansible_become_method='su' ansible_become_user='root' ansible_become_pass='user1_password' " -vvv
````
````
#切换到app_user，并执行java程序
- name: run app by java_user
  shell: java -jar hello.jar
  become: yes
  become_method: su
  become_user: app_user
````
refer to https://docs.ansible.com/ansible/latest/user_guide/become.html?highlight=become%20method