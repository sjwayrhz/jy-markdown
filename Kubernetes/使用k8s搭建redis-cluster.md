# redis-cluser 三主三从

[TOC]

## 创建nfs共享文件夹

拥有一个k8s集群，并且已经做好nfs文件共享。

```bash
每个节点都安装nfs-utils

~]# yum -y install nfs-utils

共享路径为 /data
```

为redis-cluster中的6个节点创建集群内共享文件夹，并授予777权限

```bash
[root@ghost-nfs-server ~]# mkdir -p /data/redis-cluster/cluster0
[root@ghost-nfs-server ~]# mkdir -p /data/redis-cluster/cluster1
[root@ghost-nfs-server ~]# mkdir -p /data/redis-cluster/cluster2
[root@ghost-nfs-server ~]# mkdir -p /data/redis-cluster/cluster3
[root@ghost-nfs-server ~]# mkdir -p /data/redis-cluster/cluster4
[root@ghost-nfs-server ~]# mkdir -p /data/redis-cluster/cluster5

[root@ghost-nfs-server ~]# chmod -R 777 /data/redis-cluster
```

## 创建pv

只能手动创建pv，不能使用pvc

```bash
~]# kubectl apply -f pv.yaml

~]# kubectl get pv
```

涉及的scripts请见附录

## 部署redis

直接apply本文中的redis.yaml，即可。

```bash
~]# kubectl apply -f redis-configmap.yaml
~]# kubectl apply -f redis.yaml
```

检查

```bash
~]# kubectl get svc,pod,pvc -n redis-cluster
NAME                   TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/redis          ClusterIP   None         <none>        6379/TCP   8m53s
service/redis-access   ClusterIP   10.6.93.95   <none>        6379/TCP   8m53s

NAME          READY   STATUS    RESTARTS   AGE
pod/redis-0   1/1     Running   0          2m17s
pod/redis-1   1/1     Running   0          2m14s
pod/redis-2   1/1     Running   0          119s
pod/redis-3   1/1     Running   0          104s
pod/redis-4   1/1     Running   0          90s
pod/redis-5   1/1     Running   0          76s

NAME                                 STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/data-redis-0   Bound    nfs-pv4   1Gi        RWX                           8m53s
persistentvolumeclaim/data-redis-1   Bound    nfs-pv1   1Gi        RWX                           2m14s
persistentvolumeclaim/data-redis-2   Bound    nfs-pv2   1Gi        RWX                           119s
persistentvolumeclaim/data-redis-3   Bound    nfs-pv0   1Gi        RWX                           104s
persistentvolumeclaim/data-redis-4   Bound    nfs-pv3   1Gi        RWX                           90s
persistentvolumeclaim/data-redis-5   Bound    nfs-pv5   1Gi        RWX                           76s
```

## 初始化redis集群

### 启动一个centos镜像来做设置

```bash
~]# kubectl run -it centos --image=centos:7 --restart=Never -n redis-cluster bash
```

在centos镜像里面安装redis

```bash
[root@centos /]# yum install -y epel-release 
[root@centos /]# yum install -y wget bind-utils
[root@centos /]# wget http://download.redis.io/releases/redis-5.0.13.tar.gz
[root@centos /]# tar xzf redis-5.0.13.tar.gz -C /usr/local/
[root@centos /]# cd /usr/local/redis-5.0.13/
[root@centos redis-5.0.13]# yum install -y gcc make
[root@centos redis-5.0.13]# make && make install
```

### 创建cluster-master

使用dig +short 可以获取pod的ip地址，也就是使用域名添加redis节点。

首先创建cluster的master 为 redis-0,redis-1,redis-2

```bash
[root@centos /]# redis-cli --cluster create \
  `dig +short redis-0.redis.redis-cluster.svc.cluster.local`:6379 \
  `dig +short redis-1.redis.redis-cluster.svc.cluster.local`:6379 \
  `dig +short redis-2.redis.redis-cluster.svc.cluster.local`:6379
```

日志如下

