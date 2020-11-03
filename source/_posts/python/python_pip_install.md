---
title: python第三方库安装-多种方式
date: 2018-02-26 12:24:30
tags: pip
categories: python
---

pip 是python最重要的包管理工具，今天我们来总结下pip第三方包安装的多种方式。

<!--more-->

#### 第一种方式：安装whl文件

pip install whatever.whl

#### 第二种方式：安装tar.gz文件

一般是先解压，然后进入目录之后，有setup.py文件

通过命令 python setup.py install安装。

 
#### 第三种方式：pip在线安装

pip已经收录了大部分第三方库，https://pypi.org

##### 1、如果机器可以联网：

pip install package_name==1.0.0 安装指定版本

pip install package_name 安装最新版本

##### 2、如果机器不能联网：

2.1打包已经安装的包（注意：打包的操作系统要跟目标操作系统版本一致）

pip list #查看安装的包

pip freeze >requirements.txt #将已经安装的包名称写入到文件

pip download -r requirements.txt #将文件中的包下载到当前目录

2.2离线情况下安装打包好的包

pip install --no-index --find-links=\usr\src\packs\ mysql-connector 单个安装

pip install --no-index --find-links=\usr\src\packs\ -r requirements.txt 批量安装
————————————————
版权声明：本文为CSDN博主「比永远更永远」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/byygyy/article/details/83746726