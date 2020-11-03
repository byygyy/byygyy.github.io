---
title: docker quick start
date: 2019-03-15 18:48:37
tags: dcoker
categories: docker
---
docker 官方文档：
https://docs.docker.com/get-started/

<!--more-->

### 第一个docker容器，你将可以访问docker中运行的nginx实例。
```
yum install docker
systemctl start docker
docker image pull nginx
docker run -d -p 8080:80 nginx
docker ps
curl http://127.0.0.1:8080
```