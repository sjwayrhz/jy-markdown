为了解决华为私有仓库的go_mod_vendor的问题，

在财联社api项目中执行go mod vendor出现问题

```
api]# go mod vendor
go: codehub-cn-east-2.devcloud.huaweicloud.com/jgz00001/cls-skywalking-client-go.git@v0.1.7: reading https://mirrors.aliyun.com/goproxy/codehub-cn-east-2.devcloud.huaweicloud.com/jgz00001/cls-skywalking-client-go.git/@v/v0.1.7.mod: 404 Not Found
```



解决思路如下：

```
export GOPRIVATE=*.huaweicloud.com
```



然后继续执行go mod vendor就没有问题了