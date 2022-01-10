# How to Install Jenkins on CentOS 7

[TOC]

### 安装jdk 

这将是启动Jenkins的基础依赖

```bash
~]# wget -P /tmp https://d6.injdk.cn/oraclejdk/8/jdk-8u301-linux-x64.tar.gz
```

解压

```bash
~]# tar -zxvf /tmp/jdk-8u301-linux-x64.tar.gz -C /usr/local/
```

查看jdk目录

```bash
~]# ls /usr/local/jdk1.8.0_301
bin             jre      README.html                         THIRDPARTYLICENSEREADME.txt
COPYRIGHT       lib      release
include         LICENSE  src.zip
javafx-src.zip  man      THIRDPARTYLICENSEREADME-JAVAFX.txt
```

Jdk 系统配置如下

```
export JAVA_HOME=/usr/local/jdk1.8.0_301
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```

maven下载链接

```
https://maven.apache.org/download.cgi
```

Maven  3.8.3地址

```bash
~]# wget -P /tmp https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.8.4/binaries/apache-maven-3.8.4-bin.tar.gz
```

解压

```bash
~]# tar -zxvf /tmp/apache-maven-3.8.4-bin.tar.gz -C /usr/local/
```

Maven系统配置如下

```
export MAVEN_HOME=/usr/local/apache-maven-3.8.4
export PATH=$MAVEN_HOME/bin:$PATH 
```

### 使用rpm包安装

安装依赖

```bash
~]# dnf -y install daemonize
```

目前最新是jenkins-2.319.1-1.1.noarch.rpm

```bash
~]# rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/jenkins-2.319.1-1.1.noarch.rpm
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

查看jdk位置

```bash
~]# which java
/usr/local/jdk1.8.0_301/bin/java
```

配置jdk到jenkins

```bash
~]# vim /etc/init.d/jenkins

搜索 candidates，并添加最后一行配置

......
candidates="
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-11.0/bin/java
/usr/lib/jvm/jre-11.0/bin/java
/usr/lib/jvm/java-11-openjdk-amd64
/usr/bin/java
/usr/local/jdk1.8.0_301/bin/java
"
......
```

### 部署nginx

安装

```bash
~]# dnf install nginx -y 
```

设置nginx配置文件,将`/etc/nginx/nginx.conf` 的server 模块修改为如下内容

```bash
    server {
        listen       80;
        server_name  localhost;
        #access_log /var/log/jenkins_access_log main;
        #error_log  /var/log/jenkins_error_log  debug_http;
        client_max_body_size 60M;
        client_body_buffer_size 512k;
        location / {
            proxy_pass      http://localhost:8080/;
            proxy_redirect  off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         }
     }
```

### 启动jenkins和nginx

```bash
~]# sudo systemctl enable --now jenkins
~]# sudo systemctl enable --now nginx
```

查看初始密码

```bash
~]# cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 修改默认镜像源

```bash
~]# mv /var/lib/jenkins/hudson.model.UpdateCenter.xml /var/lib/jenkins/hudson.model.UpdateCenter.xml.bak
~]# tee /var/lib/jenkins/hudson.model.UpdateCenter.xml <<-'EOF'
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





