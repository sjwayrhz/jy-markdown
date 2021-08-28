windows中环境变量中配置代理

新建系统变量
```
GO111MODULE     ->      on
GOPROXY         ->      https://goproxy.io
```

linux:
```
$ export GO111MODULE=on
$ export GOPROXY=https://goproxy.cn
```
　　or
```
$ echo "export GO111MODULE=on" >> ~/.profile
$ echo "export GOPROXY=https://goproxy.cn" >> ~/.profile
$ source ~/.profile
```
