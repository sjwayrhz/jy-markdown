# 使用 kubeadm 创建 集群

[TOC]

## 安装 kubeadm

### disable swap

```
~]# sed -ri 's/.*swap.*/#&/' /etc/fstab  
```

it is better to restart.

### Letting iptables see bridged traffic

```bash
~]# cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
~]# sudo sysctl --system
```

### Installing runtime

一般情况下，k8s采用docker的运行环境，可以通过daocloud急速安装docker环境哦，安装docker的方法如下：

```bash
~]#  wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_docker.sh | sh
```

### Installing kubeadm, kubelet and kubectl

import repo files

```csharp

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

Set SELinux in permissive mode (effectively disabling it)

```bash
~]# sudo setenforce 0
~]# sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```
Install kubelet kubeadm kubectl
```bash
latest
~]# sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
指定版本
~]# sudo yum install -y kubeadm-1.21.2 kubectl-1.21.2 kubelet-1.21.2 --disableexcludes=kubernetes
~]# sudo systemctl enable --now kubelet
```

确定主机名在/etc/hosts中

```
[root@k8s-master ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.230.8.60 k8s-master
```

注意：必须确认kubelet处于启动状态，否则执行kubeadm init会出错。

```bash
~]# sudo systemctl status kubelet
```

如果不需要自己搬运，可以尝试使用如下参数

```bash
~]# kubeadm init \
  --apiserver-advertise-address=192.168.177.60 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.21.2 \
  --service-cidr=10.7.0.0/16 \
  --pod-network-cidr=10.3.0.0/16
```

如果缺少了某个镜像，可以去 `hub.docker.com`中寻找，例如缺少`registry.aliyuncs.com/google_containers/coredns:v1.8.0`会产生如下报错

```bash
~]# kubeadm init   --apiserver-advertise-address=10.230.7.10   --image-repository registry.aliyuncs.com/google_containers   --kubernetes-version v1.21.2   --service-cidr=10.4.0.0/24   --pod-network-cidr=10.6.0.0/24
[init] Using Kubernetes version: v1.21.2
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR ImagePull]: failed to pull image registry.aliyuncs.com/google_containers/coredns:v1.8.0: output: Error response from daemon: manifest for registry.aliyuncs.com/google_containers/coredns:v1.8.0 not found: manifest unknown: manifest unknown
, error: exit status 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

在`hub.docker.com`中找到 `coredns:1.8.0`

```bash
~]# docker pull coredns/coredns:1.8.0
~]# docker tag 296 registry.aliyuncs.com/google_containers/coredns:v1.8.0
```

或者pull自己的taoistmonk中images收集的coredns

```bash
~]# docker pull registry.cn-shanghai.aliyuncs.com/taoistmonk/images:coredns-v1.8.4 
~]# docker tag 8d1 registry.aliyuncs.com/google_containers/coredns:v1.8.4
```

接下来继续执行kubeadm命令即可

如果失败可以回退

```
kubeadm reset
```

## kubeadm安装k8s_1.22.1失败解决方法

首先根据之前的经验，安装方法为

```
kubeadm init \
  --apiserver-advertise-address=192.168.177.20 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.22.1 \
  --service-cidr=10.8.0.0/16 \
  --pod-network-cidr=10.2.0.0/16
```

但是出现如下的错误

```
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.

        Unfortunately, an error has occurred:
                timed out waiting for the condition

        This error is likely caused by:
                - The kubelet is not running
                - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

        If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
                - 'systemctl status kubelet'
                - 'journalctl -xeu kubelet'

        Additionally, a control plane component may have crashed or exited when started by the container runtime.
        To troubleshoot, list all containers using your preferred container runtimes CLI.

        Here is one example how you may list all Kubernetes containers running in docker:
                - 'docker ps -a | grep kube | grep -v pause'
                Once you have found the failing container, you can inspect its logs with:
                - 'docker logs CONTAINERID'

