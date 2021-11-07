# 同步容器镜像

[TOC]



### 步骤

参考文档如下

```
https://github.com/AliyunContainerService/image-syncer/blob/master/README-zh_CN.md
```

购买一台境外的服务器，然后下载image-syncer的二进制文件，放入/usr/local/bin/目录下，即可。该二进制文件是可执行文件。

确认境外服务器中含有docker相关密钥，新建 /opt/image-sync目录，并拷贝docker密钥到此目录,制作auth.json。

```
~]# mkdir /opt/image-sync
~]# cp /.docker/config.json /opt/image-sync
~]# mv /opt/image-sync/config.json /opt/image-sync/auth.json 
```

auth.json如下

```
~]# cat auth.json
{
	"auths": {
		"https://index.docker.io/v1/": {
			"auth": "c2p3YXlyaHo6MXFhejJ3c3gj"
		},
		"registry.cn-beijing.aliyuncs.com": {
			"auth": "Y2FvYm9AY2xzOlZ3djU2dHk3"
		},
		"swr.cn-east-2.myhuaweicloud.com": {
			"auth": "Y24tZWFzdC0yQFhMQlVBUllKWDBaV0c3VThDQ09OOmRmYTcxMTJmY2YyMjU0YzVhMGQwYzQwYjkzMmY4MmNlNDI2YTIxZTAxODIwMjczOGY0YjFmMThjMTkwMDFmMTE="
		}
	},
	"HttpHeaders": {
		"User-Agent": "Docker-Client/19.03.6 (linux)"
	}
}
```

在/opt/image-sync/目录下创建image.json文件，实例如下：

```
]# cat images.json
{
    "quay.io/coreos/kube-rbac-proxy": "registry.cn-beijing.aliyuncs.com/newsfeed/kube-rbac-proxy"
}
```



将谷歌镜像同步到阿里云，范例操作如下，其中，namespace需要视情况而定：

```
]# image-syncer --proc=6 --auth=./auth.json --images=./images.json --namespace=newsfeed --registry=registry.cn-beijing.aliyuncs.com --retries=3
```

完成同步后，即可看到阿里云上已经有了同步后的镜像了。



### 账号

```
quay.io 		-> 	sjwayrhz / Vwv56ty7
docker.io		->  sjwayrhz / 1qaz2wsx#
```



