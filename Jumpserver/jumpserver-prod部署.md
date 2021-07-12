# 华为云正式环境jumpserver-v2.6.1堡垒机部署



[TOC]

## 部署基础工具准备

- 一台4c8g的虚拟机实例，操作系统为centos 7 ，用于部署jumpserver服务器
- mysql数据库，版本大于等于5.7
- redis缓存，版本大于等于5

Mysql相关信息

```
地址：172.16.33.13
端口：3306
数据库：jumpserver
用户：jumpserver/m0cSuRVGPJm7g35B
密码：root/ZXHti0cAWW94AMR4

备用用户： jump/uzCK5I9sT79D7Rzj
```

Redis相关信息

```
地址：172.16.33.14
端口：6379
密码：JkItg8OcyXgFYbrc
```

mysql 和 redis 都需要设置白名单

## 登陆jumpserver

使用key1可以登陆jumpserver服务器

```
ssh -i /Users/sjwayrhz/Keystore/test-key1.private root@jump.cls.cn
```

通过2222端口登陆

```
ssh -p2222 caobo@jump.cls.cn
```

关闭jumpserver服务器的使用密码可以登陆的权限，并在阿里云控制台测试通过root的密码不可以直接登陆服务器，但是使用密钥还是可行的。

网页登陆

```
管理员账户密码
admin / qC5uziixlykv5rhq8

自己的账户密码
caobo / ·常用密码·
```



## 部署文件准备

登陆jumpserver服务器，下载需要安装jumpserver所需的资源。

```
~]# cd /opt
~]# yum -y install wget
~]# wget https://github.com/jumpserver/installer/releases/download/v2.6.1/jumpserver-installer-v2.6.1.tar.gz
~]# tar -xf jumpserver-installer-v2.6.1.tar.gz
~]# cd jumpserver-installer-v2.6.1
~]# export DOCKER_IMAGE_PREFIX=docker.mirrors.ustc.edu.cn
~]# cat config-example.txt 
```

## 安装 、 运行 、密钥规则

```
./jmsctl.sh install
```

```
./jmsctl.sh start
./jmsctl.sh stop
./jmsctl.sh -h
```

jumpserver上的两个key

```
SECRET_KEY=CAOWInR0ZeE1aa0CisWqXGXkeRU6rd878c0gXN4AeaMhMH67gS
BOOTSTRAP_TOKEN=36ugwsbFNId4udJ1
```



密钥规则

```
key1: 使用ssh通过22端口登陆jumpserver的terminal
key2: 使用ssh通过2222端口登陆jumpserver的terminal
key3: jumpserver服务器登陆到网关——跳板机的密钥
key4: 跳板机登陆到与之相同vpc下的受体机的密钥
```

跨网段访问时，需要设置网域，在网域中设置网关，其中网关的22端口需要通过负载均衡映射到公网。 此时需要key4密钥。

key1,key2,key3,key4为四个私钥；key1.pub，key2.pub，key3.pub，key4.pub，为四组公钥。



## 密钥配置方案

所有的密钥均在jumpserver虚拟机中生成，存储于 ～/.ssh/ 目录中，生成方法

```
ssh-keygen -f key1
ssh-keygen -f key2
ssh-keygen -f key3
ssh-keygen -f key4
```

服务器配置如下：

| 虚拟机        | 内网ip地址   | 用途   | 所属vpc      |
| ------------- | ------------ | ------ | :----------- |
| jumpserver    | 172.16.33.12 | 堡垒机 | vpc-ops      |
| ops-test-0001 | 172.16.34.11 | 受体机 | vpc-ops-test |
| ops-test-0002 | 172.16.34.12 | 跳板机 | vpc-ops-test |

负载均衡配置如下：

Jumpserver映射到公网的地址：122.9.81.78

跳板机映射到公网的地址：116.63.114.187

### jumpserver控制台web页面的密钥配置

登陆地址

```
122.9.81.78
```

管理用户：需要登陆到跳板机，所以私钥是key3。

系统用户：需要通过堡垒机登陆到受体机，所以私钥也是key3。

网域列表：添加“华为云-运维-测试”这个名字的网域

网关：

```
名称：华为云-运维-测试
ip: 116.63.114.187
端口：22
用户名：root
密钥：key3
```



资产管理：

```
主机名：ops-test-0001
IP域名：172.16.34.11
系统平台：Linux
网域：华为云-运维-测试
管理用户：华为云-运维-测试（root）
节点：/default/运维组
```



### 个人pc密钥配置

个人pc登陆到jumpserver有key1和key2。

```
sjwayrhz@caobodeMac-mini ~ % ls .ssh/key*
.ssh/key1	.ssh/key2
```

jumpserver管理员通过key1登陆到linux虚拟机,管理虚拟机。

```
ssh -i .ssh/key1 root@122.9.81.78
```

普通用户通过key2登陆到jumpserver命令行终端。

```
ssh -p2222 -i .ssh/key2 caobo@122.9.81.78
```

### Jumpserver的密钥设置

此次架构，主要是jumpserver跳转到跳板机，所以jumpserver的默认私钥需要的配置key3。

管理员个人pc需要登陆jumpserver服务器，普通用户需要通过2222登陆jumpserver服务器。所以jumpserver的authorized_keys需要配置key1.pub和key2.pub。

### 跳板机的密钥配置

跳板机需要登陆到受体机,实验中的跳板机映射到负载均衡的公网ip是 116.63.114.187

跳板机的authorized_keys需要key3.pub, 同时，id_rsa需要key4私钥

### 受体机的密钥配置

受体机只需要authorized_keys里面配置key3.pub和key4.pub即可



## 添加新节点举例

在华为云中新建一个vpc，在这个vpc中添加jenkins,举例。

jenkins 服务器作为网关，并跳转回ecs 。

1. 登陆jenkins服务器，在authorized_keys里面添加key3.pub和key4.pub，同时添加私钥key4
2. 登陆jump.cls.cn，添加网域——华为云-运维-生产
3. 在上一步的网域中添加网关——华为云-运维-生产-jenkins，ip是jenkins外层负载均衡公网ip(116.63.55.71),私钥选择key3，测试链接。
4. 在资产列表中，default/运维组中添加资产。主机名，ops-prod-jenkins；ip,（172.16.35.11）也就是jenkins内网ip，网域选择华为云-运维-生产，管理用户选择“华为云-运维”
5. 最后，到授权管理中，为jenkins服务器授权即可。



## 添加kubernetes



```
kubectl describe secret -n=kube-system  kubernetes-dashboard-token-d24bl
```



### 重启jumpserver服务器

重启之后，jumpserver所有容器将不能再使用，需要删除所有容器。

```
docker stop `docker ps -aq`
docker container -prune -f
cd /opt/jumpserver-installer-v2.6.1/
./jmsctl.sh start
```

