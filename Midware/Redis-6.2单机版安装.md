# Redis-6.2单机版安装

## 简易版

Enable Remi repo using this command:

```bash
$ sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm -y
```

Remi runs in aliyun resources

```bash
$ sed -i 's/^mirrorlist=http:\/\/cdn.remirepo.net/#mirrorlist=http:\/\/cdn.remirepo.net/g' /etc/yum.repos.d/remi*.repo
$ sed -i 's/rpms.remirepo.net/mirrors.aliyun.com\/remi/g' /etc/yum.repos.d/remi**.repo
$ sed -i 's/^#baseurl=http:\/\/mirrors.aliyun.com/baseurl=http:\/\/mirrors.aliyun.com/g' /etc/yum.repos.d/remi*.repo
```

Then list redis using this:

```bash
$ sudo dnf module list redis
```

Enable redis 6.2

```bash
$ sudo dnf module enable redis:remi-6.2 -y
```

Install redis

```bash
$ sudo dnf install redis 
```

设置redis配置文件

```
bind 127.0.0.1 -::1				->		bind 0.0.0.0
# requirepass foobared		->		requirepass 123456
appendonly no						->		appendonly yes
```

启动和查看reids状态

```bash
$ systemctl enable --now redis
$ systemctl status redis
```



## 官方版

```
https://citizix.com/how-to-install-configure-redis-6-on-rocky-linux-centos-8/
```

### 1. Update Rocky Linux/Centos 8 Server

Before proceeding, ensure that the server is updated using this command:

```
sudo dnf -y update
```

Let us also ensure vim is installed using this command since we will use it later:

```
sudo dnf install -y vim
```

### 2. Installing redis

Redis 6 is not available in the default Rocky Linux/Centos 8 Servers. We will use remi release to install Redis module that will enable us to install redis 6.

Enable Remi repo using this command:

```
sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm -y
```

Then list redis using this:

```
$ sudo dnf module list redis
Last metadata expiration check: 0:02:05 ago on Fri 29 Oct 2021 08:08:41 PM UTC.
Rocky Linux 8 - AppStream
Name            Stream             Profiles             Summary
redis           5 [d]              common [d]           Redis persistent key-value database
redis           6                  common [d]           Redis persistent key-value database

Remi's Modular repository for Enterprise Linux 8 - x86_64
Name            Stream             Profiles             Summary
redis           remi-5.0           common [d]           Redis persistent key-value database
redis           remi-6.0           common [d]           Redis persistent key-value database
redis           remi-6.2           common [d]           Redis persistent key-value database
```

Enable redis 6.2

```
sudo dnf module enable redis:remi-6.2 -y
```

Install redis

```
sudo dnf install redis
```

Use this command to confirm the redis package installed:

```
$ rpm -qi redis
Name        : redis
Version     : 6.2.6
Release     : 1.el8.remi
Architecture: x86_64
Install Date: Fri 29 Oct 2021 08:14:06 PM UTC
Group       : Applications/Databases
Size        : 4522111
License     : BSD
Signature   : RSA/SHA256, Mon 04 Oct 2021 12:34:26 PM UTC, Key ID 555097595f11735a
Source RPM  : redis-6.2.6-1.el8.remi.src.rpm
Build Date  : Mon 04 Oct 2021 12:28:08 PM UTC
Build Host  : builder.remirepo.net
Relocations : (not relocatable)
Packager    : Remi Collet
Vendor      : Remi's RPM repository <https://rpms.remirepo.net/>
URL         : http://redis.io
Bug URL     : https://forum.remirepo.net/
Summary     : A persistent key-value database
Description :
Redis is an advanced key-value store. It is often referred to as a data
structure server since keys can contain strings, hashes, lists, sets and
sorted sets.

You can run atomic operations on these types, like appending to a string;
incrementing the value in a hash; pushing to a list; computing set
intersection, union and difference; or getting the member with highest
ranking in a sorted set.

In order to achieve its outstanding performance, Redis works with an
in-memory dataset. Depending on your use case, you can persist it either
by dumping the dataset to disk every once in a while, or by appending
each command to a log.

Redis also supports trivial-to-setup master-slave replication, with very
fast non-blocking first synchronization, auto-reconnection on net split
and so forth.

Other features include Transactions, Pub/Sub, Lua scripting, Keys with a
limited time-to-live, and configuration settings to make Redis behave like
a cache.

You can use Redis from most programming languages also.
```

