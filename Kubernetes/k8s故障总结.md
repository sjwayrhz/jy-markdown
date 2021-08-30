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


## failed for volume  mountfailed: exit status 32
我在k8s集群中添加了k8s-node-05,但是没有给他挂载nfs磁盘，所以出现以下错误

```
Events:
  Type     Reason       Age                  From               Message
  ----     ------       ----                 ----               -------
  Normal   Scheduled    2m57s                default-scheduler  Successfully assigned app-dev/dev-web-6b84c69c65-54rff to k8s-node-05
  Warning  FailedMount  55s                  kubelet            Unable to attach or mount volumes: unmounted volumes=[nfs], unattached volumes=[nfs kube-api-access-fvbkr]: timed out waiting for the condition
  Warning  FailedMount  50s (x9 over 2m57s)  kubelet            MountVolume.SetUp failed for volume "pvc-9e7f45b5-8e66-4df0-bef1-847f243b3a22" : mountfailed: exit status 32
Mounting command: mount
Mounting arguments: -t nfs 10.230.7.17:/data/app-dev /var/lib/kubelet/pods/34d7f1a9-8833-45a0-8283-91ad51e8d984/volumes/kubernetes.io~nfs/pvc-9e7f45b5-8e66-4df0-bef1-847f243b3a22
Output: mount: /var/lib/kubelet/pods/34d7f1a9-8833-45a0-8283-91ad51e8d984/volumes/kubernetes.io~nfs/pvc-9e7f45b5-8e66-4df0-bef1-847f243b3a22: bad option; for several filesystems (e.g. nfs, cifs) you might need a /sbin/mount.<type> helper program.
```
解决办法，给node-05添加挂载磁盘
```bash
$ dnf install -y nfs-utils
$ mkdir /data
$ echo "10.230.7.17:/data  /data  nfs  defaults  0 0" >> /etc/fstab
```