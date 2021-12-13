1、编写yaml文件

①创建集群权限的SA ，并绑定ClusterRole：cluster-admin。

```bash
$ tee /tmp/jumpserver-admin.yaml <<- 'EOF' 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jumpserver-admin
  namespace: kube-system
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: jumpserver-admin
subjects:
  - kind: ServiceAccount
    name: jumpserver-admin
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
EOF
```

部署

```bash
$ kubectl apply -f /tmp/jumpserver-admin.yaml
```

2、获取jumpserver-admin用户的secrets

```bash
$ kubectl get sa  jumpserver-admin -n kube-system  -o yaml
```

在输出中找到secrets字段，记录该字段对应的名称，本例为：jumpserver-admin-token-vxjqt
3、获取token，用上一步查找到的secret的名称

```bash
$ kubectl get secret jumpserver-admin-token-vxjqt -n kube-system -o jsonpath={".data.token"}
```

4、token转码

```bash
$ kubectl get secret jumpserver-admin-token-vxjqt -n kube-system -o jsonpath={".data.token"}| base64 -d
```

5、将获得的token加入到jumpserver应用用户里面就可以了
