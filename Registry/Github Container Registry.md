# Github Container Registry

[TOC]

## Token

账户为sjwayrhz，token如下

```
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
```

权限为

```
repo	write:packages	read:packages		delete:packages 
```

## 登陆方法

```bash
$ export CR_PAT=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
$ echo $CR_PAT | docker login ghcr.io -u sjwayrhz --password-stdin
```

## 传递镜像

```bash
$ docker pull sjwayrhz/rocky-8.4:kubectl
$ docker tag sjwayrhz/rocky-8.4:kubectl ghcr.io/sjwayrhz/rocky-8.4:kubectl
$ docker push ghcr.io/sjwayrhz/rocky-8.4:kubectl
```

### Image-syncer

```yaml
k8s.gcr.io/kube-apiserver: ghcr.io/sjwayrhz/kube-apiserver
k8s.gcr.io/kube-controller-manager: ghcr.io/sjwayrhz/kube-controller-manager
k8s.gcr.io/kube-scheduler: ghcr.io/sjwayrhz/kube-scheduler
k8s.gcr.io/kube-proxy: ghcr.io/sjwayrhz/kube-proxy
k8s.gcr.io/pause: ghcr.io/sjwayrhz/pause
k8s.gcr.io/etcd: ghcr.io/sjwayrhz/etcd
k8s.gcr.io/coredns/coredns: ghcr.io/sjwayrhz/coredns
```

