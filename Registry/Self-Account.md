### github registry

```json
{
	"auths": {
		"ghcr.io": {
			"auth": "c2p3YXlyaHo6Z2hwX083WW41ZE52VDE0bkJPVFRnb1RGTkttR1ppejFJaDE2U245Sw=="
		}
	}
}
```

### 阿里云镜像仓库

采用自己的邮箱号taoistmonk@163.com注册的阿里云账号，绑定了i@sjhz.cf邮箱，申请了阿里云镜像仓库，地址为

```bash
$ docker login --username=taoistmonk@163.com registry.cn-shanghai.aliyuncs.com
```

密码为常用的密码`Vwv56ty7`

其中，json密钥为

```json
$ cat .docker/config.json
{
	"auths": {
		"registry.cn-shanghai.aliyuncs.com": {
			"auth": "dGFvaXN0bW9ua0AxNjMuY29tOlZ3djU2dHk3"
		}
	}
}
```



### Docker官方镜像仓库

```json
$ cat .docker/config.json
{
	"auths": {
		"https://index.docker.io/v1/": {
			"auth": "c2p3YXlyaHo6MXFhejJ3c3gj"
		}
	}
}
```



### 世外小学智学通

```json
$ cat .docker/config.json
{
	"auths": {
		"registry.cn-shanghai.aliyuncs.com": {
			"auth": "aGkzMDMxNjMxNkBhbGl5dW4uY29tOlZ3djU2dHk3"
		}
	}
}
```

