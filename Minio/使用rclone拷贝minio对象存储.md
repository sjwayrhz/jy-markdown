## 使用rclone拷贝对象存储

[TOC]

### 两个minio单机集群

minio-153资源如下

```
IP: 	10.220.62.153
Port:	9000

MINIO_ACCESS_KEY=Kck0rySNBBsY1rC7
MINIO_SECRET_KEY=9dyGOElSHgAWemBefrUwK8ONsBsWXoAP
```

minio-154资源如下

```
IP: 	10.220.62.153
Port:	9000

MINIO_ACCESS_KEY=wrxFm0UC9CIJPLB5
MINIO_SECRET_KEY=GqyFwfz4IFtxezW0pr1pEogdA2ggeFvS
```

### MC操控机器

添加153的minio

```bash
$ mc config host add m153 http://10.220.62.153:9000 Kck0rySNBBsY1rC7 9dyGOElSHgAWemBefrUwK8ONsBsWXoAP --api s3v4
```

添加154的minio

```bash
$ mc config host add m154 http://10.220.62.154:9000 wrxFm0UC9CIJPLB5 GqyFwfz4IFtxezW0pr1pEogdA2ggeFvS --api s3v4
```

查看现存的host

```bash
$ mc config host list
m153
  URL       : http://10.220.62.153:9000
  AccessKey : Kck0rySNBBsY1rC7
  SecretKey : 9dyGOElSHgAWemBefrUwK8ONsBsWXoAP
  API       : s3v4
  Path      : auto

m154
  URL       : http://10.220.62.154:9000
  AccessKey : wrxFm0UC9CIJPLB5
  SecretKey : GqyFwfz4IFtxezW0pr1pEogdA2ggeFvS
  API       : s3v4
  Path      : auto

m37
  URL       : http://10.220.62.37:9000
  AccessKey : wBh26XzE7Qq5rdP
  SecretKey : wEVp8SZ225p5zVVMHQsXfHtUQGvJTx
  API       : s3v4
  Path      : auto
```

#### 创建文件

在m153中创建test桶，里面添加1.txt，内容为helloworld

```bash
$ touch /tmp/1.txt
$ echo helloworld > /tmp/1.txt
$ cat /tmp/1.txt
$ mc mb m153/test
Bucket created successfully `m153/test`.
$ mc cp /tmp/1.txt m153/test
/tmp/1.txt:                     11 B / 11 B ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 570 B/s 0s
$ mc ls m153/test
[2022-04-01 07:35:46 CST]    11B STANDARD 1.txt
```

#### 创建文件夹

在/tmp目录下创建demo文件夹，在demo文件夹中存放2.txt和3.txt，内容分别为2，3. 然后拷贝demo文件夹到test文件桶

```bash
$ mkdir /tmp/demo
$ echo 2 > /tmp/demo/2.txt
$ echo 3 > /tmp/demo/3.txt
$ mc cp --recursive /tmp/demo m153/test
/tmp/demo/3.txt:                4 B / 4 B ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 133 B/s 0s
$ mc ls m153/test
[2022-04-01 07:35:46 CST]    11B STANDARD 1.txt
[2022-04-01 07:54:27 CST]     0B demo/
$ mc ls m153/test/demo
[2022-04-01 07:53:43 CST]     2B STANDARD 2.txt
[2022-04-01 07:53:43 CST]     2B STANDARD 3.txt
```



### 安装配置rclone

安装

```bash
$ curl https://rclone.org/install.sh | sudo bash
```

使用`rclone config`可以创建rclone的配置文件，默认位置是`/root/.config/rclone/rclone.conf`

### 操作rclone

也可以手动创建配置文件

模拟 将m153中test桶文件1.hellworld克隆到m154

```bash
tee ~/.config/rclone/rclone.conf << 'EOF'
[source] 
type = s3 
provider = Minio
env_auth = false
access_key_id = Kck0rySNBBsY1rC7 
secret_access_key = 9dyGOElSHgAWemBefrUwK8ONsBsWXoAP
region = us-east-1
endpoint = http://10.220.62.153:9000
location_constraint =
server_side_encryption =
[target] 
type = s3
provider = Minio
env_auth = false
access_key_id = wrxFm0UC9CIJPLB5
secret_access_key = GqyFwfz4IFtxezW0pr1pEogdA2ggeFvS
region = us-east-1
endpoint = http://10.220.62.154:9000
location_constraint =
server_side_encryption =
acl = public-read-write
EOF
```

配置文件位于：`~/.config/rclone/rclone.conf`目录下。

#### 迁移文件

```bash
$ sudo rclone -P  sync source:test/1.txt target:test/1.txt
```

#### 迁移文件夹

```bash
$ sudo rclone -P  sync source:test/demo target:test/demo
```

#### 迁移经验

1. source和target为自定义名称，可以任意修改；
2. 迁移的source中的文件桶和路径一定需要是预先存在的，但是target中的文件桶和路径可以不存在，会自动创建的。
