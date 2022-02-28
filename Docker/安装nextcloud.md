

### 安装docker 

```bash
$ wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_docker.sh | sh
```

### 部署mysql

安装mysql

```bash
$ docker run -d -p 3306:3306 --restart=always --name mysql \
  -v /usr/local/mysql/conf:/etc/mysql/conf.d \
  -v /usr/local/mysql/logs:/logs \
  -v /usr/local/mysql/data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=123456  \
  mysql:5.7
```

配置mysql

```mysql
$ docker exec -it mysql容器ID /bin/bash

root@ID:/# mysql -uroot -p123456
mysql> GRANT ALL PRIVILEGES on *.* to root@'%' WITH GRANT OPTION;
mysql> ALTER USER 'root'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
Query OK, 0 rows affected (0.02 sec)
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.01 sec)
mysql> FLUSH PRIVILEGES;
mysql> exit
```

### 安装nextcloud 

使用如下命令运行

```bash
$ docker run -d -p 80:80 --restart=always \
 -v /usr/local/nextcloud/html:/var/www/html \
 -v /usr/local/nextcloud/apps:/var/www/html/custom_apps \
 -v /usr/local/nextlcoud/config:/var/www/html/config \
 -v /usr/local/nextcloud/nextcloud/data:/var/www/html/data \
 -v /usr/local/nextcloud/themes:/var/www/html/themes \
 nextcloud:18
```

如果安装  nextcloud:22.2.3  ，数据库需要安装 mysql:8.0  ,并且需要在  config.php 文件中 添加 'allow_local_remot_servers' => true;

打开`&host:&IP`配置

本次地址为：192.168.177.12:80

```
账号：admin
密码: 2wsx#EDC
Data flolder: /var/www/html/data
Mysql/MariaDB
Database user: root
Database password: 123456
Database name: nextcloud
Database host: 192.168.177.12:3306
```

不需要提前创建数据库，这样的配置，系统会自动创建数据库

#### 安装onlyoffice

```bash
$ docker run -i -t -d -p 6060:80 --restart=always \
  -v /app/onlyoffice/DocumentServer/logs:/var/log/onlyoffice \
  -v /app/onlyoffice/DocumentServer/data:/var/www/onlyoffice/Data \
  -v /app/onlyoffice/DocumentServer/lib:/var/lib/onlyoffice \
  -v /app/onlyoffice/DocumentServer/db:/var/lib/postgresql \
  onlyoffice/documentserver:6.4
```

#### 配置onlyoffice

1. nextcloud安装onlyoffice app

   点击右上角用户图标--》应用--》office & text--》only office --》点击下载安装并启用

2. 修改nextcloud config.php设置

   nextcloud 18版本之前不需要，19与19版本之后需要

```php
 $ vim /usr/local/nextlcoud/config/config.php
 添加如下语句   
 'allow_local_remote_servers' => true,
```

配置nextcloud onlyoffice 服务器地址

点击右上角用户图标--》设置--》onlyoffice--》配置相关信息

