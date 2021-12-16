# Minio二主机分布式集群搭建

[TOC]

### 环境

```bash
[root@minio-1 ~]# cat /etc/redhat-release
Rocky Linux release 8.4 (Green Obsidian)
```

### 服务器

```
10.225.63.5
10.225.63.6
```

### 目录

本集群由2台服务器构成（官方推荐集群最小4台服务器），每个服务器上挂载两个磁盘目录，最小数据挂载点为4个

- 数据存储目录（两台机器都创建）

```bash
[root@minio-1 ~]# mkdir -p /minio/data1
[root@minio-1 ~]# mkdir -p /minio/data2
  
[root@minio-2 ~]# mkdir -p /minio/data1
[root@minio-2 ~]# mkdir -p /minio/data2
```

- 启动脚本目录

```bash
[root@minio-1 ~]# mkdir -p /opt/minio

[root@minio-2 ~]# mkdir -p /opt/minio
```

- 集群配置文件目录

```bash
[root@minio-1 ~]# mkdir -p /etc/minio

[root@minio-2 ~]# mkdir -p /etc/minio
```

### 编写集群启动脚本

所有节点配置文件相同

节点1:

```bash
[root@minio-1 ~]# tee /opt/minio/run.sh <<- 'EOF'
#!/bin/bash
export MINIO_ACCESS_KEY=wBh26XzE7Qq5rdP
export MINIO_SECRET_KEY=wEVp8SZ225p5zVVMHQsXfHtUQGvJTx
 
/opt/minio/minio server --config-dir /etc/minio \
http://10.225.63.5/minio/data1 http://10.225.63.5/minio/data2 \
http://10.225.63.6/minio/data1 http://10.225.63.6/minio/data2 \
EOF
```

节点2:

```bash
[root@minio-2 ~]# tee /opt/minio/run.sh <<- 'EOF'
#!/bin/bash
export MINIO_ACCESS_KEY=wBh26XzE7Qq5rdP
export MINIO_SECRET_KEY=wEVp8SZ225p5zVVMHQsXfHtUQGvJTx
     
/opt/minio/minio server --config-dir /etc/minio \
http://10.225.63.5/minio/data1 http://10.225.63.5/minio/data2 \
http://10.225.63.6/minio/data1 http://10.225.63.6/minio/data2 \
EOF
```

其中，“MINIO_ACCESS_KEY”为用户名，“MINIO_SECRET_KEY”为密码，密码不能设置过于简单，不然minio会启动失败，“–config-dir”指定集群配置文件目录

### 编写服务脚本

所有节点配置文件相同

节点1:

```bash
[root@minio-1 ~]# tee /usr/lib/systemd/system/minio.service <<- 'EOF'
[Unit]
Description=Minio service
Documentation=https://docs.minio.io/
 
[Service]
WorkingDirectory=/opt/minio/
ExecStart=/opt/minio/run.sh
 
Restart=on-failure
RestartSec=5
 
[Install]
WantedBy=multi-user.target
EOF
```

节点2:

```bash
[root@minio-2 ~]# tee /usr/lib/systemd/system/minio.service <<- 'EOF'
 
[Unit]
Description=Minio service
Documentation=https://docs.minio.io/
 
[Service]
WorkingDirectory=/opt/minio/
ExecStart=/opt/minio/run.sh
 
Restart=on-failure
RestartSec=5
 
[Install]
WantedBy=multi-user.target
EOF
```

### 添加脚本权限

```bash
[root@minio-1 ~]# chmod +x /usr/lib/systemd/system/minio.service
     
[root@minio-2 ~]# chmod +x /usr/lib/systemd/system/minio.service
```

### 启动测试

将minio上传到/opt/minio目录下并赋予权限

节点1:

```bash
[root@minio-1 ~]# cd /opt/minio/
[root@minio-1 minio]# wget https://dl.minio.io/server/minio/release/linux-amd64/minio

[root@minio-1 minio]# chmod +x minio
[root@minio-1 minio]# chmod +x /opt/minio/run.sh
```

节点2:

```bash
[root@minio-2 ~]# cd /opt/minio/
[root@minio-2 minio]# wget https://dl.min.io/server/minio/release/linux-amd64/minio
 
 
[root@minio-2 minio]# chmod +x minio
[root@minio-2 minio]# chmod +x /opt/minio/run.sh
```

### 启动

节点1:

```bash
[root@minio-1 minio]# systemctl daemon-reload
[root@minio-1 minio]# systemctl enable --now minio
Created symlink from /etc/systemd/system/multi-user.target.wants/minio.service to /usr/lib/systemd/system/minio.service.
```

节点2:

```bash
[root@minio-2 minio]# systemctl daemon-reload
[root@minio-2 minio]# systemctl enable --now minio
Created symlink from /etc/systemd/system/multi-user.target.wants/minio.service to /usr/lib/systemd/system/minio.service.
```

### 测试

浏览器输入集群任意节点地址+9000端口，即可访问minio，用户名密码为前面设置的“MINIO_ACCESS_KEY”和“MINIO_SECRET_KEY”，可创建“bucket”并上传文件测试

http://10.225.63.5:9000/minio/login

### nginx代理

一定要写上 Host Header 的代理，否则请求签名两边对不上。

```nginx
root@uploadservice1 vhost]# cat minio.conf 

upstream minio.juneyaokc.com {
  server 10.225.63.5:9000 weight=5 ;
  server 10.225.63.6:9000 weight=5 ; 
}
    
server {
  listen 80;
  server_name minio.juneyaokc.com;
  client_max_body_size 20M;
  charset utf-8;

  location /health {
    proxy_pass http://minio.juneyaokc.com/health;
  }
 
  location / {
    proxy_set_header Host $http_host;
    client_body_buffer_size 10M;
    client_max_body_size 10G;
    proxy_buffers 1024 4k;
    proxy_read_timeout 300;
    proxy_pass http://minio.juneyaokc.com/;
  }
 
  location /status {
    stub_status;
  }
  error_page 500 502 503 504 /50x.html;
  
  location = /50x.html {
    root html;
  } 
}
```

