# 实战安装k8s集群

[TOC]

## 资源准备

### 虚拟机准备

准备的资源信息如下

|  IP address  |   Hostname    |   Disk    |                Application                |
| :----------: | :-----------: | :-------: | :---------------------------------------: |
| 10.220.65.65 |      vip      |           |                  ingress                  |
| 10.220.65.61 | k8s-master-01 |   557G    | docker;rancher,ingress,haproxy+keepalived |
| 10.220.65.62 | k8s-master-02 | 557G+3.6T |         minio,haproxy+keepalived          |
| 10.220.65.63 | k8s-master-03 | 557G+3.6T |       nfs-server,haproxy+keepalived       |
| 10.220.65.71 | k8s-worker-01 | 557G+8.9T |                                           |
| 10.220.65.72 | k8s-worker-02 | 40G+8.9T  |                                           |
| 10.220.65.73 | k8s-worker-03 | 40G+8.9T  |                                           |

登录每一台服务器，配置/etc/hosts解析

```bash
$  tee /etc/hosts <<-'EOF'
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.220.65.61 k8s-master-01
10.220.65.62 k8s-master-02
10.220.65.63 k8s-master-03
10.220.65.71 k8s-worker-01
10.220.65.72 k8s-worker-02
10.220.65.73 k8s-worker-03
EOF
```

然后登录每一台服务器，做以下几件事：

1. 设置root登录密钥

   ```bash
   $ mkdir ~/.ssh
   $ tee ~/.ssh/authorized_keys <<-'EOF'
   ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCtmrBfOFLF8tUByZMNbRkhCqrLxPscI1IAvv/wjJustDEsvn4W0Wxx7UIsSGhBbRkBE3pMjyNJylyqlkFixt9onGTdSVMdNuAxHSTqV7n5B4ZXXmuSTGnl2DSKYgOSleCpakAK15TBClPm1aB0c3Rhvsd6iyMfTD9MKGXENitpokToO0XSJMU/njqEkhJcwXTGHXztpitqsVnXI/4BUYya6ZYsmKASFwaPmpjvjrI8V2eVTS9bajDh7Ap8XS9d96NQYqCU2IutaYeqM7/0S9V/aCGk3M+K5ulVEpwb5bu2fjd1GTG0WaDPdxZ18DbXBZEYp2wNgYptRx8D/AM39hTT
   EOF
   $ chmod 600 ~/.ssh/authorized_keys 
   ```

2. 关闭root密码登录权限，并尝试可以使用密钥登录成功

   ```bash
   $ sed -i "s/^PasswordAuthentication.*/PasswordAuthentication no/g" /etc/ssh/sshd_config
   $ systemctl restart sshd
   ```

3. 换yum源

   ```bash
   $ sed -e 's|^mirrorlist=|#mirrorlist=|g' \
       -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.sjtug.sjtu.edu.cn/rocky|g' \
       -i.bak \
       /etc/yum.repos.d/Rocky-*.repo
   ```

4. 安装wget

   ```bash
   $ dnf install -y wget
   ```

5. 设置主机名

   ```bash
   $ vi /etc/hostname
   k8s-master-01
   ```

6. 重启

   ```bash
   $ reboot
   ```

### 安装haproxy+keepalived 

在k8s-master-01,k8s-master-02,k8s-master-03上均要安装haproxy和keepalived

```bash
$ dnf install keepalived haproxy -y
```

**haproxy**

备份原来的haproxy配置

```bash
$ mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
```

在三台用于负载均衡的机器上运行以下命令以配置 Proxy（三台机器的 Proxy 配置相同）：

```bash
tee /etc/haproxy/haproxy.cfg <<-'EOF'
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
  bind *:8443
  mode tcp
  option tcplog
  default_backend kube-apiserver
   
backend kube-apiserver
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    server k8s-master-01 10.220.65.61:6443 check # Replace the IP address with your own.
    server k8s-master-02 10.220.65.62:6443 check # Replace the IP address with your own.
    server k8s-master-03 10.220.65.63:6443 check # Replace the IP address with your own.
 
listen nodeport-proxy-server
    bind *:30000-32767
    mode tcp
    server k8s-master-01 10.220.65.61
    server k8s-master-02 10.220.65.62
    server k8s-master-03 10.220.65.63
EOF
```

### Keepalived

两台机器上必须都安装 Keepalived，但在配置上略有不同。

1. 运行以下命令以配置 Keepalived。

   ```bash
   $ mv /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf.bak
   ```
   
