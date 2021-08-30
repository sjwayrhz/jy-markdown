# How to install mysql on centos7

[TOC]

## Introduction

[MySQL](https://www.mysql.com/) is an open-source database management system, commonly installed as part of the popular [LEMP](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-centos-7) (Linux, Nginx, MySQL/MariaDB, PHP/Python/Perl) stack. It uses a relational database and SQL (Structured Query Language) to manage its data.

CentOS 7 prefers MariaDB, a fork of MySQL managed by the original MySQL developers and designed as a replacement for MySQL. If you run `yum install mysql` on CentOS 7, it is MariaDB that is installed rather than MySQL. If you're wondering about MySQL vs. MariaDB, [MariaDB will generally work seamlessly in place of MySQL](https://mariadb.com/kb/en/mariadb/mariadb-vs-mysql-compatibility/), so unless you have a specific use-case for MySQL, see the [How To Install MariaDB on Centos 7](https://www.digitalocean.com/community/tutorials/how-to-install-mariadb-on-centos-7) guide.

This tutorial will explain how to install MySQL version 5.7 on a CentOS 7 server.

## 二进制rpm安装

先登录mysql官网 下载rpm-bundle.tar包

```
~]# wget https://cdn.mysql.com//archives/mysql-5.7/mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar
```

上传完成后，先解压

```
~]# tar xvf mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar -C /tmp

~]# rm -f mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar

mysql-community-embedded-devel-5.7.26-1.el7.x86_64.rpm
mysql-community-libs-5.7.26-1.el7.x86_64.rpm
mysql-community-embedded-5.7.26-1.el7.x86_64.rpm
mysql-community-test-5.7.26-1.el7.x86_64.rpm
mysql-community-embedded-compat-5.7.26-1.el7.x86_64.rpm
mysql-community-common-5.7.26-1.el7.x86_64.rpm
mysql-community-devel-5.7.26-1.el7.x86_64.rpm
mysql-community-client-5.7.26-1.el7.x86_64.rpm
mysql-community-server-5.7.26-1.el7.x86_64.rpm
mysql-community-libs-compat-5.7.26-1.el7.x86_64.rpm
```

分析-需要安装的为以下几个版本

```
mysql-community-common-版本.el7.x86_64.rpm
mysql-community-libs-版本.el7.x86_64.rpm
mysql-community-libs-compat-版本.el7.x86_64.rpm
mysql-community-client-版本.el7.x86_64.rpm
mysql-community-server-版本.el7.x86_64.rpm
```

卸载自带的mariadb-lib

```
~]# yum remove mysql-libs -y
```

安装依赖perl

```
~]# yum install perl -y
```

然后依次安装：

```
~]# rpm -ivh /tmp/mysql-community-common-5.7.26-1.el7.x86_64.rpm
~]# rpm -ivh /tmp/mysql-community-libs-5.7.26-1.el7.x86_64.rpm
~]# rpm -ivh /tmp/mysql-community-libs-compat-5.7.26-1.el7.x86_64.rpm
~]# rpm -ivh /tmp/mysql-community-client-5.7.26-1.el7.x86_64.rpm
~]# rpm -ivh /tmp/mysql-community-server-5.7.26-1.el7.x86_64.rpm
```

启动mysqld

```
~]# systemctl enable --now mysqld
```

查看临时密码

```
~]# grep 'temporary password' /var/log/mysqld.log
```

重新设置mysqld

```
~]# mysql_secure_installation
```

创建用户和授权

```
~]# mysql -uroot -proot -e "CREATE DATABASE jumpserver char set utf8"
~]# mysql -uroot -proot -e "grant all privileges on *.* to jumpserver@'%' identified by 'jumpserver'"
~]# mysql -uroot -proot -e "FLUSH privileges"
```

如果需要设置账号sjhz,密码2wsx#EDC，并赋予所有权限，命令为

```
mysql> grant all privileges on *.* to sjhz@'%' identified by "2wsx#EDC";
```

## tar包安装mysql

卸载系统自带的Mariadb

```
~]# rpm -qa|grep mariadb
mariadb-libs-5.5.64-1.el7.x86_64
~]# rpm -e --nodeps mariadb-libs-5.5.64-1.el7.x86_64
```

检查mysql是否存在

```
~]# rpm -qa | grep mysql
mysql57-community-release-el7-9.noarch
~]# rpm -e --nodeps mysql57-community-release-el7-9.noarch
```

删除etc目录下的my.cnf文件

```
~]# rm /etc/my.cnf
```

检查mysql组和用户是否存在，如无创建

```
~]# cat /etc/group | grep mysql
~]# cat /etc/passwd | grep mysql
```

创建mysql用户组

```
~]# groupadd mysql
```

创建一个用户名为mysql的用户并加入mysql用户组

```
~]# useradd -g mysql mysql
```

制定password 为111111

```
~]# passwd mysql
```

下载并上传mysql的tar.gz包到服务器，下载地址为`https://dev.mysql.com/downloads/mysql/`

```
~]# wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.28-linux-glibc2.12-x86_64.tar.gz
```

解压

```
~]# tar zxvf mysql-5.7.28-linux-glibc2.12-x86_64.tar.gz -C /usr/local
~]# mv /usr/local/mysql-5.7.28-linux-glibc2.12-x86_64 /usr/local/mysql57
~]# rm -f mysql-5.7.28-linux-glibc2.12-x86_64.tar.gz
```

更改所属的组和用户

```
~]# chown -R mysql /usr/local/mysql57
~]# chgrp -R mysql /usr/local/mysql57
~]# mkdir -p /usr/local/mysql57/data
~]# chown -R mysql:mysql /usr/local/mysql57/data
```

在etc下新建配置文件my.cnf，并在该文件内添加以下配置

```
~]# tee /etc/my.cnf <<-'EOF'
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8 

[mysqld]
skip-name-resolve
#设置3306端口
port =3306
# 设置mysql的安装目录
basedir=/usr/local/mysql57
# 设置mysql数据库的数据的存放目录
datadir=/usr/local/mysql57/data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB 
lower_case_table_names=1
max_allowed_packet=16M
EOF
```

安装和初始化

```
~]# /usr/local/mysql57/bin/mysql_install_db --user=mysql --basedir=/usr/local/mysql57/ --datadir=/usr/local/mysql57/data/
```

拷贝启动文件

```
~]# cp /usr/local/mysql57/support-files/mysql.server /etc/init.d/mysqld

~]# chmod +x /etc/init.d/mysqld

~]# /etc/init.d/mysqld restart
```

新增环境变量

```
~]# vi /etc/profile 

添加内容为
export PATH=$PATH:/usr/local/mysql57/bin

~]# source /etc/profile
```

获得初始密码

```
~]# cat /root/.mysql_secret 
# Password set for user 'root@localhost' at 2019-11-12 15:22:26
(pf=1E&3dp_i
```

修改密码

```
~]# mysql -uroot -p
```

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

```
mysql> set PASSWORD = PASSWORD('111111');
```

Query OK, 0 rows affected, 1 warning (0.00 sec)

```
mysql> flush privileges;
```

Query OK, 0 rows affected (0.01 sec)

添加远程访问权限

```
mysql> use mysql
```

Reading table information for completion of table and column names

You can turn off this feature to get a quicker startup with -A

Database changed

```
mysql> update user set host='%' where user='root';
```

Query OK, 1 row affected (0.00 sec)

Rows matched: 1  Changed: 1  Warnings: 0

```
mysql> select host,user from user;
+-----------+---------------+
| host      | user          |
+-----------+---------------+
| %         | root          |
| localhost | mysql.session |
| localhost | mysql.sys     |
+-----------+---------------+
3 rows in set (0.00 sec)
```

```
mysql> create user 'xxx'@'%' identified by '123';  这里 @‘%’ 表示在任何主机都可以登录
```

重启生效

举例创建并授权jumpserver，数据库名，用户名，密码都是jumpserver

```
mysql> CREATE DATABASE jumpserver char set utf8;
mysql> grant all privileges on *.* to jumpserver@'%' identified by 'jumpserver';
mysql> FLUSH privileges;
mysql> exit;
```

为了在任何目录下可以登录mysql

```
~]# ln -s /usr/local/mysql57/bin/mysql  /usr/bin/mysql
```



## English installation

## Step 1 - Add New Repository

MySQL provides a repository for several Linux distributions including  rpm and deb based distributions that contain the latest stable MySQL  release. So we need to add a new MySQL repository to the system to  proceed.

Add the new MySQL repository to the CentOS 7 server with this yum command.

```
~]# yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
```

You will be asked to add a new repository, type '**y**' and press '**Enter**' to confirm.

For Ubuntu, download the MySQL deb package repository file and install it with the dpkg command and then update the repository.

```
wget https://dev.mysql.com/get/mysql-apt-config_0.8.3-1_all.debdpkg -i mysql-apt-config_0.8.3-1_all.debapt-get update
```

During the package repository installation, you will be asked which  MySQL version you want to install and which additional MySQL tools you  want to install. I will leave all at the default, so just choose '**Ok**' and press '**Enter**'.

A new repository has been added to the system.

## Step 2 - Install MySQL 5.7

The new MySQL repository is available on the system now and we are  ready to install MySQL 5.7 latest stable version from the repository.  The package name is 'mysql-community-server'.

On CentOS, install 'mysql-community-server' with yum.

```
~]# yum -y install mysql-community-server
```

For Ubuntu, please use this apt command.

```
apt-get install -y mysql-community-server
```

## Step 3 - Start MySQL and Enable Start at Boot Time

After installing MySQL, start it and add MySQL to start at boot time automatically with the systemctl command.

For CentOS server use 'mysqld' service.

```
~]# systemctl enable --now mysqld
```

For Ubuntu we can use 'mysql' service.

```
systemctl enable --now mysqld
```

MySQL started, it's using port 3306 for the connection, you can check  that on both Ubuntu and Centos server with the netstat command.

```
netstat -plntu
```

If you want to check if the mysql service automatically starts at the boot time or not, you can use the is-enabled option below.

For CentOS.

```
systemctl is-enabled mysqld
```

```
systemctl is-enabled mysql
```

## Step 4 - Configure the MySQL Root Password

MySQL 5.7 is installed and started. As you can see, the MySQL root  password for Ubuntu has been configured during the installation process,  so we do not need to configure or reset the password on Ubuntu. But  this is not the case on CentOS.

On CentOS 7, MySQL will generate a strong default password when MySQL  is started the first time, the default password is shown in the  mysqld.log file. You can use the grep command below for showing the  default MySQL password.

```
grep 'temporary' /var/log/mysqld.log
```

You will see default MySQL root password, and we should set a new default password now.

Connect to the MySQL shell with the default password.

```
mysql -u root -pTYPE DEFAULT PASSWORD
```

Now replace the default password with a new and strong password, in my case, I will set the password 'Newhakase-labs123@' with the MySQL query below.

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Newhakase-labs123@';flush privileges;
```

The default password has been changed.

You can use this query to reset the MySQL root password CentOS and Ubuntu at any time later as well.

## Step 5 - Testing

In this step, we will try to connect to the MySQL shell with the new password and then create a new database and user.

Connect to the MySQL shell with the MySQL root password, using the 'mysql' command.

```
mysql -u root -pTYPE NEW PASSWORD: Newhakase-labs123@
```

Create a new database named 'hakaselabs'. Of course, you can choose your own name here.

```
create database hakaselabs;
```

And create a new user with the name 'hakase' and the password is 'Hakase123@' (or whatever userrname and password you want to use).

```
create user hakase@localhost identified by 'Hakase123@';
```

Now grant privileges for the new database to the new user.

```
grant all privileges on hakaselabs.* to hakase@localhost identified by 'Hakase123@';flush privileges;
```

Exit from the MySQL shell and try connect again with the new user.

```
mysql -u hakase -pTYPE THE PASSWORD: Hakase123@
```

Show the database list under the 'hakase' user.

```
show databases;
```

You can see a database named 'hakaselabs'.

MySQL 5.7 has been installed successfully, it has been started and  configured to automatically start at boot time. And we created a new  user and database and connected successfully to the MySQL shell with our  new user.



如果安装mysql的时候发现已经有mariadb了，可以使用如下命令

```
~]# yum shell

> remove mariadb-libs
> run
```

即可

