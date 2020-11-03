---
title: ANSIBLE-PLAYBOOK启动的多种方式
date: 2019-03-11 14:15:37
tags: ansible-playbook
categories: ansible
---

# quick start

## start the playbook with no password, it will run "sudo su - root" at the target first.
## and only use the hosts of playbook
```
ansible-playbook -i hosts ~/ansible_playbook_test/site.yml -u lihuanhuan80 --private-key ~/robin.private -e "ansible_become_exe='sudo su -'" -vvv
```
<!--more-->
## how to start the testplaybook and input password, it will run "sudo su - root" at the target first.
## and only use the hosts of playbook
```
ansible-playbook -i hosts ~/ansible_playbook_test/site.yml -u root --ask-pass --ask-become-pass -e "ansible_become_exe='sudo su -'" -vvv
```

## how to start the playbook with hostname list and input the var with start command.
```
ansible-playbook -i 'aliyun.lihuanhuan.net,' ./ansible_playbook_test/site.yml --ask-pass --ask-become-pass -e "instance_name='robinhhli' ansible_ssh_user='root' ansible_become_exe='sudo su -'" -vvv
```

## how to use jenkins to run the ansible_playbook
```
ansible-playbook -i 'aliyun.lihuanhuan.net,' ./ansible_playbook_test/site.yml -e "instance_name='robinhhli' ansible_ssh_user='root' ansible_ssh_pass='password' ansible_become_pass='password' ansible_become_exe='sudo su -'" -vvv
```