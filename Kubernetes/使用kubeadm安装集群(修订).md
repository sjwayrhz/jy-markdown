# 实战安装k8s集群

[TOC]

## 资源准备

### 虚拟机准备

准备的资源信息如下

|   IP address    |     Hostname     |
| :-------------: | :--------------: |
| 192.168.177.205 |     Ansible      |
| 192.168.177.206 |    nfs-server    |
| 192.168.177.207 |   k8s-console    |
| 192.168.177.208 |    k8s-master    |
| 192.168.177.209 | k8s-master-left  |
| 192.168.177.210 | k8s-master-right |
| 192.168.177.211 |   k8s-node-01    |
| 192.168.177.212 |   k8s-node-02    |
|       ...       |       ...        |
| 192.168.177.220 |   k8s-node-10    |

### Ansible配置

首先进入ansible虚拟机，配置免确认ssh

```bash
$ vim  /etc/ssh/ssh_config
```

将 “#  StrictHostKeyChecking ask” 改为“StrictHostKeyChecking no”

**然后修改ansible主机的 /etc/hosts和/etc/ansible/hosts**

```bash
~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.177.208 k8s-master-left
192.168.177.209 k8s-master-right
192.168.177.210 k8s-master

192.168.177.211 k8s-node-01
192.168.177.212 k8s-node-02
192.168.177.213 k8s-node-03
192.168.177.214 k8s-node-04
192.168.177.215 k8s-node-05
192.168.177.216 k8s-node-06
192.168.177.217 k8s-node-07
192.168.177.218 k8s-node-08
192.168.177.219 k8s-node-09
192.168.177.220 k8s-node-10

-----------------------------

~]# cat /etc/ansible/hosts
[master]
k8s-master
k8s-master-left
k8s-master-right

[node]
k8s-node-01
k8s-node-02
k8s-node-03
k8s-node-04
k8s-node-05
k8s-node-06
k8s-node-07
k8s-node-08
k8s-node-09
k8s-node-10
```

使用如下命令检测ansible是否配置成功

```bash
~]# ansible all -m ping
```

### 本地域名解析

使用ansible复制/etc/hosts到所有节点

```bash
~]# ansible all -m copy -a "src=/etc/hosts dest=/etc/hosts"
```

master节点： 依次登入三个master并修改/etc/hosts的第一行

```bash
k8s-master:  # head -n 1 /etc/hosts
	127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 k8s-master
k8s-master-left:  # head -n 1 /etc/hosts
	127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 k8s-master-left
k8s-master-right:  # head -n 1 /etc/hosts
	127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 k8s-master-right
```

### 安装docker

在ansible虚拟机中执行如下命令

```bash
~]# ansible all -m shell -a "wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_docker.sh | sh"
```

### 导入kubernetes repo

首先在ansible虚拟机中导入repo

```bash
~]# cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

然后将此repo拷贝到其他虚拟机

```bash
~]# ansible all -m copy -a "src=/etc/yum.repos.d/kubernetes.repo dest=/etc/yum.repos.d/kubernetes.repo"
```

## 安装kubernetes

使用如下命令安装k8s依赖组建

```bash
~]# ansible all -m yum -a "name=kubeadm-1.21.5 state=installed"
~]# ansible all -m yum -a "name=kubectl-1.21.5 state=installed"
~]# ansible all -m yum -a "name=kubelet-1.21.5 state=installed"

~]# ansible all -m systemd -a "name=kubelet enabled=yes state=started"
```

### 安装k8s-master

进入k8s-master初始化k8s-master

```bash
~]# kubeadm init \
  --apiserver-advertise-address=192.168.177.210 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.21.5 \
  --service-cidr=10.7.0.0/16 \
  --pod-network-cidr=10.3.0.0/16
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

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.177.210:6443 --token b2zlkb.xlxohrpdvula5a73 \
        --discovery-token-ca-cert-hash sha256:9e9d01d0ef844b387e1453788b5b58fcf74497cbd0ba8e8ffefd762125a380a8
```

### 添加master节点

假设原先有k8s-master作为主节点，现在需要添加两个master分别为 k8s-master-left 和k8s-master-right

在k8s-master的kubeadm-config中添加 `controlPlaneEndpoint: ${ip}:${port}`

```
controlPlaneEndpoint: 192.168.177.210:6443
```

具体操作如下：

```yaml
~]# kubectl edit -n kube-system cm kubeadm-config
apiVersion: v1
data:
  ClusterConfiguration: |
    apiServer:
      extraArgs:
        authorization-mode: Node,RBAC
      timeoutForControlPlane: 4m0s
    apiVersion: kubeadm.k8s.io/v1beta2
    certificatesDir: /etc/kubernetes/pki
    clusterName: kubernetes
    controlPlaneEndpoint: 192.168.177.210:6443
    controllerManager: {}
    dns:
      type: CoreDNS
    etcd:
      local:
        dataDir: /var/lib/etcd
    imageRepository: registry.aliyuncs.com/google_containers
    kind: ClusterConfiguration
    kubernetesVersion: v1.21.5
    networking:
      dnsDomain: cluster.local
      podSubnet: 10.3.0.0/16
      serviceSubnet: 10.7.0.0/16
    scheduler: {}
  ClusterStatus: |
    apiEndpoints:
      k8s-master:
        advertiseAddress: 192.168.177.210
        bindPort: 6443
    apiVersion: kubeadm.k8s.io/v1beta2
    kind: ClusterStatus
