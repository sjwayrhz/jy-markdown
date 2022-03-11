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
$ git clone https://gitee.com/sjwayrhz/docker-jitsi-meet.git
$ cd docker-jitsi-meet
$ cp env.example .env
$ ./gen-passwords.sh
```

对.env做如下修改

```
TZ=Asia/Shanghai
PUBLIC_URL=https://jitsi-meet.juneyaokc.com
DOCKER_HOST_ADDRESS=10.220.62.59
```

### 启动docker-compose

```bash
$ sed -i 's/latest/stable-6173/g' docker-compose.yml
$ docker-compose up -d 
```

容器化后会生成 ~/.jitsi-meet-cfg 文件夹，里面是一些配置文件

```bash
$ vim  ~/.jitsi-meet-cfg/web/config.js
```

修改部分

```javascript
//config.resolution = 720;
//config.constraints.video.height = { ideal: 720, max: 720, min: 180 };
//config.constraints.video.width = { ideal: 1280, max: 1280, min: 320};
//config.disableSimulcast = false;
//config.startVideoMuted = 10;
//config.startWithVideoMuted = false;

config.resolution = 1080;
config.constraints.video.height = { ideal: 1080, max: 1080, min: 720 };
config.constraints.video.width = { ideal: 1920, max: 1920, min: 1280};
config.disableSimulcast = true;
config.startVideoMuted = 10;
config.startWithVideoMuted = false;
```

另外一个部分

```
capScreenshareBitrate: 0
```

重启docker-compose

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
	server_name   jitsi-meet.juneyaokc.com;
	if ($scheme != "https") {
            return 301 https://$host$request_uri;
        }
}

server {
	client_max_body_size 100m;

	listen        443	ssl;

	ssl_certificate         /etc/nginx/ssl/star_juneyaokc_com.pem;
	ssl_certificate_key     /etc/nginx/ssl/star_juneyaokc_com.key;

	server_name   jitsi-meet.juneyaokc.com;
	access_log    /var/log/nginx/jitsi-meet.juneyaokc.com.log  main;

	location / {
		proxy_pass http://10.220.62.57:8000/;
		proxy_set_header   Host    $host;
		proxy_set_header   X-Real-IP   $remote_addr;
		proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
	}
}
```

重启docker-nginx

