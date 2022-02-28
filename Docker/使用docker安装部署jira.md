# docker安装部署jira

[TOC]



## docker部署数据库

容器是 mysql:5.7.29 
```
|-- conf.d
|   |-- docker.cnf
|   |-- mysql.cnf
|   |-- mysqldump.cnf
|-- my.cnf -> /etc/alternatives/my.cnf
|-- my.cnf.fallback
|-- mysql.cnf
|-- mysql.conf.d
    |-- mysqld.cnf
```
注意：不要映射：/etc/mysql/my.cnf，该文件是个软连。即使映射成功配置文件是不会生效的。
当前页可以新建一个配置文件映射到：/etc/mysql/conf.d/ 目录下

## 编写脚本
生成脚本目录
```bash
$ cd /usr/local
$ mkdir -p mysql5.7/conf/mysql.conf.d
$ mkdir mysql5.7/scripts
```

文件：
`vim /usr/local/mysql5.7/conf/mysql.conf.d/mysqld.cnf`

```
[mysql]
default-character-set=utf8

[mysqld]
basedir=/usr/local/mysql/
datadir=/usr/local/mysql/data/

pid-file    = /var/run/mysqld/mysqld.pid
socket        = /var/run/mysqld/mysqld.sock

#log-error    = /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address    = 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

server-id=1

innodb_log_file_size=256M
max_allowed_packet=34M
transaction-isolation=READ-COMMITTED
```

文件：run.sh
 `vim /usr/local/mysql5.7/scripts/run.sh`

```bash
#!/usr/bin/env bash

# set mysql port
mysql_port=$1
if [ ! "$mysql_port" ]; then
  mysql_port=3306
fi

# scripts_path
scrpts_path=$(cd $(dirname "$0") || exit; pwd)

# mysql root path
dirpath=$(dirname "$scrpts_path")

# create mysql data dir
mkdir -p "$dirpath"/data

# run mysql server
docker rm mysql_"$mysql_port" -f

docker run -d -p "$mysql_port":3306 --name mysql_"$mysql_port" \
-v="$dirpath"/conf/mysql.conf.d/mysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf \
-v="$dirpath"/data:/usr/local/mysql/data/ \
-v="$dirpath"/data:/var/log/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:5.7.29
```

目录如下
```
$ tree mysql5.7/
mysql5.7/
|-- conf
|   `-- mysql.conf.d
|       `-- mysqld.cnf
`-- scripts
    `-- run.sh

3 directories, 2 files
```
启动MySQL服务
```bash
$ cd /usr/local/mysql5.7
$ sh scripts/run.sh
```

使用navicat，账号密码为(root/123456)创建的数据库名字为jira, 其它设置为 utf8mb4 和 utf8mb4_general_ci 

启动之后，可以更新mysql容器为自动启动

```bash
$ docker update --restart=always 容器ID(或者容器名)
```



## 所需素材

本文采用素材如下：

### Dockerfile

Docker镜像 Github链接 `https://github.com/cptactionhank/`

### atlassian-agent.jar

破解工具 `git@gitee.com:pengzhile/atlassian-agent.git` 

破解工具1.3.1蓝奏云地址：`https://wwi.lanzouo.com/iu0j9ye53ne`

采用以上工具，理论上可以破解几乎全部版本。

### server.xml

```xml
<?xml version="1.0"?>
<Server port="8000" shutdown="SHUTDOWN">
  <Service name="Tomcat-Standalone">
        <Connector port="8090" connectionTimeout="20000" redirectPort="8443"
                   maxThreads="48" minSpareThreads="10"
                   enableLookups="false" acceptCount="10" debug="0" URIEncoding="UTF-8"
                   protocol="org.apache.coyote.http11.Http11NioProtocol"
                   scheme="https" secure="true" proxyName="confluence.sjhz.tk" proxyPort="443"/>
    <Engine name="Standalone" defaultHost="localhost">
      <Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="false" startStopThreads="4">
        <Context path="" docBase="../confluence" reloadable="false" useHttpOnly="true">
          <!-- Logging configuration for Confluence is specified in confluence/WEB-INF/classes/log4j.properties -->
          <Manager pathname=""/>
          <Valve className="org.apache.catalina.valves.StuckThreadDetectionValve" threshold="60"/>
        </Context>
        <Context path="${confluence.context.path}/synchrony-proxy" docBase="../synchrony-proxy" reloadable="false" useHttpOnly="true">
          <Valve className="org.apache.catalina.valves.StuckThreadDetectionValve" threshold="60"/>
        </Context>
      </Host>
    </Engine>
  </Service>
</Server>
```



