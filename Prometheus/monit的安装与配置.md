# # monit

[TOC]

## monit的安装

### 使用yum安装

```
~]# yum install monit
```

查看版本

```
~]# monit --version
This is Monit version 5.25.1
Built with ssl, with ipv6, with compression, with pam and with large files
Copyright (C) 2001-2017 Tildeslash Ltd. All Rights Reserved.
```

### 使用rpm包安装

下载地址

```
https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/m/monit-5.25.1-1.el7.x86_64.rpm
```

启动，并创建自动启动

```
~]# systemctl enable monit
Created symlink from /etc/systemd/system/multi-user.target.wants/monit.service to /usr/lib/systemd/system/monit.service.

~]# systemctl start monit
```

检测monit状态

```
~]# systemctl status monit
● monit.service - Pro-active monitoring utility for unix systems
   Loaded: loaded (/usr/lib/systemd/system/monit.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2019-05-15 15:28:58 CST; 1s ago
  Process: 66253 ExecStop=/usr/bin/monit quit (code=exited, status=0/SUCCESS)
 Main PID: 66414 (monit)
    Tasks: 2
   Memory: 808.0K
   CGroup: /system.slice/monit.service
           └─66414 /usr/bin/monit -I

May 15 15:28:58 ajast32 systemd[1]: Started Pro-active monitoring utility for unix systems.
May 15 15:28:58 ajast32 monit[66414]: Starting Monit 5.25.1 daemon with http interface at [lo...2812
May 15 15:28:58 ajast32 monit[66414]: 'ajast32' Monit 5.25.1 started
Hint: Some lines were ellipsized, use -l to show in full.
```

需要在系统内放置一个getServerStatus.sh脚本

```
~]# vi /usr/local/bin/getServerStatus.sh
```

脚本内容为

```
#!/bin/bash
DATE=`date +%Y%m%d-%H%M%S`
DIR=`dirname $0`
ps auxf > /var/log/serverDumpLog_${DATE}.log
```

设置可执行文件

```
~]# chmod +x /usr/local/bin/getServerStatus.sh
```

### 查看配置文件

原先的配置文件查看

```
~]# cat /etc/monitrc | grep -v "^#" | grep -v "^$"
set daemon  30              # check services at 30 seconds intervals
set log syslog
set httpd port 2812 and
    use address localhost  # only accept connection from localhost
    allow localhost        # allow localhost to connect to the server and
    allow admin:monit      # require user 'admin' with password 'monit'
    #with ssl {            # enable SSL/TLS and set path to server certificate
    #    pemfile: /etc/ssl/certs/monit.pem
    #}
include /etc/monit.d/*
```

备份配置文件

```
~]# cp /etc/monitrc /etc/monitrc.backup
~]# cat /dev/null > /etc/monitrc
```

编辑配置文件

```
~]# vi /etc/monitrc
```

`/etc/monitrc`内容为

```
set logfile /var/log/monit.log

set pidfile /var/run/monit.pid


set mail-format {
      from: monit@anji-leasing.cn
   subject: monit alert --  $EVENT $SERVICE
   message: $EVENT Service $SERVICE
                 Date:        $DATE
                 Action:      $ACTION
                 Host:        $HOST
                 Description: $DESCRIPTION

            Your faithful employee,
            Monit

set httpd port 2812 and

check system $HOST
  if loadavg (1min) > 4 then exec "/usr/local/bin/getServerStatus.sh"
  if loadavg (5min) > 2 then exec "/usr/local/bin/getServerStatus.sh"
  if cpu usage > 80% for 3 cycles then exec "/usr/local/bin/getServerStatus.sh"
  if memory usage > 75% then exec "/usr/local/bin/getServerStatus.sh"
  if swap usage > 50% then alert
```

## monit的配置

### centos 7

```shell
# cat crond.conf
check process crond with pidfile /var/run/crond.pid
  start program = "/etc/init.d/crond start" with timeout 30 seconds
  stop program = "/etc/init.d/crond stop"
  if 3 restarts within 5 cycles then unmonitor
```

```shell
 # cat disk.conf
check device xvda1 with path /
    if SPACE usage > 80% then alert
```

```shell
# cat chronyd.conf
check process chronyd with pidfile /var/run/chronyd.pid
  start program = "/bin/systemctl start chronyd" with timeout 30 seconds
  stop program = "/bin/systemctl stop chronyd"
  if 3 restarts within 5 cycles then unmonitor
```

```shell
# cat crond.conf
check process crond with pidfile /var/run/crond.pid
  start program = "/bin/systemctl start crond" with timeout 30 seconds
  stop program = "/bin/systemctl stop crond"
  if 3 restarts within 5 cycles then unmonitor
```

