# 部署qfusion
## 标记节点角色

**必须**：为所有可用节点打上 qfusion/node= 标签标记，用于部署产品和调度数据库实例。

```
kubectl label node [Nodes] qfusion/node=
```

**必须**：如需要指定部署产品组件需额外打上 QFusion 平台的 “master” 节点qfusion/master=true 标签:

```
kubectl label node [Nodes] qfusion/master=true
```

**非必须**：为需要运行 Oracle 实例的节点打上 qfusion/need-oracle=true 标签，安装时将在被标记节点上部署 Oracle 所需环境：

```
kubectl label node [Nodes] qfusion/need-oracle=true
```

### 创建集群命名空间

登录任意master所在的节点，手动创建平台命名空间，用于部署产品组件，以下步骤假设使用 qfusion 命名空间：

```bash
kubectl create ns qfusion
```

##    配置安装选项

```
	标准版本，服务暴露类型为 nodeport：nodeport-sample.yaml

	标准版本，服务暴露类型为 multi-vip：multi-vip-sample.yaml

	RDS版本：rds-sample.yaml
```

以nodeport-sample.yaml 为例，下载命令为（本地文件统一命名为 ext-profile.yaml）：

```bash
$ curl https://qfusion-standard.oss-cn-hangzhou.aliyuncs.com/installation/ext-profiles/nodeport-sample.yaml --output ext-profile.yaml
```

下载得到的配置文件 ext-profile.yaml

```
	配置文件中的 iface 值必须替换为所有 “master” 节点的应用网卡名称，假设有两个节点且网卡名称为 eth0 和 eth1，应填入值 iface: 'eth0,eth1'。
	如果需要更改默认 vip，需要将所有的 10.10.150.70 替换为新的 vip 值。
注：RDS 版本无 csi-linstor、 iptables-manager、kube-keepalived-vip、vip-allocator 和 webserver 等部分的配置。
```

基于 ext-profile.yaml 创建 configmap：

```bash
$ kubectl create cm install-ext-profiles -n qfusion \
  --from-file=external.profile=ext-profile.yaml -o yaml \
  --dry-run | kubectl apply -f -
```

##   安装helm chart

添加 helm qfusion 仓库（仅在线安装场景需要）：

```bash
$ helm repo add qfusion https://helm.woqutech.com:8043/qfusion
```

在线场景的安装命令为(当前 3.12.1 版本)：

```bash
$ helm install qfusion qfusion/qfusion-installer \
  --version=v3.12.1-p1-rds -n qfusion \
  --set global.registry=registry.cn-hangzhou.aliyuncs.com \
  --set global.repository=tomc \
  --set nodeLabel=false \
  --set ext.cmName=install-ext-profiles \
  --set ext.profiles={external.profile}
```

##  检查安装情况

所有 pods 处于 Running 状态时所有组件安装完毕

```bash
$ kubectl get pods -n qfusion
```

若标记了 Oracle 节点，已标记节点拥有 qfusion/need-oracle=false 以及 qfusion/oracle= 标签时说明 Oracle 环境部署完毕。

```bash
$ kubectl get nodes --show-labels
```

## 卸载已安装的 qfusion

1.删除installer-operator

```
	找到原有的 qfusion 安装所使用的命名空间：kubectl get ns，下文假设所使用的命名空间为 qfusion
	删除该命名空间中的 qfi 资源：kubectl delete qfi qfusion -n qfusion installer-operator 将执行卸载操作，删除所有 qfusion 安装时创建的资源

```

2.删除 helm release

```
	列出 helm 已安装的 release 列表：helm list -n qfusion
	删除 qfusion release：helm delete qfusion -n qfusion
```

3.清除节点的qfusion 标签

```bash
$ kubectl label node --all qfusion/node-
$ kubectl label node --all qfusion/master-
$ kubectl label node --all qfusion/oracle-
$ kubectl label node --all qfusion/need-oracle-
```

