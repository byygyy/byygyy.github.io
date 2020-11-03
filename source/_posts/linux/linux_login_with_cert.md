---
title: 使用证书登陆Linux服务器
date: 2018-12-03 23:27:37
tags: linux
categories: linux
---

当你的运维对象为大量服务器时，使用证书登陆极为迫切。
<!--more-->

### 主要步骤：

#### 0.私钥放在client，公钥放置在server端。

#### 1.私钥在客户端可以重复使用，可以通过私钥访问多个服务器。

#### 2.公钥必须与私钥配合使用，只有server开启了证书认证，并且添加了公钥数据，client才可以访问。一个server可以添加多个公钥，每段公钥使用空行隔开。（server端修改了相关配置后，需要重启httpd服务）

#### 3.redhat系统的selinux是个很容易忽略的东东，一定要检查。

参考：CentOS 7 SSH使用证书登录 https://blog.csdn.net/long690276759/article/details/53535464