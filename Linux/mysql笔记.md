mysql  使root可以远程登录

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