## 构建jira镜像

### 制作Docker破解容器
编写Dockerfile文件：
```
FROM cptactionhank/atlassian-jira-software:latest

USER root

COPY "atlassian-agent.jar" /opt/atlassian/jira/

RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/jira/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/jira/bin/setenv.sh
```
### 下载破解文件
在github 中下载编译好的即可，放置在Dockerfile同目录下
```
- JIRA
  --Dockerfile
  --atlassian-agent.jar
```

### 构建镜像
`docker build -t sjwayrhz/jira:latest .`
结果如下：
```
Sending build context to Docker daemon  2.141MB
Step 1/4 : FROM cptactionhank/atlassian-jira-software:latest
 ---> c51100467795
Step 2/4 : USER root
 ---> Running in 3f9cea0602c7
Removing intermediate container 3f9cea0602c7
 ---> 4b9e20ba43cf
Step 3/4 : COPY "atlassian-agent.jar" /opt/atlassian/jira/
 ---> 61155470b50a
Step 4/4 : RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/jira/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/jira/bin/setenv.sh
 ---> Running in 5aed1ac41ab7
Removing intermediate container 5aed1ac41ab7
 ---> 33d0b86f8262
Successfully built 33d0b86f8262
Successfully tagged sjwayrhz/jira:latest
```
### 启动容器
```bash
$ docker run -d --name jira \
  --restart always \
  -p 80:8080 \
  -e TZ="Asia/Shanghai" \
  -m 4096M \
  -v /home/data/www/jira.sjhz.tk:/var/atlassian/jira \
  sjwayrhz/jira:1.1
```



## 授权码

### 思路

在任意一台安装java环境的linux服务器中，运行`atlassian-agent.jar`可以破解。

破解confluence

样例，本次查询到的服务器ID: `BX2E-GBMA-Q2L4-ZT0J`,破解操作如下：

```bash
$ java -jar atlassian-agent.jar -d -m i@sjhz.tk -n BAT -p jira -o https://jira.sjhz.tk -s BBML-9H9P-SOS5-QV59
```

获得的结果如下：

```
====================================================
=======     Atlassian Crack Agent v1.3.1     =======
=======           https://zhile.io           =======
=======          QQ Group: 30347511          =======
====================================================

Your license code(Don't copy this line!!!):

AAAB4g0ODAoPeJyNU12PojAUfedXkOwzWvADNGmyip0MWT7GQc3OvBW8SkcEti06zq9fBMzOjMZs0
pc259x7zrm3P7w8Ux8gUg1L1c0xQuOBqdrhQjWQYShbDpAleVEA77gshkzA4lSAT/eA7cDzyLPtT
FzF5kAly7MZlYDPRA0ZmmEpdygzEDFnxZmFl1nK9kzCWk0bghqd1ETKQoy73Y+EpdBhueJRlknIa
BYDeS8YP7XdrJGGzOoob4zTi0qyZk1p33U8Z0Fmil/uI+DBZimAC6zpF3F3ahU8X5ex7Jwvmsg38
kg5dK4K3cHSWLIDYMlL+JLl5/c79EoVtaFyzRtoG8+qanw2ZyhhGf2LsYaQA03Lehh4Q1PRlv9eK
OBbmjHR4C5J10jxlnx05E6x80xWKkmVeorZz8tzjblKoNX1SEWCPRvZD3ObHH9F3nEJlm8K9zWOy
623j6Pg9cWeOMvfKekmO5+M/syRa1lhH8pn48VZvVtb3LT4z4BCSfnZVGO1nagzw64zC4mvufqwP
0Q9yxqYRt/4siC3djIEfgBe0adTz9VGj6MnLQzCgTZfDUbKDk6X3PUhQiayej391ge5Xr2nkscJF
fD9e3wm18MpOBOt6Uo+vmGhHUutfDpZ/AWgl0bvMC0CFQCA6/fsEZoiHOeBhyWt0vS6WZjRdwIUP
FRPYVKzzH+UnkXkI8wH6OAR884=X02mu
```

输入授权码即可

## 乱码问题
在我们正常安装之后，中文可能会有乱码，我们修改一下连接字符串，在 confluence 的家目录下面，有一个配置文件confluence.cfg.xml，找到hibernate.connection.url，在数据库字符串后面加上如下字符，整体结果如下：
```
~]# cat /var/atlassian/confluence/confluence.cfg.xml | grep jdbc:mysql
    <property name="hibernate.connection.url">jdbc:mysql://10.220.62.51:3306/confluence?useUnicode=true&amp;characterEncoding=utf8</property>
```



