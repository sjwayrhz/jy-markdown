# 生成自签名 SSL 证书的具体过程

[TOC]

### 修改 openssl 配置文件

```bash
$ vim /etc/pki/tls/openssl.cnf
# CN 表示后续生成的子证书的对应项必须和创建根证书时填的值一样，否则报错。以下配置只规定子证书的 countryName 必须和根证书一致。
# commonName 表示需要申请的域名
[ policy_match ] 段配置改成如下：
...
policy          = policy_match

# For the CA policy
[ policy_match ]
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

# For the 'anything' policy
# At this point in time, you must list all acceptable 'object'
# types.
[ policy_anything ]
countryName             = CN
stateOrProvinceName     = shanghai
organizationName        = juneyao
organizationalUnitName  = juneyaokc
commonName              = juneyaokc.com
emailAddress            = i@sjhz.cf
...
# 注意：policy_match 可以不用修改，而在policy_anything 中做修改操作
```

### 进入CA工作目录

```bash
$ mkdir -p /etc/pki/CA/{certs,crl,newcerts,private}
$ cd /etc/pki
```

### 在服务器 pki 的 CA 目录下新建两个文件

```bash
$ touch CA/index.txt CA/serial && echo 00 > CA/serial
```

### 生成CA根证书和密钥

（根据提示输入信息，除了 Country Name 选项需要记住的，后面的随便填）

```bash
$ openssl req -new -x509 -days 36500 -keyout CA/ca.key -out CA/ca.crt -config tls/openssl.cnf
```

例如输入以下字符，密码设置为 `123456` , 也就是 `Enter PEM pass phrase:123456`

```
Generating a RSA private key
.............+++++
........................................+++++
writing new private key to 'CA/ca.key'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:shanghai
Locality Name (eg, city) [Default City]:juneyao
Organization Name (eg, company) [Default Company Ltd]:juneyaokc
Organizational Unit Name (eg, section) []:juneyaokc
Common Name (eg, your name or your server's hostname) []:juneyaokc.com
Email Address []:i@sjhz.cf
```

### 生成密钥文件

首先生产一个带密码的key

```bash
$ cd CA
$ openssl genrsa -out juneyaokc.com.key 2048
```

这个server.key也可以用于nginx,于是拷贝一份juneyaokc.com.key作为nginx密钥

### 生成证书请求文件（CSR）

根据提示输入信息，除了 Country Name 与前面根证书一致外，其他随便填写

密码是之前设置的 `123456` , 也就是``Enter pass phrase for ca.key: 123456`

`A challenge password`和`An optional company name`可以不填写

Common Name 填写要保护的域名，比如：*.qhh.me

```bash
$ openssl req -new -key ca.key -out ca.csr
```

输入的内容为

```
Enter pass phrase for ca.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:shanghai
Locality Name (eg, city) [Default City]:shanghai
Organization Name (eg, company) [Default Company Ltd]:juneyaokc
Organizational Unit Name (eg, section) []:juneyao
Common Name (eg, your name or your server's hostname) []:juneyaokc.com
Email Address []:i@sjhz.cf

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

### 使用 openssl 签署 CSR 请求，生成证书

```bash
$ openssl ca \
  -config ../tls/openssl.cnf \
  -in ca.csr \
  -days 36500 \
  -cert /etc/pki/CA/ca.crt \
  -keyfile /etc/pki/CA/ca.key \
  -out juneyaokc.com.crt

参数项说明：
-in: CSR 请求文件
-days: 生成的证书的有效天数
-cert: 用于签发的根 CA 证书
-keyfile: 根 CA 的私钥文件
-out: 生成证书的文件名
```

至此自签名证书生成完成，最终需要：nginx.key 和 nginx.crt

### 配置 Nginx 使用自签名证书

```
server {
        listen  80;
        server_name     test.juneyaokc.com;
        return  301     https://$host$request_uri;
}
server {
        listen  443 ssl;
        ssl_certificate       /etc/nginx/ssl/juneyaokc.com.crt;
        ssl_certificate_key   /etc/nginx/ssl/juneyaokc.com.key;
        server_name     test.juneyaokc.com;
        location / {
            root /usr/share/nginx/html ;
            index  index.html;
        }
}
```