Now that the service has been installed, let’s start it with this command:

```
sudo systemctl start redis
```

Enable the service so it starts on boot:

```
$ sudo systemctl enable redis
Created symlink /etc/systemd/system/multi-user.target.wants/redis.service → /usr/lib/systemd/system/redis.service.
```

After the service starts, use this command to check the status of the service:

```
$ sudo systemctl status redis
● redis.service - Redis persistent key-value database
   Loaded: loaded (/usr/lib/systemd/system/redis.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/redis.service.d
           └─limit.conf
   Active: active (running) since Fri 2021-10-29 20:16:17 UTC; 25s ago
 Main PID: 62643 (redis-server)
   Status: "Ready to accept connections"
    Tasks: 5 (limit: 23168)
   Memory: 7.3M
   CGroup: /system.slice/redis.service
           └─62643 /usr/bin/redis-server 127.0.0.1:6379

Oct 29 20:16:17 ip-10-2-40-54.us-west-2.compute.internal systemd[1]: Starting Redis persistent key-v>
Oct 29 20:16:17 ip-10-2-40-54.us-west-2.compute.internal systemd[1]: Started Redis persistent key-va>
```

The `Active: active (running)` means that the service has been started successfully.

### 3. Configuring Redis

The redis configuration file is located in this path `/etc/redis/redis.conf`. In this section, we are going to update the redis configuration file to allow remote access, to set an authentication password, to add a pid  file and to Set Persistent Store for Recovery.

Edit redis config file using this:

```
sudo vim /etc/redis/redis.conf
```

To allow remote access to the redis instance, bind redis to 0.0.0.0 using this line:

```
bind 0.0.0.0
```

To set password in redis, use this:

```
requirepass j2GfJuLFR8
```

To add a pid file to redis:

```
pidfile /var/run/redis/redis-server.pid
```

Set Persistent Store for Recovery by changing the appendonlyvalue to yes

```
appendonly yes
appendfilename "appendonly.aof"
```

Restart redis service to apply changes:

```
sudo systemctl restart redis
```

### 4. Connecting and performing basic operations in Redis

If you have an active firewalld service, allow port 6379

```
sudo firewall-cmd --add-port=6379/tcp --permanent
sudo firewall-cmd --reload
```

Connecting to redis locally:

```
$ redis-cli
```

To authenticate:

```
127.0.0.1:6379> auth j2GfJuLFR8
OK
```

You should receive `OK` in the output. If you input a wrong password, Authentication should fail.

Check redis information.

```
127.0.0.1:6379> INFO
```

This will output a long list of data. You can limit the output by passing Section as an argument. E.g.

```
127.0.0.1:6379> INFO Server
# Server
redis_version:6.2.6
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:b0cb03c693a4c6cc
redis_mode:standalone
os:Linux 4.18.0-305.3.1.el8_4.x86_64 x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:c11-builtin
gcc_version:8.4.1
process_id:62643
process_supervised:systemd
run_id:668853b6d043e64d7af95ab586c9aca9d6b8f49a
tcp_port:6379
server_time_usec:1635538653706020
uptime_in_seconds:76
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:8148701
executable:/usr/bin/redis-server
config_file:/etc/redis/redis.conf
io_threads_active:0
```

### 5. Performing Redis Benchmarking

Run the benchmark with `15` parallel connections, for a total of `10k` requests, against local redis to test its performance.

```
$ redis-benchmark -h 127.0.0.1 -p 6379 -n 10000 -c 15 -a j2GfJuLFR8
====== PING_INLINE ======
  10000 requests completed in 0.23 seconds
  15 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 3600 1 300 100 60 10000
  host configuration "appendonly": no
  multi-thread: no

Latency by percentile distribution:
0.000% <= 0.055 milliseconds (cumulative count 2)
50.000% <= 0.127 milliseconds (cumulative count 5509)
75.000% <= 0.159 milliseconds (cumulative count 7514)

..........

99.940% <= 0.503 milliseconds (cumulative count 9994)
100.000% <= 0.607 milliseconds (cumulative count 10000)

Summary:
  throughput summary: 74074.07 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.159     0.072     0.151     0.239     0.279     0.567
```

For more options and examples, use:

```
$ redis-benchmark --help
```

### Conclusion

We have managed to install and configure Redis 6 in Rocky Linux/Centos 8.