error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
To see the stack trace of this error execute with --v=5 or higher
```

于是，希望不通过阿里云的镜像安装，而是通过将阿里云的镜像转为官方镜像，官方需要的镜像为：

```bash
~]# kubeadm config images list
k8s.gcr.io/kube-apiserver:v1.22.1
k8s.gcr.io/kube-controller-manager:v1.22.1
k8s.gcr.io/kube-scheduler:v1.22.1
k8s.gcr.io/kube-proxy:v1.22.1
k8s.gcr.io/pause:3.5
k8s.gcr.io/etcd:3.5.0-0
k8s.gcr.io/coredns/coredns:v1.8.4
```

根据阿里云仓库下载的镜像为：

```bash
~]# docker images
REPOSITORY                                                        TAG       IMAGE ID       CREATED        SIZE
registry.aliyuncs.com/google_containers/kube-apiserver            v1.22.1   f30469a2491a   4 days ago     128MB
registry.aliyuncs.com/google_containers/kube-controller-manager   v1.22.1   6e002eb89a88   4 days ago     122MB
registry.aliyuncs.com/google_containers/kube-scheduler            v1.22.1   aca5ededae9c   4 days ago     52.7MB
registry.aliyuncs.com/google_containers/kube-proxy                v1.22.1   36c4ebbc9d97   4 days ago     104MB
registry.aliyuncs.com/google_containers/etcd                      3.5.0-0   004811815584   2 months ago   295MB
registry.aliyuncs.com/google_containers/coredns                   v1.8.4    8d147537fb7d   2 months ago   47.6MB
registry.aliyuncs.com/google_containers/pause                     3.5       ed210e3e4a5b   5 months ago   683kB
```

修改的命令为

```
docker tag f30 k8s.gcr.io/kube-apiserver:v1.22.1
docker tag 6e0 k8s.gcr.io/kube-controller-manager:v1.22.1
docker tag aca k8s.gcr.io/kube-scheduler:v1.22.1
docker tag 36c k8s.gcr.io/kube-proxy:v1.22.1
docker tag 004 k8s.gcr.io/etcd:3.5.0-0
docker tag 8d1 k8s.gcr.io/coredns/coredns:v1.8.4
docker tag ed2 k8s.gcr.io/pause:3.5
```

修改之后，安装k8s的命令为

```bash
~]# kubeadm init --kubernetes-version=v1.22.1 --pod-network-cidr=10.8.0.0/16 --service-cidr=10.2.0.0/16
```

### Creating a single control-plane cluster with kubeadm



成功安装会输出如下内容

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.86.48.227:6443 --token 8rku3h.mxyph2vxb0lvjqn0 \
    --discovery-token-ca-cert-hash sha256:3d5403156f5be5e96b098a26857c43a6fb39ad010d330f7053ac078e0d2a72fb 
```

普通用户可以按照上面的三条命令管控k8s，如果是root用户的话，可以使用如下命令

```bash
~]# export KUBECONFIG=/etc/kubernetes/admin.conf
```

当然，也可以执行如下命令，永久加入kubectl的配置文件到环境变量

```bash
~]# mkdir -p $HOME/.kube
~]# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
~]# chown $(id -u):$(id -g) $HOME/.kube/config
```

### Installing a Pod network add-on

如果安装calico网络
release will be found on this site . `https://docs.projectcalico.org/releases`

```bash
~]# kubectl apply -f https://docs.projectcalico.org/v3.19/manifests/calico.yaml
```

如果安装flannel网络

```bash
~]# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```



### Control plane node isolation

By default, your cluster will not schedule Pods on the control-plane node for security reasons. If you want to be able to schedule Pods on the control-plane node, for example for a single-machine Kubernetes cluster for development, run:

```bash
~]# kubectl taint nodes --all node-role.kubernetes.io/master-
```

With output looking something like:

