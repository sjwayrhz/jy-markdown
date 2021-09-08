# Helm操作笔记



### 安装helm3

github地址如下

```
https://github.com/helm/helm
```

下载目前最新版本v3.6.3的helm

```
~]# wget https://get.helm.sh/helm-v3.6.3-linux-amd64.tar.gz
```

将tar.gz文件解压并将helm可执行文件放到 /usr/local/bin 下即可使用

如果将.kube的config文件放在了根目录下，将会发生警告

```
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
```



### helm镜像源-国内

```
helm repo add stable http://mirror.azure.cn/kubernetes/charts/
```



