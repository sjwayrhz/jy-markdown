# Docker install

[TOC]


## docker 官方安装

### 添加仓库

添加阿里云的Docker仓库：

```
~]# yum -y install yum-utils device-mapper-persistent-data lvm2
~]# yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 安装Docker

如果是局部代理，可能无法下载gpg-key，可以取消验证

```
 ~]# sed -i 's/gpgcheck=1/gpgcheck=0/g' /etc/yum.repos.d/docker-ce.repo
```

执行以下命令，安装最新版Docker：

```
~]# yum install docker-ce -y
```

运行`docker --version`,可以看到安装了截止目前最新的18.09.4版本：
```
~]# docker --version
Docker version 18.09.4, build d14af54266
```

### 加速Docker

可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```
~]# mkdir -p /etc/docker
~]# tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"],
  # "graph": "/opt/docker",
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

~]# systemctl daemon-reload
~]# systemctl enable --now docker
```
### 启动Docker

启动Docker服务并激活开机启动：  
```
~]# systemctl enable --now docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.

~]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2019-04-01 10:43:12 CST; 5s ago
     Docs: https://docs.docker.com
 Main PID: 16708 (dockerd)
    Tasks: 10
   Memory: 28.8M
   CGroup: /system.slice/docker.service
           └─16708 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

### 配置docker上网代理

创建docker代理服务

```
~]# mkdir -p /etc/systemd/system/docker.service.d
```

配置docker代理文件

```
~]# vim /etc/systemd/system/docker.service.d/http-proxy.conf
```

```
配置文件方法
   
   [Service]
     Environment="HTTP_PROXY=http://username:password@ip:port"

配置文件范例

   [Service]
        Environment="HTTP_PROXY=http://192.168.61.55:3333"
        
        
如果是https代理，需要将HTTP_PROXY改为HTTPS_PROXY
```

### 修改Docker默认存储位置的方法

```
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"],
  "graph": "/opt/docker",
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

```

### 重启docker生效

```
~]# systemctl daemon-reload
~]# systemctl restart docker
```


### 安装 docker-compose
克隆项目

```
~]#  curl -L https://get.daocloud.io/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```
Apply executable permissions to the binary:

```
~]# sudo chmod +x /usr/local/bin/docker-compose

~]# sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Test the installation.
```
~]#  docker-compose --version
```

安装Docker-Compose：

```
~]# pip install docker-compose -i https://pypi.tuna.tsinghua.edu.cn/simple
```

如果有需要使用代理安装docker-compose

```
~]# pip install docker-compose -i https://pypi.tuna.tsinghua.edu.cn/simple --proxy http://192.168.61.55:3333
```

检查是是否成功：

```
~]# docker-compose -version
```
