# 同步容器镜像

[TOC]



## 步骤

参考文档如下

```
https://github.com/AliyunContainerService/image-syncer/blob/master/README-zh_CN.md
```

购买一台境外的服务器，然后下载image-syncer的二进制文件，放入/opt/目录下，即可。该二进制文件是可执行文件。

确认境外服务器中含有docker相关密钥，新建 /opt/image-sync目录，并拷贝docker密钥到此目录,制作auth.json。

账号

```
quay.io 		-> 	sjwayrhz / Vwv56ty7
docker.io		->  sjwayrhz / 1qaz2wsx#
```

### image-syncer

```bash
$ wget -P /tmp https://github.com/AliyunContainerService/image-syncer/releases/download/v1.3.1/image-syncer-v1.3.1-linux-amd64.tar.gz
$ tar -zxvf /tmp/image-syncer-v1.3.1-linux-amd64.tar.gz -C /tmp
$ mv /tmp/image-syncer /opt
```

### auth.yaml

```bash
tee /opt/auth.yaml <<- 'EOF'
docker.io:
  username: sjwayrhz
  password: 1qaz2wsx#
  insecure: true
quay.io:
  username: sjwayrhz
  password: Vwv56ty7
  insecure: true
EOF
```

### image.yaml

在/opt/目录下创建image.yaml文件，单个版本的镜像

```bash
tee /opt/images.yaml <<- 'EOF'
k8s.gcr.io/ingress-nginx/controller:v0.34.0: docker.io/sjwayrhz/controller
EOF
```

全版本镜像

```bash
tee /opt/images.yaml <<- 'EOF'
k8s.gcr.io/ingress-nginx/controller: docker.io/sjwayrhz/controller
EOF
```

将谷歌镜像同步到阿里云，范例操作如下，其中，namespace需要视情况而定：

```bash
nohup /opt/image-syncer --proc=6 --retries=3 --auth=/opt/auth.yaml --images=/opt/images.yaml &
```







