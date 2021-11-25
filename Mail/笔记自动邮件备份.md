# 自动发邮件

[TOC]

## 手动发邮件

### 安装mail

```bash
~]# dnf -y install mailx
```

### 配置文件 `~/.mailrc`

这里配置的发邮件方是 a@sjhz.cf

    set smtp=smtps://smtp.sjhz.cf:465   
    set smtp-auth=login                  
    set smtp-auth-user=a@sjhz.cf        
    set smtp-auth-password=Vwv56ty7     
    set ssl-verify=ignore              
    set nss-config-dir=/etc/pki/nssdb    
    set from=a@sjhz.cf                   
    #set smtp-use-starttls=yes           

### 发送邮件

这里配置的收邮件方是 b@sjhz.cf

```bash
$ echo 'content' | mail -s 'title' b@sjhz.cf
OR
$ cat content.txt | mail -s 'title' b@sjhz.cf
OR
$ mail -s 'title' b@sjhz.cf < content.txt
```

## 编写自动发邮件脚本

### 压缩打包，带时间戳

```bash
$ zip -r /usr/local/games/devops_$(date +%Y%m%d).zip /usr/local/src/devops
```

### 邮件内容

```bash
$ vi /opt/nxsuccess.txt

Daily_Note_Backup

    Send mail Address : taoistmonk@126.com
    Receiver  Address : taoistmonk@163.com,sjwayrhz@hotmail.com,i@sjhz.cf

Thank you for marking sure you are received !

$ vi /opt/nxfailed.txt

If you have recieved this message ,that means the backup_sever is broken down.

```

### 脚本

```
vi /opt/sendmail.sh
```

内容为

```shell
#!/bin/bash

cd /usr/local/src/devops && /usr/bin/git pull

sleep 3

cd /usr/local/src/markdown && /usr/bin/git pull

sleep 3

cd /usr/local/src && zip -r /usr/local/games/devops_$(date +%Y%m%d).zip *

sleep 8

cd /usr/local/src && zip -r /usr/local/games/markdown_$(date +%Y%m%d).zip *

sleep 8

if [ -f "/usr/local/games/devops_$(date +%Y%m%d).zip" ];then
    mail -s devops_and_markdown_backup_$(date +%Y%m%d) -a /usr/local/games/devops_$(date +%Y%m%d).zip -a /usr/local/games/markdown_$(date +%Y%m%d).zip taoistmonk@163.com < /opt/nxsuccess.txt
else
    mail -s back_up_failed taoistmonk@163.com < /opt/nxfailed.txt
fi

find /usr/local/games -ctime +40 -type f | xargs rm -rf
```



### 定时任务

```
# crontab -l 

0 17 * * * /opt/sendmail.sh
```

opt文件夹下的文件

```
# ls /opt
nxfailed.txt  nxsuccess.txt  sendmail.sh
```