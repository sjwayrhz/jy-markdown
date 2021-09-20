## Jenkins相关配置

[TOC]

### 修改默认启动配置

```bash
$ cat  /etc/sysconfig/jenkins|grep -Ev '^$|#'
JENKINS_HOME="/var/lib/jenkins"
JENKINS_JAVA_CMD=""
JENKINS_USER="jenkins"
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true"
JENKINS_PORT="8080"
JENKINS_LISTEN_ADDRESS=""
JENKINS_HTTPS_PORT=""
JENKINS_HTTPS_KEYSTORE=""
JENKINS_HTTPS_KEYSTORE_PASSWORD=""
JENKINS_HTTPS_LISTEN_ADDRESS=""
JENKINS_HTTP2_PORT=""
JENKINS_HTTP2_LISTEN_ADDRESS=""
JENKINS_DEBUG_LEVEL="5"
JENKINS_ENABLE_ACCESS_LOG="no"
JENKINS_HANDLER_MAX="100"
JENKINS_HANDLER_IDLE="20"
JENKINS_EXTRA_LIB_FOLDER=""
JENKINS_ARGS=""
```

在引号中填入你的路径。

### 设置语言为英文

安装插件“Locale”

jenkins->【系统管理】->【系统设置】->【Locale】，输入：

zh_CN	中文

en_US	英文

勾选`Ignore browser preference and force this language to all users`

可自行选择

### 安装插件

Git Parameter
Gitlab
GitHub Authentication
Publish Over SSH

### 添加密钥

```
Manage Jenkins -> Manage Credentials -> Jenkins -> Global credentials (unrestricted) -> Add Credentials
```

一般情况下，添加ssh密钥即可

### 配置maven

```
Manage Jenkins -> Global Tool Configuration -> Maven

Name: m2
MAVEN_HOME: /usr/local/apache-maven-3.8.2
```

之后在代码中引用maven

```
mvnHome = tool "m2"
sh """
  cd ${warehouse}                      
  ${mvnHome}/bin/mvn clean install
  cp -fr ${warehouse}/target/demo-0.0.1-SNAPSHOT.jar ${warehouse}/docker
"""
```





