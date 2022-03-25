# 3master+3worker

[TOC]

## 准备主机

| IP 地址      | 服务名        | 角色                 |
| ------------ | ------------- | -------------------- |
| 10.220.62.43 | internal-VIP  | 内网端口转发         |
| 10.220.62.27 | k8s-lb-01     | Keepalived & HAproxy |
| 10.220.62.28 | k8s-lb-02     | Keepalived & HAproxy |
| 10.220.62.29 | k8s-lb-03     | Keepalived & HAproxy |
| 10.220.65.21 | k8s-master-01 |                      |
| 10.220.65.22 | k8s-master-02 |                      |
| 10.220.65.23 | k8s-master-03 |                      |
| 10.220.65.24 | k8s-worker-01 |                      |
| 10.220.65.25 | k8s-worker-02 |                      |
| 10.220.65.26 | k8s-worker-03 |                      |
| 10.220.65.30 | global-VIP    | 公网虚拟 IP 地址     |



## 配置负载均衡

[Keepalived](https://www.keepalived.org/) 提供 VRRP 实现，并允许您配置 Linux 机器使负载均衡，预防单点故障。[HAProxy](https://www.haproxy.org/) 提供可靠、高性能的负载均衡，能与 Keepalived 完美配合。

由于 k8s-lb-01，k8s-lb-02，k8s-lb-03 上安装了 Keepalived 和 HAproxy，如果其中一个节点故障，虚拟 IP 地址（即浮动 IP  地址）将自动与另一个节点关联，使集群仍然可以正常运行，从而实现高可用。若有需要，也可以此为目的，添加更多安装 Keepalived 和  HAproxy 的节点。

先在三台linux中运行以下命令安装 Keepalived 和 HAproxy。

```bash
$ dnf install keepalived haproxy -y
```

### HAproxy

1. 在三台主机上修改hosts创建本地域名解析

   ```bash
   $ tee /etc/hosts <<-'EOF'
   127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
   ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
   10.220.62.27 k8s-lb-01
   10.220.62.28 k8s-lb-02
   10.220.62.29 k8s-lb-03
   10.220.65.21 k8s-master-01
   10.220.65.22 k8s-master-02
   10.220.65.23 k8s-master-03
   EOF
   ```

2. 在三台用于负载均衡的机器上运行以下命令以配置 Proxy（两台机器的 Proxy 配置相同）：

   ```bash
   $ mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
   ```
   
3. 以下是示例配置，供您参考（请注意 `server` 字段。请记住 `6443` 是 `apiserver` 端口）：

   ```
   tee vim /etc/haproxy/haproxy.cfg <<- 'EOF'
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
       server k8s-master-01 10.220.65.21:6443 check # Replace the IP address with your own.
       server k8s-master-02 10.220.65.22:6443 check # Replace the IP address with your own.
       server k8s-master-03 10.220.65.23:6443 check # Replace the IP address with your own.
    
   listen nodeport-proxy-server
       bind *:30000-32767
       mode tcp
       server k8s-master-01 10.220.65.21
       server k8s-master-02 10.220.65.22
       server k8s-master-03 10.220.65.23
   EOF
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

2. 以下是示例配置 (k8s-lb-01)，供您参考：

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
     interface ens192                     # Network card
     virtual_router_id 60
     advert_int 1
     authentication {
       auth_type PASS
       auth_pass 1111
     }
     unicast_src_ip 10.220.62.27      # The IP address of this machine
     unicast_peer {
       10.220.62.28                  # The IP address of peer machines
       10.220.62.29
     }
      
     virtual_ipaddress {
       10.220.62.30/24                  # The VIP address
     }
      
     track_script {
       chk_haproxy
     }
   }
   ```

   以下是示例配置 (k8s-lb-02)，供您参考：

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
     interface ens192                     # Network card
     virtual_router_id 60
     advert_int 1
     authentication {
       auth_type PASS
       auth_pass 1111
     }
     unicast_src_ip 10.220.62.28     # The IP address of this machine
     unicast_peer {
       10.220.62.27                  # The IP address of peer machines
       10.220.62.29
     }
      
     virtual_ipaddress {
       10.220.62.30/24                  # The VIP address
     }
      
     track_script {
       chk_haproxy
     }
   }
   ```

   以下是示例配置 (k8s-lb-03)，供您参考：

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
     interface ens192                     # Network card
     virtual_router_id 60
     advert_int 1
     authentication {
       auth_type PASS
       auth_pass 1111
     }
     unicast_src_ip 10.220.62.29      # The IP address of this machine
     unicast_peer {
       10.220.62.27                   # The IP address of peer machines
       10.220.62.28
     }
      
     virtual_ipaddress {
       10.220.62.30/24                  # The VIP address
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

   

### 验证高可用

在开始创建 Kubernetes 集群之前，请确保已经测试了高可用。

目标：三台负载均衡的机器上，出现inet配置的，只有其中一台虚拟机出现两条，另外两台虚拟机只有一条。

出现两条inet地址配置的虚拟机，有一条是其自身ip地址，另外一条是vip，虚拟ip地址。

1. 在机器 `k8s-lb-03` 上，运行以下命令：

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
       inet 10.220.62.29/24 brd 10.225.63.255 scope global dynamic noprefixroute ens3
          valid_lft 86359sec preferred_lft 86359sec
       inet 10.220.62.30/24 scope global secondary ens3
          valid_lft forever preferred_lft forever
       inet6 fe80::f816:3eff:fe2a:3ced/64 scope link
          valid_lft forever preferred_lft forever
   ```

