# docker镜像仓库

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
  -p 5000:443 \
  -v /var/lib/registry:/var/lib/registry \
  -v /etc/pki/ssl:/certs \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/docker.sjhz.tk_bundle.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/docker.sjhz.tk.key \
  --restart=always \
  --name registry \
  registry:2
```

搭建完成后，镜像保存在/var/lib/registry目录下
