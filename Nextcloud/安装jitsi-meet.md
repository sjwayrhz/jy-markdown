### 安装docker

```bash
$ wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_docker.sh | sh
```

### 安装docker-compose

```bash
$ sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

### 加载compose配置文件

```bash
$ dnf install -y git
$ git clone https://github.com/jitsi/docker-jitsi-meet
$ cd docker-jitsi-meet
$ cp env.example .env
$ echo "ENABLE_XMPP_WEBSOCKET=0" >> .env
$ ./gen-passwords.sh
```

### 启动docker-compose

```bash
$ sed -i 's/latest/stable-6173/g' docker-compose.yml
$ docker-compose up -d 
```



### 启动docker-nginx

```bash
$ docker run -d \
--name nginx \
--restart=always \
--net=host \
-v /opt/docker-nginx/log:/var/log/nginx \
-v /opt/docker-nginx/ssl:/etc/nginx/ssl \
-v /opt/docker-nginx/conf.d:/etc/nginx/conf.d \
nginx:1.21.6
```

拷贝证书到ssl目录下

```bash
$ ll /opt/docker-nginx/ssl
total 8
-rw------- 1 root root 1700 Mar  4 13:07 star_juneyaokc_com.key
-rw------- 1 root root 4012 Mar  4 13:07 star_juneyaokc_com.pem
```

创建 /opt/docker-nginx/conf.d/x.conf 文件

```nginx
server {
	listen        80;
	server_name   test.juneyaokc.com;
	if ($scheme != "https") {
            return 301 https://$host$request_uri;
        }
}

server {
	client_max_body_size 100m;

	listen        443	ssl;

	ssl_certificate         /etc/nginx/ssl/star_juneyaokc_com.pem;
	ssl_certificate_key     /etc/nginx/ssl/star_juneyaokc_com.key;

	server_name   test.juneyaokc.com;
	access_log    /var/log/nginx/test.juneyaokc.com.log  main;

	location / {
		proxy_pass http://10.220.62.57:8000/;
		proxy_set_header   Host    $host;
		proxy_set_header   X-Real-IP   $remote_addr;
		proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
	}
}
```

重启docker-nginx

