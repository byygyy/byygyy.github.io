---
title: ansible playbook用法
date: 2019-12-12 22:53:37
tags: 
  - ansible-playbook
categories: ansible
---

playbook作为ansible重要的企业实战解决方案。
<!--more-->

### playbook 基本使用
创建playbook文件
````
touch playbooktest.yml
vim playbooktest.yml
````

编写playbook
````
---
- hosts: websrvs
  remote_user: root

  task: 
    - name: hello
      command: hostname
````

执行playbook
`ansible-playbook playbooktest.yml`

#### palybook构成
palybook采用YAML语言编写，有多个剧本组成，每个剧本由若干个task组成，每个task调用不同的模块，最终在目标机器上执行。ansible playbook相当于linux shell脚本。
`yaml.org 查看更多的语法详情`

#### playbook之Hosts
第一种编写方式
````
---
- hosts: websrvs
  remote_user: root

  tasks:
    - name: create new file
      file: name=/data/newfile state=touch
    - name: create a new user
      user: name=test2 system=yes
    - name: insall package
      yum: name=httpd
    - name: copy files
      copy: src=/var/www/html/index.html
````


第二种编写方式
````
---
- hosts: websrvs
  remote_user: root

  tasks:
    - name: create new file
      file: 
        name: /data/newfile
        state: touch

    - name: create a new user
      user:
        name: test2
        system: yes

    - name: insall package
      yum: 
        name:httpd
    
    - name: copy files
      copy:
        src:/var/www/html/index.html
````


只进行检查，不真正执行
`ansible-playbook test.yml --check`
`ansible-playbook -C test.yml`

#### handlers和notify结合使用条件触发
* handlers相当于一个触发器，配合notify使用。
````
---
- hosts: websrvs
  remote_user: root

  tasks:
     - name: install httpd
       yum: name=httpd
       tags: installed

     - name: config file
       copy: src=files/httpd.conf dest=/etc/httpd/conf/ backup=yes
       notify: 
        - restart service
        - check the nginx process
    
     - name: restart httpd
       service: name=httpd state=start enableed=yes
       tags: rshttpd

  handlers:
    - name: restart service
      service: name=httpd state=restarted
    - name: check the nginx process
      shell: killall -0 nginix> /tmp/nginx.log
````

#### ansible tags的使用
添加tags后，可以通过tags调用，通过指定标签执行特定task.
`ansible-playbook -t installed,rshttpd httpd.yml`

#### playbook中变量的使用
ansible变量只能数字字母下划线组成，且只能字母开头，包括系统变量和自定义变量。

* 查询某个关键字的系统变量。
`ansible websrvs -m setup -a 'filter=ansible_fqdn'`

* 引用外部的变量
````
---
- hosts: websrvs
  remote_user: root

  tasks:
    - name: create new file
      file: 
        name: /data/newfile
        state: touch

    - name: create a new user
      user:
        name: test2
        system: yes

    - name: insall package
      yum: 
        name: {{ pkname }}
````


执行playbook并通过-e传入变量
`ansbible-playbook app.yml -e 'pkname=httpd pkname2=vbss'`

* 在playbook中定义变量
````
---
- hosts: websrvs
  remote_user: root
  vars:
    - pkname1: httpd
    - pkname2: vbs
     
  tasks:
    - name: create new file
      file: 
        name: /data/newfile
        state: touch

    - name: insall package
      yum: 
        name: {{ pkname }}
````


执行playbook自动引用内部定义的变量
`ansbible-playbook app.yml`



* 在ansible的hosts文件中定义变量，
`/etc/ansible/hosts`
````
[webserver]
192.168.1.2 http_port=81
192.168.1.4 http_port=82
````

`playbook引用hosts中的变量`
````
---
- hosts: websrvs
  remote_user: root
  vars:
    - pkname1: httpd
    - pkname2: vbs
  
  tasks:
    - name: set hostname
      hostname: name=wwww.{{http_port}}.baidu.com
````
