# docker搭建mysql:5.7.29

## 容器中目录结构
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
```
$ cd /usr/local
$ mkdir -p mysql5.7/conf/mysql.conf.d
$ mkdir mysql5.7/scripts
```

文件：
``vim /usr/local/mysql5.7/conf/mysql.conf.d/mysqld.cnf``
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
```

文件：run.sh
 ``vim /usr/local/mysql5.7/scripts/run.sh``

```
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
```
$ cd /usr/local/mysql5.7
$ sh scripts/run.sh
```

进入MySQL服务容器内
```
$ docker exec -it mysql_3306 bash
```

执行：docker ps 查看服务是否启动成功

>注意：data 目录不要随便删除 ,绝对路径是/usr/local/mysql5.7/data
>注意：该镜像自动创建远程账号：root 密码就是初始密码

连接mysql
```
mysql -uroot -p

# 初始密码：123456
```
### 按需操作
```
# 修改本地连接账号-置空
alter user 'root'@'localhost' identified by '';

# 修改远程账号密码-置空
alter user 'root'@'%' identified by '';
```
> 注意：此时连接mysql服务器root密码不再是初始密码，而是刚才设定的密码