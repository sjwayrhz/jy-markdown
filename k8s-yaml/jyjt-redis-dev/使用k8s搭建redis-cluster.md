# redis-cluser 三主三从

[TOC]

## 集群规划

| hostname                                   | port | data                      | master or slave |
| ------------------------------------------ | ---- | ------------------------- | --------------- |
| redis-0.redis.app-office.svc.cluster.local | 6379 | /data/app-office/cluster0 | master1         |
| redis-1.redis.app-office.svc.cluster.local | 6379 | /data/app-office/cluster1 | master2         |
| redis-2.redis.app-office.svc.cluster.local | 6379 | /data/app-office/cluster2 | master3         |
| redis-3.redis.app-office.svc.cluster.local | 6379 | /data/app-office/cluster3 | slaveof master1 |
| redis-4.redis.app-office.svc.cluster.local | 6379 | /data/app-office/cluster4 | slaveof master2 |
| redis-5.redis.app-office.svc.cluster.local | 6379 | /data/app-office/cluster5 | slaveof master3 |



## 创建nfs共享文件夹

拥有一个k8s集群，并且已经做好nfs文件共享。

```bash
每个节点都安装nfs-utils

~]# yum -y install nfs-utils

共享路径为 /data
```

为app-office中的6个节点创建集群内共享文件夹，并授予777权限

```bash
[root@ghost-nfs-server ~]# mkdir -p /data/app-office/cluster0
[root@ghost-nfs-server ~]# mkdir -p /data/app-office/cluster1
[root@ghost-nfs-server ~]# mkdir -p /data/app-office/cluster2
[root@ghost-nfs-server ~]# mkdir -p /data/app-office/cluster3
[root@ghost-nfs-server ~]# mkdir -p /data/app-office/cluster4
[root@ghost-nfs-server ~]# mkdir -p /data/app-office/cluster5

[root@ghost-nfs-server ~]# chmod -R 777 /data/app-office
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
~]# kubectl get svc,pod,pvc -n app-office
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
~]# kubectl run -it rockylinux --image=rockylinux/rockylinux:8.4 -n app-office bash
```

在centos镜像里面安装redis

```bash
[root@rockylinux /]# dnf install -y bind-utils redis
```

### 创建cluster-master

使用dig +short 可以获取pod的ip地址，也就是使用域名添加redis节点。

首先创建cluster的master 为 redis-0,redis-1,redis-2

```bash
[root@rockylinux /]# /usr/bin/redis-cli --cluster create \
  `dig +short redis-0.redis.app-office.svc.cluster.local`:6379 \
  `dig +short redis-1.redis.app-office.svc.cluster.local`:6379 \
  `dig +short redis-2.redis.app-office.svc.cluster.local`:6379
```

日志如下

```
>>> Performing hash slots allocation on 3 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
M: 9aaa3073d82732628a3940e8cb89f80ba8acb8d2 10.2.44.253:6379
   slots:[0-5460] (5461 slots) master
M: d6f16b1bb015eaf69f98d3c506dfdf6e6ec92823 10.2.154.228:6379
   slots:[5461-10922] (5462 slots) master
M: 1d7920f8d2de8a1f249c12feb041c4b6c8836aec 10.2.127.196:6379
   slots:[10923-16383] (5461 slots) master
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 10.2.44.253:6379)
M: 9aaa3073d82732628a3940e8cb89f80ba8acb8d2 10.2.44.253:6379
   slots:[0-5460] (5461 slots) master
M: 1d7920f8d2de8a1f249c12feb041c4b6c8836aec 10.2.127.196:6379
   slots:[10923-16383] (5461 slots) master
M: d6f16b1bb015eaf69f98d3c506dfdf6e6ec92823 10.2.154.228:6379
   slots:[5461-10922] (5462 slots) master
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
[root@rockylinux /]# /usr/bin/redis-cli  --cluster add-node `dig +short redis-3.redis.app-office.svc.cluster.local`:6379 `dig +short redis-0.redis.app-office.svc.cluster.local`:6379 --cluster-slave --cluster-master-id 9aaa3073d82732628a3940e8cb89f80ba8acb8d2
[root@rockylinux /]# /usr/bin/redis-cli  --cluster add-node `dig +short redis-4.redis.app-office.svc.cluster.local`:6379 `dig +short redis-1.redis.app-office.svc.cluster.local`:6379 --cluster-slave --cluster-master-id 1d7920f8d2de8a1f249c12feb041c4b6c8836aec
[root@rockylinux /]# /usr/bin/redis-cli  --cluster add-node `dig +short redis-5.redis.app-office.svc.cluster.local`:6379 `dig +short redis-2.redis.app-office.svc.cluster.local`:6379 --cluster-slave --cluster-master-id d6f16b1bb015eaf69f98d3c506dfdf6e6ec92823
```

### 查找cluster id

继续在临时centos中，做以下操作，验证cluster nodes 和cluster info

