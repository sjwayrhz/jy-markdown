# 安装postgres

下载docker

```bash
$ wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_docker.sh | sh
```

安装postgres

```bash
$ docker run -d \
  --restart=always \
  -p 5432:5432 --name=postgresql \
  -v /var/lib/postgresql/data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=123 \
  postgres:13.6-alpine3.15
```

开放端口

```bash
$ firewall-cmd --permanent --add-port=5432/tcp
$ firewall-cmd --reload
```

创建效果

数据库名称：postgres

端口： 5432

用户名： postgres

密码：123