2. 以下是示例配置 (k8s-master-01)，供您参考：

   ```bash
   tee /etc/keepalived/keepalived.conf <<-'EOF'
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
     interface eno3                       # Network card
     virtual_router_id 60
     advert_int 1
     authentication {
       auth_type PASS
       auth_pass 1111
     }
     unicast_src_ip 10.220.65.61      # The IP address of this machine
     unicast_peer {
       10.220.65.62                   # The IP address of peer machines
       10.220.65.63
     }
      
     virtual_ipaddress {
       10.220.65.65/24                  # The VIP address
     }
      
     track_script {
       chk_haproxy
     }
   }
   EOF
   ```

   以下是示例配置 (k8s-master-02)，供您参考：
   
   ```
     ---
     unicast_src_ip 10.220.65.62      # The IP address of this machine
     unicast_peer {
       10.220.65.61                   # The IP address of peer machines
       10.220.65.63
     }
      
     virtual_ipaddress {
       10.220.65.65/24                  # The VIP address
     }
     ---
   ```
   
   以下是示例配置 (k8s-master-03)，供您参考：
   
   ```
    ---
    unicast_src_ip 10.220.65.63      # The IP address of this machine
     unicast_peer {
       10.220.65.61                   # The IP address of peer machines
       10.220.65.62
     }
      
     virtual_ipaddress {
       10.220.65.65/24                  # The VIP address
     }
     ---
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

1. 在机器 `k8s-master-03` 上，运行以下命令：

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
   2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
       link/ether b4:05:5d:c8:48:c4 brd ff:ff:ff:ff:ff:ff
   3: eno2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
       link/ether b4:05:5d:c8:48:c5 brd ff:ff:ff:ff:ff:ff
   4: eno3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
       link/ether b4:05:5d:c8:48:c6 brd ff:ff:ff:ff:ff:ff
       inet 10.220.65.63/24 brd 10.220.65.255 scope global noprefixroute eno3
          valid_lft forever preferred_lft forever
       inet 10.220.65.65/24 scope global secondary eno3
          valid_lft forever preferred_lft forever
       inet6 fe80::b605:5dff:fec8:48c6/64 scope link noprefixroute
          valid_lft forever preferred_lft forever
   ```

2. 在`k8s-master-01`和`k8s-master-03`上显示的都是自己的ip

   k8s-master-01

   ```
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
   2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
       link/ether b4:05:5d:c7:3e:a6 brd ff:ff:ff:ff:ff:ff
   3: eno2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
       link/ether b4:05:5d:c7:3e:a7 brd ff:ff:ff:ff:ff:ff
       inet6 fe80::d844:8653:d044:a309/64 scope link noprefixroute
          valid_lft forever preferred_lft forever
   4: eno3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
       link/ether b4:05:5d:c7:3e:a8 brd ff:ff:ff:ff:ff:ff
       inet 10.220.65.61/24 brd 10.220.65.255 scope global noprefixroute eno3
          valid_lft forever preferred_lft forever
       inet6 fe80::b605:5dff:fec7:3ea8/64 scope link noprefixroute
          valid_lft forever preferred_lft forever
   ```

   k8s-master-02

   ```
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
   2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
       link/ether b4:05:5d:c8:57:04 brd ff:ff:ff:ff:ff:ff
   3: eno2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
       link/ether b4:05:5d:c8:57:05 brd ff:ff:ff:ff:ff:ff
       inet6 fe80::c03d:c37a:56c1:5b1a/64 scope link noprefixroute
          valid_lft forever preferred_lft forever
   4: eno3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
       link/ether b4:05:5d:c8:57:06 brd ff:ff:ff:ff:ff:ff
       inet 10.220.65.62/24 brd 10.220.65.255 scope global noprefixroute eno3
          valid_lft forever preferred_lft forever
       inet6 fe80::b605:5dff:fec8:5706/64 scope link noprefixroute
          valid_lft forever preferred_lft forever
   ```

3. 尝试在含有vip的虚拟机上执行如下：

   ```bash
   $ systemctl stop keepalived
   ```

   这里只可以用启停keepalived来做实验，而不是haproxy来做实验。

4. 再次检查浮动 IP 地址，您可以看到该地址在 `k8s-master-03` 上消失了。

   

5. 理论上讲，若配置成功，该虚拟 IP 会漂移到另一台机器 上。

### Ansible配置

在infra虚拟机，配置免确认ssh

```bash
$ vim  /etc/ssh/ssh_config
```

将 “#  StrictHostKeyChecking ask” 改为“StrictHostKeyChecking no”

导入私钥到infra服务器`~/.ssh/id_rsa`中

**然后修改ansible主机的 /etc/hosts和/etc/ansible/hosts**

```bash
$ cat /etc/ansible/hosts
[k8s]
k8s-master-01
k8s-master-02
k8s-master-03
k8s-worker-01
k8s-worker-02
k8s-worker-03
```

使用如下命令检测ansible是否配置成功

```bash
$ ansible k8s -m ping
```

### 安装containerd

