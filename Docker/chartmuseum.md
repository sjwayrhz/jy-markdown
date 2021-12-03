## 官方部署方法

需要在根目录新建/charts文件夹

```bash
$ mkdir /charts
$ chown -R 1000:0 /charts
```

启动charts项目

```bash
$ docker run -d \
  -p 8080:8080 \
  -e STORAGE=local \
  -e STORAGE_LOCAL_ROOTDIR=/charts \
  -v /charts:/charts \
  sjwayrhz/chartmuseum:v0.13.1
```

添加repo
```bash
$ helm repo add chartrepo http://121.46.238.133:8080
```

以 “redis-single-6.2.6” 为例子，备份这个helm chart
```bash
$ helm package redis-single-6.2.6
Successfully packaged chart and saved it to: /root/redis-0.0.2.tgz
```

拷贝/root/redis-0.0.2.tgz到chartmuseum主机的/charts目录下
```
$ scp /root/redis-0.0.2.tgz root@10.220.62.36:/charts
```

登录到 charmuseum主机，查看到放入gz包的文件
```
$ ll /charts/
total 4
-rw-r--r-- 1 root root 3999 Dec  3 12:12 redis-0.0.2.tgz
```

在安装helm的主机上更新repo，然后可以查询到这个repo
```bash
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "chartrepo" chart repository
Update Complete. ⎈Happy Helming!⎈

$ helm search repo
NAME           	CHART VERSION	APP VERSION	DESCRIPTION
chartrepo/redis	0.0.2        	6.2.6      	A Helm chart for Redis
```

安装helm push 插件
```
$ helm env | grep HELM_PLUGINS
HELM_PLUGINS="/root/.local/share/helm/plugins"

$ mkdir -p ~/.local/share/helm/plugins/helm-push.git

$ tar -zxvf helm-push_0.10.1_linux_amd64.tar.gz -C  ~/.local/share/helm/plugins/helm-push.git

$ helm plugin list
NAME   	VERSION	DESCRIPTION
cm-push	0.10.1 	Push chart package to ChartMuseum
```
helm push命令将chart发布到chartmuseum上
```
$ helm cm-push --insecure redis chartreop

OR: chart dir or chart tgz
 
$ helm cm-push --insecure redis-0.0.2.tgz chartrepo
```
更新helm repo，搜索刚刚上传的chart。
```
$ helm repo upgrade

$ helm search repo chartrepo
NAME           	CHART VERSION	APP VERSION	DESCRIPTION
chartrepo/redis	0.0.2        	6.2.6      	A Helm chart for Redis
```