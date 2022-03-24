安装docker 

```bash
 $ wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_docker.sh | sh
```

新建 /opt/docker-nginx/conf.d/default.conf

```nginx
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

部署nginx

```bash
$ docker run -d \
--name nginx \
--restart=always \
-p 80:80 -p 443:443 \
-v /opt/docker-nginx/log:/var/log/nginx \
-v /opt/docker-nginx/ssl:/etc/nginx/ssl \
-v /opt/docker-nginx/conf.d:/etc/nginx/conf.d \
nginx:1.21.6
```

创建软连接

```bash
$ ln -s  /opt/docker-nginx/conf.d/ .
$ ln -s  /opt/docker-nginx/ssl .
$ ln -s  /opt/docker-nginx/log .
```