```bash
~]# kubectl exec -it rockylinux -n app-office -- sh
[root@rockylinux /]# /usr/bin/redis-cli -h `dig +short redis-0.redis.app-office.svc.cluster.local` -p 6379
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
~]# kubectl delete po centos -n app-office
```


## 高可用实现

由于k8s本身对容器的运行已经具备较强的可用性保障，可以保障容器一直按照我们期望的状态或实例数运行，在这里我们就不再阐述容器自身存活的可用性保障。

理解k8s的朋友应该已经发现，我们使用的是podip而非服务名，k8s中statefulset是可以保障每个pod的服务访问地址唯一的，但由于redis自身对域名解析不好的原因，无法填写域名，故pod出现各类异常问题的情况，k8s将对容器重新调度，重新调度的容器大概率会更换ip地址，此时可能会对集群的高可用造成影响。

### 非全体节点故障

redis的nodes.conf配置文件，维护了redis集群的状态信息，其中重点维护了redis自身节点的id和redis集群的ip信息，在redis自身实例启动过程中，将读取该文件，本实例的nodes.conf文件以持久化的方式存储在了/var/lib中。

redis实例在启动过程中，会根据nodes.conf信息寻找集群地址，只要满足一个节点可以被寻址到，并匹配成功相关的id信息，此redis实例就将加入到该集群中，这种机制极大满足了在kubernetes场景中的ip更换问题。

因为在nodes.conf中维护了所有node节点的状态，即便大多数节点ip变更出现节点丢失，但在集群加入的过程中， 只要保障有一个节点和原本节点相同，即可保障所有节点的加入。

基于这种 理论，我们可以模拟容器故障的场景，测试下非全体节点故障的集群可用性。

首先我们查看所有节点的容器ip地址：

```
[root@kubemaster redis-5.0.5-cluster]# kubectl get pod -owide --namespace=app-office |awk '{print $6}' |grep -v IP
10.244.2.245
10.244.0.187
10.244.1.49
10.244.1.50
10.244.2.246
10.244.0.188
```

记录这些ip地址后，我们手动删除5个pod，仅保留ip为10.244.0.188的pod:redis-app-5

```
kubectl delete pod -n app-office redis-app-0
kubectl delete pod -n app-office redis-app-1
kubectl delete pod -n app-office redis-app-2
kubectl delete pod -n app-office redis-app-3
kubectl delete pod -n app-office redis-app-4
```

全部删除成功后查看下现在的容器状态：

```
[root@kubemaster redis-5.0.5-cluster]# kubectl get pod -owide -n app-office
NAME          READY     STATUS    RESTARTS   AGE       IP             NODE
redis-app-0   1/1       Running   0          1m        10.244.2.247   slave2
redis-app-1   1/1       Running   0          1m        10.244.0.190   kubemaster
redis-app-2   1/1       Running   0          57s       10.244.1.67    slave1
redis-app-3   1/1       Running   0          47s       10.244.1.68    slave1
redis-app-4   1/1       Running   0          36s       10.244.2.248   slave2
redis-app-5   1/1       Running   0          1h        10.244.0.188   kubemaster
```

处理发现除了redis-app-5 之外，所有的podip都已经改变

此时我们在进入容器内查看集群的状态:

```
127.0.0.1:6379> CLUSTER INFO
cluster_state:ok
127.0.0.1:6379> CLUSTER NODES
cda0c5b72a8cde9e93e01089c7092a9a710431c5 10.244.0.190:6379@16379 slave a66c30807d2857cfd0d50e4c10587744045aa0be 0 1567154253529 8 connected
a66c30807d2857cfd0d50e4c10587744045aa0be 10.244.0.188:6379@16379 master - 0 1567154254030 8 connected 5461-10922
c05c483527bdbcaf7636e275dc8976e02dc41619 10.244.1.67:6379@16379 slave bfb98a4d55c77b86686c8900ffa105cde7020c13 0 1567154254000 9 connected
bfb98a4d55c77b86686c8900ffa105cde7020c13 10.244.1.68:6379@16379 master - 0 1567154254000 9 connected 10923-16383
c0fcd18205a3455f9877d3c36ea0a53e47091619 10.244.2.248:6379@16379 slave 8434b3117c9e3304c51a7f1b463a8b5a3a6dcfaa 0 1567154254530 11 connected
8434b3117c9e3304c51a7f1b463a8b5a3a6dcfaa 10.244.2.245:6379@16379 myself,master - 0 1567154254000 11 connected 0-5460
```

发现集群状态为监控，所有节点都已经加入的集群中，同事发现nodes.conf配置文件已经被修改为新的ip地址

