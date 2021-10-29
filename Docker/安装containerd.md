### 下载containerd

containerd在gihub中的镜像地址
```
https://hub.fastgit.org/containerd/containerd
```
wget 下载并解压containerd到/tmp目录
```
$ wget -P /tmp https://download.fastgit.org/containerd/containerd/releases/download/v1.5.7/cri-containerd-cni-1.5.7-linux-amd64.tar.gz

$ tar -zxvf /tmp/cri-containerd-cni-1.5.7-linux-amd64.tar.gz -C /
```

### 启动containerd

```
$ systemctl enable --now containerd
```
### 查看版本
```
$ ctr version
```
### 创建默认配置文件
```
$ mkdir /etc/containerd
$ containerd config default > /etc/containerd/config.yoml
```

### 镜像加速
镜像加速的配置就在 cri 插件配置块下面的 registry 配置块，所以需要修改的部分如下：
```
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://dockerhub.mirrors.nwafu.edu.cn"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."k8s.gcr.io"]
          endpoint = ["https://registry.aliyuncs.com/k8sxio"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."gcr.io"]
          endpoint = ["xxx"]
```
- registry.mirrors."xxx" : 表示需要配置 mirror 的镜像仓库。例如，registry.mirrors."docker.io" 表示配置 docker.io 的 mirror。

- endpoint : 表示提供 mirror 的镜像加速服务。例如，这里推荐使用西北农林科技大学提供的镜像加速服务作为 docker.io 的 mirror。

### 存储配置
Containerd 有两个不同的存储路径，一个用来保存持久化数据，一个用来保存运行时状态。
```
root = "/var/lib/containerd"
state = "/run/containerd"
```
root用来保存持久化数据，包括 Snapshots, Content, Metadata 以及各种插件的数据。每一个插件都有自己单独的目录，Containerd 本身不存储任何数据，它的所有功能都来自于已加载的插件，真是太机智了。
```
$ tree -L 2 /var/lib/containerd/
$ tree -L 2 /run/containerd/
```
