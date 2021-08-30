kubeadm初始化时，产生如下警告

```
[init] Using Kubernetes version: v1.18.2
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
```



[官方文档](https://kubernetes.io/zh/docs/setup/production-environment/container-runtimes/)表示，更改设置，令容器运行时和kubelet使用systemd作为cgroup驱动，以此使系统更为稳定。 请注意在docker下设置native.cgroupdriver=systemd选项。

两种解决方式：

一、编辑docker配置文件/etc/docker/daemon.json

```
vi /etc/docker/daemon.json
 
"exec-opts": ["native.cgroupdriver=systemd"]

systemctl daemon-reload
systemctl restart docker
```

二、编辑/usr/lib/systemd/system/docker.service

```
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd

systemctl daemon-reload
systemctl restart docker
```

设置完成后通过docker info命令可以看到Cgroup Driver为systemd

```
docker info | grep Cgroup
```