需要rocky linux安装wget ，更换过国内源并且`.ssh/authorized_keys`中储存好公钥，然后可以使用如下脚本初始化环境

```shell
$ ansible k8s -m shell -a "wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/rocky_linux_8.5_init.sh | sh"
```

如果已经是模版rocky linux了，仅需在ansible虚拟机中执行如下命令安装1.21.10版本运行时

```bash
$ ansible k8s -m shell -a "wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_containerd.sh | bash -s 1.21.10"
```

## 安装kubernetes

### 安装k8s-master-01

使用kubeadm之前，可以提前导入能生成10年有效期证书的kubeadm文件,蓝奏云分享链接和wget url如下：

```bash
https://wwi.lanzouo.com/i5oWKxerf3c
$ rm -f /usr/bin/kubeadm
$ wget -P /usr/bin/ -N http://121.46.238.135:9000/k8s-tools/kubeadm-1.21/kubeadm
```

用这个kubeadm替换master系统目录下的 /usr/bin/kubeadm

编辑kubeadm-config.yaml文件

```bash
$ kubeadm config print init-defaults > kubeadm-config.yaml
```

在 `clusterName: kubernetes` 下面, `controllerManager: {}`上面添加一行 controlPlaneEndpoint

```bash
$ vim kubeadm-config.yaml
```

查看 kubeadm-config.yaml文件

```shell
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
  advertiseAddress: 10.220.65.61
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
controlPlaneEndpoint: "10.220.65.65:8443"
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.21.10
networking:
  dnsDomain: cluster.local
  podSubnet: 10.4.0.0/16
  serviceSubnet: 10.6.0.0/16
scheduler: {}
```

进入k8s-master-01初始化k8s集群

```bash
$ kubeadm init --config kubeadm-config.yaml 
```

得到如下输出

```
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

  kubeadm join 10.220.65.31:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:d7b0780a21d34591268906a7855a685d093d14fda2699832ad4f65c4e9091296 \
	--control-plane

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.220.65.31:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:d7b0780a21d34591268906a7855a685d093d14fda2699832ad4f65c4e9091296
```



### kubectl命令自动补全

安装 bash-completion

```bash
$ dnf install bash-completion -y
```

添加环境变量

```bash
$ echo "source <(kubectl completion bash)" >> ~/.bashrc
$ source ~/.bashrc
```

### 添加master节点

假设原先有k8s-master-01作为主节点，现在需要添加两个master分别为 k8s-master-02 和k8s-master-03

登录 k8s-master-02, 安装containerd

```bash
$ wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_containerd.sh | bash -s 1.21.10
```

安装containerd之后，替换10年证书的kubeadm

```bash
https://wwi.lanzouo.com/i5oWKxerf3c

$ wget -P /usr/bin/ -N http://121.46.238.135:9000/k8s-tools/kubeadm-1.21/kubeadm
```

登录k8s-master-01, 获得加入master的命令

```bash
$ kubeadm token create --print-join-command
kubeadm join 10.220.65.31:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:d7b0780a21d34591268906a7855a685d093d14fda2699832ad4f65c4e9091296

$ kubeadm init phase upload-certs --upload-certs
W0117 17:38:56.059723    4405 version.go:114] could not obtain client version; using remote version: v1.23.1
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
c309edc3ccf996d37cc5abeb84df33a80899f0e9a698ad270b52cbc5c8876b5f
```

于是， k8s-master-02加入k8s-master-01到集群的命令为

```bash
$ kubeadm join 10.220.65.31:6443 \
--token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:d7b0780a21d34591268906a7855a685d093d14fda2699832ad4f65c4e9091296 \
--control-plane --certificate-key c309edc3ccf996d37cc5abeb84df33a80899f0e9a698ad270b52cbc5c8876b5f
```

可以登陆k8s-master-02和k8s-master-03执行上述指令，然后拷贝证书

需要将k8s-master-01上的证书打包拷贝到其他的master，证书位置

```
/etc/kubernetes/pki/
```

在k8s-master-01上安装登录k8s-master-02和k8s-master-03的私钥拷贝脚本如下：

```shell
# tee /tmp/cpkey.sh <<- "EOF"
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

然后登陆到ansible，让所有的worker节点加入到k8s集群

```bash
$ kubeadm join 10.220.65.31:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:d7b0780a21d34591268906a7855a685d093d14fda2699832ad4f65c4e9091296
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
VERSION="v1.8.2"
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

k8s-1.21.8 对应与 rook-ceph-1.8.2，可以兼容

登陆infrastructure，下载安装git，克隆gitee上的k8s项目

```bash
$ cd /tmp
$ git clone git@gitee.com:sjwayrhz/k8s.git
$ cd k8s/storage/rook-ceph/ceph-1.7/
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
