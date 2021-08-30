# Redis Sentinel 高可用服务架构搭建

[TOC]

## 环境规划

|       IP       | port  | Services |            配置文件            |
| :------------: | :---: | :------: | :----------------------------: |
| 192.168.177.53 | 16379 |  master  |  /etc/redis-master-16379.conf  |
| 192.168.177.53 | 26379 |  slave   |  /etc/redis-slave-26379.conf   |
| 192.168.177.53 | 36379 | sentinel | /etc/redis-sentinel-36379.conf |

密码设置为 

`nfA9pZwC`

源码编译安装

先从 [redis.io/releases](http://download.redis.io/releases/)，下载最新版本的 Redis（需要进行编译）。三个虚拟机都需要安装redis。

```bash
~]# wget http://download.redis.io/releases/redis-5.0.13.tar.gz
~]# tar xzf redis-5.0.13.tar.gz -C /usr/local/
~]# rm -f redis-5.0.13.tar.gz
~]# cd /usr/local/redis-5.0.13/
redis-4.0.8]# dnf install -y gcc make
redis-4.0.8]# make && make install
```

正常安装完成的输出如下

```
make[1]: Leaving directory '/usr/local/redis-5.0.13/src'
cd src && make install
make[1]: Entering directory '/usr/local/redis-5.0.13/src'
    CC Makefile.dep

Hint: It's a good idea to run 'make test' ;)

    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
make[1]: Leaving directory '/usr/local/redis-5.0.13/src'
```

## 具体配置

### redis-master

新建日志目录，配置文件目录并拷贝初始配置文件

```bash
~]# mkdir /etc/redis
~]# mkdir /var/log/redis
~]# cp /usr/local/redis-5.0.13/redis.conf /etc/redis/redis-master-16379.conf
```

编辑redis.conf

修改如下：

|       redis.conf初始配置        |            修改redis.conf之后             |
| :-----------------------------: | :---------------------------------------: |
|         bind 127.0.0.1          |            bind 192.168.177.53            |
|            port 6379            |                port 16379                 |
|          daemonize no           |               daemonize yes               |
|           logfile ""            | logfile "/var/log/redis/redis-master.log" |
| stop-writes-on-bgsave-error yes |      stop-writes-on-bgsave-error no       |
| # masterauth <master-password>  |            masterauth nfA9pZwC            |
|     # requirepass foobared      |           requirepass nfA9pZwC            |
|       # maxclients 10000        |              maxclients 5000              |

创建redis的systemd服务

```bash
~]# tee /etc/systemd/system/redis-master.service << 'EOF'
[unit]
Description=redis-master
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /etc/redis/redis-master-16379.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

启动redis服务

```bash
~]# systemctl enable --now redis-master
```

### redis-slave

新建日志目录，配置文件目录并拷贝初始配置文件,等同于redis-master

```bash
~]# cp /usr/local/redis-5.0.13/redis.conf /etc/redis/redis-slave-26379.conf
```

编辑redis.conf

修改如下：

|        redis.conf初始配置         |            修改redis.conf之后            |
| :-------------------------------: | :--------------------------------------: |
|          bind 127.0.0.1           |           bind 192.168.177.53            |
|             port 6379             |                port 26379                |
|           daemonize no            |              daemonize yes               |
|            logfile ""             | logfile "/var/log/redis/redis-slave.log" |
|  stop-writes-on-bgsave-error yes  |      stop-writes-on-bgsave-error no      |
|  # masterauth <master-password>   |           masterauth nfA9pZwC            |
|      # requirepass foobared       |           requirepass nfA9pZwC           |
|        # maxclients 10000         |             maxclients 50000             |
| # slaveof <masterip> <masterport> |       slaveof 192.168.177.53 16379       |

创建redis的systemd服务 和 启动redis服务 

```bash
~]# tee /etc/systemd/system/redis-slave.service << 'EOF'
[unit]
Description=redis-slave
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /etc/redis/redis-slave-26379.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

启动redis服务

```bash
~]# systemctl enable --now redis-slave
```

### redis-sentinel

新建日志目录，配置文件目录并拷贝初始配置文件

```bash
~]# cp /usr/local/redis-5.0.13/sentinel.conf /etc/redis/redis-sentinel-36379.conf
```

修改如下：

|             sentinel.conf初始配置             |               修改sentinel.conf之后                |
| :-------------------------------------------: | :------------------------------------------------: |
|         # bind 127.0.0.1 192.168.1.1          |                bind 192.168.177.53                 |
|                       -                       |                   daemonize yes                    |
|  sentinel monitor mymaster 127.0.0.1 6379 2   |  sentinel monitor mymaster 192.168.177.53 16379 1  |
| # sentinel auth-pass <master-name> <password> |        sentinel auth-pass mymaster nfA9pZwC        |
|                       -                       | sentinel known-slave mymaster 192.168.177.53 26379 |

启动服务为

```bash
~]# tee /etc/systemd/system/redis-sentinel.service << 'EOF'
[unit]
Description=redis-sentinel
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-sentinel /etc/redis/redis-sentinel-36379.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

开机自启动

```bash
~]# systemctl enable --now redis-sentinel
```

## 单机无密码版

先从 [redis.io/releases](http://download.redis.io/releases/)，下载最新版本的 Redis（需要进行编译）。

```bash
~]# wget http://download.redis.io/releases/redis-5.0.13.tar.gz
~]# tar xzf redis-5.0.13.tar.gz -C /usr/local/
~]# rm -f redis-5.0.13.tar.gz
~]# cd /usr/local/redis-5.0.13/
redis-4.0.8]# dnf install -y gcc make
redis-4.0.8]# make && make install
```

新建日志目录，配置文件目录并拷贝初始配置文件

```bash
~]# mkdir /etc/redis
~]# mkdir /var/log/redis
~]# cp /usr/local/redis-5.0.13/redis.conf /etc/redis/redis.conf
```

编辑redis.conf

修改如下：

| redis.conf初始配置 |         修改redis.conf之后         |
| :----------------: | :--------------------------------: |
|   bind 127.0.0.1   |        bind 192.168.177.53         |
|    daemonize no    |           daemonize yes            |
|     logfile ""     | logfile "/var/log/redis/redis.log" |
| # maxclients 10000 |          maxclients 5000           |