```shell
# cat logging
# log to monit.log
set logfile /var/log/monit.log
```

```shell
# cat mysql.conf
check process mysqld with pidfile /var/run/mariadb/mariadb.pid
  start program = "/bin/systemctl start mariadb" with timeout 30 seconds
  stop program = "/bin/systemctl stop mariadb"
  if failed unixsocket /var/lib/mysql/mysql.sock with timeout 10 seconds then restart
  if 3 restarts within 5 cycles then unmonitor
```

```shell
# cat nginx.conf
check process nginx with pidfile /var/run/nginx.pid
  start program = "/bin/systemctl start nginx" with timeout 30 seconds
  stop program = "/bin/systemctl stop nginx"
  if failed port 80 protocol http
     and request "/" status = 200
     then restart
  if 3 restarts within 5 cycles then unmonitor
```

```shell
# cat php-fpm.conf
check process php-fpm with pidfile /var/run/php-fpm/php-fpm.pid
  start program = "/bin/systemctl start php-fpm" with timeout 30 seconds
  stop program = "/bin/systemctl stop php-fpm"
  if 3 restarts within 5 cycles then unmonitor
```

```shell
# cat sendmail.conf
check process sendmail with pidfile /var/run/sendmail.pid
  start program = "/bin/systemctl start sendmail" with timeout 30 seconds
  stop program = "/bin/systemctl stop sendmail"
  if 3 restarts within 5 cycles then unmonitor
```

```shell
# cat sshd.conf
check process sshd with pidfile /var/run/sshd.pid
  start program = "/bin/systemctl start sshd" with timeout 30 seconds
  stop program = "/bin/systemctl stop sshd"
  if 3 restarts within 5 cycles then unmonitor
```

```shell
# cat zabbix_agentd.conf
check process zabbix-agent with pidfile /var/run/zabbix/zabbix_agentd.pid
  start program = "/bin/systemctl start zabbix-agent" with timeout 30 seconds
  stop program = "/bin/systemctl stop zabbix-agent"
  if 3 restarts within 5 cycles then unmonitor
```

### Centos 6
```shell
# cat crond.conf
check process crond with pidfile /var/run/crond.pid
  start program = "/etc/init.d/crond start" with timeout 30 seconds
  stop program = "/etc/init.d/crond stop"
  if 3 restarts within 5 cycles then unmonitor
```

```shell
# cat disk.conf
check device xvda1 with path /
    if SPACE usage > 80% then alert
```

```shell
# cat fail2ban.conf
check process fail2ban with pidfile /var/run/fail2ban/fail2ban.pid
  start program = "/etc/init.d/fail2ban start" with timeout 30 seconds
  stop program = "/etc/init.d/fail2ban stop"
```
```shell
# cat jetty.conf
check process jetty with pidfile /var/run/jetty.pid
  start program = "/etc/init.d/jetty start" with timeout 30 seconds
  stop program  = "/etc/init.d/jetty stop"
  if failed port 8080 protocol http with timeout 15 seconds then restart
```
```shell
# cat mysql.conf
check process mysql with pidfile /var/run/mysqld/mysqld.pid
  start program = "/etc/init.d/mysqld start" with timeout 30 seconds
  stop program = "/etc/init.d/mysqld stop"
  if failed port 3306 protocol mysql with timeout 10 seconds then restart
  if 3 restarts within 5 cycles then unmonitor
```
```powershell
# cat nginx.conf
check process nginx with pidfile /var/run/nginx.pid
  start program = "/etc/init.d/nginx start" with timeout 30 seconds
  stop program  = "/etc/init.d/nginx stop"
  if failed port 80 protocol http
     and request "/index.php"
     then restart
  if 3 restarts within 5 cycles then unmonitor
```
```shell
# cat ntpd.conf
check process ntpd with pidfile /var/run/ntpd.pid
  start program = "/etc/init.d/ntpd start" with timeout 30 seconds
  stop program = "/etc/init.d/ntpd stop"
  if 3 restarts within 5 cycles then unmonitor
```
```shell
# cat sendmail.conf
check process sendmail with pidfile /var/run/sendmail.pid
  start program = "/etc/init.d/sendmail start" with timeout 30 seconds
  stop program = "/etc/init.d/sendmail stop"
  if 3 restarts within 5 cycles then unmonitor
```
```shell
# cat sshd.conf
check process sshd with pidfile /var/run/sshd.pid
  start program = "/etc/init.d/sshd start" with timeout 30 seconds
  stop program = "/etc/init.d/sshd stop"
  if 3 restarts within 5 cycles then unmonitor
```
```shell
# cat td-agent.conf
check process td-agent with pidfile /var/run/td-agent/td-agent.pid
  start program = "/etc/init.d/td-agent start" with timeout 30 seconds
  stop program = "/etc/init.d/td-agent stop"
```

