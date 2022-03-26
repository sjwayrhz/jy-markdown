# 实战安装k8s集群

[TOC]

## 资源准备

### 虚拟机准备

准备的资源信息如下

|  IP address  |     Hostname      |   Disk   |    Application     |
| :----------: | :---------------: | :------: | :----------------: |
| 10.220.62.41 |  internal-devops  |   40G    |    内网端口转发    |
| 10.220.62.32 | loadbalance-left  |   40G    | haproxy+keepalived |
| 10.220.62.33 | loadbalance-right |   40G    | haproxy+keepalived |
| 10.220.62.21 | devops-master-01  |   40G    |                    |
| 10.220.62.22 | devops-master-02  |   40G    |                    |
| 10.220.62.23 | devops-master-03  |   40G    |                    |
| 10.220.62.24 | devops-worker-01  | 40G+400G |                    |
| 10.220.62.25 | devops-worker-02  | 40G+400G |                    |
| 10.220.62.26 | devops-worker-03  | 40G+400G |                    |
| 10.220.62.31 |    global-vip     |          |   ingress-nginx    |

### 安装haproxy+keepalived 

haproxy 关键配置如下

```
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
```

### 局部DNS配置

在infra虚拟机中部署应用dnsmasq

```bash
$ dnf install -y dnsmasq
```

生成并编辑dnsmasq配置文件

```bash
$ vim /etc/dnsmasq.d/juneyao.conf
```

添加的主机信息为

```bash
# devops k8s
address=/devops-master-01/10.220.62.21
address=/devops-master-02/10.220.62.22
address=/devops-master-03/10.220.62.23
address=/devops-worker-01/10.220.62.24
address=/devops-worker-02/10.220.62.25
address=/devops-worker-03/10.220.62.26

address=/loadbalance-left/10.220.62.32
address=/loadbalance-right/10.220.62.33
```

启动并设置dnsmasq为自启动,检查启动状态

```bash
$ systemctl enable --now dnsmasq
$ systemctl status dnsmasq
```

在infra机器中配置局部dns

```bash
$ sed -i '1inameserver 10.220.62.30' /etc/resolv.conf
```

完成在首行追加后，使用`ping ceph-master`就可以测试是否生效，其他虚拟机也可以添加这一条局部dns。

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
[devops]
devops-master-01
devops-master-02
devops-master-03
devops-worker-01
devops-worker-02
devops-worker-03
```

使用如下命令检测ansible是否配置成功

```bash
$ ansible devops -m ping
```

### 局部域名解析

使用ansible复制/etc/hosts到所有节点

```bash
$ ansible devops -m copy -a "src=/etc/resolv.conf dest=/etc/resolv.conf"
```

### 安装containerd

需要rocky linux安装wget ，更换过国内源并且`.ssh/authorized_keys`中储存好公钥，然后可以使用如下脚本初始化环境

```shell
$ ansible all -m shell -a "wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/rocky_linux_8.5_init.sh | sh"
```

初始化之后，或许需要重启linux系统，但是vmware中的镜像系统，只需执行下面的一步：

如果已经是模版rocky linux了，仅需在ansible虚拟机中执行如下命令安装1.21.7版本运行时

```bash
$ ansible all -m shell -a "modprobe br_netfilter"
$ ansible all -m shell -a "echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables"

$ ansible all -m shell -a "wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_containerd_1.5.sh | bash -s 1.21.8"
```

## 安装kubernetes

### 安装devops-master-01

使用kubeadm之前，可以提前导入能生成10年有效期证书的kubeadm文件,蓝奏云分享链接和wget url如下：

```bash
https://wwi.lanzouo.com/i5oWKxerf3c

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
  advertiseAddress: 10.220.62.21
  bindPort: 6443
nodeRegistration:
  # criSocket: /var/run/dockershim.sock
  criSocket: /run/containerd/containerd.sock
  name: devops-master-01
  taints: 
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: "10.220.62.31:6443"
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.21.8
networking:
  dnsDomain: cluster.local
  podSubnet: 10.4.0.0/16
  serviceSubnet: 10.6.0.0/16
scheduler: {}
```

进入devops-master-01初始化k8s集群

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

  kubeadm join 10.220.62.31:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:d7b0780a21d34591268906a7855a685d093d14fda2699832ad4f65c4e9091296 \
	--control-plane

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.220.62.31:6443 --token abcdef.0123456789abcdef \
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

假设原先有devops-master-01作为主节点，现在需要添加两个master分别为 devops-master-02 和devops-master-03

登录 devops-master-02, 安装containerd

```bash
$ wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_containerd_1.5.sh | bash -s 1.21.8
```

安装containerd之后，替换10年证书的kubeadm

```bash
https://wwi.lanzouo.com/i5oWKxerf3c

$ wget -P /usr/bin/ -N http://121.46.238.135:9000/k8s-tools/kubeadm-1.21/kubeadm
```

登录devops-master-01, 获得加入master的命令

```bash
$ kubeadm token create --print-join-command
kubeadm join 10.220.62.31:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:d7b0780a21d34591268906a7855a685d093d14fda2699832ad4f65c4e9091296

$ kubeadm init phase upload-certs --upload-certs
W0117 17:38:56.059723    4405 version.go:114] could not obtain client version; using remote version: v1.23.1
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
c309edc3ccf996d37cc5abeb84df33a80899f0e9a698ad270b52cbc5c8876b5f
```

于是， devops-master-02加入devops-master-01到集群的命令为

```bash
$ kubeadm join 10.220.62.31:6443 \
--token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:d7b0780a21d34591268906a7855a685d093d14fda2699832ad4f65c4e9091296 \
--control-plane --certificate-key c309edc3ccf996d37cc5abeb84df33a80899f0e9a698ad270b52cbc5c8876b5f
```

可以登陆devops-master-02和devops-master-03执行上述指令，然后拷贝证书

需要将devops-master-01上的证书打包拷贝到其他的master，证书位置

```
/etc/kubernetes/pki/
```

在devops-master-01上安装登录devops-master-02和devops-master-03的私钥拷贝脚本如下：

```shell
# tee /tmp/cpkey.sh <<- "EOF"
USER=root
CONTROL_PLANE_IPS="10.220.62.22 10.220.62.23"  # IP of k8s-master-02 and k8s-master-03
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
$ kubeadm join 10.220.62.31:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:d7b0780a21d34591268906a7855a685d093d14fda2699832ad4f65c4e9091296
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

登陆infrastructure，下载安装git，克隆gitee上的devops项目

```bash
$ cd /tmp
$ git clone git@gitee.com:sjwayrhz/devops.git
$ cd devops/storage/rook-ceph/ceph-1.7/
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
