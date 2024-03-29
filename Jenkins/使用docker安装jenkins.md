# 使用docker安装jenkins

[TOC]

## 安装docker

```bash
$ wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_docker.sh | sh
```

## 部署jenkins

下载maven

```bash
$ wget -P /tmp https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.8.5/binaries/apache-maven-3.8.5-bin.tar.gz
```

解压

```bash
$ tar -zxvf /tmp/apache-maven-3.8.5-bin.tar.gz -C /usr/local/
```

启动脚本

```bash
$ docker run -d \
	-u root \
	--name jenkins \
	--restart=always \
	-p 8080:8080 -p 50000:5000 \
	-v /etc/localtime:/etc/localtime \
	-v /usr/local/apache-maven-3.8.5/:/usr/local/maven \
	-v /var/jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

启动之后，打开 ip:8080即可浏览jenkins

获得jenkins密码

```bash
$ cat /var/jenkins_home/secrets/initialAdminPassword
37eea732480e4d498a6c474d5e7b6dc6
```

输入密码后，选择安装推荐的插件即可。

nginx反向代理配置文件

```nginx
server {
	listen        80;
	server_name   jenkins.sjhz.tk;
	if ($scheme != "https") {
            return 301 https://$host$request_uri;
        }
}

server {
	client_max_body_size 100m;

	listen        443	ssl;

	ssl_certificate         /etc/nginx/ssl/sjhz.tk.cer;
	ssl_certificate_key     /etc/nginx/ssl/sjhz.tk.key;

	server_name   jenkins.sjhz.tk;
	access_log    /var/log/nginx/jenkins.sjhz.tk.log  main;

	location / {
		proxy_pass http://10.220.62.44:8080/;
		proxy_set_header   Host    $host;
		proxy_set_header   X-Real-IP   $remote_addr;
		proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
	}
}
```



## 优化安装

安装插件

- Locale

- publish over ssh

- Maven Integration

安装插件完成后我们需要配置Maven环境和JDK环境。

点击全局配置后，配置maven为刚才挂载的maven路径，也就是`/usr/local/maven`

参考

```
https://juejin.cn/post/6862497968973742094
```

