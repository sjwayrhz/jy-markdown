# openstack中的jenkins服务器丢失文件夹

[TOC]

## 概述

jenkins服务器在做自动发布均瑶集团app项目的第970个job时，出现文件夹丢失的异常，此种情况足以证实九州云提供的ceph存储不安全。

## 配置信息

资源池： JYJT-PAAS

IP:  192.168.177.57

主机名： jyjt-jenkins

安装应用： java , maven, nodejs ,jenkins

管理控制台地址：`http://192.168.177.57:8080/jenkins/`

管理控制台账号： admin

管理控制台密码： bSnXKviBNDomA7KQ

## 阶段状态

### 2021年7月，计划编写jenkins脚本

6个月前，创建部署的jyjt-jenkins项目，服务与均瑶app的cicd自动发布工作

### 2021年9月7日，完成基本jenkins脚本

这是这个jenkins执行的第30个任务的时间点，表面在这个时候，cicd已经正常运行，再次之后，cicd脚本再也没有做大变更。

```
http://192.168.177.57:8080/jenkins/job/publish-jyjt-app/30/console
```

查看日志连接如上

初始设置的系统盘为 100 GB

### 2021年12月7日，磁盘空间不足

在今天执行的第832个任务的时候，发现原先设置的100GB系统盘空间不足，执行任务的连接为

```
http://192.168.177.57:8080/jenkins/job/publish-jyjt-app/832/console
```

在此日志中发现如下描述

```
Step 4/14 : COPY ${JAR_FILE} application.jar
failed to copy files: failed to copy file: Error processing tar file(exit status 1): write /application.jar: no space left on device
[Pipeline] }
```

然后又登陆了192.168.177.57这台服务器，使用 df 命令，查看确实磁盘不足。

### 2021年12月7日，增加系统盘空间

登陆192.168.177.57这台服务器，关机，然后在openstack平台找到这台linux的云硬盘，将磁盘从100GB修改为230GB，然后开机。

注意：关键操作在于在openstack平台提升系统盘空间，而不是挂载第二块新硬盘。

完成之后，执行第833条jenkins任务，正常，连接为：

```
http://192.168.177.57:8080/jenkins/job/publish-jyjt-app/833/console
```

从第833条任务到969任务，也基本上没有失败，这一个月以来，jenkins发布机恢复正常。

### 2022年1月6日，磁盘文件夹丢失

发现磁盘文件夹丢失，执行任务的连接为

```
http://192.168.177.57:8080/jenkins/job/publish-jyjt-app/970/
```

查看到的关键性日志：

````
[Pipeline] sh
+ cd /opt/jenkins/workspace/publish-jyjt-app/juneyaoYun
/opt/jenkins/workspace/publish-jyjt-app@tmp/durable-8681ef24/script.sh: line 2: cd: /opt/jenkins/workspace/publish-jyjt-app/juneyaoYun: No such file or directory
[Pipeline] }
[Pipeline] // script
````



## 关键信息

### 事件简述

- 这台linux的密钥只有我一个人拥有，jenkins平台的搭建，部署，jenkins cicd的设计完全由我一个人完成。
- 接近上千次的使用，基本上交给开发和测试的同事使用，都没有发现问题。
- 1月6日出现问题，1月8日是集团年会，这期间jenkins发布机出现问题，影响了1月8日年会之前的关键性发布。如果我自己没有记录笔记，如果不能在此状态下成功改为手动发布，则有可能影响到年会之前到最后一版集团app的发布。
- 在此申明，就事论事，个人不可能做出任何人为进入发布机删除文件夹的行为，不可能给自己挖坑，也不可能陷害别人。一切操作日志，仍然保留在服务器上，一切操作命令都没有清理，事发现场仍然处于保留状态。

### 九州云检查工作的不配合

当我用邮件详细描述该问题之后，还和九州云客服提到了关于磁盘扩容的事情。

但是九州云的检查工作不配合证据如下：

