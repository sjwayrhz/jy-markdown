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
Name		-> 	m2
MAVEN_HOME	->	/usr/local/apache-maven-3.8.2
```

maven作为变量的使用方法一

```groovy
tools {
    maven 'apache-maven-3.0.1' 
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
