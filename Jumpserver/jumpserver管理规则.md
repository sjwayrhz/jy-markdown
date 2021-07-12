# jumpserver管理规则

[TOC]

## 安全设置

### 密码安全

新建的用户，首次登陆时，需要设置密码，下表是设置密码必须满足的条件。

| 选择             | 数值   |
| ---------------- | ------ |
| SSH最大空闲时间  | 30分钟 |
| 密码过期时间     | 30天   |
| 密码最小长度     | 12     |
| 必须包含大写字母 | ture   |
| 必须包含小写字母 | ture   |
| 必须包含数字字符 | ture   |
| 必须包含特殊字符 | ture   |

### 开启MFA

简述： MFA认证是在安卓或者苹果手机里安装一个身份验证器，该验证器会在30秒左右更改一次数值，数值为6位数。每次登陆jumpserver管理控制台时，需要输入的正确密码搭配MFA码才可以。

| 选项             | 数值                                                      |
| ---------------- | --------------------------------------------------------- |
| MFA 二次认证     | 用户登录必须使用MFA二次认证                               |
| 批量命令         | 允许用户批量执行命令                                      |
| 终端注册         | 允许使用bootstrap token注册终端, 当终端注册成功后可以禁止 |
| 限制登录失败次数 | 7次                                                       |
| 禁止登录时间间隔 | 30分                                                      |

 ### 密钥定义

在jumpserver中牵涉三种密钥。

| 密钥名 | 功能                                        | 是否需要密码验证 |
| ------ | ------------------------------------------- | ---------------- |
| key1   | 使用ssh通过22端口登陆jumpserver的terminal   | 是               |
| key2   | 使用ssh通过2222端口登陆jumpserver的terminal | 是               |
| key3   | jumpserver使用该密钥ssh到其它服务器         | 否               |

由于key1的使用频率并不高，只有管理员需要，用户不需要，所以仅设置om账户拥有key1

新建的用户都需要有key2和key3，区别在于Key2的公钥由用户提交给管理员，Key3的公钥由管理员在Jumpserver服务器生成                                              

## 开通账户流程

以曹波(caobo)为例，开通jumpserver账号

### 创建用户

使用admin用户登陆jumpserver管理控制台，创建新用户（普通用户），并添加到OM组，MFA已经设置为自动开启。

勾选首次登陆需要重置密码，这样，首次登陆的时候，便要设置密码为规定的密码。

jumpserver的管理控制台登陆地址为：`http://10.19.5.16`

登陆Jumpserver服务器，创建用户caobo

```
~]# useradd -g om caobo
```

### 创建key2

在自己的windows上安装的ssh软件上生成密钥，要求如下：

| key2要求           | 数值                        |
| ------------------ | --------------------------- |
| 字节长度           | 2048                        |
| 是否为key2配置密码 | 是 ，并且需要满足密码复杂度 |
| 格式               | RSA ，推荐RSA2              |
| 公钥和私钥名称     | id_rsa.pub 和 id_rsa        |

使用admin用户登陆jumpserver管理控制台，将曹波生成的”id_rsa.pub “公钥内容填入ssh公钥认证栏。

然后使用”id_rsa“即可通过2222端口登陆jumpserver服务器了。

### 创建key3

在jumpserver的caobo用户目录下创建key3，需要登陆jumpserver后，做如下操作

```
~]# su caobo
~]# ssh-keygen
按回车键三次

~]$ ll .ssh/
total 8
-rw------- 1 caobo om 1679 Dec 11 11:29 id_rsa
-rw-r--r-- 1 caobo om  402 Dec 11 11:29 id_rsa.pub
```

### 受管服务器批量创建caobo用户

解释一下jumpserver的ansible配置文件

```
~]# ll /etc/ansible/
total 56
-rw-r--r-- 1 root root 19984 Dec  9 15:08 ansible.cfg
-rw-r--r-- 1 root root  6721 Dec 10 19:17 hosts.bak			#所有环境root密码的备份
-rw-r--r-- 1 root root  4866 Dec 10 11:57 hosts-prd			#生成环境root密码备份
-rw-r--r-- 1 root root  1838 Dec 10 11:55 hosts-pre			#预生成环境root密码备份
-rw-r--r-- 1 om   om    4866 Dec 10 19:21 om-hosts-prd		#生成环境om用户密码备份
-rw-r--r-- 1 om   om     217 Dec 10 19:31 push-ssh.yaml		#受管机推送密钥的yaml文件
drwxr-xr-x 2 root root  4096 Nov 14 12:25 roles
```

暂时只有om组推送的密码备份，所以应该创建一个caobo用户的密码文件

```
~]# touch /etc/ansible/caobo-hosts-prd	
```

使用root用户批量在prd环境创建caobo用户

```
~]# ansible all -i hosts-prd -m shell -a 'useradd -g om caobo' 
```

使用root用户批量在prd环境新建` /home/caobo/.ssh`文件夹

```
~]# ansible all -i hosts-prd -m shell -a 'mkdir /home/caobo/.ssh'
```
本地拷贝id_rsa.pub，并重命名为authorized_key

```
~]# cat /home/caobo/.ssh/id_rsa.pub >> /opt/authorized_keys
```

使用root用户批量推送key3公钥到受管服务器

```
~]# ansible all -i hosts-prd -m copy -a 'src=/opt/authorized_keys backup=yes dest=/home/caobo/.ssh/authorized_keys'
```
使用root用户批量在prd环境更改` /home/caobo/.ssh`文件夹的属主

```
~]# ansible all -i hosts-prd -m shell -a 'chown -R caobo:om /home/caobo/.ssh'
```

到此，jumpserver服务器上的caobo用户已经可以免密登陆生产环境服务器的caobo用户

注意：/opt/authorized_keys 中，每一行都会代表一个用户，需要妥善保存，每次copy到目标机器的时候，都会覆盖原先的authorized_keys文件

### 受管服务器接收caobo用户转变为root权限

目前caobo用户可以登陆受管服务器，但是不可以使用sudo -i 切换到root权限的。

现在需要使caobo在登陆目标服务器后可以通过sudo切换为root

使用root用户批量在prd环境

```
~]# ansible all -i hosts-prd -m shell -a 'echo "caobo    ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers.d/caobo'
~]# ansible all -i hosts-prd -m shell -a 'systemctl restart sshd'
```

### 配置用户caobo可以通过2222端口登陆jumpserver

使用admin用户登陆jumpserver管理控制台，做如下操作：

创建管理用户和系统用户，名称为caobo，导入key2的私钥和私钥所需的密码

在资产授权中，授权服务器权限给caobo账户，权限名称为”生产环境-运维组授权-caobo“