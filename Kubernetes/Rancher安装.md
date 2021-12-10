快速启动

```
docker run -d --restart=unless-stopped \
  --privileged \
  -p 8080:80 -p 8443:443 \
  -v /var/lib/rancher:/var/lib/rancher/ \
  rancher/rancher:v2.5.11-patch1
```

配置nginx
```
$ dnf install yum-utils -y
```
设置nginx repo
```
$ tee /etc/yum.repos.d/nginx.repo <<-'EOF'
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
EOF
```
安装nginx
```
dnf install nginx
```
配置nginx
```
map $http_upgrade $connection_upgrade {
        default Upgrade;
        ''      close;
}
 
 
server {
    listen               443 ssl;
    server_name          rancher.juneyaokc.com;
 
    ssl_certificate      /etc/nginx/cert/star_juneyaokc_com.pem;
    ssl_certificate_key  /etc/nginx/cert/star_juneyaokc_com.key;
 
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;
 
    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
 
    add_header Access-Control-Allow-Origin *;
    
    location / {
        proxy_pass https://127.0.0.1:8443;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_read_timeout 900s;
    }   
}
```




