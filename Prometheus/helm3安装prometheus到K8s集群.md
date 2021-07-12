#### 1. 安装 helm3

```
# helm3 下载地址
  https://github.com/helm/helm/releases
  
# 下载后解压获取二进制文件即可使用

# 添加 bitnami chart仓库
~]# helm repo add bitnami https://charts.bitnami.com/bitnami

# 更新仓库
~]# helm repo update
```

#### 2. 安装 Prometheus

首先确定有monit的namespace，如果没有，需要创建 `kubectl create namespace monit`

```
# 安装 prometheus
~]# helm install prometheus bitnami/prometheus-operator \
--namespace=monit \
--set prometheus.service.type=NodePort \
--set prometheus.service.nodePort=30090

# 访问 prometheus
curl http://localhost:30090
```

#### 3. 安装 Grafana

```
~]# helm install grafana bitnami/grafana \
--namespace=monit \
--set persistence.enabled=false \
--set service.type=NodePort \
--set service.nodePort=30080

# 获取 grafana 用户名密码 , -n monit指的是monit的命名空间
~]# echo "User: admin" && echo "Password: $(kubectl get secret grafana-admin -n monit -o jsonpath="{.data.GF_SECURITY_ADMIN_PASSWORD}" | base64 --decode)"

# 访问 grafana
~]# curl http://localhost:30080
```

Grafana  数据源填入

prometheus-operated:9090

dashboard模版选择13105

#### 卸载Prometheus和Grafana

```
]# helm ls
NAME   	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
grafana	default  	1       	2021-01-29 08:17:20.803098162 +0000 UTC	deployed	grafana-5.1.0	7.3.7

]# helm uninstall grafana
```

