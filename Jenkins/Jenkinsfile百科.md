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

