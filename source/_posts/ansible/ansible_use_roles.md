---
title: ansible 条件，循环，roles用法
date: 2019-12-16 22:53:37
tags: 
  - ansible-roles
categories: ansible
---

roles作为ansible重要的企业实战解决方案。
<!--more-->

#### 批量创建文件，使用循环
````
---
- hosts: websrvs
  remote_user: root

  tasks:
    - name: create some files
      file: name=/data/{{ item }} state=touch
      with_items:
        - file1
        - file2
      when: ansible_distribution_major_version == "7"
````


#### 迭代嵌套自变量，使用循环
````
---
- hosts: websrvs
  remote_user: root

  tasks:
    - name: create some groups
      group: name={{item}}
      when: ansible_distribution_major_version == "7" 
      with_items:
        - g1
        - g2
        - g3

    - name: create some users
      user: name={{item.name}} group={{item.group}}
      when: ansible_distribution_major_version == "7"
      with_items:
        - { name: 'user1',group: 'g1'}
        - { name: 'user1',group: 'g2'}
        - { name: 'user1',group: 'g3'}
      
````

#### 使用for循环
````
---
- hosts websrvs
  remote_user: root
  vars:
    #定义一个列表类型的变量，名称为ports
    ports: 
      - 81
      - 82
      - 83

  tasks:
    - name: copy conf file
      template: src=for1.conf.j2 dest=/data/for1.conf
````


`touch for1.conf.j2`
循环创建配置，并且可以使用if条件判断
````
{% for port in ports %}
server{
    listem {{ port }}
}
{% endfor %}
````

#### roles的使用
* 将playbook中使用的功能拆开，分们别类，在playbook中引入即可，实现了功能复用，适合大型项目。


* 在一个playbook中调用多个roles
````
- hosts: websrvs
  remote_user: root
  roles:
    - role: httpd
    - role: nginx
````


* 在一个roles的task中调用其他roles中的tasks.
main.yml
````
- include: temp.yml
- include: roles/htpd/tasks/copyfile.yml
````

* 在一个playbook，给每个roles打上tag
````
- hosts: websrvs
  remote_user: root
  roles:
    - { role: httpd, tasg: ['web',httpd'] }
    - { role: nginx, tags: ['web','nginx'],when: ansible_distribution_major_version == "7" }
````

* 调用带有tag的任务
`ansible-playbook -t web some_role.yml`