2. 在`k8s-lb-01`和`k8s-lb-02`上显示的都是自己的ip

   k8s-lb-01

   ```
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
   2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
       link/ether fa:16:3e:3e:b9:87 brd ff:ff:ff:ff:ff:ff
       inet 10.220.62.27/24 brd 10.225.63.255 scope global dynamic noprefixroute ens3
          valid_lft 85030sec preferred_lft 85030sec
       inet6 fe80::f816:3eff:fe3e:b987/64 scope link
          valid_lft forever preferred_lft forever
   ```

   k8s-lb-02

   ```
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
   2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
       link/ether fa:16:3e:a1:41:16 brd ff:ff:ff:ff:ff:ff
       inet 10.220.62.28/24 brd 10.225.63.255 scope global dynamic noprefixroute ens3
          valid_lft 86386sec preferred_lft 86386sec
       inet6 fe80::f816:3eff:fea1:4116/64 scope link
          valid_lft forever preferred_lft forever
   ```

   

3. 尝试在含有vip的虚拟机上执行如下：

   ```bash
   $ systemctl stop keepalived
   ```

   这里只可以用启停keepalived来做实验，而不是haproxy来做实验。

4. 再次检查浮动 IP 地址，您可以看到该地址在 `k8s-lb-03` 上消失了。

   

5. 理论上讲，若配置成功，该虚拟 IP 会漂移到另一台机器 上。

   

## 部署k8s集群

