---
title: jenkins之-pipeline基础知识
date: 2020-04-05 22:55:08
tags: pipeline
categories: jenkins
---

在讲过了jenkins的简单使用后，我们今天来讲讲jenkins是如何支持pipeline即代码的。
<!--more-->

# 1.pipeline的两种类型
我们在jenkins编辑流水线pipeline时，有两种不同的语法样式，脚本式语法（scripts syntax )和申明式语法（declarative syntax)。
## 1.1脚本式流水线
这是Jenkins最开始实现的流水线即代码方式，类似于编程的方式实现，它依赖于groovy语言和结构，特别是对于错误检查和异常处理来说。
```
node("cm-linux") {
    properties([pipelineTriggers([cron('H/10 * * * *')])])
    stage('Build'){
        echo 'I am doing the build' 
        sh 'echo "hello world"'
        sh '''
          echo "multiline shell step works too"
        '''
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
***node用于引用jenkins的节点，stage是jenkins任务的基本单位，可以包括各种jenkins指令。***

## 1.2声明式流水线
这是Jenkins提供的一种新的选择。声明式风格的流水线代码被编排在清晰的段落中，条例很清楚，很容里理解。
```
pipeline{
    agent { label 'cm-linux-003' }
    stages{
        stage('Build'){
          steps{
            echo 'start to do Build'
            sh 'pwd'
            sh 'whoami'
          }
        }
        stage('Test'){
              steps{
                echo 'start to do Test'
                sh 'pwd'
                sh 'whoami'
              }
        }
        stage('Deploy'){
              steps{
                echo 'start to do Deploy'
                sh 'pwd'
                sh 'whoami'
              }
        }
    }
}
```
***stages是一个集合，可以存储多个stage，每个stage内部有一个steps，每个steps里有好多步骤。***

# 2.理解pipeline步骤语法
pipeline每个步骤，会执行一个命令。Jenkins中的步骤总是希望每个参数对应一个名称。每个命令如果不用太多的参数，可以使用简化版，如果有较多参数则需要使用完整版。
例如，这是一个简化版的步骤。
```
sh 'whoami
git 'https://github.com/byygyy/ansible-examples.git''
```
例如，这个是一个完整版示例。
```
sh([script:'whoami'])
git([branch:'master',url:'https://github.com/byygyy/ansible-examples.git'])
```

# 3.理解pipeline参数
## 3.1使用parameters指令
脚本式：
```
node("cm-linux") {
    properties([pipelineTriggers([cron('H/10 * * * *')])])
    
    parameters{
      string(name: 'USERID', defaultValue:'',description: 'input or select your userid')
    }
    
    stage('Build'){
        echo 'I am doing the build' 
        sh 'echo "hello world"'
        sh '''
          echo "multiline shell step works too"
          ls -alh
          pwd
          whoami
        '''
        git 'https://github.com/byygyy/ansible-examples.git'
    }
}
```
声明式：
```
pipeline{
    agent { label 'cm-linux-003' }

    parameters{
      string(name: 'USERID2', defaultValue:'',description: 'input or select your userid')
    }
    
    stages{
        stage('Build'){
            steps{
              echo 'start to do Build'
              sh 'whoami'
              git 'https://github.com/byygyy/ansible-examples.git'
            }
        }
        stage('Test'){
              steps{
                echo 'start to do Test'
                sh 'whoami'
                git([branch:'master',url:'https://github.com/byygyy/ansible-examples.git'])
              }
        }
    }
}
```

## 3.2使用properties定义参数
脚本式：
```
node("cm-linux") {    
    properties([
      parameters([
        string(name:'USERID', defaultValue:'', description:'input or select your userid'),
        string(name:'USERNAME', defaultValue:'', description:'input or select your username'),
      ])
    ])

    stage('Build'){
        echo 'I am doing the build' 
        sh 'echo "hello world"'
        sh '''
          echo "multiline shell step works too"
          ls -alh
          pwd
          whoami
        '''
        git 'https://github.com/byygyy/ansible-examples.git'
    }
}
```

# 4.pipeline执行流程
## 4.1 pipeline触发任务
### 4.1.1 脚本式pipeline触发任务
如果是脚本式pipeline，可以在脚本中指定一个properties代码块（通常在流水线开始前）来定义触发条件，这里设置的内容将和web界面合并处理，并且web页面优先级高，例如：
```
node("cm-linux") {
    properties([
      pipelineTriggers([cron('H/10 * * * *')])
      ])
    stage('Build'){
      echo 'I am doing the Test'
    }
}
```

如果还有其他的properties代码块，应该合并到一起写，例如：
```
node("cm-linux") {
    properties([
      pipelineTriggers([cron('H/10 * * * *')]),
      parameters([
        string(name:'NODENAME', defaultValue:'cm-linux', description:'input the node name'),
        string(name:'USERID', defaultValue:'', description:'input or select your userid'),
        string(name:'USERNAME', defaultValue:'', description:'input or select your username'),
      ])
    ])

    stage('Build'){
      echo 'I am doing the Test'
    }
}
```

### 4.1.2 声明式pipeline触发任务
如果式声明式流水线，可以使用triggers指令定义一个流水线的触发任务。
```
pipeline{
    agent { label 'cm-linux' }
    triggers{ cron('H/10 * * * *') }
    parameters{
      string(name: 'USERID2', defaultValue:'',description: 'input or select your userid')
      string(name: 'USENAME2', defaultValue:'',description: 'input or select your username')
    }
    stages{
        stage('Build'){
            steps{
              echo 'start to do Build'
              sh 'whoami'
              git 'https://github.com/byygyy/ansible-examples.git'
            }
        }
    }
}
```
***声明式pipline，同一个指定内部每条命令是独立的一样，不用加什么分隔符。***

## 4.2 githu钩子触发器触发jenkins GitSCM轮询
仅支持脚本式pipeline，当piepine配置的github代码发生push提交后，会触发这个pipeline的build操作。
```
node("cm-linux") {
      properties([
        pipelineTriggers([
            githubPush(),
        ]),
      parameters([
        string(name:'NODENAME', defaultValue:'cm-linux', description:'input the node name'),
        string(name:'USERID', defaultValue:'', description:'input or select your userid'),
      ])
    ])

    stage('Build'){
      echo 'I am doing the Test'
    }
}
```

## 4.3 SCM轮询
这个是标准的功能，pipeline会周期性地扫描源码版本控制系统的变更，当发现任何更新，任务就会处理这些变化。

SCM轮询使用与“周期性构建”相同的jenkins core语法指令，脚本式语法如下：
```
node("cm-linux") {
      properties([
        pipelineTriggers([
            pollSCM('*/11 * * * *'),
        ]),
      parameters([
        string(name:'NODENAME', defaultValue:'cm-linux', description:'input the node name')
      ])
    ])

    stage('Build'){
      echo 'I am doing the Test'
    }
}
```

声明式语法如下：
```
pipeline{
    agent { label 'cm-linux' }
    triggers{ 
      pollSCM('*/11 * * * *')
    }
    parameters{
      string(name: 'USERID2', defaultValue:'',description: 'input or select your userid')
      string(name: 'USENAME2', defaultValue:'',description: 'input or select your username')
    }
    stages{
        stage('Build'){
            steps{
              echo 'start to do Build'
              sh 'whoami'
              git 'https://github.com/byygyy/ansible-examples.git'
            }
        }
    }
}
```

## 4.4 远程调用构建任务
详见: [https://blog.csdn.net/byygyy/article/details/105259356](https://blog.csdn.net/byygyy/article/details/105259356).


