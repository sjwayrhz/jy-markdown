# Mysql操作笔记

[TOC]

## MySQL创建用户与授权

### 创建用户

```mariadb
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

说明：
username：你将创建的用户名
host：指定该用户在哪个主机上可以登陆，如果是本地用户可用localhost，如果想让该用户可以从任意远程主机登陆，可以使用通配符%
password：该用户的登陆密码，密码可以为空，如果为空则该用户可以不需要密码登陆服务器
例子：

```mysql
CREATE USER 'dog'@'localhost' IDENTIFIED BY '123456';
CREATE USER 'pig'@'192.168.1.101_' IDENDIFIED BY '123456';
CREATE USER 'pig'@'%' IDENTIFIED BY '123456';
CREATE USER 'pig'@'%' IDENTIFIED BY '';
CREATE USER 'pig'@'%';
```

### 授权:

```mariadb
GRANT privileges ON databasename.tablename TO 'username'@'host'
```

说明:
privileges：用户的操作权限，如SELECT，INSERT，UPDATE等，如果要授予所的权限则使用ALL
databasename：数据库名
tablename：表名，如果要授予该用户对所有数据库和表的相应操作权限则可用*表示，如*.*
例子:

```mariadb
GRANT SELECT, INSERT ON test.user TO 'pig'@'%';
GRANT ALL ON *.* TO 'pig'@'%';
GRANT ALL ON maindataplus.* TO 'pig'@'%';
```

注意:
用以上命令授权的用户不能给其它用户授权，如果想让该用户可以授权，用以下命令:

```mariadb
GRANT privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION;
```

例子：

```mariadb
 grant ALL privileges on juneyao_party_test.* to 'luocheng'@'%' WITH GRANT OPTION;
```

### 设置与更改用户密码

```mariadb
SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');
```

如果是当前登陆用户用:

```mariadb
SET PASSWORD = PASSWORD("newpassword");
```

例子:

```mariadb
SET PASSWORD FOR 'pig'@'%' = PASSWORD("123456");
```

### 撤销用户权限

```mariadb
REVOKE privilege ON databasename.tablename FROM 'username'@'host';
```

说明:
privilege, databasename, tablename：同授权部分

例子:

```mariadb
REVOKE SELECT ON *.* FROM 'pig'@'%';
```

注意:
假如你在给用户'pig'@'%'授权的时候是这样的（或类似的）：`GRANT SELECT ON test.user TO 'pig'@'%'`，则在使用`REVOKE SELECT ON *.* FROM  'pig'@'%'`;命令并不能撤销该用户对test数据库中user表的SELECT 操作。相反，如果授权使用的是`GRANT SELECT ON  *.* TO 'pig'@'%';则REVOKE SELECT ON test.user FROM  'pig'@'%'`;命令也不能撤销该用户对test数据库中user表的Select权限。

具体信息可以用命令`SHOW GRANTS FOR 'pig'@'%'`; 查看。

### 刷新列表

```mariadb
flush privileges
```

### 删除用户

```mariadb
DROP USER 'username'@'host';
```

## mysql  使root可以远程登录

```mysql
 mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
 mysql> FLUSH PRIVILEGES;
```

可以通过mysql的user查看

```mysql
mysql> use mysql 
mysql> select user,host from user;
+---------------+-----------+
| user          | host      |
+---------------+-----------+
| repl          | %         |
| root          | %         |
| mysql.session | localhost |
| mysql.sys     | localhost |
| root          | localhost |
+---------------+-----------+
5 rows in set (0.00 sec)
```