### k8s-master  

 Linux初始化

   ```bash
   $ wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/rocky_linux_8.5_init.sh | sh
   ```

   使用如下命令安装

   ```bash
   $ wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_containerd.sh | bash -s 1.21.11
   ```

   用这个kubeadm替换master系统目录下的 /usr/bin/kubeadm

   编辑kubeadm-config.yaml文件

   ```bash
   $ kubeadm config print init-defaults > kubeadm-config.yaml
   ```

   在 `clusterName: kubernetes` 下面, `controllerManager: {}`上面添加一行 controlPlaneEndpoint

   ```bash
   $ vi kubeadm-config.yaml
   ```

   查看 kubeadm-config.yaml文件

   ```yaml
   apiVersion: kubeadm.k8s.io/v1beta2
   bootstrapTokens:
   - groups:
     - system:bootstrappers:kubeadm:default-node-token
     token: abcdef.0123456789abcdef
     ttl: 24h0m0s
     usages:
     - signing
     - authentication
   kind: InitConfiguration
   localAPIEndpoint:
     advertiseAddress: 10.220.65.21
     bindPort: 6443
   nodeRegistration:
     # criSocket: /var/run/dockershim.sock
     criSocket: /run/containerd/containerd.sock
     name: k8s-master-01
     taints: 
     - effect: NoSchedule
       key: node-role.kubernetes.io/master
   ---
   apiServer:
     timeoutForControlPlane: 4m0s
   apiVersion: kubeadm.k8s.io/v1beta2
   certificatesDir: /etc/kubernetes/pki
   clusterName: kubernetes
   controlPlaneEndpoint: "10.220.62.30:6443"
   controllerManager: {}
   dns:
     type: CoreDNS
   etcd:
     local:
       dataDir: /var/lib/etcd
   imageRepository: docker.io/sjwayrhz
   kind: ClusterConfiguration
   kubernetesVersion: v1.21.11
   networking:
     dnsDomain: sjhz.tk
     podSubnet: 10.4.0.0/16
     serviceSubnet: 10.6.0.0/16
   scheduler: {}
   ```

   如果服务器重启过了，需要做如下操作

   ```bash
   $ modprobe br_netfilter
   $ echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
   ```

   进入k8s-master-01初始化k8s集群

   ```bash
   $ kubeadm init --config kubeadm-config.yaml 
   ```

   获得以下结果

   ```bash
   Your Kubernetes control-plane has initialized successfully!
   
   To start using your cluster, you need to run the following as a regular user:
   
     mkdir -p $HOME/.kube
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     sudo chown $(id -u):$(id -g) $HOME/.kube/config
   
   Alternatively, if you are the root user, you can run:
   
     export KUBECONFIG=/etc/kubernetes/admin.conf
   
   You should now deploy a pod network to the cluster.
   Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
     https://kubernetes.io/docs/concepts/cluster-administration/addons/
   
   You can now join any number of control-plane nodes by copying certificate authorities
   and service account keys on each node and then running the following as root:
   
     kubeadm join 10.220.62.30:6443 --token abcdef.0123456789abcdef \
           --discovery-token-ca-cert-hash sha256:59116345e04464f759713ee7abe5906ba630506e7ac4b54638da2e87706d28ed \
           --control-plane
   
   Then you can join any number of worker nodes by running the following on each as root:
   
   kubeadm join 10.220.62.30:6443 --token abcdef.0123456789abcdef \
           --discovery-token-ca-cert-hash sha256:59116345e04464f759713ee7abe5906ba630506e7ac4b54638da2e87706d28ed
   ```

   

### 添加master节点

假设原先有k8s-master-01作为主节点，现在需要添加两个master分别为 k8s-master-02 和k8s-master-03

登录 k8s-master-02, 安装containerd

```bash
$ wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_containerd.sh | bash -s 1.21.11
```

安装containerd之后，替换10年证书的kubeadm

```bash
https://wwi.lanzouo.com/i5oWKxerf3c

$ wget -P /usr/bin/ -N http://121.46.238.135:9000/k8s-tools/kubeadm-1.21/kubeadm
```

登录k8s-master-01, 获得加入master的命令

```bash
$ kubeadm token create --print-join-command
kubeadm join 10.220.62.30:6443 --token noxy10.4rq8sl2rujhpp0s2 --discovery-token-ca-cert-hash sha256:59116345e04464f759713ee7abe5906ba630506e7ac4b54638da2e87706d28ed 

$ kubeadm init phase upload-certs --upload-certs
W0322 05:21:31.805370   35220 version.go:114] could not obtain client version; using remote version: v1.23.5
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
91307fe4f4327a8fd5425cb87579ef34c1f3e6d313d0bf2e0e72766125760505
```

于是， k8s-master-02加入k8s-master-01到集群的命令为

```bash
$ kubeadm join 10.220.62.30:6443 \
--token noxy10.4rq8sl2rujhpp0s2 --discovery-token-ca-cert-hash sha256:59116345e04464f759713ee7abe5906ba630506e7ac4b54638da2e87706d28ed \
--control-plane --certificate-key 91307fe4f4327a8fd5425cb87579ef34c1f3e6d313d0bf2e0e72766125760505
```

