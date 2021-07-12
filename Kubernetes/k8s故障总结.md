# k8s故障总结

[TOC]

### k8s 删除pv一直处于terminating 解决方法

现状

```
kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS        CLAIM                STORAGECLASS   REASON   AGE
pvc-357c3dcd-79f8-4b64-9e6d-52de33a12685   1Mi        RWX            Delete           Terminating   default/nfs-sc-pvc   nfs-client              5d19h
```

发现这个pv，一直处于Terminating无法终结

删除方案

```
kubectl patch pv pvc-357c3dcd-79f8-4b64-9e6d-52de33a12685 -p '{"metadata":{"finalizers":null}}'
```

再次检查pv发现已经被删除

