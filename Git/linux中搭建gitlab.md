# 在linux中搭建linux

[TOC]

操作系统 : rockylinux

## 环境准备

### 下载postfix

```bash
$ dnf install postfix -y 
$ systemctl enable --now postfix
```

### 配置环境变量

```bash
$ vim /etc/profile
# 将以下内容添加到配置文件末尾
export LC_CTYPE=en_US.UTF-8
export LC_ALL=en_US.UTF-8
# 重读环境变量
$ source /etc/profile
```

## 部署gitlab

### 在线安装gitlab 

安装gitlab的repo

```bash
$ curl -f -sSL https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash 
```

现在可以使用dnf或者yum安装gitlab

```bash
$ dnf install gitlab-ce -y
```

### 离线安装gitlab

下载

```bash
$ wget -P /tmp/ --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/8/gitlab-ce-14.5.2-ce.0.el8.x86_64.rpm/download.rpm
```

 安装

```bash
$ rpm -ivh /tmp/gitlab-ce-14.5.2-ce.0.el8.x86_64.rpm
```

### 配置 GitLab URL

编辑gitlab配置文件

```bash
$ cp /etc/gitlab/gitlab.rb /etc/gitlab/gitlab.rb.bak
$ vim /etc/gitlab/gitlab.rb
```

And change HTTP to HTTPS on the external_url line.

```bash
$ cat /etc/gitlab/gitlab.rb | grep "external_url 'http://gitlab.example.com'"
external_url 'http://gitlab.example.com'

$ sed -i "s/external_url 'http:\/\/gitlab.example.com'/external_url 'https:\/\/gitlab.juneyaokc.com'/g" /etc/gitlab/gitlab.rb 

$ cat /etc/gitlab/gitlab.rb | grep "external_url 'https://gitlab.juneyaokc.com'"
```

Paste the ssl certificate into the gitlab server

```bash
$ ll /etc/ssl/juneyaokc
total 8
-rw-r--r-- 1 root root 1700 Dec  2 09:12 star_juneyaokc_com.key
-rw-r--r-- 1 root root 4012 Dec  2 09:12 star_juneyaokc_com.pem
```

Then paste the following configuration under the 'external_url' line configuration.

```bash
$ vim /etc/gitlab/gitlab.rb

nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/ssl/star_juneyaokc_com.pem"
nginx['ssl_certificate_key'] = "/etc/ssl/star_juneyaokc_com.key"
```

Finally, apply the GitLab configuration using the following command.

```bash
$ gitlab-ctl reconfigure
```

checkout the password

```bash
$ cat /etc/gitlab/initial_root_password
```

## 部署gitlab-runner

### 在线安装gitlab-runner

安装gitlab的repo

```bash
$ curl -f -sSL https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash 
```

现在可以使用dnf或者yum安装gitlab

```bash
$ dnf install gitlab-runner -y
```

### 离线安装gitlab

下载

```bash
$ wget -P /tmp/ --content-disposition https://packages.gitlab.com/runner/gitlab-runner/packages/el/8/gitlab-runner-14.6.0-1.x86_64.rpm/download.rpm
```

 安装

```bash
$ rpm -ivh /tmp/gitlab-runner-14.6.0-1.x86_64.rpm
```

### 指定gitlab-runner

```bash
$ useradd staff

$ gitlab-runner uninstall 
Runtime platform                                    arch=amd64 os=linux pid=25137 revision=ac8e767a version=12.6.0

$ gitlab-runner install --working-directory /home/staff --user staff
Runtime platform                                    arch=amd64 os=linux pid=25169 revision=ac8e767a version=12.6.0

[root@k8s-node02 staff]# vim /etc/systemd/system/gitlab-runner.service
[root@k8s-node02 staff]# cat /etc/systemd/system/gitlab-runner.service
[Unit]
Description=GitLab Runner
After=syslog.target network.target
ConditionFileIsExecutable=/usr/lib/gitlab-runner/gitlab-runner

[Service]
StartLimitInterval=5
StartLimitBurst=10
ExecStart=/usr/lib/gitlab-runner/gitlab-runner "run" "--working-directory" "/home/staff" "--config" "/etc/gitlab-runner/config.toml" "--service" "gitlab-runner" "--syslog" "--user" "staff"

Restart=always
RestartSec=120

[Install]
WantedBy=multi-user.target
```

### 启动gitlab服务

```bash
$ systemctl  daemon-reload                        #重新加载配置
$ /bin/systemctl start gitlab-runner              #启动服务
$ /bin/systemctl enable gitlab-runner             #设置开机启动
$ /bin/systemctl restart gitlab-runner            #重启服务
```

验证服务是否启动

```bash
$ ps -ef |grep gitlab
```

设置权限

```bash
$ chown -R staff.staff /home/gitlab-runner
```

### gitlab runner注册服务

```bash
$ gitlab-runner register
```

注册过程

```bash
$ gitlab-runner register
Runtime platform                                    arch=amd64 os=linux pid=6245 revision=ac8e767a version=12.6.0
Running in system-mode.                            

#输入公司的gitlab公网地址                                                   
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://192.168.200.133/

#输入gitlab的token，找到第三步token，然后选择第四步，复制token
Please enter the gitlab-ci token for this runner:
8sjydnrsPuSRhKiQTQky

#输入描述这个runner名称
Please enter the gitlab-ci description for this runner:
[k8s-node02]: my-runner

#输入runner的标签
Please enter the gitlab-ci tags for this runner (comma separated):
my-tag,another-tag
Registering runner... succeeded                     runner=8sjydnrs

#输入runner执行器的环境
Please enter the executor: custom, docker-ssh, parallels, kubernetes, docker-ssh+machine, docker, shell, ssh, virtualbox, docker+machine:
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```

操作命令

| **命令**                 | **描述**                                                     |
| ------------------------ | ------------------------------------------------------------ |
| gitlab-runner run        | 运行一个runner服务                                           |
| gitlab-runner register   | 注册一个新的runner                                           |
| gitlab-runner start      | 启动服务                                                     |
| gitlab-runner stop       | 关闭服务                                                     |
| gitlab-runner restart    | 重启服务                                                     |
| gitlab-runner status     | 查看各个runner的状态                                         |
| gitlab-runner unregister | 注销掉某个runner                                             |
| gitlab-runner list       | 显示所有运行着的runner                                       |
| gitlab-runner verify     | 检查已注册的运行程序是否可以连接到GitLab，但它不验证GitLab Runner服务是否正在使用运行程序。 |
