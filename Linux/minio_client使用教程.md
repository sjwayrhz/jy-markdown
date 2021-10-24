# minio client使用教程

[TOC]
详细教程可以访问如下网址：
```
http://docs.minio.org.cn/docs/master/minio-client-complete-guide
```
## 下载MinIO Client

下载二进制文件
### (GNU/Linux)

下载之后要赋予mc可执行权限
```
$ wget -P /usr/local/bin http://dl.minio.org.cn/client/mc/release/linux-amd64/mc
$ chmod +x /usr/local/bin/mc
$ mc -h
```

### (Microsoft Windows)
```
http://dl.minio.org.cn/client/mc/release/windows-amd64/mc.exe
```
需要将mc.exe加入到环境变量，或者放到`C:\WINDOWS\System32\`目录下

## 添加一个云存储服务
公式为
```
$ mc config host add <ALIAS> <YOUR-S3-ENDPOINT> <YOUR-ACCESS-KEY> <YOUR-SECRET-KEY> [--api API-SIGNATURE]
```
示例-Amazon S3云存储
```
$ mc config host add s3 https://s3.amazonaws.com BKIKJAA5BMMU2RHO6IBB V7f1CwQqAcwo80UEIJEjc5gVQUSSx5ohQ9GSrr12 --api s3v4
```
目前210上的k8s可用如下方式添加mino存储服务
```
$ mc config host add minio http://192.168.177.210:30090 root123 adminadmin --api s3v4
```
返回值为
```
Added `minio` successfully.
```
## 验证
### 内置play文件桶
mc预先配置了云存储服务URL：https://play.min.io，别名“play”。它是一个用于研发和测试的MinIO服务。如果想测试Amazon S3,你可以将“play”替换为“s3”。

示例:

列出https://play.min.io上的所有存储桶。
```
$ mc ls play
```
### 验证minio文件桶
参考内置play文件桶的验证方法，得知验证minio文件桶的方法为：
```
$ mc ls minio

[2021-10-14 13:53:13 CST]     0B bucket01/
```
### 文件桶的增删
创建bucket02文件桶
```
$ mc mb minio/bucket02
Bucket created successfully `minio/bucket02`.
```
显示现在拥有的文件桶
```
$ mc ls minio
[2021-10-14 13:53:13 CST]     0B bucket01/
[2021-10-24 15:31:20 CST]     0B bucket02/
```
删除bucket02文件桶
```
$ mc rm --dangerous --force minio/bucket02
Removing `minio/bucket02`.
```
拷贝 1.txt到bucket01
```
$ echo "test" > 1.txt
$ mc cp 1.txt minio/bucket01
1.txt:         5 B / 5 B ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 68 B/s 0s
```
删除bucket01中的1.txt
```
$ mc rm minio/bucket01/1.txt
Removing `minio/bucket01/1.txt`.
```
## 设置文件桶策略
先在做如下模拟操作
1. 新建minio存储桶名称为test
2. 设置minio/test文件桶为可下载
3. 新建helloworld.txt，内容为“helloworld”，并将该文件放入文件桶minio/test 
4. 获取minio/test/helloworld.txt的下载链接，并尝试访问
5. 删除minio/test文件桶

使用mc的操作命令如下：
```
$ mc mb minio/test
$ mc policy set download minio/test
$ echo "helloworld" > helloworld.txt
$ mc cp helloworld.txt minio/test/helloworld.txt
```
获取helloworld.txt的访问链接为：
```
http://192.168.177.210:30090/test/helloworld.txt
```
删除minio/test文件桶目前只能通过浏览器打开管理控制台删除