可以登陆k8s-master-02和k8s-master-03执行上述指令，然后拷贝证书

需要将k8s-master-01上的证书打包拷贝到其他的master，证书位置

```
/etc/kubernetes/pki/
```

在k8s-master-01上安装登录k8s-master-02和k8s-master-03的私钥拷贝脚本如下：

```shell
$ tee /tmp/cpkey.sh <<- "EOF"
USER=root
CONTROL_PLANE_IPS="10.220.65.22 10.220.65.23"  # IP of k8s-master-02 and k8s-master-03
dir=/etc/kubernetes/pki/
for host in ${CONTROL_PLANE_IPS}; do
    scp /etc/kubernetes/pki/ca.crt "${USER}"@$host:${dir}
    scp /etc/kubernetes/pki/ca.key "${USER}"@$host:${dir}
    scp /etc/kubernetes/pki/sa.key "${USER}"@$host:${dir}
    scp /etc/kubernetes/pki/sa.pub "${USER}"@$host:${dir}
    scp /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@$host:${dir}
    scp /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@$host:${dir}
    scp /etc/kubernetes/pki/etcd/ca.crt "${USER}"@$host:${dir}etcd
    scp /etc/kubernetes/pki/etcd/ca.key "${USER}"@$host:${dir}etcd
done
EOF
```

执行该脚本

```bash
$ sh /tmp/cpkey.sh
```

### 添加worker节点

让所有的worker节点加入到k8s集群

```bash
$ kubeadm join 10.220.62.30:6443 \
--token noxy10.4rq8sl2rujhpp0s2 --discovery-token-ca-cert-hash sha256:59116345e04464f759713ee7abe5906ba630506e7ac4b54638da2e87706d28ed 
```

如何需要使用rancher，三个master还需要做如下操作

```bash
$ sed -i 's|- --port=0|#- --port=0|' /etc/kubernetes/manifests/kube-scheduler.yaml
$ sed -i 's|- --port=0|#- --port=0|' /etc/kubernetes/manifests/kube-controller-manager.yaml

$ systemctl restart kubelet
```

### 安装kube-ovn网络

```bash
$ wget https://raw.githubusercontent.com/kubeovn/kube-ovn/release-1.8/dist/images/install.sh
```

修改service,pod,join关键信息，编辑 install.sh

```
REGISTRY="kubeovn"
VERSION="v1.8.3"
IMAGE_PULL_POLICY="IfNotPresent"
POD_CIDR="10.4.0.0/16"                # Do NOT overlap with NODE/SVC/JOIN CIDR
POD_GATEWAY="10.4.0.1"
SVC_CIDR="10.6.0.0/16"                # Do NOT overlap with NODE/POD/JOIN CIDR
JOIN_CIDR="10.5.0.0/16"              # Do NOT overlap with NODE/POD/SVC CIDR
```

执行安装

```bash
$ bash install.sh
```

### 安装calico网络

release will be found on this site . `https://docs.projectcalico.org/releases`

```bash
$ kubectl apply -f https://docs.projectcalico.org/v3.20/manifests/calico.yaml
```

## 配置ceph存储

克隆项目

k8s-1.21.11 对应与 rook-ceph-1.8.2，可以兼容

登陆infrastructure，下载安装git，克隆gitee上的k8s项目

```bash
$ cd /tmp
$ git clone git@gitee.com:sjwayrhz/devops.git
$ cd devops/storage/rook-1.8.2/
```

部署到k8s集群

```bash
$ kubectl create -f crds.yaml -f common.yaml -f operator.yaml
$ kubectl create -f cluster.yaml
```

部署cephfs

```bash
$ kubectl  apply -f filesystem.yaml
$ kubectl  apply -f csi/cephfs/storageclass.yaml
```



## 安装k8s-console

### 选择rancher为console

登陆k8s-console

安装docker

```bash
$ wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_docker.sh | sh
```

启动rancher

```bash
$ docker run -d --restart=unless-stopped \
  --privileged \
  -p 80:80 -p 443:443 \
  -v /opt/rancher:/var/lib/rancher/ \
  rancher/rancher:stable
```

添加集群即可