- 只是查看一下ceph监控，提供ceph监控截图，完全不登陆jenkins服务器排查问题。
- 他们检查完成之后，我发现linux密码未被修改，history历史记录完全没有九州云同事留下的痕迹。
- 系统日志他们不帮忙分析，history不帮忙查看，让我自己查看。
- jenkins脚本不帮忙分析，让我自己分析。
- 在我百分百确定他们的ceph出现丢失文件bug之后，毫无责任感的甩锅给我，连linux服务器都不登陆，就可以妄自猜测是我人为删除了文件夹。



### 特别申明

这是均瑶科创内部使用的集团app项目，因为jenkins服务器完全由我一个人恢复，丢失数据文件，凭我一人之力，仍然可以恢复jenkins发布机。

但是如果ceph底层存储不稳定性，九州云同事不能从技术层面给出一个合理的解释，不能深刻分析可打出相应的补丁，做出的回复仍然无法让真理接受。那么，之后如果有均瑶集团其他子公司的存储一旦使用九州云的存储平台所涉及的任何文件丢失问题，与均瑶科创的运维工程师无关。



## 问题推测

依据各个阶段的jenkins服务器的状态来看，此次故障的可能性分析如下几条

### 推测一：九州云的ceph无法确保系统磁盘直接扩容

因为直接修改系统盘的大小，会有涉及到物理卷和逻辑卷的重组与划分，十分复杂，怀疑九州云的ceph无法完成此项操作。

此种推测概率较大

### 推测二：九州云的ceph自身不稳定，与是否扩容无关

根据目前使用情况看来，尚未发现其他服务器出现如此严重的文件夹丢失问题，所以如果就此事断定所有九州云的ceph存储都不稳定，证据不足。

但是，也不排除有这种可能性。因为如果是事实，那么后果就相当恐怖。

此种推测概率较小



## jenkinsfile

在集团app中,jenkins代码量极小，不存在恶意删除文件夹的可能性。编写的jenkinsfile如下：

```groovy
#!groovy
def createVersion() {
  return new Date().format('yyyyMMddHHmm')
}


def road = [
  "api": "juneyaoYun",
  "web": "juneyaoYun",
  "vue": "group-app-manage",
  "redbag": "redBag",
  "meet": "meet"
]

def projectName

pipeline {
    agent any

    parameters {
        choice(name: 'branch', choices: ['develop', 'master'], description: 'select the branch')
        choice(name: 'app', choices: ['api', 'web', 'vue','redbag','meet'], description: 'select the app')
    }

    environment {
        def tag = createVersion()
        def repositry = "registry.cn-shanghai.aliyuncs.com/jyjt/${params.branch}-${params.app}"
    }

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
        
        stage('compile') {
            steps {
                sh """
                    if [[ ${params.app} == "api" || ${params.app} == "web" ]] ;then
                        cd $WORKSPACE/juneyaoYun
                        /usr/local/apache-maven-3.8.1/bin/mvn clean install
                    elif [[ ${params.app} == "vue" || ${params.app} == "redbag" || ${params.app} == "meet" ]] ;then
                        echo "vue don't need compile"
                    fi                  
                """
            }
        }

        stage ('operation by docker') {
            steps{
                script {
                    sh """
                        if [[ ${params.app} == "api" || ${params.app} == "web" ]] ;then
                            cd $WORKSPACE/juneyaoYun/juneyaoYun-${params.app}
                        elif [ ${params.app} == "vue" ];then
                            cd $WORKSPACE/group-app-manage
                        elif [ ${params.app} == "redbag" ];then
                            cd $WORKSPACE/redBag
                        elif [ ${params.app} == "meet" ];then
                            cd $WORKSPACE/meet
                        fi     
                        
                        docker build -t ${repositry}:${tag} .
                        docker push ${repositry}:${tag}
                        docker rmi -f ${repositry}:${tag}
                    """
                }                
            }
        }
        stage ('kubernetes update apps') {
            steps{
                sh """
                 if [ "${params.branch}" == "develop" ];then {
                    ssh root@192.168.177.90 "kubectl set image deployment/${params.app} ${params.app}=${repositry}:${tag} -n app-dev"
                    }
                 elif [ "${params.branch}" == "master" ];then {
                    ssh root@103.192.254.15 -p 30022 "kubectl set image deployment/${params.app} ${params.app}=${repositry}:${tag} -n app-prod"
                    }
                 fi
                """
            }
        }
    }
}

```

