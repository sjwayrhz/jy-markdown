# 某些版本的docker是没有/etc/sysconfig/docker，这个配置文件的，所以需要做如下的配置操作：

------

## 1、 vim  /lib/systemd/system/docker.service

```
[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/docker
ExecStart=/usr/bin/dockerd $OPTIONS
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity
MountFlags=slave
```

## 2、vim /etc/sysconfig/docker

```
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false  -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375'
if [ -z "${DOCKER_CERT_PATH}" ]; then
    DOCKER_CERT_PATH=/etc/docker
fi
```

## 3、重启docker

```
   systemctl daemon-reload
    systemctl restart docker
```