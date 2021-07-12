安装 prometheus-community 源

```bash
~]# helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

下载charts表到本地

```bash
~]# helm pull prometheus-community/prometheus
```

解压并修改chart表到数值

```bash
~]# tar zxvf prometheus-13.7.0.tgz
```

查看需要pull的镜像与标签

```bash
~]# cat prometheus/values.yaml | grep repository
    repository: quay.io/prometheus/alertmanager
      repository: jimmidyson/configmap-reload
      repository: jimmidyson/configmap-reload
    repository: quay.io/prometheus/node-exporter
    repository: quay.io/prometheus/prometheus
    repository: prom/pushgateway
~]# cat prometheus/values.yaml | grep tag
    tag: v0.21.0
      tag: v0.5.0
      tag: v0.5.0
    tag: v1.0.1
    tag: v2.26.0
    tag: v1.3.1
    
    
 发现还有一个
 k8s.gcr.io/kube-state-metrics/kube-state-metrics:v1.9.8
```

登陆境外服务器，同步墙外镜像

```bash
~]# docker pull quay.io/prometheus/alertmanager:v0.21.0

~]# docker tag quay.io/prometheus/alertmanager:v0.21.0 sjwayrhz/image:alertmanager-v0.21.0

~]# docker push sjwayrhz/image:alertmanager-v0.21.0
```

依次内推，可以拉取以下image

```
sjwayrhz/image:alertmanager-v0.21.0
sjwayrhz/image:configmap-reload-v0.5.0
sjwayrhz/image:node-exporter-v1.0.1
sjwayrhz/image:prometheus-v2.26.0
sjwayrhz/image:pushgateway-v1.3.1

sjwayrhz/kube-state-metrics-v1.9.8
```

helm安装prometheus

```bash
~]# helm install prometheus prometheus/
NAME: prometheus-1618216412
LAST DEPLOYED: Mon Apr 12 16:33:34 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The Prometheus server can be accessed via port 80 on the following DNS name from within your cluster:
prometheus-1618216412-server.default.svc.cluster.local


Get the Prometheus server URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9090


The Prometheus alertmanager can be accessed via port 80 on the following DNS name from within your cluster:
prometheus-1618216412-alertmanager.default.svc.cluster.local


Get the Alertmanager URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=alertmanager" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9093
#################################################################################
######   WARNING: Pod Security Policy has been moved to a global property.  #####
######            use .Values.podSecurityPolicy.enabled with pod-based      #####
######            annotations                                               #####
######            (e.g. .Values.nodeExporter.podSecurityPolicy.annotations) #####
#################################################################################


The Prometheus PushGateway can be accessed via port 9091 on the following DNS name from within your cluster:
prometheus-1618216412-pushgateway.default.svc.cluster.local


Get the PushGateway URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=pushgateway" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9091

For more information on running Prometheus, visit:
https://prometheus.io/
```

