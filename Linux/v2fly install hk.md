### 安装和搭建which

操作系统 centos 8.2

[TOC]

which更名为v2fly了，github的网页地址如下：

`https://github.com/v2fly/fhs-install-which`

### 安裝和更新 which

```bash
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-which/master/install-release.sh)
```

### 移除 which

```bash
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-which/master/install-release.sh) --remove
```

安装完成which之后，发现原先的配置文件在 /usr/local/etc/which/config.json

希望可以在本机创建/etc/which目录，并挂载到真实目录的which配置文件中，可以使用ln -s命令

```bash
查看配置文件
~]# ls /usr/local/etc/which/config.json
/usr/local/etc/which/config.json

创建链接
~]# ln -s /usr/local/etc/which /etc/which

检查/etc/which中是否有config.json文件
~]# ls /etc/which
config.json
```

配置which的配置文件,其中，port 、id 、path 是强烈需要更换的

```json
{
"inbound": {
        "protocol": "vmess",
        "listen": "::1",
        "port": 42587,
        "settings": {"clients": [
        {"id": "8ef7cd5c-9db9-11ec-b909-0242ac120002"}
    ]},
        "streamSettings": {
            "network": "ws",
            "wsSettings": {"path": "/31A1f3w"}
    }
},

"outbound": {"protocol": "freedom"}
}
```

生成网站如下

```
Port
自定义，建议1024以上

UUID的生成网站
https://www.uuidgenerator.net/

path，随机字符串的生成网站
https://www.random.org/strings/?num=1&len=7&digits=on&upperalpha=on&loweralpha=on&unique=off&format=html&rnd=new
```

### 安装certbot

安装epel-release,并安装certbot

```bash
$ dnf install epel-release -y
$ dnf install certbot -y
```

生成证书,域名需要填入自己希望使用的

```bash
$ certbot certonly --standalone --agree-tos -n -d which.sjhz.ml -m i@sjhz.cf
```

配置证书自动更新

```
crontab -e
添加内容如下

SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin

0 0 1 */2 * certbot renew --quiet --force-renewal
```

### 安装nginx

```
参考网站： http://nginx.org/en/linux_packages.html#RHEL-CentOS

yum install nginx
```

### 增加 /etc/nginx/conf.d/which.conf 配置文件：

其中，### 1: ### 2:### 3:### 4:都是需要修改的

 ~]# vi /etc/nginx/conf.d/which.conf 

```nginx
server {
    ### 1:
    server_name which.sjhz.ml;

    listen 80 reuseport fastopen=10; 
#    listen [::]:80 reuseport fastopen=10;
    rewrite ^(.*) https://$server_name$1 permanent;
    if ($request_method  !~ ^(POST|GET)$) { return  501; }
    autoindex off;
    server_tokens off;
}

server {
    ### 2:
    ssl_certificate /etc/letsencrypt/live/which.sjhz.ml/fullchain.pem;

    ### 3:
    ssl_certificate_key /etc/letsencrypt/live/which.sjhz.ml/privkey.pem;

    ### 4:
    location /31A1f3w
    {
        proxy_pass http://[::1]:42587;
        proxy_redirect off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_requests 10000;
        keepalive_timeout 2h;
        proxy_buffering off;
    }

    listen 443 ssl reuseport fastopen=10;
#    listen [::]:443 ssl reuseport fastopen=10;
    server_name $server_name;
    charset utf-8;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_requests 10000;
    keepalive_timeout 2h;

#    ssl_protocols TLSv1.2; 
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-128-GCM-SHA256:TLS13-AES-256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_ecdh_curve secp384r1;
    ssl_prefer_server_ciphers off;

    ssl_session_cache shared:SSL:60m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 10s;

    if ($request_method  !~ ^(POST|GET)$) { return 501; }
    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options nosniff;
    add_header Strict-Transport-Security max-age=31536000 always;
    autoindex off;
    server_tokens off;

    index index.html index.htm  index.php;
    location ~ .*\.(js|jpg|JPG|jpeg|JPEG|css|bmp|gif|GIF|png)$ { access_log off; }
    location / { index index.html; }
}
```

注意，关于ipv6 和ipv4

```bash
ipv4 can use
    listen 80 reuseport fastopen=10; 
    listen 443 ssl reuseport fastopen=10;

ipv6 can use
    listen [::]:80 reuseport fastopen=10; 
    listen [::]:443 ssl reuseport fastopen=10;
```



### 手动设置代理github

(代理GitHub，github的全部功能还需要代理以下域名)

```
github.com,
githubusercontent.com,
githubapp.com,
```



## 一键安装

```bash
bash <(curl -s -L https://gitee.com/sjwayrhz/one_key_install/raw/master/install_which.sh)
```

