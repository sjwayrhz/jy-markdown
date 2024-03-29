# 如何通过 JumpServer 堡垒机管控 Kubernetes 集群

**1. 跳转原理**

Jumpserver是个好东西，特别是对于线上设备的管控，基本跳转原理如下图所示。详细的系统设计，可以参考其文档

```
              http                       ssh
    [user] <---------> [jumpserver] <----------> [remote machine]
```

然而，随着**kubernetes**的普及，越来越多的线上服务采用了**kubernetes**集群部署。如何通过**Jumpserver**原理进行**kubernetes**集群管控就是本文要解决的问题。

**2. K8S跳转**

**kubernetes**的管控原理，和管控远程机器的原理基本类似。只是需要在集群内部部署一个持久的POD, 针对 **Jumpserver** 该POD能够提供 **SSHD** 服务，其次该POD内部应该自带 **kubectl** 工具。

**2.1 简单的网络架构图**

```
              http                       ssh      /------------------------------------------------\
    [user] <---------> [jumpserver] <---------->  |    [kubectl pod] <=> [ kubernetes resource ]   |
                                                  \------------------------------------------------/      
```

**2.2 构建POD的IMAGE**

按以上原理，构建中间跳转POD的IMAGE。具体**Dockerfile**如下:

```
FROM sickp/centos-sshd:latest

#安装kubectl
RUN curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
RUN chmod +x ./kubectl
RUN mv ./kubectl /usr/local/bin/kubectl

#安装helm
ADD helm /usr/local/bin/helm
RUN chmod +x /usr/local/bin/helm

#提供默认的ssh key
RUN usermod -p "!" root
ADD id_rsa.pub /root/.ssh/id_rsa.pub
RUN cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys

# 生成/var/log/lastlog文件
RUN touch /var/log/lastlog
```

按此Dockerfile请提前准备好对应的**ssh key**。并将次IMAGE推送的自己的**Docker Registry**中。

**2.3 在K8S集群中部署**

有了中间POD的IMAGE，部署具体的K8S服务很简单。具体定义文件，参考以下**manifest**定义：

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: jump
  labels:
    app: jump
spec:
  serviceName: jump
  replicas: 1
  selector:
    matchLabels:
      app: jump
  template:
    metadata:
      labels:
        app: jump
    spec:
      imagePullSecrets:
        - name: <YOUR-PULL-SECRET>
      serviceAccountName: jump
      containers:
        - name: jump
          image: <YOUR-POD-IMAGE>
          ports:
          - name: ssh
            containerPort: 22
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jump
imagePullSecrets: 
  - name: <YOUR-PULL-SECRET>
---
apiVersion: v1
kind: Service
metadata:
  name: jump
spec:
  type: LoadBalancer
  selector:
    app: jump
  ports:
    - name: ssh
      port: 22
      targetPort: ssh
      protocol: TCP
```

注意替换相应的集群参数配置。从定义文件中可以看出，中间POD是以**LoadBalancer**的方式对外提供服务的。需要查出具体的外网服务IP。

```
$: kubectl get service | grep jump
jump      LoadBalancer   [内网IP]   [外网IP]   22:30525/TCP     75d
```

该[外网IP]就是**JumpServer**连接的地址了。

**2.4 配置JumpServer**

**JumpServer**中的配置和普通的远程主机配置基本一至，不再赘述了。

```
https://www.ucloud.cn/yun/32997.html
```

