# haproxy转发ingress端口



如果ingress虚拟ip为10.220.62.60

内网转发的haproxy端口为10.220.62.52，那么可以在10.220.62.52上安装haproxy

```bash
$ dnf install -y haproxy
```

配置内容如下

```bash
tee /etc/haproxy/haproxy.cfg <<- 'EOF'
global
        ulimit-n  51200
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        pidfile /var/run/haproxy.pid
        user haproxy
        group haproxy
        daemon

defaults
        log     global
        mode    tcp
        option  dontlognull
        timeout connect 600
        timeout client 5m
        timeout server 5m

frontend traffic-in-80
        bind *:80
        default_backend traffic-out-80
backend traffic-out-80
        server server1 10.220.62.60:30080 maxconn 20480

frontend traffic-in-443
        bind *:443
        default_backend traffic-out-443
backend traffic-out-443
        server server1 10.220.62.60:30443 maxconn 20480
EOF
```

启动

```bash
$ systemctl enable --now haproxy
```

