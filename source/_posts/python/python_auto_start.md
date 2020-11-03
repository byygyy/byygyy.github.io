---
title: python脚本在Linux Redhat7开机启动
date: 2018-12-24 23:16:30
tags: linux
categories: python
---

有时候，python程序需要确保linux server重启的时候可以自动加载。

<!--more-->

````
cd /usr/lib/systemd/system
touch proxy.service
````

````
###################################################################
[Unit]
Description=proxy
After=multi-user.target
 
[Service]
Type=idle
ExecStart=/usr/bin/python /appvol/ProxyServer/proxyserver.py 11081
PrivateTmp=true
 
[Install]
WantedBy=multi-user.target
````
````
systemctl enable proxy
systemctl start proxy
````