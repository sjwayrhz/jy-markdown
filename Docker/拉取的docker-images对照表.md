### Prometheus-Operator

来源于云效仓库

```
git@codeup.aliyun.com:60e566f6f00dfa2fa4cabdd6/prometheus-operator.git
```

仓库的tag 为 20210826 适用于安装prometheus

| old                                                     | new                                                          |
| :------------------------------------------------------ | :----------------------------------------------------------- |
| quay.io/prometheus/alertmanager:v0.22.2                 | registry.cn-shanghai.aliyuncs.com/taoistmonk/images:alertmanager-v0.22.2 |
| quay.io/prometheus/blackbox-exporter:v0.19.0            | registry.cn-shanghai.aliyuncs.com/taoistmonk/images:blackbox-exporter-v0.19.0 |
| grafana/grafana:8.1.2                                   | registry.cn-shanghai.aliyuncs.com/taoistmonk/images:grafana-8.1.2 |
| k8s.gcr.io/kube-state-metrics/kube-state-metrics:v2.2.0 | registry.cn-shanghai.aliyuncs.com/taoistmonk/images:kube-state-metrics-v2.2.0 |
| quay.io/prometheus/node-exporter:v1.2.2                 | registry.cn-shanghai.aliyuncs.com/taoistmonk/images:node-exporter-v1.2.2 |
| k8s.gcr.io/prometheus-adapter/prometheus-adapter:v0.9.0 | registry.cn-shanghai.aliyuncs.com/taoistmonk/images:prometheus-adapter-v0.9.0 |
| quay.io/prometheus/prometheus:v2.29.1                   | registry.cn-shanghai.aliyuncs.com/taoistmonk/images:prometheus-v2.29.1 |
| quay.io/prometheus-operator/prometheus-operator:v0.50.0 | registry.cn-shanghai.aliyuncs.com/taoistmonk/images:prometheus-operator-v0.50.0 |



nfs provisioner

| old                                                          | new                                                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2 | registry.cn-shanghai.aliyuncs.com/taoistmonk/images:nfs-subdir-external-provisioner-v4.0.2 |

