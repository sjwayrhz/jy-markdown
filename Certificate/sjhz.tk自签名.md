# 如何创建一个自签名的SSL证书(X509)

[TOC]

## 使用openssl工具生成一个RSA私钥

```bash
 $ openssl genrsa -des3 -out server.key 2048
```

*说明：生成rsa私钥，des3算法，2048位强度，server.key是秘钥文件名。*

注意：生成私钥，需要提供一个至少4位的密码。假设密码为 `1234`。

## 第2步：生成CSR（证书签名请求）

此时可以有两种选择。理想情况下，可以将证书发送给证书颁发机构（CA），CA验证过请求者的身份之后，会出具签名证书（很贵）。另外，如果只是内部或者测试需求，也可以使用OpenSSL实现自签名，具体操作如下：

```bash
$ openssl req -new -key server.key -out server.csr
```

说明：需要依次输入国家，地区，城市，组织，组织单位，Common Name和Email。其中Common Name，可以写自己的名字或者域名，如果要支持https，Common Name应该与域名保持一致，否则会引起浏览器警告。

```
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Shanghai
Locality Name (eg, city) []:Shanghai
Organization Name (eg, company) [Internet Widgits Pty Ltd]:sjwayrhz
Organizational Unit Name (eg, section) []:sjhz
Common Name (e.g. server FQDN or YOUR name) []:sjhz.tk
Email Address []:i@sjhz.cf
```

## 第3步：删除私钥中的密码

在第1步创建私钥的过程中，由于必须要指定一个密码。而这个密码会带来一个副作用，那就是在每次Apache启动Web服务器时，都会要求输入密码，这显然非常不方便。要删除私钥中的密码，操作如下：

```bash
$ cp server.key server.key.org
$ openssl rsa -in server.key.org -out server.key
$ rm -f server.key.org
```

## 第4步：生成自签名证书

如果你不想花钱让CA签名，或者只是测试SSL的具体实现。那么，现在便可以着手生成一个自签名的证书了。

需要注意的是，在使用自签名的临时证书时，浏览器会提示证书的颁发机构是未知的。

```bash
$ openssl x509 -req -days 36500 -in server.csr -signkey server.key -out server.crt
```

*说明：crt上有证书持有人的信息，持有人的公钥，以及签署者的签名等信息。当用户安装了证书之后，便意味着信任了这份证书，同时拥有了其中的公钥。证书上会说明用途，例如服务器认证，客户端认证，或者签署其他证书。当系统收到一份新的证书的时候，证书会说明，是由谁签署的。如果这个签署者确实可以签署其他证书，并且收到证书上的签名和签署者的公钥可以对上的时候，系统就自动信任新的证书。*

验证

```bash
$ openssl x509 -noout -modulus -in server.csr | openssl md5

unable to load certificate
140126698313536:error:0909006C:PEM routines:get_name:no start line:crypto/pem/pem_lib.c:745:Expecting: TRUSTED CERTIFICATE
(stdin)= d41d8cd98f00b204e9800998ecf8427e

$ openssl x509 -noout -modulus -in server.key | openssl md5

unable to load certificate
140264055646016:error:0909006C:PEM routines:get_name:no start line:crypto/pem/pem_lib.c:745:Expecting: TRUSTED CERTIFICATE
(stdin)= d41d8cd98f00b204e9800998ecf8427e
```



## 第5步：安装私钥和证书

将私钥和证书文件复制到Apache的配置目录下即可。

```nginx
server {
        listen  80;
        server_name     test.sjhz.tk;
        return  301     https://$host$request_uri;
}
server {
        listen  443 ssl;
        ssl_certificate       /etc/nginx/ssl/sjhz.tk.crt;
        ssl_certificate_key   /etc/nginx/ssl/sjhz.tk.key;
        server_name     test.sjhz.tk;
        location / {
            root /usr/share/nginx/html ;
            index  index.html;
        }
}
```

