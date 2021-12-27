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

在没有loadbalance的环境条件下，无法做出此配置

### 确认外部访问权限

由于私有云服务器不支持loadbalance  所以只能使用nodeport方式访问应用

查询访问地址如下：

```bash
$ kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                      AGE
istio-ingressgateway   LoadBalancer   10.233.43.203   <pending>     15021:31639/TCP,80:31037/TCP,443:31905/TCP,31400:30263/TCP,15443:31319/TCP   2d22h
```

访问bookinfo的路径应该为

```
http://$GATEWAY_URL/productpage
```

本例的访问路径为

```
http://10.225.63.20:31037/productpage
```

## 查看dashboard

### Install `Kiali and the other addons `and wait for them to be deployed.

```bash
$ kubectl apply -f /usr/local/istio-1.12.1/samples/addons
$ kubectl rollout status deployment/kiali -n istio-system
Waiting for deployment "kiali" rollout to finish: 0 of 1 updated replicas are available...
deployment "kiali" successfully rolled out
```

### Access the Kiali dashboard.

由于原生的kiali是clusterIp的方式，所以无法直接在集群外部浏览器访问

```bash
$ kubectl get svc -n istio-system | grep kiali
kiali                  ClusterIP      10.233.0.190    <none>        20001/TCP,9090/TCP                                               9m
```

现在增加nodeport方式，yaml如下：

```
apiVersion: v1
kind: Service
metadata:
  name: kiali-nodeport
  namespace: istio-system
  labels:
    app: kiali
spec:
  type: NodePort
  ports:
  - name: http
    port: 20001
    targetPort: 20001
    nodePort: 30021
  - name: http-metrics
    port: 9090
    targetPort: 9090
    nodePort: 30023
  selector:
    app: kiali
```

再次查看svc

```bash
$ kubectl get svc -n istio-system | grep kiali
kiali                  ClusterIP      10.233.0.190    <none>        20001/TCP,9090/TCP                                                           18m
kiali-nodeport         NodePort       10.233.29.23    <none>        20001:30021/TCP,9090:30023/TCP                                               9s
```

在局域网其它主机可以通过浏览器访问kiali，路径为：

```
http://10.225.63.20:30021
```

### In the left navigation menu, select *Graph* and in the *Namespace* drop down, select *default*.

查看流量的官方教程 和本次案例如下：

```bash
$ for i in $(seq 1 100); do curl -s -o /dev/null "http://$GATEWAY_URL/productpage"; done

$ for i in $(seq 1 100); do curl -s -o /dev/null "http://10.225.63.20:31037/productpage"; done
```

The Kiali dashboard shows an overview of your mesh with the relationships between the services in the `Bookinfo` sample application. It also provides filters to visualize the traffic flow.