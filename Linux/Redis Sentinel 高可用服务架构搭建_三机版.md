# Redis Sentinel 高可用服务架构搭建

[TOC]

## 环境规划

|       IP       |    Hostname    | Services |        配置文件        |
| :------------: | :------------: | :------: | :--------------------: |
| 192.168.177.87 |  redis-master  |  master  | /etc/redis-master.conf |
| 192.168.177.88 |  redis-slave   |  slave   | /etc/redis-slave.conf  |
| 192.168.177.89 | redis-sentinel | sentinel |   /etc/sentinel.conf   |

源码编译安装

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

## 具体配置

### redis-master

新建日志目录，配置文件目录并拷贝初始配置文件

```bash
~]# mkdir /etc/redis

~]# mkdir /var/log/redis

~]# cp /usr/local/redis-4.0.8/redis.conf /etc/redis/redis.conf
```

编辑redis.conf

修改如下：

|       redis.conf初始配置        |         修改redis.conf之后         |
| :-----------------------------: | :--------------------------------: |
|         bind 127.0.0.1          |        bind 192.168.177.87         |
|          daemonize no           |           daemonize yes            |
|           logfile ""            | logfile "/var/log/redis/redis.log" |
| stop-writes-on-bgsave-error yes |   stop-writes-on-bgsave-error no   |
| # masterauth <master-password>  |        masterauth nfA9pZwC         |
|     # requirepass foobared      |        requirepass nfA9pZwC        |
|       # maxclients 10000        |          maxclients 50000          |

创建redis的systemd服务

```bash
~]# tee /etc/systemd/system/redis.service << 'EOF'
[unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

启动redis服务

```bash
~]# systemctl enable --now redis
```

### redis-slave

新建日志目录，配置文件目录并拷贝初始配置文件,等同于redis-master

编辑redis.conf

修改如下：

|        redis.conf初始配置         |         修改redis.conf之后         |
| :-------------------------------: | :--------------------------------: |
|          bind 127.0.0.1           |        bind 192.168.177.88         |
|           daemonize no            |           daemonize yes            |
|            logfile ""             | logfile "/var/log/redis/redis.log" |
|  stop-writes-on-bgsave-error yes  |   stop-writes-on-bgsave-error no   |
|  # masterauth <master-password>   |        masterauth nfA9pZwC         |
|      # requirepass foobared       |        requirepass nfA9pZwC        |
|        # maxclients 10000         |          maxclients 50000          |
| # slaveof <masterip> <masterport> |    slaveof 192.168.177.87 6379     |

创建redis的systemd服务 和 启动redis服务 ，等同于redis-master。

### redis-sentinel

新建日志目录，配置文件目录并拷贝初始配置文件

```
~]# mkdir /etc/redis

~]# mkdir /var/log/redis

~]# cp /usr/local/redis-4.0.8/sentinel.conf /etc/redis/sentinel.conf
```

修改如下：

|             sentinel.conf初始配置             |               修改sentinel.conf之后               |
| :-------------------------------------------: | :-----------------------------------------------: |
|         # bind 127.0.0.1 192.168.1.1          |                bind 192.168.177.89                |
|                       -                       |                   daemonize yes                   |
|  sentinel monitor mymaster 127.0.0.1 6379 2   |  sentinel monitor mymaster 192.168.177.87 6379 1  |
| # sentinel auth-pass <master-name> <password> |       sentinel auth-pass mymaster nfA9pZwC        |
|                       -                       | sentinel known-slave mymaster 192.168.177.88 6379 |

启动服务为

```bash
~]# tee /etc/systemd/system/sentinel.service << 'EOF'
[unit]
Description=redis-sentinel
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-sentinel /etc/redis/sentinel.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

开机自启动

```
~]# systemctl enable --now sentinel
```

