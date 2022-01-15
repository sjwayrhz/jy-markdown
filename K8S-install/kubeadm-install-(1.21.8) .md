# 实战安装k8s集群

[TOC]

## 资源准备

### 虚拟机准备

准备的资源信息如下

|  IP address  |     Hostname      |   Disk   |    Application     |
| :----------: | :---------------: | :------: | :----------------: |
| 10.220.62.31 |        vip        |          |                    |
| 10.220.62.32 | loadbalance-left  |   40G    | haproxy+keepalived |
| 10.220.62.33 | loadbalance-right |   40G    | haproxy+keepalived |
| 10.220.62.21 | devops-master-01  |   40G    |                    |
| 10.220.62.22 | devops-master-02  |   40G    |                    |
| 10.220.62.23 | devops-master-03  |   40G    |                    |
| 10.220.62.24 | devops-worker-01  | 40G+400G |                    |
| 10.220.62.25 | devops-worker-02  | 40G+400G |                    |
| 10.220.62.26 | devops-worker-03  | 40G+400G |                    |

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

$ ansible all -m shell -a "wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_containerd.sh | bash -s 1.21.8"
```

## 安装kubernetes

### 安装k8s-master

使用kubeadm之前，可以提前导入能生成10年有效期证书的kubeadm文件,蓝奏云分享链接和wget url如下：

```
https://wwi.lanzouo.com/i5oWKxerf3c
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
	--discovery-token-ca-cert-hash sha256:732a5bb54e5dbbbc4c61e96dc6e88974dfd6f09f95eafbe260ae1195f2f804d2 \
	--control-plane

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.220.62.31:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:732a5bb54e5dbbbc4c61e96dc6e88974dfd6f09f95eafbe260ae1195f2f804d2
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

假设原先有k8s-master作为主节点，现在需要添加两个master分别为 k8s-master-left 和k8s-master-right

登录k8s-master, 获得加入master的命令

```bash
$ kubeadm token create --print-join-command
kubeadm join 10.220.62.31:6443 --token 8dnb5o.ce7g7yzmw1f7lsm3 --discovery-token-ca-cert-hash sha256:732a5bb54e5dbbbc4c61e96dc6e88974dfd6f09f95eafbe260ae1195f2f804d2

$ kubeadm init phase upload-certs --upload-certs
W0115 08:59:53.150220   15905 version.go:114] could not obtain client version; using remote version: v1.23.1
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
64c46206ede6bb29c1024fdcf80bbdc28341ae45a1d4da955826459efb3cf6f8
```

于是，加入master到集群的命令为

```bash
$ kubeadm join 10.220.62.31:6443 \
--token 8dnb5o.ce7g7yzmw1f7lsm3 --discovery-token-ca-cert-hash sha256:732a5bb54e5dbbbc4c61e96dc6e88974dfd6f09f95eafbe260ae1195f2f804d2 \
--control-plane --certificate-key 64c46206ede6bb29c1024fdcf80bbdc28341ae45a1d4da955826459efb3cf6f8
```

可以登陆k8s-master-left和k8s-master-right执行上述指令

需要将k8s-master上的证书打包拷贝到其他的master，证书位置

```
/etc/kubernetes/pki/
```

然后登陆到ansible，让所有的node节点加入到k8s集群

```bash
$ ansible node -m shell -a "kubeadm join 10.220.62.31:6443 --token 8dnb5o.ce7g7yzmw1f7lsm3 --discovery-token-ca-cert-hash sha256:732a5bb54e5dbbbc4c61e96dc6e88974dfd6f09f95eafbe260ae1195f2f804d2"
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
