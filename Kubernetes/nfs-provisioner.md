# nfs-client-provisioner

[TOC]

参考项目链接

```
https://github.com/helm/charts/tree/master/stable/nfs-client-provisioner
```



## 搭建NFS-server

安装配置 nfs

```bash
~]# yum -y install nfs-utils rpcbind
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
    100003    3   udp   2049  nfs
    100003    4   udp   2049  nfs
    100227    3   udp   2049  nfs_acl
```

查看nfs共享文件夹

```bash
~]# showmount -e
Export list for localhost.localdomain:
/data *
```

## 配置nfs-client

切换登陆到nfs-client虚拟机

安装showmount

```bash
~]# yum install -y nfs-utils
```

假设nfs-server的ip地址为 10.230.7.21

在nfs-client虚拟机中查看开启的 nfs目录

```bash
~]# showmount -e 10.230.7.21
Export list for 10.230.7.21:
/data *
```

创建data目录，并挂载nfs-server中的/data目录

```bash
~]# mkdir /data
~]# mount 10.230.7.21:/data /data

~]# df -h | grep data
10.230.7.21:/data           1.0T  7.2G 1017G   1% /data
```

测试完成后，卸载挂载

```bash
~]# umount 10.230.7.21:/data
```



## 创建nfs-client-provisioner

首先需要在每个k8s的node节点上都需要挂在nfs目录，现在以k8s-node-01为例

```bash
~]# mkdir /data
~]# yum install -y showmount
~]# vi /etc/fstab
10.230.7.21:/data        /data                    nfs     defaults              0 0

~]# reboot
```

重启检查挂载生效

使用ansible脚本如下：

```bash
~]# ansible all -m shell -a " echo '192.168.177.69:/data        /data                    nfs     defaults              0 0' >> /etc/fstab"
```

github上的语法为

```bash
$ helm install --set nfs.server=x.x.x.x --set nfs.path=/exported/path stable/nfs-client-provisioner 
```

本次实验样例

```bash
国际源
~]# helm repo add stable https://kubernetes-charts.storage.googleapis.com
国内源
~]# helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
本次采用
~]# helm repo add stable https://charts.helm.sh/stable

检测repo
~]# helm search repo nfs-client-provisioner
NAME                         	CHART VERSION	APP VERSION	DESCRIPTION
stable/nfs-client-provisioner	1.2.11       	3.1.0      	DEPRECATED - nfs-client is an automatic provisi...

创建nfs-provisioner
~]# helm install --set nfs.server=10.230.7.21 --set nfs.path=/data stable/nfs-client-provisioner --namespace kube-system --generate-name
ARNING: This chart is deprecated
NAME: nfs-client-provisioner-1624345828
LAST DEPLOYED: Tue Jun 22 15:12:43 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

卸载nfs-provisioner
~]# helm list
NAME                             	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART                        APP VERSION
nfs-client-provisioner-1617246242	default  	1       	2021-04-01 11:04:04.517570448 +0800 CST	deployed	nfs-client-provisioner-1.2.113.1.0
~]# helm uninstall nfs-client-provisioner-1617246242
release "nfs-client-provisioner-1617246242" uninstalled
```

查询helm中的chart表

```bash
~]# helm search repo nfs-client-provisioner
NAME                         	CHART VERSION	APP VERSION	DESCRIPTION                                       
stable/nfs-client-provisioner	1.2.8        	3.1.0      	nfs-client is an automatic provisioner that use...
```

下载，然后更改对应的values.yaml：

```bash
~]# helm pull stable/nfs-client-provisioner
~]# tar xf nfs-client-provisioner-1.2.8.tgz
备份原有的values.yaml
~]# cp values.yaml{,.ori}
```

修改values.yaml，只列出了修改的部分：

```
image:
  repository: docker.io/sjwayrhz/nfs-client-provisioner

nfs:
  server: 10.230.7.21
  path: /data

resources:
  limits:
   cpu: 100m
   memory: 128Mi
  requests:
   cpu: 100m
   memory: 128Mi
```

查看渲染后的yaml：

```bash
cd nfs-client-provisioner
nfs-client-provisioner]# helm -n kube-system install nfs-client-provisioner ./ --dry-run --debug
```

安装

```bash
nfs-client-provisioner]# helm -n kube-system install nfs-client-provisioner ./
```

检查状态

```bash
~]# helm -n kube-system ls
```

查看日志发现报错

```
kubectl -n kube-system logs nfs-client-provisioner-7559966458-clhpr
.......
E0401 05:56:59.756595       1 controller.go:1004] provision "default/test-claim" class "nfs-client": unexpected error getting claim reference: selfLink was empty, can't make reference
```

这是由于k8s 1.20.0版本的api-server不支持,可以修改

Current workaround is to edit /etc/kubernetes/manifests/kube-apiserver.yaml

```bash
~]# vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

Under here:

```
spec:
  containers:
  - command:
    - kube-apiserver
```

Add this line:

```
    - --feature-gates=RemoveSelfLink=false
```

The do this:
kubectl apply -f /etc/kubernetes/manifests/kube-apiserver.yaml
kubectl apply -f /etc/kubernetes/manifests/kube-apiserver.yaml

检查nfs-provisioner日志是否有报错

```
 ~]# kubectl -n kube-system logs nfs-client-provisioner-7559966458-clhpr
```

如果没有报错，查看storageclass，名称是 nfs-client ，后续需要用到

```bash
~]# kubectl get sc
NAME         PROVISIONER                            RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client   cluster.local/nfs-client-provisioner   Delete          Immediate           true                   61m
```

