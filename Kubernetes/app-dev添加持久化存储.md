计划分100G存储空间给集团app
由于之前已经制作好了 app-web-dev-withconfigmap.yaml ,于是在此基础上，copy为 app-web-dev-withpv.yaml即可。

在此之前，安装部署nfs-subdir
```
https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner
```

创建storeageclass,名字为nfs-client

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: k8s-sigs.io/nfs-subdir-external-provisioner # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  pathPattern: "${.PVC.namespace}" # waits for nfs.io/storage-path annotation, if not specified will accept as empty string.
  onDelete: delete
```

添加pvc.yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dev-web-pvc
  namespace: app-dev
spec:
  storageClassName: nfs-client
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 20G
```

在 spec.container 内部与 -name 同列下添加

volumeMounts: 
  - name: nfs
    mountPath: /logs

在 spec.container 与 container 同列下添加
volumes:
  - name: nfs
    persistentVolumeClaim:
      claimName: dev-web-pvc