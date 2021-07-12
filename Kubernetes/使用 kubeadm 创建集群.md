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

```
~]# curl -sSL https://get.daocloud.io/docker | sh
~]# systemctl enable --now docker 
```

### Installing kubeadm, kubelet and kubectl

import repo files

```bash
~]# cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```
或者导入国内源

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
~]# sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
~]# sudo systemctl enable --now kubelet
```

### Creating a single control-plane cluster with kubeadm

使用kubeadm init 就可以安装k8s了

```bash
~]# kubeadm init --apiserver-advertise-address=<ip-address> --service-cidr=172.10.4.0/16 --pod-network-cidr=172.10.6.0/16 --service-dns-domain=sjhz.tk
```

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

```bash
~]# kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
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
~]# wget https://get.helm.sh/helm-v3.2.4-linux-amd64.tar.gz
~]# tar -zxvf helm-v3.2.4-linux-amd64.tar.gz
~]# mv linux-amd64/helm /usr/local/bin/helm

~]# rm -rf helm-v3.2.4-linux-amd64.tar.gz linux-amd64/
```

确认helm安装成功

```bash
~]# helm version
version.BuildInfo{Version:"v3.2.4", GitCommit:"0ad800ef43d3b826f31a5ad8dfbb4fe05d143688", GitTreeState:"clean", GoVersion:"go1.13.12"}
```





## 添加nodes节点

The nodes are where your workloads (containers and Pods, etc) run. To add new nodes to your cluster do the following for each machine:

- SSH to the machine
- Become root (e.g. `sudo su -`)
- Run the command that was output by `kubeadm init`. For example:

```bash
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```

If you do not have the token, you can get it by running the following command on the control-plane node:

```bash
kubeadm token list
```

The output is similar to this:

```console
TOKEN                    TTL  EXPIRES              USAGES           DESCRIPTION            EXTRA GROUPS
8ewj1p.9r9hcjoqgajrj4gi  23h  2018-06-12T02:51:28Z authentication,  The default bootstrap  system:
                                                   signing          token generated by     bootstrappers:
                                                                    'kubeadm init'.        kubeadm:
                                                                                           default-node-token
```

By default, tokens expire after 24 hours. If you are joining a node to the cluster after the current token has expired, you can create a new token by running the following command on the control-plane node:

```bash
kubeadm token create
```

The output is similar to this:

```console
5didvk.d09sbcov8ph2amjw
```

If you don't have the value of `--discovery-token-ca-cert-hash`, you can get it by running the following command chain on the control-plane node:

```bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
```

The output is similar to:

```console
8cb2de97839780a412b93877f8507ad6c94f73add17d5d7058e91741c9d5ec78
```

> **Note:** To specify an IPv6 tuple for `<control-plane-host>:<control-plane-port>`, IPv6 address must be enclosed in square brackets, for example: `[fd00::101]:2073`.

The output should look something like:

```
[preflight] Running pre-flight checks

... (log output of join workflow) ...

Node join complete:
* Certificate signing request sent to control-plane and response
  received.
* Kubelet informed of new secure connection details.

Run 'kubectl get nodes' on control-plane to see this machine join.
```

A few seconds later, you should notice this node in the output from `kubectl get nodes` when run on the control-plane node

## 参考链接

### 安装kubeadm

```
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
```

### 使用kubeadm创建集群

```
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
```

## 国内方法

参考链接

```
https://www.jianshu.com/p/d42ef0eff63f
```

### 查看kubeadm config所需的镜像

```
$ kubeadm config images list

k8s.gcr.io/kube-apiserver:v1.13.1
k8s.gcr.io/kube-controller-manager:v1.13.1
k8s.gcr.io/kube-scheduler:v1.13.1
k8s.gcr.io/kube-proxy:v1.13.1
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.2.24
k8s.gcr.io/coredns:1.2.6
```

### 第一种：中转

首先从[Kubernetes国内Docker镜像]拉取镜像，然后修改镜像的tag

```
sudo docker tag registry.cn-beijing.aliyuncs.com/imcto/kube-controller-manager:v1.13.1 k8s.gcr.io/kube-controller-manager:v1.13.1
sudo docker tag registry.cn-beijing.aliyuncs.com/imcto/kube-apiserver:v1.13.1 k8s.gcr.io/kube-apiserver:v1.13.1
sudo docker tag registry.cn-beijing.aliyuncs.com/imcto/kube-proxy:v1.13.1 k8s.gcr.io/kube-proxy:v1.13.1
sudo docker tag registry.cn-beijing.aliyuncs.com/imcto/kube-scheduler:v1.13.1 k8s.gcr.io/kube-scheduler:v1.13.1
sudo docker tag registry.cn-beijing.aliyuncs.com/imcto/etcd:3.2.24 k8s.gcr.io/etcd:3.2.24
sudo docker tag registry.cn-beijing.aliyuncs.com/imcto/pause:3.1 k8s.gcr.io/pause:3.1
sudo docker tag registry.cn-beijing.aliyuncs.com/imcto/coredns:1.2.6 k8s.gcr.io/coredns:1.2.6
```



### 第二种：修改配置

使用kubeadm配置文件，通过在配置文件中指定docker仓库地址，便于内网快速部署。

生成配置文件

```
kubeadm config print init-defaults ClusterConfiguration > kubeadm.conf
```

修改kubeadm.conf

```
vi kubeadm.conf
# 修改 imageRepository: k8s.gcr.io
# 改为 registry.cn-beijing.aliyuncs.com/imcto
imageRepository: registry.cn-beijing.aliyuncs.com/imcto
# 修改kubernetes版本kubernetesVersion: v1.13.0
# 改为kubernetesVersion: v1.13.1
kubernetesVersion: v1.13.1
```

再次查看kubeadm config所需的镜像

```
$ kubeadm config images list --config kubeadm.conf
registry.cn-beijing.aliyuncs.com/imcto/kube-apiserver:v1.13.1
registry.cn-beijing.aliyuncs.com/imcto/kube-controller-manager:v1.13.1
registry.cn-beijing.aliyuncs.com/imcto/kube-scheduler:v1.13.1
registry.cn-beijing.aliyuncs.com/imcto/kube-proxy:v1.13.1
registry.cn-beijing.aliyuncs.com/imcto/pause:3.1
registry.cn-beijing.aliyuncs.com/imcto/etcd:3.2.24
registry.cn-beijing.aliyuncs.com/imcto/coredns:1.2.6
```

拉取镜像并初始化

```
kubeadm config images pull --config kubeadm.conf
kubeadm init --config kubeadm.conf
```

更多kubeadm配置文件参数详见

```
kubeadm config print init-defaults
```