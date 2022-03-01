# Mysql_5.7的安装与配置

[TOC]

## 前置条件

安装在 rocky-linux 8.5中 ，安装mysql 5.7,需要移除系统自带的mysql模块

Disable MySQL default AppStream repository:

```bash
$ sudo dnf remove @mysql
$ sudo dnf module reset mysql
$ sudo dnf module disable mysql
```

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
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql57-community-el7/
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

### 查看初使默认的mysql密码

```bash
$ cat /var/log/mysqld.log  | grep password
2022-02-14T01:41:15.321262Z 1 [Note] A temporary password is generated for root@localhost: isFiTl/vo8yb
```

### 初始化mysql

```bash
$ mysql_secure_installation

Enter password for user root: isFiTl/vo8yb
New password: 2wsx#EDC
Re-enter new password: 2wsx#EDC
Change the password for root ? ((Press y|Y for Yes, any other key for No) : n
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

mysql > grant all privileges on *.* to root@'%' identified by '2wsx#EDC';

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

mysql>  set global validate_password_policy=LOW;

mysql>  create user 'oral3_dba'@'%' identified by'MxE79joA';

mysql>  create database oral3;

授予全部权限

mysql>  grant all privileges on oral3.* to oral3_dba@'%';

刷新系统权限表

mysql>  flush privileges;
```
