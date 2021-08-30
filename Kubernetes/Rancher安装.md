快速启动

```
docker run -d --restart=unless-stopped \
  --privileged \
  -p 80:80 -p 443:443 \
  -v /opt/rancher:/var/lib/rancher/ \
  rancher/rancher:latest
```

官方文档

```
sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 --privileged rancher/rancher
```





