# rook-ceph for storageclass

[TOC]

创建一个k8s集群用于研究rook-ceph

## 资源准备

一主三从的k8s集群

| 主机名      | 系统盘           | 附加磁盘         | IP地址         |
| ----------- | ---------------- | ---------------- | -------------- |
| ceph-master | /dev/vda: 30 GiB | 无               | 192.168.177.26 |
| ceph-node-1 | /dev/vda: 50 GiB | /dev/vdb: 30 GiB | 192.168.177.80 |
| ceph-node-2 | /dev/vda: 50 GiB | /dev/vdb: 30 GiB | 192.168.177.55 |
| ceph-node-3 | /dev/vda: 50 GiB | /dev/vdb: 30 GiB | 192.168.177.14 |

> 注意：附加磁盘，都必须是空磁盘，并且不能格式化。

## 部署步骤

### 拉取代码

克隆代码

```bash
$ git clone git@gitee.com:sjwayrhz/devops.git
```

进入rook-ceph目录

```bash
$ cd devops/rook/ceph-1.7
```

### 部署rook-ceph

部署如下应用：

```bash
$ kubectl create -f crds.yaml -f common.yaml -f operator.yaml
$ kubectl create -f cluster.yaml
```

查询rook-ceph状态

```bash
$ kubectl -n rook-ceph get pod
```

### Ceph Dashboard

创建nodeport访问

```bash
$  kubectl create -f dashboard-external-https.yaml
```

查看端口

```bash
$ kubectl -n rook-ceph get svc 
```

运行 Rook 的命名空间中生成了一个名为 `rook-ceph-dashboard-admin-password` 的 Secret，要获取密码，可以运行以下命令：

```bash
$ kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo
xjDadabefO
```

### 创建storageclass

执行如下命令创建

```bash
$ kubectl apply -f devops/rook/ceph-1.7/csi/rbd/storageclass.yaml
```

查看现有的storage class 名称

```bash
$ kubectl get sc
NAME              PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com   Delete          Immediate           true                   46m
```

### 使用storageclass

这里用一个busybox样例来使用storageclass

```bash
$ kubectl apply -f devops/rook/busybox-ceph-sc-pvc.yaml
```



## 清理步骤

rook-ceph O版结合k8s部署，部署失败后,一定要彻底清理，否则磁盘osd识别不到硬盘。

### 删除k8s上rook-ceph下的所有资源

按照部署的顺序反着操作。

```bash
$ kubectl delete -f cluster.yaml
$ kubectl delete -f operator.yaml
$ kubectl delete -f common.yaml
$ kubectl delete -f crds.yaml
```

（其实后面的common和crds可以不删，为了稳妥一点还是删了吧）
删资源的过程中，可能会出现命名空间rook-ceph卡在Terminating状态，网上能找的通过curl或者--grace-period强删，如果结果不了，直接进etcd里删除。
etcdctl del /registry/namespaces/rook-ceph

### 清理所有osd节点的磁盘

#检查硬盘路径
fdisk -l

#删除硬盘分区信息
DISK="/dev/sdb"

sgdisk --zap-all $DISK

#清理硬盘数据（hdd硬盘使用dd，ssd硬盘使用blkdiscard）

```
dd if=/dev/zero of="$DISK" bs=1M count=100 oflag=direct,dsync

blkdiscard $DISK
```

#删除原osd的lvm信息（如果单个节点有多个osd，那么就不能用*拼配模糊删除，而根据lsblk -f查询出明确的lv映射信息再具体删除，参照第5项操作）

```
ls /dev/mapper/ceph-* | xargs -I% -- dmsetup remove %

rm -rf /dev/ceph-*
```

操作完成之后重启服务器，再次安装rook-ceph基本没什么问题。
