# k8s证书更新

[TOC]

### 查看各个证书过期时间

```bash
~]# for item in `find /etc/kubernetes/pki -maxdepth 2 -name "*.crt"`;do openssl x509 -in $item -text -noout| grep Not;echo ======================$item===============;done
```

### 生成集群的配置文件

```bash
~]# kubectl get cm -o yaml -n kube-system kubeadm-config > /tmp/cluster.yaml
```

### 备份原有证书

```bash
~]# cp -rp /etc/kubernetes /etc/kubernetes.bak
```

### 备份etcd数据目录

```bash
~]# cp -r /var/lib/etcd /var/lib/etcd.bak
```

### 更新证书

```bash
~]# kubeadm certs renew all
```

输出如下内容

```
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed

Done renewing certificates. You must restart the kube-apiserver, kube-controller-manager, kube-scheduler and etcd, so that they can use the new certificates. 
```

### 

### 更新kube-config

```bash
~]# kubeadm init phase kubeconfig all
I0907 17:17:16.653605 2807201 version.go:254] remote version is much newer: v1.22.1; falling back to: stable-1.21
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/admin.conf"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/controller-manager.conf"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/scheduler.conf"
```

将新生成的 admin 配置文件覆盖掉原本的 admin 文件:

```bash
$ mv $HOME/.kube/config $HOME/.kube/config.old
$ cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ chown $(id -u):$(id -g) $HOME/.kube/config
```

完成后重启 kube-apiserver、kube-controller、kube-scheduler、etcd 这4个容器即可，我们可以查看 apiserver 的证书的有效期来验证是否更新成功：

### 重启docker使得更新的证书生效

```bash
~]# docker ps |grep -E 'k8s_kube-apiserver|k8s_kube-controller-manager|k8s_kube-scheduler|k8s_etcd_etcd' | awk -F ' ' '{print $1}' |xargs docker restart
```

### 更新前后比较

更新前

```bash
~]# kubeadm  certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Aug 28, 2022 23:09 UTC   355d                                    no
apiserver                  Aug 28, 2022 23:09 UTC   355d            ca                      no
apiserver-etcd-client      Aug 28, 2022 23:09 UTC   355d            etcd-ca                 no
apiserver-kubelet-client   Aug 28, 2022 23:09 UTC   355d            ca                      no
controller-manager.conf    Aug 28, 2022 23:09 UTC   355d                                    no
etcd-healthcheck-client    Aug 28, 2022 23:09 UTC   355d            etcd-ca                 no
etcd-peer                  Aug 28, 2022 23:09 UTC   355d            etcd-ca                 no
etcd-server                Aug 28, 2022 23:09 UTC   355d            etcd-ca                 no
front-proxy-client         Aug 28, 2022 23:09 UTC   355d            front-proxy-ca          no
scheduler.conf             Aug 28, 2022 23:09 UTC   355d                                    no

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Aug 22, 2031 23:24 UTC   9y              no
etcd-ca                 Aug 22, 2031 23:24 UTC   9y              no
front-proxy-ca          Aug 22, 2031 23:24 UTC   9y              no  
```

更新后

```bash
~]# kubeadm  certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Sep 07, 2022 09:19 UTC   364d                                    no
apiserver                  Sep 07, 2022 09:19 UTC   364d            ca                      no
apiserver-etcd-client      Sep 07, 2022 09:19 UTC   364d            etcd-ca                 no
apiserver-kubelet-client   Sep 07, 2022 09:19 UTC   364d            ca                      no
controller-manager.conf    Sep 07, 2022 09:19 UTC   364d                                    no
etcd-healthcheck-client    Sep 07, 2022 09:19 UTC   364d            etcd-ca                 no
etcd-peer                  Sep 07, 2022 09:19 UTC   364d            etcd-ca                 no
etcd-server                Sep 07, 2022 09:19 UTC   364d            etcd-ca                 no
front-proxy-client         Sep 07, 2022 09:19 UTC   364d            front-proxy-ca          no
scheduler.conf             Sep 07, 2022 09:19 UTC   364d                                    no

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Aug 22, 2031 23:24 UTC   9y              no
etcd-ca                 Aug 22, 2031 23:24 UTC   9y              no
front-proxy-ca          Aug 22, 2031 23:24 UTC   9y              no 
```

