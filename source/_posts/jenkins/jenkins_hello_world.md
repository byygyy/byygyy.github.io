---
title: jenkins之-创建第一个pipeline
date: 2020-04-01 08:08:08
tags: jenkins
categories: jenkins
---

从今天开始，我们来整体学习下jenkins相关内容。jenkins是一款用java开发的开源持续集成和持续交付工具，他也是实现Dev OPS的基础工具。
<!--more-->

# 1.什么是pipeline
部署流水线（deployment pipeline）是指软件从原始code到用户这一过程的自动化表现形式，我们经常简称pipeline。jenkins2.x 开始已经支持了pipeline as code的模式，也就是说我们可以像写代码一样，去定义部署流水线的各种工作。

# 2.在jenkins中使用pipeline
## 2.1 pipeline支持的语法
既然jenkins2.x已经支持了pipeline as code，那我们首先要简单了解下pipeline支持的语法。jenkins一开始考虑实现jenkins pipeline的时候，就考虑使用简单易用的groovy脚本语言去实现pipeline。在写脚本式的pipeline时，很像实在写groovy代码。
```
node {
    stage('Build'){
      echo 'I am doing the build!'
    }
    stage('Test'){
      echo 'I am doing the Test'
    }
    stage('Deploy'){
       try{
         //执行部署任务
         echo 'I am try to do the code deployment'
       }
       catch(error){
         //捕捉错误并通知用户
       }
    }
}
```

jenkins pipelien 还支持另外一种语法，申明式语法。
```
pipeline{
    agent any
    stages{
        stage('Build'){
          steps{
            echo 'hello world!'
          }
        }
    }
}
```

## 2.2 创建第一个pipeline
要创建pipeline，首先需要jenkins已经安装了pipeline插件，笔者安装的jenkins默认已经安装了pipeline。
```mermaid
flowchat
st=>start: 新建任务
e=>end: 结束
op=>operation: 流水线
cond=>condition: 确认
st->op->cond
cond(yes)->e
```

默认情况下，流水线定义的默认值为pipeline scripts，我们输入示例。
```
node {
   echo 'Hello World'
}
```
点击确认后，就到了pipeline的主页面，点击左侧的立即构建，就可以看到结果了。
```
控制台输出
Started by user robinhhli
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/lib/jenkins/workspace/my fist pipeline
[Pipeline] {
[Pipeline] echo
Hello World
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```

## 2.3 从github拉取pipeline
明日再续。