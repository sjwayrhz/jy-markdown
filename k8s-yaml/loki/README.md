添加数据到database

```
/ $ cat /etc/hosts
# Kubernetes-managed hosts file.
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
10.2.154.222    loki-0.loki-headless.default.svc.cluster.local  loki-0
```

注意点：在volumeMounts的mountPath和volumes的hostPath中，应该加入k8s的node节点的docker存储目录和Pod存储目录，
/var/log/pods是k8s默认的pod存储目录，但是我的docker目录修改为/opt/docker了
```
- name: docker
  mountPath: /opt/docker/containers
  readOnly: true
- name: pods
  mountPath: /var/log/pods

- name: docker
  hostPath:
    path: /opt/docker/containers
    type: ""
- name: pods
  hostPath:
    path: /var/log/pods
    type: ""
```