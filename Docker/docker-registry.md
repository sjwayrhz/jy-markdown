# docker镜像仓库

[TOC]

## 自带证书搭建

在/etc/pki/ssl中放入购买的ssl证书
```bash
$ ll /etc/pki/ssl/
total 20
-rw-rw-rw- 1 root root 1700 Dec  2 02:26 docker.sjhz.tk.key
-rw-rw-rw- 1 root root 3901 Dec  2 02:26 docker.sjhz.tk_bundle.crt
```

使用如下命令可以搭建镜像仓库

```bash
$ docker run -d \
  -p 443:443 \
  -v /var/lib/registry:/var/lib/registry \
  -v /etc/pki/ssl:/certs \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/docker.sjhz.tk_bundle.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/docker.sjhz.tk.key \
  --restart=always \
  --name registry \
  registry:2.7.1
```

搭建完成后，镜像保存在/var/lib/registry目录下

## Nginx反向代理搭建

使用nginx反向代理，可以将证书存放与nginx中，启动如下：

```bash
$ docker run -d \
	--name registry \
	--restart=always \
  -p 5000:5000 \
  -v /var/lib/registry:/var/lib/registry \  
  registry:2.8.2
```



## 存放文件

> 描述

在docker-registry之外的其它linux系统中，推送镜像到docker-registry服务器

> 准备证书

注意： 证书是购买的与证书是自签名有区别

建议使用购买的证书，而不是自签名证书，否则无法通过安全检查，需要对docker做进一步的设置。

```bash
$ mkdir -p /etc/docker/certs.d/docker.sjhz.tk
$ cp docker.sjhz.tk_bundle.crt /etc/docker/certs.d/docker.sjhz.tk

$ ls /etc/docker/certs.d/docker.sjhz.tk
docker.sjhz.tk_bundle.crt
```

> 重打标签

将自建的maven包重新打包成私有镜像包

```bash
$ docker tag sjwayrhz/maven:3.8.4-jdk-8 docker.sjhz.tk/maven:3.8.4-jdk-8
```

> 推送镜像

```bash
$ docker push sjwayrhz/maven:3.8.4-jdk-8 docker.sjhz.tk/maven:3.8.4-jdk-8
```



## 查看仓库镜像

推送了maven镜像之后，在docker-registry服务器上可以看到

```bash
$ ls /var/lib/registry/docker/registry/v2/repositories/
maven  redis
```

推送了maven镜像之后，在docker客户端服务器上可以看到

```bash
$ curl -XGET https://docker.sjhz.tk/v2/_catalog
{"repositories":["maven","redis"]}

$ curl -XGET https://docker.sjhz.tk/v2/maven/tags/list
{"name":"maven","tags":["3.8.4-jdk-8"]}
$ curl -XGET https://docker.sjhz.tk/v2/redis/tags/list
{"name":"redis","tags":["4.0.1"]}
```

追加推送 docker:20.10.12 和 docker:20.10.12-dind , 在和客户端服务器可以看到

```bash
$ curl -XGET https://docker.sjhz.tk/v2/_catalog
{"repositories":["docker","maven","redis"]}
$ curl -XGET https://docker.sjhz.tk/v2/docker/tags/list
{"name":"docker","tags":["20.10.12","20.10.12-dind"]}
```

