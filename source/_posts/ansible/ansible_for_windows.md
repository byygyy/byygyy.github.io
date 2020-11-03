---
title: windows server使用ansible管理
date: 2019-11-26 22:15:37
tags: 
  - windows
  - ansible-module
categories: ansible
---

windows 作为运维的半壁江山，虽然ansible对其支持不太好，但是也不能忽略他。
<!--more-->

#### 1.在Windows 下载并执行ansible onboard 脚本。

https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1
如果必要的话，允许app用户远程连接和读写本地文件的权限。

#### 2.ansible core server 安装必要的包

```shell
pip install pywinrm2
```
#### 3.尝试使用下载ansible playbook到ansible core server并使用。

```shell
read -s ansible_password
```

```shell
ansible-playbook -i windows_ip_or_domain_name, /home/user1/Ansible/deploy/site.yml -e "ansible_user=Administrator ansible_password=$ansible_password ansible_port=5986 ansible_connection=winrm ansible_winrm_transport=ntlm ansible_winrm_server_cert_validation=ignore ansible_winrm_read_timeout_sec=180 remote_user=Administrator"

```

ansible 2.6.4已经测试通过，windows server 2016测试通过。