# 实战单master安装k8s集群

[TOC]

## 资源准备

### 虚拟机准备

准备的资源信息如下

|  IP address  |       Hostname       |  Disk   | Application |
| :----------: | :------------------: | :-----: | :---------: |
| 10.225.63.10 | openstack-controller |  100G   |   ansible   |
| 10.225.63.27 |     cb21-master      |   50G   |   Ingress   |
| 10.225.63.28 |    cb21-worker-01    | 50G+70G |    demo     |
| 10.225.63.29 |    cb21-worker-02    | 50G+70G |    demo     |
| 10.225.63.30 |    cb21-worker-03    | 50G+70G |    demo     |

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
address=/cb21-master/10.225.63.27
address=/cb21-worker-01/10.225.63.28
address=/cb21-worker-02/10.225.63.29
address=/cb21-worker-03/10.225.63.30
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

完成在首行追加后，使用`ping demo-master`就可以测试是否生效，其他虚拟机也可以添加这一条局部dns。

### Ansible配置

在openstack-controller虚拟机，配置免确认ssh

```bash
$ vim  /etc/ssh/ssh_config
```

将 “#  StrictHostKeyChecking ask” 改为“StrictHostKeyChecking no”

导入私钥到infra服务器`~/.ssh/id_rsa`中

**然后修改ansible主机的 /etc/hosts和/etc/ansible/hosts**

```bash
$ cat /etc/ansible/hosts
[cb21]
cb21-master
cb21-worker-01
cb21-worker-02
cb21-worker-03
```

使用如下命令检测ansible是否配置成功

```bash
$ ansible cb21 -m ping
```

### 局部域名解析

使用ansible复制/etc/hosts到所有节点

```bash
$ ansible cb21 -m copy -a "src=/etc/resolv.conf dest=/etc/resolv.conf"
```

### 安装containerd

需要rocky linux安装wget ，更换过国内源并且`.ssh/authorized_keys`中储存好公钥，然后可以使用如下脚本初始化环境

```shell
$ ansible cb21 -m shell -a 'sh -c "$(curl -fsSL https://gitee.com/sjwayrhz/one_key_install/raw/master/rocky_linux_8.5_init.sh)"'
```

初始化之后，或许需要重启linux系统，但是vmware中的镜像系统，只需执行下面的一步：

如果已经是模版rocky linux了，仅需在ansible虚拟机中执行如下命令安装1.21.7版本运行时

```bash
$ ansible cb21 -m shell -a "modprobe br_netfilter"
$ ansible cb21 -m shell -a "echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables"

$ ansible cb21 -m shell -a "wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_containerd_1.5.sh | bash -s 1.21.10"
```

## 安装kubernetes

### 安装k8s-master

使用kubeadm之前，可以提前导入能生成10年有效期证书的kubeadm文件,蓝奏云分享链接和wget url如下：

```
https://www.lanzouv.com/igiqFxjqq8h

$ wget http://121.46.238.135:9000/k8s-tools/kubeadm-1.21/kubeadm
```

进入k8s-master初始化k8s-master

```bash
$ kubeadm init \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.21.10 \
  --service-cidr=10.6.0.0/16 \
  --pod-network-cidr=10.4.0.0/16 \
  --apiserver-advertise-address=10.225.63.27 \
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

kubeadm join 10.225.63.27:6443 --token uda4zh.lvbekkprj1v22eyd \
	--discovery-token-ca-cert-hash sha256:577828dc647a412f94c52dc6405061352f71b9c392540da8ae22ae2b3de0108f
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

### 添加worker节点

于是，加入master到集群的命令为

然后登陆worker节点，让所有的worker节点（woker-01,woker-02,worker-03）加入到k8s集群

```bash
$ kubeadm join 10.225.63.27:6443 --token uda4zh.lvbekkprj1v22eyd --discovery-token-ca-cert-hash sha256:577828dc647a412f94c52dc6405061352f71b9c392540da8ae22ae2b3de0108f
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

登陆openstack-controller，下载安装git，克隆gitee上的devops项目

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
