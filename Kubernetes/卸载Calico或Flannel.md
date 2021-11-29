# 网络插件的调整

[TOC]

## 卸载CalicoCNI

### 卸载Calico组件

可以执行以下脚本内容

```shell
#!/usr/bin/env bash
set -ex

# cni flag
kubectl -n kube-public delete cm cni-calico

kubectl -n kube-system delete cm calico-config

# calico crd
kubectl delete crd felixconfigurations.crd.projectcalico.org
kubectl delete crd ipamblocks.crd.projectcalico.org
kubectl delete crd blockaffinities.crd.projectcalico.org
kubectl delete crd ipamhandles.crd.projectcalico.org
kubectl delete crd ipamconfigs.crd.projectcalico.org
kubectl delete crd bgppeers.crd.projectcalico.org
kubectl delete crd bgpconfigurations.crd.projectcalico.org
kubectl delete crd ippools.crd.projectcalico.org
kubectl delete crd hostendpoints.crd.projectcalico.org
kubectl delete crd clusterinformations.crd.projectcalico.org
kubectl delete crd globalnetworkpolicies.crd.projectcalico.org
kubectl delete crd globalnetworksets.crd.projectcalico.org
kubectl delete crd networkpolicies.crd.projectcalico.org
kubectl delete crd networksets.crd.projectcalico.org

# delete calico tunnel
for pod in $(kubectl -n kube-system get pod -l k8s-app=calico-node --no-headers | awk '{print $1}')
do
  kubectl exec -n kube-system "$pod" -- modprobe -r ipip
done

# calico-node
kubectl -n kube-system delete ds calico-node
kubectl -n kube-system delete sa calico-node
kubectl delete clusterrole calico-node
kubectl delete clusterrolebinding calico-node

# calico-kube-controllers
kubectl -n kube-system delete deploy calico-kube-controllers
kubectl -n kube-system delete sa calico-kube-controllers
kubectl delete clusterrole calico-kube-controllers
kubectl delete clusterrolebinding calico-kube-controllers
```

### 删除每台机器上的残留文件并重启

- 根据环境具体情况，在每台机器执行删除

  ```bash
  $ rm -f /etc/cni/net.d/10-calico.conflist /etc/cni/net.d/calico-kubeconfig
  ```
  
- 执行reboot命令，重启清理 calico 残留 iptables 和 ipset 规则



## 卸载Flannel 

新建脚本，将以下内容拷入，执行脚本，删除Flannel的配置信息。注意过滤条件 -lapp=galaxy 需要修改为适配自己情况的条件，或者直接查出来全部的Flannel Pod，分步骤执行命令。

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "[Step -1] Delete flannel and galaxy resource on host"
for galaxy in $(kubectl get pod --no-headers -n kube-system -lapp=galaxy | awk '{print $1}')
do
  kubectl exec -n kube-system "$galaxy" -- ip link del flannel.1
  kubectl exec -n kube-system "$galaxy" -- rm -rf /host/etc/cni/net.d/00-galaxy.conf
  kubectl exec -n kube-system "$galaxy" -- rm -rf /etc/cni/net.d/00-galaxy.conf
done
echo "-------------------------------"
echo ""

echo "[Step 0] delete flannel and galaxy resource in kubernetes"
kubectl delete ds flannel galaxy-daemonset -n kube-system --ignore-not-found=true
kubectl delete cm cni-etc galaxy-etc kube-flannel-cfg -n kube-system --ignore-not-found=true
kubectl delete sa flannel galaxy --ignore-not-found=true
kubectl delete clusterrole flannel --ignore-not-found=true
kubectl delete clusterrolebindings flannel galaxy --ignore-not-found=true
kubectl annotate no --all flannel.alpha.coreos.com/backend-data-
kubectl annotate no --all flannel.alpha.coreos.com/backend-type-
kubectl annotate no --all flannel.alpha.coreos.com/kube-subnet-manager-
kubectl annotate no --all flannel.alpha.coreos.com/public-ip-
echo "-------------------------------"
echo ""
```



## 安装kube-ovn网络

```bash
$ wget https://raw.githubusercontent.com/kubeovn/kube-ovn/release-1.8/dist/images/install.sh
```

修改service,pod,join关键信息，编辑 install.sh

```
REGISTRY="kubeovn"
VERSION="v1.8.2"
IMAGE_PULL_POLICY="IfNotPresent"
POD_CIDR="10.4.0.0/16"                # Do NOT overlap with NODE/SVC/JOIN CIDR
POD_GATEWAY="10.4.0.1"
SVC_CIDR="10.6.0.0/16"                # Do NOT overlap with NODE/POD/JOIN CIDR
JOIN_CIDR="10.5.0.0/16"              # Do NOT overlap with NODE/POD/SVC CIDR
```

执行安装

```bash
$ bash install.sh
```
