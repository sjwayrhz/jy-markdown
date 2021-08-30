依靠如下问题没有解决问题

```
export GOPRIVATE=*.huaweicloud.com
go env -w GOSUMDB=off
go env -w GOSUMDB="sum.golang.google.cn"
go env -w GOSUMDB="sum.golang.org"


https://www.jianshu.com/p/e0c878d4ca19
```





依靠如下配置解决了问题

```
git config --global url."git@codehub-cn-east-2.devcloud.huaweicloud.com:".insteadOf "https://codehub-cn-east-2.devcloud.huaweicloud.com/"
```





另外 go build 出现问题的时候

```
[root@ops-prod-jenkins deploy]# CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o cailianpress-vod ../main.go
# codehub-cn-east-2.devcloud.huaweicloud.com/jgz00001/cls-skywalking-client-go.git
/root/go/pkg/mod/codehub-cn-east-2.devcloud.huaweicloud.com/jgz00001/cls-skywalking-client-go.git@v0.1.5/contextlocal.go:61:4: t.Reset undefined (type *time.Ticker has no field or method Reset)
```

需要升级go版本到1.15就可以了