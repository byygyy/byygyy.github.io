---
title: ansible下载文件的多种方式
date: 2019-11-27 21:46:30
tags: 
  - github
  - http
  - https
  - ansible-module
categories: ansible
---

对于ansible来说，下载文件是一个很重要的课题，这是build或者deploy的第一步，通常来讲由于不同项目的差异，可能我们的代码包或者资源文件保存在于http,github,nexus,ftp,nas等等。

<!--more-->

#### http文件下载,前提是http允许匿名用户下载
```yml
- name: download war file
  get_url:
    url: "{{ https_url }}/start.war"
    dest: /tmp
    mode: 0644
    force: yes
    validate_certs: no
```

#### github 文件下载,前提是已经在github申请了token
```yml
- name: donwload docker rpm
  get_url:
    validate_certs: no
    url: https://github.com/raw/org_name/project/master/docker.rpm
    dest: /tmp/docker.rpm
    mode: 0755
    force: yes
    headers:
      Authorization: token xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

#### 如果想让文件下载到ansible master端,只要增加一条
`delegate_to: localhost`

* 一个完整的task，如下：
```yml
- name: donwload docker rpm
  get_url:
    validate_certs: no
    url: https://github.com/raw/org_name/project/master/docker.rpm
    dest: /tmp/docker.rpm
    mode: 0755
    force: yes
    headers:
      Authorization: token xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  delegate_to: localhost
```

#### 关于https证书验证

大家知道，凡是我们使用浏览器或者脚本方式访问https资源的时候，客户端默认会检查https证书是否合法，例如：https证书的绑定的地址是否跟访问的地址匹配，证书颁发机构是否​合法等。对于ansible下载文件来说，也会遇到这个问题。

​不验证证书：
```
validate_certs: no
```

验证证书：
```
validate_certs: yes
```