```
>>> Performing hash slots allocation on 3 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
M: 7ffd3d68a40e2bba485862b88161fdb6553184c3 10.4.116.152:6379
   slots:[0-5460] (5461 slots) master
M: 27bbfc2655466912c7d58f6661b8dcbaeca6814a 10.4.177.98:6379
   slots:[5461-10922] (5462 slots) master
M: 9a14a8b5d8365f5b8bd7d5b29cfc4e55b0b30f35 10.4.218.18:6379
   slots:[10923-16383] (5461 slots) master
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 10.4.116.152:6379)
M: 7ffd3d68a40e2bba485862b88161fdb6553184c3 10.4.116.152:6379
   slots:[0-5460] (5461 slots) master
M: 27bbfc2655466912c7d58f6661b8dcbaeca6814a 10.4.177.98:6379
   slots:[5461-10922] (5462 slots) master
M: 9a14a8b5d8365f5b8bd7d5b29cfc4e55b0b30f35 10.4.218.18:6379
   slots:[10923-16383] (5461 slots) master
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

### 添加cluster-slave

redis-0,redis-1,redis-2对应的slave依次为redis-3,redis-4,redis-5

redis-cli  --cluster add-node的命令规则为

**add-node: 后面的分别跟着新加入的slave和slave对应的master**

**cluster-slave：表示加入的是slave节点**

**--cluster-master-id：表示slave对应的master的node ID**

```bash
[root@centos /]# redis-cli  --cluster add-node `dig +short redis-3.redis.redis-cluster.svc.cluster.local`:6379 `dig +short redis-0.redis.redis-cluster.svc.cluster.local`:6379 --cluster-slave --cluster-master-id 7ffd3d68a40e2bba485862b88161fdb6553184c3
[root@centos /]# redis-cli  --cluster add-node `dig +short redis-4.redis.redis-cluster.svc.cluster.local`:6379 `dig +short redis-1.redis.redis-cluster.svc.cluster.local`:6379 --cluster-slave --cluster-master-id 27bbfc2655466912c7d58f6661b8dcbaeca6814a
[root@centos /]# redis-cli  --cluster add-node `dig +short redis-5.redis.redis-cluster.svc.cluster.local`:6379 `dig +short redis-2.redis.redis-cluster.svc.cluster.local`:6379 --cluster-slave --cluster-master-id 9a14a8b5d8365f5b8bd7d5b29cfc4e55b0b30f35
```

### 查找cluster id

继续在临时centos中，做以下操作，验证cluster nodes 和cluster info

```bash
[root@centos /]# redis-cli -h `dig +short redis-0.redis.redis-cluster.svc.cluster.local` -p 6379
```

查看集群节点

```mysql
10.4.116.152:6379> cluster nodes
8e2685f235a499025006ed013e1065bee08e59d9 10.4.148.30:6379@16379 slave 27bbfc2655466912c7d58f6661b8dcbaeca6814a 0 1630452502394 2 connected
9a14a8b5d8365f5b8bd7d5b29cfc4e55b0b30f35 10.4.218.18:6379@16379 master - 0 1630452502595 3 connected 10923-16383
7ffd3d68a40e2bba485862b88161fdb6553184c3 10.4.116.152:6379@16379 myself,master - 0 1630452503000 1 connected 0-5460
27bbfc2655466912c7d58f6661b8dcbaeca6814a 10.4.177.98:6379@16379 master - 0 1630452502000 2 connected 5461-10922
de941b709ba4637220cbd3fbe49f1636c530b5d4 10.4.212.85:6379@16379 slave 7ffd3d68a40e2bba485862b88161fdb6553184c3 0 1630452502896 1 connected
88b80acdb7e9e88f6ba5f931269f4176655244f5 10.4.116.153:6379@16379 slave 9a14a8b5d8365f5b8bd7d5b29cfc4e55b0b30f35 0 1630452502896 3 connected
```

查看集群信息

```mysql
10.4.116.152:6379> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:3
cluster_my_epoch:1
cluster_stats_messages_ping_sent:470
cluster_stats_messages_pong_sent:485
cluster_stats_messages_sent:955
cluster_stats_messages_ping_received:482
cluster_stats_messages_pong_received:470
cluster_stats_messages_meet_received:3
cluster_stats_messages_received:955
```

完毕，推出临时centos并删除该pod

```bash
~]# kubectl delete po centos -n redis-cluster
```



## 附录：scripts

### pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv0
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 192.168.177.33
    path: /data/redis-cluster/cluster0

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 192.168.177.33
    path: /data/redis-cluster/cluster1

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv2
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 192.168.177.33
    path: /data/redis-cluster/cluster2

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv3
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 192.168.177.33
    path: /data/redis-cluster/cluster3

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv4
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 192.168.177.33
    path: /data/redis-cluster/cluster4

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv5
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 192.168.177.33
    path: /data/redis-cluster/cluster5
```

### namespace-redis-cluster.yaml

```
apiVersion: v1
kind: Namespace
metadata:
  name: redis-cluster
```

### redis.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: redis-cluster
  labels:
    app: redis
spec:
  selector:
    app: redis
    appCluster: redis-cluster
  ports:
  - name: redis
    port: 6379
  clusterIP: None

---
apiVersion: v1
kind: Service
metadata:
  name: redis-access
  namespace: redis-cluster
  labels:
    app: redis
spec:
  selector:
    app: redis
    appCluster: redis-cluster
  ports:
  - name: redis-access
    protocol: TCP
    port: 6379
    targetPort: 6379

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
  namespace: redis-cluster
spec:
  serviceName: redis
  replicas: 6
  selector:
    matchLabels:
      app: redis
      appCluster: redis-cluster
  template:
    metadata:
      labels:
        app: redis
        appCluster: redis-cluster
    spec:
      terminationGracePeriodSeconds: 20
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - redis
              topologyKey: kubernetes.io/hostname
      containers:
      - name: redis
        image: redis:5.0.13-alpine
        command:
          - "redis-server"
        args:
          - "/etc/redis/redis.conf"
          - "--protected-mode"
          - "no"
        resources:
          requests:
            cpu: "500m"
            memory: "500Mi"
        ports:
        - containerPort: 6379
          name: redis
          protocol: TCP
        - containerPort: 16379
          name: cluster
          protocol: TCP
        volumeMounts:
        - name: conf
          mountPath: /etc/redis
        - name: data
          mountPath: /var/lib/redis
      volumes:
      - name: conf
        configMap:
          name: redis-conf
          items:
          - key: redis.conf
            path: redis.conf
  volumeClaimTemplates:
  - metadata:
      name: data
      namespace: redis-cluster
    spec:
      accessModes: [ "ReadWriteMany" ]
      resources:
        requests:
          storage: 1Gi
```

### redis-configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-conf
  namespace: redis-cluster
data:
  redis.conf: |
    appendonly yes
    cluster-enabled yes
    cluster-config-file /var/lib/redis/nodes.conf
    cluster-node-timeout 5000
    dir /var/lib/redis
    port 6379
```

