由于使用的是国内源，所以查看kubeadm所需的包，是k8s.gcr.io的，这是无法通过kubeadm下载的。

```bash
~]# kubeadm config images list
k8s.gcr.io/kube-apiserver:v1.20.5
k8s.gcr.io/kube-controller-manager:v1.20.5
k8s.gcr.io/kube-scheduler:v1.20.5
k8s.gcr.io/kube-proxy:v1.20.5
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.13-0
k8s.gcr.io/coredns:1.7.0
```

生成配置文件

```
kubeadm config print init-defaults > kubeadm.conf
```

修改kubeadm.conf

```
vi kubeadm.conf
# 修改 imageRepository: k8s.gcr.io
# 改为 registry.cn-shanghai.aliyuncs.com/sjhz
imageRepository: registry.cn-shanghai.aliyuncs.com/sjhz
# 修改kubernetes版本kubernetesVersion: v1.20.0
# 改为kubernetesVersion: v1.20.0
kubernetesVersion: v1.20.0
```

再次查看kubeadm config所需的镜像

```
kubeadm config images list --config kubeadm.conf
registry.cn-shanghai.aliyuncs.com/sjhz/kube-apiserver:v1.20.0
registry.cn-shanghai.aliyuncs.com/sjhz/kube-controller-manager:v1.20.0
registry.cn-shanghai.aliyuncs.com/sjhz/kube-scheduler:v1.20.0
registry.cn-shanghai.aliyuncs.com/sjhz/kube-proxy:v1.20.0
registry.cn-shanghai.aliyuncs.com/sjhz/pause:3.2
registry.cn-shanghai.aliyuncs.com/sjhz/etcd:3.4.13-0
registry.cn-shanghai.aliyuncs.com/sjhz/coredns:1.7.0
```

拉取镜像并初始化

```
kubeadm config images pull --config kubeadm.conf
kubeadm init --config kubeadm.conf
```

如果不需要自己搬运，可以尝试使用如下参数

```
kubeadm init \
  --apiserver-advertise-address=10.230.8.60 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.20.0 \
  --service-cidr=10.4.0.0/24 \
  --pod-network-cidr=10.6.0.0/24
```

如果失败可以回退

```
kubeadm resets
```