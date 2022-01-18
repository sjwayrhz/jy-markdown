# haproxy+keepalived

## 准备主机

| IP 地址      | 主机名            | 角色                 |
| ------------ | ----------------- | -------------------- |
| 10.220.62.32 | loadbalance-left  | Keepalived & HAproxy |
| 10.220.62.33 | loadbalance-right | Keepalived & HAproxy |
| 10.220.62.21 | devops-master-01  | master, etcd         |
| 10.220.62.22 | devops-master-02  | master, etcd         |
| 10.220.62.23 | devops-master-03  | master, etcd         |
| 10.220.62.24 | devops-node-01    | worker               |
| 10.220.62.25 | devops-node-02    | worker               |
| 10.220.62.26 | devops-node-03    | worker               |
| 10.220.62.31 |                   | 虚拟 IP 地址         |

![](https://kubesphere.io/images/docs/installing-on-linux/high-availability-configurations/set-up-ha-cluster-using-keepalived-haproxy/architecture-ha-k8s-cluster.png)

## 配置负载均衡

[Keepalived](https://www.keepalived.org/) 提供 VRRP 实现，并允许您配置 Linux 机器使负载均衡，预防单点故障。[HAProxy](https://www.haproxy.org/) 提供可靠、高性能的负载均衡，能与 Keepalived 完美配合。

由于 `lb1` 和 `lb2` 上安装了 Keepalived 和 HAproxy，如果其中一个节点故障，虚拟 IP 地址（即浮动 IP  地址）将自动与另一个节点关联，使集群仍然可以正常运行，从而实现高可用。若有需要，也可以此为目的，添加更多安装 Keepalived 和  HAproxy 的节点。

先运行以下命令安装 Keepalived 和 HAproxy。

```bash
$ dnf install keepalived haproxy -y
```

### HAproxy

1. 在两台用于负载均衡的机器上运行以下命令以配置 Proxy（两台机器的 Proxy 配置相同）：

   ```bash
   $ mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
   
   $ vim /etc/haproxy/haproxy.cfg
   ```

2. 以下是示例配置，供您参考（请注意 `server` 字段。请记住 `6443` 是 `apiserver` 端口）：

   ```
   global
       log /dev/log  local0 warning
       chroot      /var/lib/haproxy
       pidfile     /var/run/haproxy.pid
       maxconn     4000
       user        haproxy
       group       haproxy
       daemon
      
      stats socket /var/lib/haproxy/stats
      
   defaults
     log global
     option  httplog
     option  dontlognull
           timeout connect 5000
           timeout client 50000
           timeout server 50000
      
   frontend kube-apiserver
     bind *:6443
     mode tcp
     option tcplog
     default_backend kube-apiserver
      
   backend kube-apiserver
       mode tcp
       option tcplog
       option tcp-check
       balance roundrobin
       default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
       server devops-master-01 10.220.62.21:6443 check # Replace the IP address with your own.
       server devops-master-02 10.220.62.22:6443 check # Replace the IP address with your own.
       server devops-master-03 10.220.62.23:6443 check # Replace the IP address with your own.
    
   frontend in-ingress-nginx-30080
       bind *:30080
       default_backend ingress-nginx-30080
   backend ingress-nginx-30080
       server devops-master-01 10.220.62.21:30080 check # Replace the IP address with your own.
       server devops-master-02 10.220.62.22:30080 check # Replace the IP address with your own.
       server devops-master-03 10.220.62.23:30080 check # Replace the IP address with your own.
   ```

3. 保存文件并运行以下命令以开启 HAproxy。

   ```bash
   $ systemctl enable --now haproxy
   ```

4. 检查 HAproxy 当前运行状态：

   ```bash
   $ systemctl status haproxy
   ```

5. 确保您在另一台机器 (`lb2`) 上也配置了 HAproxy。

### Keepalived

两台机器上必须都安装 Keepalived，但在配置上略有不同。

1. 运行以下命令以配置 Keepalived。

   ```bash
   $ mv /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf.bak
   
   $ vim /etc/keepalived/keepalived.conf
   ```

2. 以下是示例配置 (loadbalance-left)，供您参考：

   ```
   global_defs {
     notification_email {
     }
     router_id LVS_DEVEL
     vrrp_skip_check_adv_addr
     vrrp_garp_interval 0
     vrrp_gna_interval 0
   }
      
   vrrp_script chk_haproxy {
     script "killall -0 haproxy"
     interval 2
     weight 2
   }
      
   vrrp_instance haproxy-vip {
     state BACKUP
     priority 100
     interface ens192                       # Network card
     virtual_router_id 60
     advert_int 1
     authentication {
       auth_type PASS
       auth_pass 1111
     }
     unicast_src_ip 10.220.62.32      # The IP address of this machine
     unicast_peer {
       10.220.62.33                   # The IP address of peer machines
     }
      
     virtual_ipaddress {
       10.220.62.31/24                  # The VIP address
     }
      
     track_script {
       chk_haproxy
     }
   }
   ```

   以下是示例配置 (loadbalance-right)，供您参考：

   ```
   global_defs {
     notification_email {
     }
     router_id LVS_DEVEL
     vrrp_skip_check_adv_addr
     vrrp_garp_interval 0
     vrrp_gna_interval 0
   }
      
   vrrp_script chk_haproxy {
     script "killall -0 haproxy"
     interval 2
     weight 2
   }
      
   vrrp_instance haproxy-vip {
     state BACKUP
     priority 100
     interface ens192                       # Network card
     virtual_router_id 60
     advert_int 1
     authentication {
       auth_type PASS
       auth_pass 1111
     }
     unicast_src_ip 10.220.62.33      # The IP address of this machine
     unicast_peer {
       10.220.62.32                   # The IP address of peer machines
     }
      
     virtual_ipaddress {
       10.220.62.31/24                  # The VIP address
     }
      
     track_script {
       chk_haproxy
     }
   }
   ```

   备注

   - 对于 `interface` 字段，您必须提供自己的网卡信息。您可以在机器上运行 `ifconfig` 以获取该值。
   - 为 `unicast_src_ip` 提供的 IP 地址是您当前机器的 IP 地址。对于也安装了 HAproxy 和 Keepalived 进行负载均衡的其他机器，必须在字段 `unicast_peer` 中输入其 IP 地址。

3. 保存文件并运行以下命令以开启  Keepalived。

   ```bash
   $ systemctl enable --now keepalived
   ```

4. 检查Keepalived 当前运行状态：

   ```bash
   $ systemctl status keepalived
   ```

5. 确保您在另一台机器 (`loadbalance-right`) 上也配置了 Keepalived。

## 验证高可用

在开始创建 Kubernetes 集群之前，请确保已经测试了高可用。

1. 在机器 `loadbalance-left` 上，运行以下命令：

   ```
   [root@loadbalance-left ~]#  ip a s
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
   2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
       link/ether 00:50:56:8f:8c:5d brd ff:ff:ff:ff:ff:ff
       inet 10.220.62.32/24 brd 10.220.62.255 scope global noprefixroute ens192
          valid_lft forever preferred_lft forever
       inet 10.220.62.31/24 scope global secondary ens192
          valid_lft forever preferred_lft forever
       inet6 fe80::250:56ff:fe8f:8c5d/64 scope link noprefixroute
          valid_lft forever preferred_lft forever
   ```

2. 如上图所示，虚拟 IP 地址已经成功添加。模拟此节点上的故障：

   ```bash
   $ systemctl stop keepalived
   ```

   这里只可以用启停keepalived来做实验，而不是haproxy来做实验。

3. 再次检查浮动 IP 地址，您可以看到该地址在 `loadbalance-left` 上消失了。

   ```
   [root@loadbalance-left ~]# ip a s
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
   2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
       link/ether 00:50:56:8f:8c:5d brd ff:ff:ff:ff:ff:ff
       inet 10.220.62.32/24 brd 10.220.62.255 scope global noprefixroute ens192
          valid_lft forever preferred_lft forever
       inet6 fe80::250:56ff:fe8f:8c5d/64 scope link noprefixroute
          valid_lft forever preferred_lft forever
   ```

4. 理论上讲，若配置成功，该虚拟 IP 会漂移到另一台机器 (`loadbalance-right`) 上。在 `loadbalance-right` 上运行以下命令，这是预期的输出：

   ```
   [root@loadbalance-right ~]# ip a s
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
   2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
       link/ether 00:50:56:8f:a1:07 brd ff:ff:ff:ff:ff:ff
       inet 10.220.62.33/24 brd 10.220.62.255 scope global noprefixroute ens192
          valid_lft forever preferred_lft forever
       inet 10.220.62.31/32 scope global ens192
          valid_lft forever preferred_lft forever
       inet6 fe80::250:56ff:fe8f:a107/64 scope link noprefixroute
          valid_lft forever preferred_lft forever
   ```

5. 如上所示，高可用已经配置成功。