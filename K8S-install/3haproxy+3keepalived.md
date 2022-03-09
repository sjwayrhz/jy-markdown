# haproxy+keepalived

## 准备主机

| IP 地址      | 服务名    | 主机名        | 角色                 |
| ------------ | --------- | ------------- | -------------------- |
| 10.220.62.41 | lb-test-1 | k8s-master-01 | Keepalived & HAproxy |
| 10.220.62.42 | lb-test-2 | k8s-master-02 | Keepalived & HAproxy |
| 10.220.62.43 | lb-test-3 | k8s-master-03 | Keepalived & HAproxy |
| 10.220.62.40 | VIP       |               | 虚拟 IP 地址         |



## 配置负载均衡

[Keepalived](https://www.keepalived.org/) 提供 VRRP 实现，并允许您配置 Linux 机器使负载均衡，预防单点故障。[HAProxy](https://www.haproxy.org/) 提供可靠、高性能的负载均衡，能与 Keepalived 完美配合。

由于 lb-test-1，lb-test-2，lb-test-3 上安装了 Keepalived 和 HAproxy，如果其中一个节点故障，虚拟 IP 地址（即浮动 IP  地址）将自动与另一个节点关联，使集群仍然可以正常运行，从而实现高可用。若有需要，也可以此为目的，添加更多安装 Keepalived 和  HAproxy 的节点。

先在三台linux中运行以下命令安装 Keepalived 和 HAproxy。

```bash
$ dnf install keepalived haproxy -y
```

### HAproxy

1. 在三台主机上修改hosts创建本地域名解析

   ```bash
   $ vim /etc/hosts
   127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
   ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
   10.225.63.41	k8s-master-01
   10.225.63.42	k8s-master-02
   10.225.63.43	k8s-master-03
   ```

2. 在三台用于负载均衡的机器上运行以下命令以配置 Proxy（两台机器的 Proxy 配置相同）：

   ```bash
   $ mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
   
   $ vim /etc/haproxy/haproxy.cfg
   ```

3. 以下是示例配置，供您参考（请注意 `server` 字段。请记住 `6443` 是 `apiserver` 端口）：

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
       server k8s-master-01 10.225.63.41:6443 check # Replace the IP address with your own.
       server k8s-master-02 10.225.63.42:6443 check # Replace the IP address with your own.
       server k8s-master-03 10.225.63.43:6443 check # Replace the IP address with your own.
    
   listen nodeport-proxy-server
       bind *:30000-32767
       mode tcp
       server k8s-master-01 10.225.63.41
       server k8s-master-02 10.225.63.42
       server k8s-master-03 10.225.63.43
   ```

4. 保存文件并运行以下命令以开启 HAproxy。

   ```bash
   $ systemctl enable --now haproxy
   ```

5. 检查 HAproxy 当前运行状态：

   ```bash
   $ systemctl status haproxy
   ```

6. 确保您在三台机器上做了同样的 HAproxy 配置。

### Keepalived

两台机器上必须都安装 Keepalived，但在配置上略有不同。

1. 运行以下命令以配置 Keepalived。

   ```bash
   $ mv /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf.bak
   
   $ vim /etc/keepalived/keepalived.conf
   ```

2. 以下是示例配置 (lb-test-1)，供您参考：

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
     interface ens3                       # Network card
     virtual_router_id 60
     advert_int 1
     authentication {
       auth_type PASS
       auth_pass 1111
     }
     unicast_src_ip 10.225.63.41      # The IP address of this machine
     unicast_peer {
       10.225.63.42                   # The IP address of peer machines
       10.225.63.43
     }
      
     virtual_ipaddress {
       10.225.63.40/24                  # The VIP address
     }
      
     track_script {
       chk_haproxy
     }
   }
   ```

   以下是示例配置 (lb-test-2)，供您参考：

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
     interface ens3                       # Network card
     virtual_router_id 60
     advert_int 1
     authentication {
       auth_type PASS
       auth_pass 1111
     }
     unicast_src_ip 10.225.63.42      # The IP address of this machine
     unicast_peer {
       10.225.63.41                   # The IP address of peer machines
       10.225.63.43
     }
      
     virtual_ipaddress {
       10.225.63.40/24                  # The VIP address
     }
      
     track_script {
       chk_haproxy
     }
   }
   ```

   以下是示例配置 (lb-test-3)，供您参考：

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
     interface ens3                       # Network card
     virtual_router_id 60
     advert_int 1
     authentication {
       auth_type PASS
       auth_pass 1111
     }
     unicast_src_ip 10.225.63.43      # The IP address of this machine
     unicast_peer {
       10.225.63.41                   # The IP address of peer machines
       10.225.63.42
     }
      
     virtual_ipaddress {
       10.225.63.40/24                  # The VIP address
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

   

## 验证高可用

在开始创建 Kubernetes 集群之前，请确保已经测试了高可用。

目标：三台负载均衡的机器上，出现inet配置的，只有其中一台虚拟机出现两条，另外两台虚拟机只有一条。

出现两条inet地址配置的虚拟机，有一条是其自身ip地址，另外一条是vip，虚拟ip地址。

1. 在机器 `lb-test-3` 上，运行以下命令：

   ```bash
   $ ip a s
   ```

   结果如下

   ```
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
   2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
       link/ether fa:16:3e:2a:3c:ed brd ff:ff:ff:ff:ff:ff
       inet 10.225.63.43/24 brd 10.225.63.255 scope global dynamic noprefixroute ens3
          valid_lft 86359sec preferred_lft 86359sec
       inet 10.225.63.40/24 scope global secondary ens3
          valid_lft forever preferred_lft forever
       inet6 fe80::f816:3eff:fe2a:3ced/64 scope link
          valid_lft forever preferred_lft forever
   ```

2. 在`lb-test-1`和`lb-test-2`上显示的都是自己的ip

   lb-test-1

   ```
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
   2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
       link/ether fa:16:3e:3e:b9:87 brd ff:ff:ff:ff:ff:ff
       inet 10.225.63.41/24 brd 10.225.63.255 scope global dynamic noprefixroute ens3
          valid_lft 85030sec preferred_lft 85030sec
       inet6 fe80::f816:3eff:fe3e:b987/64 scope link
          valid_lft forever preferred_lft forever
   ```

   lb-test-2

   ```
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
   2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
       link/ether fa:16:3e:a1:41:16 brd ff:ff:ff:ff:ff:ff
       inet 10.225.63.42/24 brd 10.225.63.255 scope global dynamic noprefixroute ens3
          valid_lft 86386sec preferred_lft 86386sec
       inet6 fe80::f816:3eff:fea1:4116/64 scope link
          valid_lft forever preferred_lft forever
   ```

   

3. 尝试在含有vip的虚拟机上执行如下：

   ```bash
   $ systemctl stop keepalived
   ```

   这里只可以用启停keepalived来做实验，而不是haproxy来做实验。

4. 再次检查浮动 IP 地址，您可以看到该地址在 `lb-test-3` 上消失了。

   

5. 理论上讲，若配置成功，该虚拟 IP 会漂移到另一台机器 上。

   

   