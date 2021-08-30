JDK下载地址

```
~]# wget https://repo.huaweicloud.com/java/jdk/8u192-b12/jdk-8u192-linux-x64.tar.gz
```

解压

```
~]# tar -zxvf jdk-8u192-linux-x64.tar.gz -C /usr/local/
```

查看jdk目录

```bash
~]# ls /usr/local/jdk1.8.0_192
bin             jre      README.html                         THIRDPARTYLICENSEREADME.txt
COPYRIGHT       lib      release
include         LICENSE  src.zip
javafx-src.zip  man      THIRDPARTYLICENSEREADME-JAVAFX.txt
```

Jdk 系统配置如下

```
export JAVA_HOME=/usr/local/jdk1.8.0_192
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```

maven下载链接

```
https://maven.apache.org/download.cgi
```

Maven  3.8.1地址

```
~]# wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.8.1/binaries/apache-maven-3.8.1-bin.tar.gz
```

Maven系统配置如下

```
export MAVEN_HOME=/usr/local/apache-maven-3.8.1
export PATH=$MAVEN_HOME/bin:$PATH 
```

tomcat下载地址

```
~]# wget https://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.69/bin/apache-tomcat-8.5.69.tar.gz 
```

设置tomcat的pid

```
~]# touch /var/run/tomcat.pid
```

设置tomcat的systemd

```bash
~]# cat /usr/lib/systemd/system/tomcat.service
[Unit]
Description=tomcat
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/local/jdk1.8.0_192
Environment=CATALINA_PID=/var/run/tomcat.pid
Environment=CATALINA_HOME=/usr/local/apache-tomcat-8.5.69/
Environment=CATALINA_BASE=/usr/local/apache-tomcat-8.5.69/
Environment=CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC

WorkingDirectory=/usr/local/apache-tomcat-8.5.69/


PIDFile=/var/run/tomcat.pid
ExecStart=/usr/local/apache-tomcat-8.5.69/bin/catalina.sh start
ExecReload=/usr/local/apache-tomcat-8.5.69/bin/catalina.sh stop && /usr/local/apache-tomcat-8.5.69/bin/catalina.sh start
ExecStop=/usr/local/apache-tomcat-8.5.69/bin/catalina.sh stop
PrivateTmp=true
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

