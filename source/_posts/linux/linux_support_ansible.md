---
title: 使用ansible管理Redhat8服务器
date: 2020-02-04 23:59:37
tags: ansible
categories: linux
---

redhat7默认安装了python，天生就支持ansible管理，什么都不用做。
redhat8默认不安装python，因此无法通过python去管理，直接上解决方案。
dnf install python3 -y
alternatives --set python /usr/bin/python3
yum install python3-libselinux_x86_64

<!--more-->