```
root@redis-app-0:/data# cat /var/lib/redis/nodes.conf
cda0c5b72a8cde9e93e01089c7092a9a710431c5 10.244.0.190:6379@16379 slave a66c30807d2857cfd0d50e4c10587744045aa0be 0 1567154110000 8 connected
a66c30807d2857cfd0d50e4c10587744045aa0be 10.244.0.188:6379@16379 master - 0 1567154110516 8 connected 5461-10922
c05c483527bdbcaf7636e275dc8976e02dc41619 10.244.1.67:6379@16379 slave bfb98a4d55c77b86686c8900ffa105cde7020c13 0 1567154111000 9 connected
bfb98a4d55c77b86686c8900ffa105cde7020c13 10.244.1.68:6379@16379 master - 0 1567154111121 9 connected 10923-16383
c0fcd18205a3455f9877d3c36ea0a53e47091619 10.244.2.248:6379@16379 slave 8434b3117c9e3304c51a7f1b463a8b5a3a6dcfaa 0 1567154111236 11 connected
8434b3117c9e3304c51a7f1b463a8b5a3a6dcfaa 10.244.2.245:6379@16379 myself,master - 0 1567154111000 11 connected 0-5460
```

### 全体节点故障

当全体节点故障的情况下，任何一个pod不可以对集群进行寻址，此时集群将不能自动恢复，需要手动告诉集群的所有节点，集群的位置在哪里，redis-cli命令可以实现此需求，

在实例启动之后，可以选取任意节点为集群的发现节点，此节点充当上面自动恢复集群过程的中心节点，如上面使用的redis-app-5 节点。

模拟全节点故障

删除所有pod

```
[root@kubemaster ~]# kubectl delete pod --namespace=app-office $(kubectl get pod --namespace=app-office | awk '{print $1}'|grep -v NAME)
```

等待容器全部恢复后，进入到任一一台节点

```
root@redis-app-0:/data# redis-cli
127.0.0.1:6379> CLUSTER INFO
cluster_state:fail
```

可以发现集群已经出现故障:

依次发现其他节点，加入的redis集群

```
127.0.0.1:6379> CLUSTER MEET $IP 6379
```

全部加入后，查看集群状态

```
127.0.0.1:6379> CLUSTER INFO
cluster_state:ok
```

所以，若想将redis集群在k8s环境下实现高可用的话，在整个机房或中心发生故障后，需要手动或编写相关的脚本执行完成集群加入的操作即可。

### 增加节点:

增加节点非常简单，首先我们将statefulset副本调整为7，addons中的yaml文件原本为6.

调整好后，查看pod状态:

```
[root@kubemaster redis-5.0.5-cluster]# kubectl get pod -owide -n app-office
NAME          READY     STATUS    RESTARTS   AGE       IP             NODE
redis-app-0   1/1       Running   0          13m       10.244.0.191   kubemaster
redis-app-1   1/1       Running   0          13m       10.244.2.8     slave2
redis-app-2   1/1       Running   0          12m       10.244.1.69    slave1
redis-app-3   1/1       Running   0          12m       10.244.0.192   kubemaster
redis-app-4   1/1       Running   0          12m       10.244.1.70    slave1
redis-app-5   1/1       Running   0          12m       10.244.2.9     slave2
redis-app-6   1/1       Running   0          3m        10.244.2.10    slave2
```

此时我们加入新增的pod容器中执行加入集群指令

```
127.0.0.1:6379> CLUSTER MEET 10.244.0.191 6379
```

加入成功后，我们可以查看集群状态信息观察是否加入:

```
127.0.0.1:6379> CLUSTER INFO
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:7
cluster_size:3
cluster_current_epoch:11
cluster_my_epoch:0
cluster_stats_messages_ping_sent:451
cluster_stats_messages_pong_sent:463
cluster_stats_messages_meet_sent:6
cluster_stats_messages_sent:920
cluster_stats_messages_ping_received:463
cluster_stats_messages_pong_received:457
cluster_stats_messages_received:920
```

### 清理环境

```
kubectl delete -f addons/.
kubectl delete pvc -n app-office $(kubectl get pvc -n app-office |awk '{print $1}' |grep -v NAME)
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
    path: /data/app-office/cluster0

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
    path: /data/app-office/cluster1

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
    path: /data/app-office/cluster2

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
    path: /data/app-office/cluster3

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
    path: /data/app-office/cluster4

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
    path: /data/app-office/cluster5
```

### namespace-app-office.yaml

```
apiVersion: v1
kind: Namespace
metadata:
  name: app-office
```

### redis.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: app-office
  labels:
    app: redis
spec:
  selector:
    app: redis
    appCluster: app-office
  ports:
  - name: redis
    port: 6379
  clusterIP: None

---
apiVersion: v1
kind: Service
metadata:
  name: redis-access
  namespace: app-office
  labels:
    app: redis
spec:
  selector:
    app: redis
    appCluster: app-office
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
  namespace: app-office
spec:
  serviceName: redis
  replicas: 6
  selector:
    matchLabels:
      app: redis
      appCluster: app-office
  template:
    metadata:
      labels:
        app: redis
        appCluster: app-office
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
      namespace: app-office
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
  namespace: app-office
data:
  redis.conf: |
    appendonly yes
    cluster-enabled yes
    cluster-config-file /var/lib/redis/nodes.conf
    cluster-node-timeout 5000
    dir /var/lib/redis
    port 6379
```

