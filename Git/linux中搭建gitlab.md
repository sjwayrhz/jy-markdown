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

## 安装步骤

### 安装gitlab 

安装gitlab的repo

```bash
$ curl -f -sSL https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash 
```

现在可以使用dnf或者yum安装gitlab

```bash
$ dnf install gitlab-ce
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

