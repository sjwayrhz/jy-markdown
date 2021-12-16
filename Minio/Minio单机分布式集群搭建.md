# Minio二主机分布式集群搭建

[TOC]

### 环境

系统

```bash
[root@minio ~]# cat /etc/redhat-release
Rocky Linux release 8.4 (Green Obsidian)
```

ip地址

```
10.225.63.4
```

### 目录

此台服务器共有5块盘，分区为1块系统盘加4块minio存储盘。

- 数据存储目录

```bash
[root@minio ~]# mkdir -p /minio/data1
[root@minio ~]# mkdir -p /minio/data2
[root@minio ~]# mkdir -p /minio/data3
[root@minio ~]# mkdir -p /minio/data4
```

- 启动脚本目录

```bash
[root@minio ~]# mkdir -p /minio/bin
```

- 集群配置文件目录

```bash
[root@minio ~]# mkdir -p /etc/minio
```

### 编写集群启动脚本

```bash
[root@minio ~]# tee /minio/bin/run.sh <<- 'EOF'
#!/bin/bash
export MINIO_ACCESS_KEY=wBh26XzE7Qq5rdP
export MINIO_SECRET_KEY=wEVp8SZ225p5zVVMHQsXfHtUQGvJTx
     
/minio/bin/minio server --config-dir /etc/minio \
http://10.225.63.4/minio/data1 http://10.225.63.4/minio/data2 \
http://10.225.63.4/minio/data3 http://10.225.63.4/minio/data4 \
EOF
```

其中，“MINIO_ACCESS_KEY”为用户名，“MINIO_SECRET_KEY”为密码，密码不能设置过于简单，不然minio会启动失败，“–config-dir”指定集群配置文件目录

添加脚本权限

```bash
[root@minio ~]# chmod +x /minio/bin/run.sh
```

### 编写服务脚本

```bash
[root@minio ~]# tee /usr/lib/systemd/system/minio.service <<- 'EOF'
[Unit]
Description=Minio service
Documentation=https://docs.minio.io/
 
[Service]
WorkingDirectory=/minio/bin/
ExecStart=/minio/bin/run.sh
 
Restart=on-failure
RestartSec=5
 
[Install]
WantedBy=multi-user.target
EOF
```

添加脚本权限

```bash
[root@minio ~]# chmod +x /usr/lib/systemd/system/minio.service
```

### 启动测试

将minio上传到/opt/minio目录下并赋予权限

```bash
[root@minio minio]# wget -P /minio/bin/ https://dl.minio.io/server/minio/release/linux-amd64/minio
[root@minio minio]# chmod +x /minio/bin/minio
```

### 启动

```bash
[root@minio minio]# systemctl daemon-reload
[root@minio minio]# systemctl enable --now minio
Created symlink from /etc/systemd/system/multi-user.target.wants/minio.service to /usr/lib/systemd/system/minio.service.
```

### 测试

浏览器输入集群地址+9000端口，即可访问minio，用户名密码为前面设置的“MINIO_ACCESS_KEY”和“MINIO_SECRET_KEY”

```
http://10.225.63.4:9000/minio/login
```







