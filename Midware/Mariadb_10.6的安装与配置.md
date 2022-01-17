# mariadb_10.6的安装与配置

[TOC]

## 安装

### 启动Mariadb

```bash
$ sudo dnf install MariaDB-server -y
$ sudo systemctl enable --now mariadb
```

### 挂载第二块硬盘到/var/lib/mysql

```bash
$ fdisk /dev/sdb

$ mkfs.xfs -f /dev/sdb

$ vim /etc/fstab
/dev/sdb /var/lib/mysql xfs defaults 0 0
```

挂载硬盘后，重启

### 验证是否挂载成功

```bash
$ df -h | grep mysql
/dev/sdb             100G  746M  100G   1% /var/lib/mysql
```

### 配置mariadb安装源

```bash
$ tee /etc/yum.repos.d/MariaDB.repo <<- "EOF"
# MariaDB 10.6 CentOS repository list - created 2022-01-17 01:46 UTC
# https://mariadb.org/download/
[mariadb]
name = MariaDB
baseurl = https://mirrors.aliyun.com/mariadb/yum/10.6/centos8-amd64
module_hotfixes=1
gpgkey=https://mirrors.aliyun.com/mariadb/yum/RPM-GPG-KEY-MariaDB
gpgcheck=1
EOF
```

### 安装mariadb

使用如下命令安装

```bash
$ dnf install MariaDB-server -y
```

使用如下命令启动

```bash
$ systemctl enable --now mariadb
```

检查是否启动成功

```bash
$ systemctl status mariadb
```

## 初始化

使用如下命令初始化

```bash
$ mariadb-secure-installation
```

关键信息如下：

```
Switch to unix_socket authentication [Y/n] n
Change the root password? [Y/n] y    # secret is  "2wsx#EDC"
Remove anonymous users? [Y/n] y
Disallow root login remotely? [Y/n] n
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y
```

## 授权

### 授权root用户远程登录

登录mysql

```
mysql -uroot -p
```

授权root用户

```mariadb
MariaDB [(none)]> grant all privileges on *.* to 'root'@'%' identified by '2wsx#EDC';

MariaDB [(none)]> flush privileges;
```

## 创建数据库并授权

### 教程

```mariadb
创建用户和数据库

MariaDB [(none)]> create user '用户名'@'%' identified by'密码';

MariaDB [(none)]> create database 数据库名;

授予全部权限

MariaDB [(none)]> grant all privileges on 数据库名.* to 用户名@localhost identified by '密码';

授予一部分权限

MariaDB [(none)]> grant select,insert,delete,update,create,drop, alter, index, create view, show view on jpress.* to 用户名@"%" identified by "密码";

刷新系统权限表

MariaDB [(none)]> flush privileges;
```

### jumpserver数据库

```mariadb
创建用户和数据库

MariaDB [(none)]> create user 'jumpserveruser'@'%' identified by'1qaz@WSX';

MariaDB [(none)]> create database jumpserverdb;

授予全部权限

MariaDB [(none)]> grant all privileges on jumpserverdb.* to jumpserveruser@localhost identified by '1qaz@WSX';

刷新系统权限表

MariaDB [(none)]> flush privileges;
```

