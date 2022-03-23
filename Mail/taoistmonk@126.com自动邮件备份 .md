# 自动发邮件

[TOC]

## 手动发邮件

### 安装mail

```bash
~]# dnf -y install mailx zip unzip
```

### 配置文件 `~/.mailrc`

这里配置的发邮件方是`taoistmonk@126.com`

```bash
$ tee ~/.mailrc <<- 'EOF'
set smtp=smtps://smtp.126.com:465
set smtp-auth=login
set smtp-auth-user=taoistmonk@126.com
set smtp-auth-password=OVPWVQPNJAWURLKL
set ssl-verify=ignore
set nss-config-dir=/etc/pki/nssdb
set from=taoistmonk@126.com
#set smtp-use-starttls=yes
EOF
```

### 发送测试邮件

这里配置的收邮件方是`taoistmonk@163.com`

```bash
$ echo 'content' | mail -s 'title' taoistmonk@163.com
OR
$ cat content.txt | mail -s 'title' taoistmonk@163.com
OR
$ mail -s 'title' taoistmonk@163.com < content.txt
```

## 编写自动发邮件脚本

首先确保devops文件夹放在 `/usr/local/src/devops`

### 压缩打包，带时间戳

```bash
$ zip -r /usr/local/games/devops_$(date +%Y%m%d).zip /usr/local/src/devops
```

### 邮件内容

成功发送

```bash
$ cat /opt/nxsuccess.txt

Daily_Note_Backup

    Send mail Address : taoistmonk@126.com
    Receiver  Address : taoistmonk@163.com

Thank you for marking sure you are received !
```

失败发送

```bash
$ cat /opt/nxfailed.txt

If you have recieved this message ,that means the backup_sever is broken down.
```

### 脚本

```bash
$ vim /opt/sendmail.sh
```

内容为

```shell
#!/bin/bash
cd /usr/local/src/devops && /usr/bin/git pull && /usr/bin/git push
sleep 3
cd /usr/local/src/markdown && /usr/bin/git pull && /usr/bin/git push
sleep 3
cd /usr/local/src/devops && zip -r /usr/local/games/devops_$(date +%Y%m%d).zip *
sleep 8
cd /usr/local/src/markdown && zip -r /usr/local/games/markdown_$(date +%Y%m%d).zip *
sleep 8
if [ -f "/usr/local/games/devops_$(date +%Y%m%d).zip" ];then
    mail -s devops_and_markdown_backup_$(date +%Y%m%d) -a /usr/local/games/devops_$(date +%Y%m%d).zip -a /usr/local/games/markdown_$(date +%Y%m%d).zip taoistmonk@163.com < /opt/nxsuccess.txt
else
    mail -s back_up_failed taoistmonk@163.com < /opt/nxfailed.txt
fi
find /usr/local/games -ctime +40 -type f | xargs rm -rf
```

添加可执行权限

```bash
$ chmod +x /opt/sendmail.sh
```

### 定时任务

```bash
$ crontab -l 

0 2 * * * /opt/sendmail.sh
```

opt文件夹下的文件

```bash
$ ls /opt
nxfailed.txt  nxsuccess.txt  sendmail.sh
```

