# docker镜像仓库

使用如下命令可以搭建镜像仓库

```bash
$ docker run -d \
  -p 5000:443 \
  -v /var/lib/registry:/var/lib/registry \
  -v /var/lib/certs:/certs \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/server.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/server.key \
  --restart=always \
  --name registry \
  registry:2
```

