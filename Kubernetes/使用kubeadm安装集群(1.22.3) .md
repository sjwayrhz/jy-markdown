# 实战安装k8s集群

[TOC]

## 资源准备

### 虚拟机准备

准备的资源信息如下

| IP address  |    Hostname    |  Disk   |  Application  |
| :---------: | :------------: | :-----: | :-----------: |
| 10.230.7.35 | infrastructure |   30G   | nginx rancher |
| 10.230.7.30 |  ceph-master   |   30G   |    Ingress    |
| 10.230.7.31 |  ceph-node-01  | 40G+70G |     ceph      |
| 10.230.7.32 |  ceph-node-02  | 40G+70G |     ceph      |
| 10.230.7.33 |  ceph-node-03  | 40G+70G |     ceph      |

### 局部DNS配置

在infra虚拟机中部署应用dnsmasq

```bash
~]# dnf install -y dnsmasq
```

生成并编辑dnsmasq配置文件

```bash
~]# vim /etc/dnsmasq.d/juneyao.conf
```

添加的主机信息为

```bash
address=/ceph-master/10.230.7.30
address=/ceph-node-01/10.230.7.31
address=/ceph-node-02/10.230.7.32
address=/ceph-node-03/10.230.7.33
```

启动并设置dnsmasq为自启动,检查启动状态

```bash
~]# systemctl enable --now dnsmasq
~]# systemctl status dnsmasq
```

在infra机器中配置局部dns

```bash
~]# sed -i '1inameserver 10.230.7.35' /etc/resolv.conf
```

完成在首行追加后，使用`ping ceph-master`就可以测试是否生效，其他虚拟机也可以添加这一条局部dns。

### Ansible配置

在infra虚拟机，配置免确认ssh

```bash
~]# vim  /etc/ssh/ssh_config
```

将 “#  StrictHostKeyChecking ask” 改为“StrictHostKeyChecking no”

导入私钥到infra服务器`~/.ssh/id_rsa`中

**然后修改ansible主机的 /etc/hosts和/etc/ansible/hosts**

```bash
~]# cat /etc/ansible/hosts
[ceph]
ceph-master
ceph-node-01
ceph-node-02
ceph-node-03
```

使用如下命令检测ansible是否配置成功

```bash
~]# ansible ceph -m ping
```

### 局部域名解析

使用ansible复制/etc/hosts到所有节点

```bash
~]# ansible ceph -m copy -a "src=/etc/resolv.conf dest=/etc/resolv.conf"
```

### 安装containerd

如果不是用k8s模板部署的rocky linux，可以使用如下脚本初始化环境

```shell
~]# ansible all -m shell -a "wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/k8s_init.sh | sh"
```

如果已经是模版rocky linux了，仅需在ansible虚拟机中执行如下命令安装1.22.3版本运行时

```bash
~]# ansible all -m shell -a "wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_containerd.sh | bash -s 1.22.3"
```

## 安装kubernetes

### 安装k8s-master

进入k8s-master初始化k8s-master

```bash
~]# kubeadm init \
  --apiserver-advertise-address=10.230.7.30 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.22.3 \
  --service-cidr=10.6.0.0/16 \
  --pod-network-cidr=10.4.0.0/16
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

kubeadm join 10.230.7.30:6443 --token c3zgmg.hcaicmx4q9m24lmv \
	--discovery-token-ca-cert-hash sha256:fbc433da4e43882e3af046704753c3c37e1e532a9d30fa6515fd8e2558f1a23b
```

### 添加master节点

假设原先有k8s-master作为主节点，现在需要添加两个master分别为 k8s-master-left 和k8s-master-right

在k8s-master的kubeadm-config中添加 `controlPlaneEndpoint: ${ip}:${port}`

```
controlPlaneEndpoint: 10.230.7.30:6443
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
    controlPlaneEndpoint: 10.230.7.30:6443
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
      podSubnet: 10.4.0.0/16
      serviceSubnet: 10.6.0.0/16
    scheduler: {}
  ClusterStatus: |
    apiEndpoints:
      k8s-master:
        advertiseAddress: 10.230.7.30
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
kubeadm join 10.230.7.30:6443 --token q96fs4.px2fojztfcr2bxyd --discovery-token-ca-cert-hash sha256:9e9d01d0ef844b387e1453788b5b58fcf74497cbd0ba8e8ffefd762125a380a8 

~]# kubeadm init phase upload-certs --upload-certs
I1001 12:21:20.438453   32513 version.go:254] remote version is much newer: v1.22.2; falling back to: stable-1.21
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
6a76fe7da098717cd2871a0f65ea5bce2a135e0f9208fc4659847b33fb87805d
```

于是，加入master到集群的命令为

```bash
~]# kubeadm join 10.230.7.30:6443 \
--token q96fs4.px2fojztfcr2bxyd --discovery-token-ca-cert-hash sha256:9e9d01d0ef844b387e1453788b5b58fcf74497cbd0ba8e8ffefd762125a380a8 \
--control-plane --certificate-key 6a76fe7da098717cd2871a0f65ea5bce2a135e0f9208fc4659847b33fb87805d
```

可以登陆k8s-master-left和k8s-master-right执行上述指令

然后登陆到ansible，让所有的node节点加入到k8s集群

```bash
~]# ansible node -m shell -a "kubeadm join 10.230.7.30:6443 --token q96fs4.px2fojztfcr2bxyd --discovery-token-ca-cert-hash sha256:9e9d01d0ef844b387e1453788b5b58fcf74497cbd0ba8e8ffefd762125a380a8"
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
~]# kubectl apply -f https://docs.projectcalico.org/v3.20/manifests/calico.yaml
```

## 配置ceph存储

克隆项目

登陆infrastructure，下载安装git，克隆gitee上的devops项目

```bash
~]# cd /tmp
tmp]# git clone git@gitee.com:sjwayrhz/devops.git
tmp]# cd devops/rook/ceph-1.7
```

部署到k8s集群

```bash
tmp]# kubectl create -f crds.yaml -f common.yaml -f operator.yaml
tmp]# kubectl create -f cluster.yaml
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