```shell
# cat vsftpd.conf
check host vsftpd with address 127.0.0.1
  start program = "/etc/init.d/vsftpd start" with timeout 30 seconds
  stop program = "/etc/init.d/vsftpd stop"
  if failed port 21 protocol ftp with timeout 15 seconds then restart
  if 3 restarts within 5 cycles then unmonitor
```

 ```shell
# cat php-fpm.conf
check process php-fpm with pidfile /usr/local/php/var/run/php-fpm.pid
  start program = "/etc/init.d/php-fpm start" with timeout 30 seconds
  stop program = "/etc/init.d/php-fpm stop"
  if 3 restarts within 5 cycles then unmonitor
 ```


### 设置monit自动发邮件

#### 修改之前的monitrc配置文件

```
备份 
# cp /etc/monitrc /etc/monitrc.bak.20181029
确认备份成功
# ll /etc/monitrc
-rw-r--r-- 1 root root 12293 Oct 29 14:57 /etc/monitrc
# ll /etc/monitrc.bak.20181029
-rw------- 1 root root 12974 Dec 27  2017 /etc/monitrc.bak.20181029


查看修改之前的monit配置文件
# cat /etc/monitrc.bak.20181029 | grep -v '#'

set log syslog



set httpd port 2812 and

include /etc/monit.d/*
```

#### 修改之后的monitrc配置文件

```
# vi /etc/monitrc
# cat /etc/monitrc | grep -v '#'

set logfile /var/log/monit.log

set pidfile /var/run/monit.pid


set mail-format {
      from: monit@wechat-test.daikin.net.cn
   subject: monit alert --  $EVENT $SERVICE
   message: $EVENT Service $SERVICE
                 Date:        $DATE
                 Action:      $ACTION
                 Host:        $HOST
                 Description: $DESCRIPTION

            Your faithful employee,
            Monit

set httpd port 2812 and

check system $HOST
  if loadavg (1min) > 4 then exec "/usr/local/script/getServerStatus.sh"
  if loadavg (5min) > 2 then exec "/usr/local/script/getServerStatus.sh"
  if cpu usage > 80% for 3 cycles then exec "/usr/local/script/getServerStatus.sh"
  if memory usage > 75% then exec "/usr/local/script/getServerStatus.sh"
  if swap usage > 50% then alert
```

#### 比较不同

```shell
# diff /etc/monitrc /etc/monitrc.bak.20181029
27c27
< set logfile /var/log/monit.log
---
> set log syslog
34c34
< set pidfile /var/run/monit.pid
---
> # set pidfile /var/run/monit.pid
116,127c116,127
< set mail-format {
<       from: monit@$HOST
<    subject: monit alert --  $EVENT $SERVICE
<    message: $EVENT Service $SERVICE
<                  Date:        $DATE
<                  Action:      $ACTION
<                  Host:        $HOST
<                  Description: $DESCRIPTION
<
<             Your faithful employee,
<             Monit
<
---
> ## set mail-format {
> ##   from:    Monit <monit@$HOST>
> ##   subject: monit alert --  $EVENT $SERVICE
> ##   message: $EVENT Service $SERVICE
> ##                 Date:        $DATE
> ##                 Action:      $ACTION
> ##                 Host:        $HOST
> ##                 Description: $DESCRIPTION
> ##
> ##            Your faithful employee,
> ##            Monit
> ## }
146c146
< set alert itc.zxg@daikin.net.cn not on { action }   # receive all alerts without action
---
> # set alert your-name@your.domain not on { instance, action }
173,178c173,178
< check system $HOST
<   if loadavg (1min) > 4 then exec "/usr/local/script/getServerStatus.sh"
<   if loadavg (5min) > 2 then exec "/usr/local/script/getServerStatus.sh"
<   if cpu usage > 80% for 3 cycles then exec "/usr/local/script/getServerStatus.sh"
<   if memory usage > 75% then exec "/usr/local/script/getServerStatus.sh"
<   if swap usage > 50% then alert
---
> #  check system $HOST
> #    if loadavg (1min) > 4 then alert
> #    if loadavg (5min) > 2 then alert
> #    if cpu usage > 95% for 10 cycles then alert
> #    if memory usage > 75% then alert
> #    if swap usage > 25% then alert
303a304,305
> #
> include /etc/monit.d/*
```

