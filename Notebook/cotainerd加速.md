## cotainerd加速

1.4版本需要的配置

```bash
sed -i "s#k8s.gcr.io#registry.cn-hangzhou.aliyuncs.com/google_containers#g"  /etc/containerd/config.toml
sed -i "s#https://registry-1.docker.io#https://registry.cn-hangzhou.aliyuncs.com#g"  /etc/containerd/config.toml
sed -i '/containerd.runtimes.runc.options/a\ \ \ \ \ \ \ \ \ \ \ \ SystemdCgroup = true' /etc/containerd/config.toml
```

1.5版本需要的配置

```bash
sed -i "s#k8s.gcr.io#registry.cn-hangzhou.aliyuncs.com/google_containers#g"  /etc/containerd/config.toml
sed -i "s#SystemdCgroup = false#SystemdCgroup = true#g" /etc/containerd/config.toml
sed -i '/registry.mirrors/a\ \ \ \ \ \ \ \ [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]' /etc/containerd/config.toml
sed -i '/docker.io/a\ \ \ \ \ \ \ \ \ \ endpoint = ["https://registry.cn-hangzhou.aliyuncs.com"]' /etc/containerd/config.toml
```

