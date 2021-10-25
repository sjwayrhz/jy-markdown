## windows中环境变量中配置代理

新建系统变量
```
GO111MODULE     ->      on
GOPROXY         ->      https://goproxy.io
```

## linux安装go

### 下载 Go 二进制文件

```shell
~]# cd /tmp
tmp]# wget https://dl.google.com/go/go1.17.linux-amd64.tar.gz
```

解压缩下载的 tar，然后安装到系统中的所需位置。但是通常遵循文档最好将其安装在 /user/local/go 下。在终端中运行以下命令进行安装。

```bash
tmp]# sudo tar -xvf go1.17.linux-amd64.tar.gz -C /usr/local/
```

我们正在设置的三个 Go 语言环境变量是 GOROOT，GOPATH 和 PATH。 GOROOT 是 Go 在机器中安装的路径 GOPATH 是工作目录的位置。

编辑.bashrc，添加以下几行

```
# loads go
export GOROOT="/usr/local/go"
export GOPATH="$HOME/go"
export PATH="$GOPATH/bin:$GOROOT/bin:$PATH"
export GOPROXY="https://goproxy.cn"
```

应该已经成功安装在机器上，并检查它是否在以下命令下运行

```shell
~]# go version
go version go1.17 linux/amd64
~]# go env
```

## Linux安装node

下载node

```bash
~]# cd /tmp
tmp]# wget https://nodejs.org/dist/v14.17.5/node-v14.17.5-linux-x64.tar.xz
tmp]# tar -xf node-v14.17.5-linux-x64.tar.xz -C /usr/local
```

配置环境变量

```bash
~]# echo "export PATH=$PATH:/usr/local/node-v14.17.5-linux-x64/bin" >> /etc/profile
~]# source /etc/profile
~]# node -v
v14.17.5
```

配置加速

#### 临时使用 

```
 ~]# npm --registry https://registry.npm.taobao.org install express 
```

#### 持久使用

```
~]# npm config set registry https://registry.npm.taobao.org
```


  配置后可通过下面方式来验证是否成功 

```
~]# npm config get registry 或 npm info express 
```

#### 通过cnpm使用 

```
~]# npm install -g cnpm --registry=https://registry.npm.taobao.org
```



