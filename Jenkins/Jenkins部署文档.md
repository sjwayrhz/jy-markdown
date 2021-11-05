# How to Install Jenkins on CentOS 7

[TOC]

安装openjdk 

这将是启动Jenkins的基础依赖

```
~]# sudo dnf install java-1.8.0-openjdk java-devel -y
```

### 使用rpm包安装

目前最新是jenkins-2.303.3-1.1.noarch.rpm

```
~]# rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/jenkins-2.303.3-1.1.noarch.rpm
```

### 修改jenkins默认配置

```bash
~]# cat  /etc/sysconfig/jenkins|grep -Ev '^$|#'
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

例如可以修改 

```bash
JENKINS_HOME="/var/lib/jenkins"		--> 	JENKINS_HOME="/opt/jenkins"
JENKINS_USER="jenkins"			    -->  	JENKINS_USER="root"
```

新建jenkins家目录

```bash
~]# mkdir /opt/jenkins
~]# chown jenkins:jenkins /opt/jenkins
```

### 启动jenkins

```bash
~]# sudo systemctl enable --now jenkins
```

查看初始密码

```bash
~]# cat /opt/jenkins/secrets/initialAdminPassword
```

### 修改默认镜像源

```bash
~]# mv /opt/jenkins/hudson.model.UpdateCenter.xml /opt/jenkins/hudson.model.UpdateCenter.xml.bak
~]# tee /opt/jenkins/hudson.model.UpdateCenter.xml <<-'EOF'
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json</url>
  </site>
</sites>
EOF
```

修改镜像源之后，就可以安装插件了。





