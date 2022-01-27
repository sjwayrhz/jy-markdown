# Mysql_8.0的安装与配置

[TOC]

## 安装

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

### 配置安装源

```bash
$ tee /etc/yum.repos.d/mysql-community.repo<<EOF
[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el8/
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
EOF
```

### 下载mysql

```bash
$ dnf install mysql-server -y 
```

### 启动mysql并查看状态

启动

```bash
$ systemctl enable --now mysqld
```

查看状态

```bash
$ systemctl status mysqld
```

### 初始化mysql

```bash
$ mysql_secure_installation

Press y|Y for Yes, any other key for No: y
Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 0
New password: 2wsx#EDC
Re-enter new password: 2wsx#EDC
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : n
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
```

## 授权

### 授权root用户远程登录

登录mysql

```
mysql -uroot -p
```

授权root用户

```mysql
mysql > use mysql;

mysql > update user set host='%' where user='root';

mysql > grant all privileges on *.* to root@'%'

mysql>  flush privileges;
```

## 创建数据库并授权

### 教程

```mysql
创建用户和数据库

mysql>  create user '用户名'@'%' identified by'密码';

mysql>  create database 数据库名;

授予全部权限

mysql>  grant all privileges on 数据库名.* to 用户名@localhost ;

授予一部分权限

mysql>  grant select,insert,delete,update,create,drop, alter, index, create view, show view on jpress.* to 用户名@"%" identified by "密码";

刷新系统权限表

mysql>  flush privileges;
```

### oral3数据库

```mysql
创建用户和数据库

mysql>  create user 'oral3_dba'@'%' identified by'MxE79joA';

mysql>  create database oral3;

授予全部权限

mysql>  grant all privileges on oral3.* to oral3_dba@'%';

刷新系统权限表

mysql>  flush privileges;
```
