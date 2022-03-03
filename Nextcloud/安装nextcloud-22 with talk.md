

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
  mysql:8.0
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
 -v /usr/local/nextcloud/config:/var/www/html/config \
 -v /usr/local/nextcloud/data:/var/www/html/data \
 -v /usr/local/nextcloud/themes:/var/www/html/themes \
 nextcloud:22.2.5
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

### 安装talk

点击右上角，在设置app中搜索talk并下载。