```
node "test-01" untainted
taint "node-role.kubernetes.io/master:" not found
taint "node-role.kubernetes.io/master:" not found
```

This will remove the `node-role.kubernetes.io/master` taint from any nodes that have it, including the control-plane node, meaning that the scheduler will then be able to schedule Pods everywhere.

## 安装helm

下载helm的tar包，并解压到环境变量文件夹

```bash
~]# wget https://get.helm.sh/helm-v3.6.1-linux-amd64.tar.gz
~]# tar -zxvf helm-v3.6.1-linux-amd64.tar.gz
~]# mv linux-amd64/helm /usr/local/bin/helm

~]# rm -rf helm-v3.6.1-linux-amd64.tar.gz linux-amd64/
```

确认helm安装成功

```bash
~]# helm version
version.BuildInfo{Version:"v3.6.1", GitCommit:"61d8e8c4a6f95540c15c6a65f36a6dd0a45e7a2f", GitTreeState:"clean", GoVersion:"go1.16.5"}
```





## 增删nodes节点

### 添加master

从master-02节点获得添加命令

```bash
~]# kubeadm token create --print-join-command
kubeadm join 192.168.177.62:6443 --token phk5xp.8mrxl7lhr8sic87a --discovery-token-ca-cert-hash sha256:3cfc7ee96916ca08a8338bb67b36bc849d2e001e763cf10c40bdbfefb8350a38

~]# kubeadm init phase upload-certs --upload-certs
W0701 08:24:58.612527   16598 version.go:102] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get "https://storage.googleapis.com/kubernetes-release/release/stable-1.txt": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
W0701 08:24:58.612618   16598 version.go:103] falling back to the local client version: v1.21.2
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
9a00e629b37bc9bce1254bb108035451d15e9cf1b8c15b254c194238743cad7e
```

前往master-01,master-03节点执行指令

```
kubeadm join 192.168.177.62:6443 --token phk5xp.8mrxl7lhr8sic87a \
  --discovery-token-ca-cert-hash sha256:3cfc7ee96916ca08a8338bb67b36bc849d2e001e763cf10c40bdbfefb8350a38 \
  --control-plane --certificate-key 9a00e629b37bc9bce1254bb108035451d15e9cf1b8c15b254c194238743cad7e
```

### 添加node

从master节点获得添加命令

```bash
~]# kubeadm token create --print-join-command
kubeadm join 192.168.177.61:6443 --token us3alf.6k9czgakn8bzvjlr --discovery-token-ca-cert-hash sha256:caef6cf676c481e6c3269f8d5d257052ae1330036fee964114d3b4594bf0faf3
```

前往node节点执行指令

```
kubeadm join 192.168.177.61:6443 --token us3alf.6k9czgakn8bzvjlr \
  --discovery-token-ca-cert-hash sha256:caef6cf676c481e6c3269f8d5d257052ae1330036fee964114d3b4594bf0faf3 \
```

新添加的node如果发现状态为 NotReady，可以尝试重启kubelet

```
 systemctl restart kubelet
```

安装完成之后发现 scheduler和controller-manager为unhealthy

```
~]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused
controller-manager   Unhealthy   Get "http://127.0.0.1:10252/healthz": dial tcp 127.0.0.1:10252: connect: connection refused
etcd-0               Healthy     {"health":"true"}
```

查看清单文件

```
ls /etc/kubernetes/manifests/
etcd.yaml kube-apiserver.yaml kube-controller-manager.yaml kube-scheduler.yaml
```

需要删除 kube-controller-manager.yaml 和  kube-scheduler.yaml 中的 ‘--port=0’这一行

```bash
~]# sed -i 's/^[^#].*--port=0*/#&/g' /etc/kubernetes/manifests/kube-scheduler.yaml
~]# sed -i 's/^[^#].*--port=0*/#&/g' /etc/kubernetes/manifests/kube-controller-manager.yaml
```

