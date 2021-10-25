# 快速部署confluence

[TOC]

## 不使用SSL证书

### nossl_docker启动
```
docker run -d --name confluence   \
    --restart always   \
    -p 8090:8090 -p 8091:8091  \
    -e TZ="Asia/Shanghai"   \
    -v /data:/var/atlassian/confluence   \
    sjwayrhz/confluence:1.1
```

### nossl_nginx配置
```
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
```

## 使用SSL证书

### 思路

如果需要使用ssl证书，则首先需要自定义dockerfile文件，修改confluence默认的tomcat文件中的conf/server.xml，添加https域名的关键配置`proxyName="confluence.sjhz.tk" proxyPort="443" scheme="https"`

重新构建docker镜像，启动docker镜像。

然后再配置nginx，使得其支持https。

### 添加server.xml
```
<?xml version="1.0"?>
<Server port="8000" shutdown="SHUTDOWN">
  <Service name="Tomcat-Standalone">
    <Connector port="8090" 
        connectionTimeout="20000" 
        redirectPort="8443" 
        maxThreads="48" 
        minSpareThreads="10" 
        enableLookups="false" 
        acceptCount="10" 
        URIEncoding="UTF-8" 
        protocol="org.apache.coyote.http11.Http11NioProtocol"
        proxyName="confluence.sjhz.tk" proxyPort="443" scheme="https"/>
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
关键是，修改了这一行
```
proxyName="confluence.sjhz.tk" proxyPort="443" scheme="https"/>
```

### 制作Dockerfile
```
FROM cptactionhank/atlassian-confluence:latest
USER root
COPY "atlassian-agent.jar" /opt/atlassian/confluence/
COPY server.xml /opt/atlassian/confluence/conf/server.xml
RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/confluence/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/confluence/bin/setenv.sh
```
### 制作docker镜像
```
docker build -t sjwayrhz/confluence:confluence.sjhz.tk .
```

withssl_docker启动
```
docker run -d --name confluence   \
    --restart always   \
    -p 8090:8090 -p 8091:8091  \
    -e TZ="Asia/Shanghai"   \
    -v /data:/var/atlassian/confluence   \
    sjwayrhz/confluence:confluence.sjhz.tk
```

配置nginx 
```
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

### 查看数据库的配置
```
~]# grep -C 10 "mariadb" /etc/my.cnf.d/server.cnf
# Optional setting
#wsrep_slave_threads=1
#innodb_flush_log_at_trx_commit=0

# this is only for embedded server
[embedded]

# This group is only read by MariaDB servers, not by MySQL.
# If you use the same .cnf file for MySQL and MariaDB,
# you can put MariaDB-only options here
[mariadb]
innodb_log_file_size=256M
max_allowed_packet=34M
transaction-isolation=READ-COMMITTED

# This group is only read by MariaDB-10.6 servers.
# If you use the same .cnf file for MariaDB of different versions,
# use this group for options that older servers don't understand
[mariadb-10.6]
```

## 乱码问题
在我们正常安装之后，中文可能会有乱码，我们修改一下连接字符串，在 confluence 的家目录下面，有一个配置文件confluence.cfg.xml，找到hibernate.connection.url，在数据库字符串后面加上如下字符，整体结果如下：
```
~]# cat /home/data/www/confluence.sjhz.tk/confluence.cfg.xml | grep jdbc:mysql
    <property name="hibernate.connection.url">jdbc:mysql://10.230.7.33:3306/confdb?useUnicode=true&amp;characterEncoding=utf8</property>
```