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

### 下载github代码

```bash
~]# git clone https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner.git
~]# cd nfs-subdir-external-provisioner
```

### 选择命名空间

```bash
# Set the subject of the RBAC objects to the current namespace where the provisioner is being deployed
nfs-subdir-external-provisioner]# NAMESPACE=k
nfs-subdir-external-provisioner]# sed -i'' "s/namespace:.*/namespace: $NAMESPACE/g" ./deploy/rbac.yaml ./deploy/deployment.yaml
nfs-subdir-external-provisioner]# kubectl create -f deploy/rbac.yaml
```

### 修改deployment的yaml文件

需要修改两处变量 <YOUR NFS SERVER HOSTNAME>和 <YOUR NFS SERVER SHARED DIR>

需要修改image:镜像地址为`registry.cn-shanghai.aliyuncs.com/taoistmonk/images:nfs-subdir-external-provisioner-v4.0.2`

`vim ./deploy/deployment.yaml`

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nfs-client-provisioner
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: registry.cn-shanghai.aliyuncs.com/taoistmonk/images:nfs-subdir-external-provisioner-v4.0.2
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: k8s-sigs.io/nfs-subdir-external-provisioner
            - name: NFS_SERVER
              value: <YOUR NFS SERVER HOSTNAME>
            - name: NFS_PATH
              value: <YOUR NFS SERVER SHARED DIR>
      volumes:
        - name: nfs-client-root
          nfs:
            server: <YOUR NFS SERVER HOSTNAME>
            path: <YOUR NFS SERVER SHARED DIR>
```

然后部署

```bash
nfs-subdir-external-provisioner]# kubectl apply -f ./deploy/deployment.yaml  
```

### 创建StoreageClass

`vim ./deploy/class.yaml`

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: k8s-sigs.io/nfs-subdir-external-provisioner 
parameters:
  pathPattern: "${.PVC.namespace}" 
  onDelete: delete
```

然后部署

```bash
nfs-subdir-external-provisioner]# kubectl apply -f ./deploy/class.yaml  
```

### 使用busybox来检测

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: busybox-pvc
spec:
  storageClassName: nfs-client
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1M
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-nfs
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox-nfs
  template:
    metadata:
      labels:
        name: busybox-nfs
    spec:
      containers:
      - image: busybox
        command:
          - sh
          - -c
          - 'while true; do date > /mnt/index.html; hostname >> /mnt/index.html; sleep 10m; done'
        imagePullPolicy: IfNotPresent
        name: busybox
        volumeMounts:
          - name: nfs
            mountPath: "/mnt"
      volumes:
      - name: nfs
        persistentVolumeClaim:
          claimName: busybox-pvc
```



