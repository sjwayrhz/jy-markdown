# Redis Sentinel 高可用服务架构搭建

[TOC]

## 环境规划

|     IP      |    Hostname    |           Services            |             配置文件              |
| :---------: | :------------: | :---------------------------: | :-------------------------------: |
| 10.230.7.31 |  redis-master  | redis server + redis-sentinel | redis-master.conf + sentinel.conf |
| 10.230.7.32 |  redis-slave   | redis server + redis-sentinel | redis-slave.conf + sentinel.conf  |
| 10.230.7.33 | redis-sentinel |        redis-sentinel         |           sentinel.conf           |

## 源码编译安装

先从 [redis.io/releases](http://download.redis.io/releases/)，下载最新版本的 Redis（需要进行编译）。三个虚拟机都需要安装redis。

```bash
~]# wget http://download.redis.io/releases/redis-4.0.8.tar.gz
~]# tar xzf redis-4.0.8.tar.gz -C /usr/local/
~]# rm -f redis-4.0.8.tar.gz
~]# cd /usr/local/redis-4.0.8/
redis-4.0.8]# dnf install -y gcc make
redis-4.0.8]# make && make install
```

正常安装完成的输出如下

```
make[1]: Leaving directory '/usr/local/redis-4.0.8/src'
cd src && make install
make[1]: Entering directory '/usr/local/redis-4.0.8/src'
    CC Makefile.dep

Hint: It's a good idea to run 'make test' ;)

    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
make[1]: Leaving directory '/usr/local/redis-4.0.8/src'
```

## 生成配置文件

### redis-sentinel

分别登陆10.230.7.31，10.230.7.32，10.230.7.33，新建redis-sentinel.conf配置文件

```bash
~]# tee /usr/local/redis-4.0.8/redis-sentinel.conf << 'EOF'
port 26379
bind 10.230.7.31
daemonize yes
logfile "/var/log/redis/redis-4.0.8/redis-sentinel.log"

sentinel monitor manager1 10.230.7.31 6379 2
sentinel auth-pass manager1 123456
sentinel down-after-milliseconds manager1 60000
sentinel failover-timeout manager1 180000
sentinel parallel-syncs manager1 1
EOF
```

分别登陆10.230.7.31，10.230.7.32，10.230.7.33，新建redis-sentinel.service

```bash
~]# tee /etc/systemd/system/redis-sentinel.service << 'EOF'
[Unit]
Description=Redis sentinel Server
After=network.target
[Service]
Type=simple
LimitNOFILE=100000
ExecStart=/usr/local/bin/redis-sentinel /usr/local/redis-4.0.8/sentinel.conf
ExecStop=/usr/local/bin/redis-cli -p 26379 shutdown
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
```

为日志文件生成目录

```bash
~]# mkdir -p /var/log/redis/redis-4.0.8/
```

添加redis-sentinel到系统自动启动

```bash
~]# systemctl enable redis-sentinel
```

redis-sentinel这样配置就完成了，但是对于redis-master和redis-slave来说，还需要新增其他配置文件。

### redis-master

登陆10.230.7.31，新建redis-master.conf配置文件

```bash
~]# tee /usr/local/redis-4.0.8/redis-master.conf << 'EOF'
daemonize yes
protected-mode no
port 6379
bind 10.230.7.31
loglevel notice
logfile /var/log/redis/redis-4.0.8/redis-server.log
requirepass 123456
EOF
```

此外，新增redis服务

```bash
~]# tee /etc/systemd/system/redis.service << 'EOF'
[unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /usr/local/redis-4.0.8/redis-master.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

添加redis-master到系统自动启动

```bash
~]# systemctl enable redis
```



### redis-slave

登陆10.230.7.32，新建redis-master.conf配置文件

```bash
~]# tee /usr/local/redis-4.0.8/redis-slave.conf << 'EOF'
daemonize yes
protected-mode no
port 6379
bind 10.230.7.32
slaveof 10.230.7.31 6379
loglevel notice
logfile /var/log/redis/redis-4.0.8/redis-server.log
masterauth 123456
EOF
```

此外，新增redis服务

```bash
~]# tee /etc/systemd/system/redis.service << 'EOF'
[unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /usr/local/redis-4.0.8/redis-slave.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

添加redis-slave到系统自动启动

```
~]# systemctl enable redis
```