再重启kubelet服务

```
systemctl restart kubelet
```

给node命名roles为worker

```
~]# kubectl label node k8s-node-01 node-role.kubernetes.io/worker=worker
```

### 添加master

假设原先有k8s-master作为主节点，现在需要添加两个master分别为 k8s-master-left 和k8s-master-right

在k8s-master的kubeadm-config中添加 `controlPlaneEndpoint: ${ip}:${port}`

```
controlPlaneEndpoint: 10.230.7.20:6443
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
    controlPlaneEndpoint: 10.230.7.20:6443
    controllerManager: {}
    dns:
      type: CoreDNS
    etcd:
      local:
        dataDir: /var/lib/etcd
    imageRepository: registry.aliyuncs.com/google_containers
    kind: ClusterConfiguration
    kubernetesVersion: v1.21.4
    networking:
      dnsDomain: cluster.local
      podSubnet: 10.2.0.0/16
      serviceSubnet: 10.8.0.0/16
    scheduler: {}
  ClusterStatus: |
    apiEndpoints:
      k8s-master:
        advertiseAddress: 10.230.7.20
        bindPort: 6443
    apiVersion: kubeadm.k8s.io/v1beta2
    kind: ClusterStatus
kind: ConfigMap
metadata:
  creationTimestamp: "2021-08-24T23:24:35Z"
  name: kubeadm-config
  namespace: kube-system
  resourceVersion: "207"
  uid: 333d1199-2bad-4bef-bac1-80b512f6e1fc
```

登录k8s-master, 获得加入master的命令

```bash
~]# kubeadm token create --print-join-command
kubeadm join 10.230.7.20:6443 --token jksr2p.33e1qdh1uca8rlag --discovery-token-ca-cert-hash sha256:ad7a32886d6f58339ebb7b5e3b4cc1c12bc6e319e2b2b10dcaec78b9bf27c10e 

~]# kubeadm init phase upload-certs --upload-certs
I0829 07:16:29.077778 1023310 version.go:254] remote version is much newer: v1.22.1; falling back to: stable-1.21
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
54ab65724ca5fae6173a40d2d90df456f8356fd2da8c22252ac47933856bf2bb
```

如果是在国内网络，可能需要pull阿里云coredns镜像

```bash
~]# docker pull coredns/coredns:1.8.0
~]# docker tag 296 registry.aliyuncs.com/google_containers/coredns:v1.8.0
```

于是，加入master到集群的命令为

```bash
~]# kubeadm join 10.230.7.20:6443 \
--token nwl9bh.tueegchv2du3t3dv --discovery-token-ca-cert-hash sha256:ad7a32886d6f58339ebb7b5e3b4cc1c12bc6e319e2b2b10dcaec78b9bf27c10e \
--control-plane --certificate-key 54ab65724ca5fae6173a40d2d90df456f8356fd2da8c22252ac47933856bf2bb
```

如果需要导入rancher，还需要做以下操作

```bash
~]# sed -i 's|- --port=0|#- --port=0|' /etc/kubernetes/manifests/kube-scheduler.yaml
~]# sed -i 's|- --port=0|#- --port=0|' /etc/kubernetes/manifests/kube-controller-manager.yaml

~]# systemctl restart kubelet
```

参考文档

```
https://www.reddit.com/r/kubernetes/comments/kwj1jx/adding_additional_master_nodes_to_an_existing/
https://blog.csdn.net/u012586326/article/details/117742500
```

### 删除node

删除一个节点前，先驱赶掉上面的pod

```
kubectl drain k8s-node-03 --delete-local-data --force --ignore-daemonsets
```

此时节点上面的pod开始迁移,查看节点状态

```
kubectl get nodes
```

最后删除节点

```
kubectl delete node k8s-node-03
```



## 参考链接

### 安装kubeadm

```
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
```

### 使用kubeadm创建集群

```
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
```
