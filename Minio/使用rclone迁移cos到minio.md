## 使用rclone迁移cos到minio

[TOC]

### 资源准备

1. mc操纵机器，10.220.62.50
2. minio私有服务器，10.220.62.37
3. 腾讯云cos



### 腾讯云资料整理

key

```
access_key_id = AKIDb5zlRz1YlrlT2QFfU7ZptnbIn6EALSo0 
secret_access_key = 8sUuCJF0Zx8BSdcSALRL3386B9wX1JQL
```

访问域名

```
APP_ID = 1255850199L
BucketName = windfindtech

<BucketName-APPID>.cos.ap-shanghai.myqcloud.com
windfindtech-1255850199L.cos.ap-shanghai.myqcloud.com
```

region

```
region = sh
```



### rclone配置文件

```bash
tee ~/.config/rclone/rclone.conf << 'EOF'
[cos] 
type = s3 
provider = Other
env_auth = false
access_key_id = AKIDb5zlRz1YlrlT2QFfU7ZptnbIn6EALSo0 
secret_access_key = 8sUuCJF0Zx8BSdcSALRL3386B9wX1JQL
region = sh
endpoint = cos.ap-shanghai.myqcloud.com
location_constraint =
server_side_encryption =
[m37] 
type = s3
provider = Minio
env_auth = false
access_key_id = wBh26XzE7Qq5rdP
secret_access_key = wEVp8SZ225p5zVVMHQsXfHtUQGvJTx
region = us-east-1
endpoint = http://10.220.62.37:9000
location_constraint =
server_side_encryption =
acl = public-read-write
EOF
```

#### 克隆单个文件

```bash
sudo rclone -P  sync cos:windfindtech-1255850199/JuneYaoAPP.apk m37:/test/JuneYaoAPP.apk
```

#### 克隆文件桶

```bash
sudo rclone -P  sync cos:h5-1255850199 m37:/h5t
sudo rclone -P  sync cos:windfindtech-1255850199 m37:/windfindtech
```

