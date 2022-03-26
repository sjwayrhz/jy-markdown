# 同步容器镜像

[TOC]



## 步骤

参考文档如下

```
https://github.com/AliyunContainerService/image-syncer/blob/master/README-zh_CN.md
```

购买一台境外的服务器，然后下载image-syncer的二进制文件，放入/opt/目录下，即可。该二进制文件是可执行文件。

确认境外服务器中含有docker相关密钥，新建 /opt/image-sync目录，并拷贝docker密钥到此目录,制作auth.json。

账号

```
quay.io 		-> 	sjwayrhz / Vwv56ty7
docker.io		->  sjwayrhz / 1qaz2wsx#
gchr.io			->	sjwayrhz / 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
registry.cn-shanghai.aliyuncs.com -> taoistmonk@163.com	/ Vwv56ty7
```

### image-syncer

```bash
$ wget -P /tmp https://github.com/AliyunContainerService/image-syncer/releases/download/v1.3.1/image-syncer-v1.3.1-linux-amd64.tar.gz
$ tar -zxvf /tmp/image-syncer-v1.3.1-linux-amd64.tar.gz -C /tmp
$ mv /tmp/image-syncer /opt
```

### auth.yaml

```bash
tee /opt/auth.yaml <<- 'EOF'
docker.io:
  username: sjwayrhz
  password: 1qaz2wsx#
quay.io:
  username: sjwayrhz
  password: Vwv56ty7
  insecure: true
gchr.io:
  username: sjwayrhz
  password: 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
registry.cn-shanghai.aliyuncs.com:
  username: taoistmonk@163.com
  password: Vwv56ty7
EOF
```

### image.yaml

在/opt/目录下创建image.yaml文件，单个版本的镜像

```bash
tee /opt/images.yaml <<- 'EOF'
k8s.gcr.io/ingress-nginx/controller:v0.34.0: docker.io/sjwayrhz/controller
EOF
```

全版本镜像

```bash
tee /opt/images.yaml <<- 'EOF'
k8s.gcr.io/ingress-nginx/controller: docker.io/sjwayrhz/controller
EOF
```

将谷歌镜像同步到阿里云，范例操作如下，其中，namespace需要视情况而定：

```bash
nohup /opt/image-syncer --proc=6 --retries=3 --auth=/opt/auth.yaml --images=/opt/images.yaml &
```



## 同步请求举例

### Prometheus

```yaml
quay.io/prometheus-operator/prometheus-operator: docker.io/sjwayrhz/prometheus-operator
quay.io/prometheus/alertmanager: docker.io/sjwayrhz/alertmanager
quay.io/prometheus/node-exporter: docker.io/sjwayrhz/node-exporter
quay.io/prometheus/prometheus: docker.io/sjwayrhz/prometheus
quay.io/prometheus/blackbox-exporter: docker.io/sjwayrhz/blackbox-exporter
quay.io/brancz/kube-rbac-proxy: docker.io/sjwayrhz/kube-rbac-proxy
jimmidyson/configmap-reload: docker.io/sjwayrhz/configmap-reload
grafana/grafana: docker.io/sjwayrhz/grafana
k8s.gcr.io/kube-state-metrics/kube-state-metrics: docker.io/sjwayrhz/kube-state-metrics
k8s.gcr.io/prometheus-adapter/prometheus-adapter: docker.io/sjwayrhz/prometheus-adapter
quay.io/prometheus-operator/prometheus-config-reloader: docker.io/sjwayrhz/prometheus-config-reloader
```

### ingress-nginx

```yaml
k8s.gcr.io/ingress-nginx/controller: docker.io/sjwayrhz/controller
k8s.gcr.io/ingress-nginx/kube-webhook-certgen: docker.io/sjwayrhz/kube-webhook-certgen
```

### kubernetes

```yaml
k8s.gcr.io/kube-apiserver: docker.io/sjwayrhz/kube-apiserver
k8s.gcr.io/kube-controller-manager: docker.io/sjwayrhz/kube-controller-manager
k8s.gcr.io/kube-scheduler: docker.io/sjwayrhz/kube-scheduler
k8s.gcr.io/kube-proxy: docker.io/sjwayrhz/kube-proxy
k8s.gcr.io/pause: docker.io/sjwayrhz/pause
k8s.gcr.io/etcd: docker.io/sjwayrhz/etcd
k8s.gcr.io/coredns/coredns: docker.io/sjwayrhz/coredns
```





