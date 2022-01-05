# docker安装部署confluence

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

创建的数据库名字为confluence, 其它设置为 utf8mb4 和 utf8mb4_bin 。

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



## 构建confluence镜像

### Dockefile

```dockerfile
FROM sjwayrhz/confluence:7.9.3

USER root

# 将代理破解包加入容器
COPY "atlassian-agent.jar" /opt/atlassian/confluence/

# 添加自定义server.xml到容器，可添加支持ssl
COPY server.xml /opt/atlassian/confluence/conf/server.xml

# 设置启动加载代理包
RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/confluence/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/confluence/bin/setenv.sh
```

### 下载破解文件

在github 中下载编译好的即可，放置在Dockerfile同目录下

```
- Confluence
  --Dockerfile
  --server.xml
  --atlassian-agent.jar
```

server.xml直接拷贝至文件夹即可，无需修改。

### 构建镜像

```bash
$ docker build -t sjwayrhz/confluence:confluence.sjhz.tk .
```

### 推送镜像到仓库

```bash
$ docker push sjwayrhz/confluence:confluence.sjhz.tk
```

## docker部署confluence

### 挂载磁盘

后续将数据写入`/var/atlassian/confluence`，于是可以挂在第二磁盘到`/var/atlassian`

### 启动容器

先用如下

```bash
$ docker run -d --name confluence \
  --restart always \
  -p 8090:8090 \
  -p 8091:8091 \
  -e TZ="Asia/Shanghai" \
  -v /var/atlassian/confluence:/var/atlassian/confluence \
  sjwayrhz/confluence:confluence.sjhz.tk
```

### 使用nginx代理confluence

nginx的配置文件

`vim /etc/nginx/conf.d/confluence.sjhz.tk.conf`

内容如下

```nginx
# with no ssl 

upstream confluence_8090 {
    server 127.0.0.1:8090;
}
upstream confluence_8091 {
    server 127.0.0.1:8091;
}

server {
    listen 80;
    server_name confluence.sjhz.tk;
    server_tokens off;
    client_max_body_size 100m;
    access_log /var/log/nginx/confluence_access.log;
    error_log /var/log/nginx/confluence_error.log;

    location / {
        proxy_pass http://confluence_8090;
        proxy_read_timeout 300;
        proxy_connect_timeout 300;
        proxy_redirect off;
        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto http;
    }
    location /synchrony {
	    proxy_pass http://confluence_8091;
 	    proxy_set_header   Host    $host;
	    proxy_set_header   X-Real-IP   $remote_addr;
	    proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
	    proxy_http_version 1.1;
	    proxy_set_header Upgrade $http_upgrade;
	    proxy_set_header Connection "upgrade";
    }
}

# with ssl

server {
	listen        80;
	server_name   confluence.sjhz.tk;
	if ($scheme != "https") {
            return 301 https://$host$request_uri;
        }
}

server {
	client_max_body_size 100m;

	listen        443	ssl;

	ssl_certificate         /etc/nginx/ssl/1_confluence.sjhz.tk_bundle.crt;
	ssl_certificate_key     /etc/nginx/ssl/2_confluence.sjhz.tk.key;

	server_name   confluence.sjhz.tk;
	access_log    /var/log/nginx/confluence.sjhz.tk.log  main;

	location / {
		proxy_pass http://127.0.0.1:8090/;
		proxy_set_header   Host    $host;
		proxy_set_header   X-Real-IP   $remote_addr;
		proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
	}

	location /synchrony {
		proxy_pass http://127.0.0.1:8091;
		proxy_set_header   Host    $host;
		proxy_set_header   X-Real-IP   $remote_addr;
		proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "upgrade";
	}
}
```

上传ssl证书

启动nginx

```bash
$ systemctl enable --now nginx
```

### 连接外部数据库

通过连接字符串， url -> `jdbc:mysql://10.220.62.51/confluence`

完整字符串，url -> `jdbc:mysql://10.220.62.51/confluence?useUnicode=true&amp;characterEncoding=utf8`

可能更换的位置为 ： 

ip地址 - 10.220.62.51

数据库名 - confluence

## 授权码

### 思路

在任意一台安装java环境的linux服务器中，运行`atlassian-agent.jar`可以破解。

破解confluence

样例，本次查询到的服务器ID: `BX2E-GBMA-Q2L4-ZT0J`,破解操作如下：

```bash
$ java -jar atlassian-agent.jar -d -m i@sjhz.tk -n BAT -p conf -o https://confluence.sjhz.tk -s BX2E-GBMA-Q2L4-ZT0J
```

获得的结果如下：

```
====================================================
=======     Atlassian Crack Agent v1.3.1     =======
=======           https://zhile.io           =======
=======          QQ Group: 30347511          =======
====================================================

Your license code(Don't copy this line!!!):

AAABmw0ODAoPeJxtUdtuqzAQfPdXIPXxiNSG5ipZKgFOkxaSBtL08ubQJfiEGGSbNMnXlxCiSkeV/
LKz65nZ2ZtX+DQemTBw1yD2yB6ObNtw46VhYctCrgSmeSE8poGeERMTE3eRv2d51XRoynIFyAOVS
F42yIvI+Y7rmjfnCQgFxvpoZFqXanR7e8p4Dh1eoLncMMHVheTaTQqR5hWIBDrqX3bq6C06Qx2Wa
L4HqmUFyC2Erms/ZDyn/P4612pNmMpo6H65fx+Avfa31XBTOacyVNHnR+xHb7k3h66TLSbbMHHfo
4hYmz9pkE68r6fisFpNDrCg9CIaayY1yHbDBgouIstjCTO2A+rOw9CP3KkToNqO0CBY7d0/lFwe2
8wGQxP364fav1OPBlMv9mdmQHp3xO4R0re7eIBikHuQdXv8Zvnmwzh0zIUV3JkfS/x4Ua8ZmQvi7
KlJYgvHFUh1DpD0MO7jgW2Tq87vJp4rmWRMwf8HbdO70lkortY/F23UGguzarcGOU9fVD1JTYLqR
egvy7RHakIaO8tvROXJJjAuAhUAlwVtY8KS32Vmej6P1+X4fyAnFfMCFQCFCARTbetvdwGold2vo
coX3n0JnQ==X02k0
```

输入授权码即可

## 乱码问题
在我们正常安装之后，中文可能会有乱码，我们修改一下连接字符串，在 confluence 的家目录下面，有一个配置文件confluence.cfg.xml，找到hibernate.connection.url，在数据库字符串后面加上如下字符，整体结果如下：
```
~]# cat /var/atlassian/confluence/confluence.cfg.xml | grep jdbc:mysql
    <property name="hibernate.connection.url">jdbc:mysql://10.220.62.51:3306/confluence?useUnicode=true&amp;characterEncoding=utf8</property>
```

## confluence支持ssl
添加自定义的server.xml文件到Dockerfile根目录
Dockerfile修改如下：

```
FROM sjwayrhz/confluence:7.9.3
USER root
COPY "atlassian-agent.jar" /opt/atlassian/confluence/
COPY server.xml /opt/atlassian/confluence/conf/server.xml
RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/confluence/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/confluence/bin/setenv.sh
```