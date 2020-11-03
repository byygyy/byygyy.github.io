---
title: ANSIBLE 通过网络下载和上传文件
date: 2019-06-22 14:53:37
tags: 
  - ansible-module
categories: ansible
---

refer to get_url – Downloads files from HTTP, HTTPS, or FTP to node
https://docs.ansible.com/ansible/latest/modules/get_url_module.html?highlight=get_url

* 1.通过http下载文件，并且不验证证书
```shell
- name: download files by https
  get_url:
    url: https://robin.org.cn/test.zip
    dest: /appvol/ansible-test/
    validate_certs: no
```