kind: ConfigMap
metadata:
  creationTimestamp: "2021-10-01T04:15:10Z"
  name: kubeadm-config
  namespace: kube-system
  resourceVersion: "214"
  uid: 57ef5ae9-085c-44c2-8e4c-8733e2f2b4e7
```

登录k8s-master, 获得加入master的命令

```bash
~]# kubeadm token create --print-join-command
kubeadm join 192.168.177.210:6443 --token q96fs4.px2fojztfcr2bxyd --discovery-token-ca-cert-hash sha256:9e9d01d0ef844b387e1453788b5b58fcf74497cbd0ba8e8ffefd762125a380a8 

~]# kubeadm init phase upload-certs --upload-certs
I1001 12:21:20.438453   32513 version.go:254] remote version is much newer: v1.22.2; falling back to: stable-1.21
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
6a76fe7da098717cd2871a0f65ea5bce2a135e0f9208fc4659847b33fb87805d
```

于是，加入master到集群的命令为

```bash
~]# kubeadm join 192.168.177.210:6443 \
--token q96fs4.px2fojztfcr2bxyd --discovery-token-ca-cert-hash sha256:9e9d01d0ef844b387e1453788b5b58fcf74497cbd0ba8e8ffefd762125a380a8 \
--control-plane --certificate-key 6a76fe7da098717cd2871a0f65ea5bce2a135e0f9208fc4659847b33fb87805d
```

可以登陆k8s-master-left和k8s-master-right执行上述指令

然后登陆到ansible，让所有的node节点加入到k8s集群

```bash
~]# ansible node -m shell -a "kubeadm join 192.168.177.210:6443 --token q96fs4.px2fojztfcr2bxyd --discovery-token-ca-cert-hash sha256:9e9d01d0ef844b387e1453788b5b58fcf74497cbd0ba8e8ffefd762125a380a8"
```

如何需要使用rancher，三个master还需要做如下操作

```bash
~]# sed -i 's|- --port=0|#- --port=0|' /etc/kubernetes/manifests/kube-scheduler.yaml
~]# sed -i 's|- --port=0|#- --port=0|' /etc/kubernetes/manifests/kube-controller-manager.yaml

~]# systemctl restart kubelet
```

如果安装calico网络
release will be found on this site . `https://docs.projectcalico.org/releases`

```bash
~]# kubectl apply -f https://docs.projectcalico.org/v3.19/manifests/calico.yaml
```

## 配置nfs-storage

登陆nfs-server，创建nfs共享存储

### 配置nfs-server

```bash
~]# dnf -y install nfs-utils rpcbind
```

配置 nfs，nfs 的默认配置文件在 /etc/exports 文件下，在该文件中添加下面的配置信息：

```
~]# vi /etc/exports
/data  *(rw,sync,no_root_squash)
```

配置说明：

```
/nfs：是共享的数据目录
*：表示任何人都有权限连接，当然也可以是一个网段，一个 IP，也可以是域名
rw：读写的权限
sync：表示文件同时写入硬盘和内存
no_root_squash：当登录 NFS 主机使用共享目录的使用者是 root 时，其权限将被转换成为匿名使用者，通常它的 UID 与 GID，都会变成 nobody 身份
```

启动rpcbind 、nfs

```bash
~]# systemctl enable --now rpcbind
~]# systemctl enable --now nfs-server
```

确认nfs开启

```bash
~]# rpcinfo -p|grep nfs
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
```

创建并查看nfs共享文件夹

```bash
~]# mkdir /data
~]# showmount -e
Export list for nfs-server:
/data *
```

### 配置nfs-client

使用ansible为k8s集群安装nfs-utils

```bash
~]# ansible all -m yum -a "name=nfs-utils state=installed"
```

创建挂载的/data目录

```bash
~]# ansible all -m shell -a "mkdir /data"
```

假设nfs-server的ip地址为 192.168.177.206

抽取k8s-node-10作为实验，在k8s-node-10虚拟机中查看开启的 nfs目录

```bash
~]# showmount -e 192.168.177.206
Export list for 192.168.177.206:
/data *
```

创建data目录，并挂载nfs-server中的/data目录

```bash
~]# mount 192.168.177.206:/data /data
```

测试完成后，卸载挂载

```bash
~]# umount 192.168.177.206:/data
```

登陆到ansible虚拟机，修改集群的/etc/fstab

```bash
~]# ansible all -m shell -a "echo '192.168.177.206:/data /data nfs defaults 0 0' >> /etc/fstab"
```

先尝试重启k8s-node-10，观测是否可以正常启动，如果可以，则重启k8s集群所有虚拟机

```bash
~]# ansible all -m shell -a "reboot"
```

等待启动完成后，使用如下命令检测挂载成功

```bash
~]# ansible all -m shell -a "df -h | grep data" 
```

## 安装k8s-console

### 选择rancher为console

登陆k8s-console

安装docker

```bash
~]# wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_docker.sh | sh
```

启动rancher

```bash
~]# docker run -d --restart=unless-stopped \
  --privileged \
  -p 80:80 -p 443:443 \
  -v /opt/rancher:/var/lib/rancher/ \
  rancher/rancher:stable
```

添加集群即可
