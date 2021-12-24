# Getting Started

[TOC]

## 下载istio

### 查看版本链接

https://github.com/istio/istio/releases

### 依据以下链接下载

```bash
$ wget https://github.com/istio/istio/releases/download/1.12.1/istio-1.12.1-linux-amd64.tar.gz -P /tmp 
```

### 解压并移动istio到/usr/local文件夹下

```bash
$ tar -zxvf /tmp/istio-1.12.1-linux-amd64.tar.gz -C /usr/local
```

### 设置istioctl到环境变量

```bash
$ echo 'export PATH=/usr/local/istio-1.12.1/bin:$PATH' >> /etc/profile
$ cat /etc/profile
$ source /etc/profile

$ istioctl version
no running Istio pods in "istio-system"
1.12.1
```

## 安装isito

### 使用demo测试文件安装

```bash
$ istioctl install --set profile=demo -y
```

### 添加一个命名空间让isito自动注入

这里以default命名空间为例

```bash
$ kubectl label namespace default istio-injection=enabled
```

### 查看被istio标签注入的namespace

```bash
$ kubectl get ns --show-labels | grep istio
default                           Active   9d      field.cattle.io/projectId=p-cmm56,istio-injection=enabled,kubernetes.io/metadata.name=default,kubesphere.io/namespace=default,kubesphere.io/workspace=system-workspace
istio-system                      Active   3m30s   kubernetes.io/metadata.name=istio-system,kubesphere.io/namespace=istio-system
```

## 部署sample application

### 部署Bookinfo样例应用

```bash
$ kubectl apply -f /usr/local/istio-1.12.1/samples/bookinfo/platform/kube/bookinfo.yaml
```

### 发现该应用拥有sidecar

```bash
$ kubectl get services
```

和

```bash
$ kubectl get pods
```

### 检查该应用可以提供 HTML

等待istio部署完毕，检测bookinfo容器是否提供以下html页面

```bash
$ kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```

预期的输出结果是 `<title>Simple Bookstore App</title>`

### 打开应用程序的外部访问权限

打开网关

```bash
$ kubectl apply -f /usr/local/istio-1.12.1/samples/bookinfo/networking/bookinfo-gateway.yaml
```

确保网管打开没有问题

```bash
$ istioctl analyze
```

### 定义ingress的ip和端口

### 确认外部访问权限