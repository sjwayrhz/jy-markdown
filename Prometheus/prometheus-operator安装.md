# 使用prometheus-operator安装prometheus
[TOC]

参考github链接`https://github.com/prometheus-operator/kube-prometheus`

增加nodeport部署与替换镜像后，上传到codeup链接：

```
git@codeup.aliyun.com:60e566f6f00dfa2fa4cabdd6/configure.git
```

在configure项目中的tag 2021086

快速部署方法：

```bash
~]#git clone git@codeup.aliyun.com:60e566f6f00dfa2fa4cabdd6/configure.git

~]#kubectl apply -f configure/prometheus-operator/manifests/setup/
~]#kubectl apply -f configure/prometheus-operator/manifests/
```

即可部署成功

nodeport开放端口为 

```
prometheus	30001
grafana		30002
altermanage	 30003
```



