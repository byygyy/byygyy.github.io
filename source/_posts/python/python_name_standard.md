---
title: python命名标准
date: 2019-02-24 23:24:30
tags: standard
categories: python
---
python 命名标准，个人开发参考。

<!--more-->

#### 1.package name
全部小写字母，中间可以由点分隔开，作为命名空间，包名应该具有唯一性，推荐采用公司或组织域名的倒置，如com.apple.quicktime.v2

#### 2.module name
全部小写字母，如果是多个单词构成，可以用下划线隔开，如dummy_threading.

#### 3.class name
采用大驼峰法命名，如SplitViewController

#### 4.var name
全部小写字母，如果由多个单词构成，可以用下划线隔开。如果变量应用于模块或函数内部，则变量名可以用单下划线开头；变量类内部私有变量使用变量名可以双下划线开头。另外，避免使用小写L、大写O和大写I作为变量名。

#### 5.函数名和方法名
命名同变量命名，如balance_account

#### 6 .常量名
全部大写字母，如果是由多个单词构成，可以用下划线隔开，如YEAR和 WEEK_OF_MONTH