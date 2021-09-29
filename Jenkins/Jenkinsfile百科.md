# Jenkinsfile百科

[TOC]

## 环境

### 基础配置

项目在Linux中的目录 `/usr/local/src/project/spring-k8s-configmap-demo`

jenkins工作空间目录 `/opt/jenkins/workspace/demo-spring`

项目与工作空间简历软连接，设置目录名称为warehouse

```bash
~]# ln -s /usr/local/src/project/spring-k8s-configmap-demo /opt/jenkins/workspace/demo-spring/warehouse
```

### 配置maven

在 "Manage Jenkins" -> "Global Tool Configuration"当中配置maven

```
Name		-> 	maven
MAVEN_HOME	->	/usr/local/apache-maven-3.8.2
```

maven作为变量的使用方法一

```groovy
tools {
    maven 'maven' 
}
stages {
    stage('Example') {
        steps {
            sh 'mvn --version'
        }
    }
}
```

maven作为变量的使用方法二

```groovy
stage('Complie') {
            steps {
                script{
                  mvnHome = tool "m2"
                  sh """
                    cd ${warehouse}     
                    ${mvnHome}/bin/mvn --version                 
                  """
                }        
            }
        }
```



### 配置sharelibrary

在 "Manage Jenkins" -> "Configure System "->"Global Pipeline Libraries"当中配置sharedlibrary

```json
Name : Jenkinslib
Default version : develop
Project Repository ：git@gitee.com:sjwayrhz/devops.git
Library Path (optional) : Jenkinslib
```

然后在jenkinsfile中添加如下

```groovy
#!groovy
@Library('Jenkinslib') _     
def mytools = new org.devops.tools()


stage('git') {
            steps {
                git "git@gitee.com:sjwayrhz/devops.git"
                script {
                    mytools.PrintMes("拉取代码",'green')  
                }  
            }
        }
```

即可使用jenkinslib



### 插件

build user vars

```
https://plugins.jenkins.io/build-user-vars-plugin/

Variables provided

The plugin provides the following environment variables:
Variable 	Description
BUILD_USER 	Full name (first name + last name)
BUILD_USER_FIRST_NAME 	First name
BUILD_USER_LAST_NAME 	Last name
BUILD_USER_ID 	Jenkins user ID
BUILD_USER_GROUPS 	Jenkins user groups
BUILD_USER_EMAIL 	Email address

Pipeline Examples

node {
  wrap([$class: 'BuildUser']) {
    def user = env.BUILD_USER_ID
  }
}

```

### 生成token

点击登录的账户，比如 admin ,点击configure,可以看到“API Token”，点击“Add new token” ，记得生成的token需要save 保存

### 定义和使用map

```groovy
@Library('Jenkinslib') _

def country_capital = [
    Australia: [
        best: 'xx1',
        good: 'xx2',
        bad: 'xx3'
    ],
    America: [
        best: 'yy1',
        good: 'yy2',
        bad: 'yy3'
    ]
]

def map = [
  "demo-spring-a": "hwy-1",
  "demo-spring-b": "hwy-2"
]

pipeline {
    agent any    
    stages {
        stage('Test Map') {
            steps {
                script {
                    echo country_capital.Australia.best
                    echo map."demo-spring-a"
                }
            }
        }
    }
}

```

### 定时任务

```
cron
这里采用和Linux系统一样的定时任务管理方案，加入一些简单的参数项，以应对某些需要定期执行的场景。

pipeline {
    agent any
    triggers {
        cron('* * * * *')
    }
    stages {
        stage('cron job') {
            steps {
                echo 'cron job test'
            }
        }
    }
}

这里参数可以参考linux cron来配置。

pollSCM
这里表示定期对代码仓库进行检测，如果有变化，则自动触发构建。

pipeline {
    agent any
    triggers {
        pollSCM('* * * * *')
    }
    stages {
        stage('cron job') {
            steps {
                echo '每一小时检测一次仓库的变化'
            }
        }
    }
}

upstream
当B项目的执行依赖A项目的执行结果是，A就是B的上游项目，在Jenkins2.22以上的版本中，可以通过upstream关键字进行这种关系的表示。

triggers { 
      // job1,job2都是任务名称
      upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS) 
}
```

### projectName与app对应

在使用Jenkins的过程中，会牵涉到workspace的概念，一般我会在workspace中创建与项目对应的软链接。

例如 ` ln -s /usr/local/src/devops /opt/jenkins/workspace/syncGit`

那么，如何创建app与projectName对应呢

```groovy
def road = [
  "api": "juneyaoYun",
  "web": "juneyaoYun",
  "vue": "group-app-manage"
]

def projectName
pipeline {
    agent any

    stages {                 
         stage ('git pull') {
            steps { 
                script {
                    projectName = road."${params.app}"
                    sh """
                        cd $WORKSPACE/$projectName
                        git reset --hard HEAD
                        git switch ${params.branch}
                        git fetch --all
                        git reset --hard origin/${params.branch}
                        git pull origin ${params.branch}
                    """
                }              
                
            }
        }
    }
}
```

