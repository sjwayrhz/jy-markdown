# Nextcloud使用MinIO对象存储作为主存储

nextcloud支持s3对象存储或者任何兼容s3的实现，例如minio、腾讯cos等，将其配置为主要存储，替代默认的文件存储。当对象存储用作主存储时，对象存储中不存储元数据（名称、目录结构等）。元数据仅存储在数据库中，对象存储仅通过唯一标识符保存文件内容。

将对象存储用作主存储通常比使用相同的对象存储作为外部存储时表现更好，但同时也有一些弊端。这种配置只适合全新安装的nextcloud，如果在现有nextcloud上配置的话，当前实例上的所有现有文件将无法访问。同时保存到bucket里面的数据只能通过nextcloud来访问，这样就失去了从nextcloud外部访问文件的能力。

我博客其实写过很多篇关于nextcloud的文章了，但是现在回头看看我之前的配置，或多或少都有一些问题。最近又部署了一次nextcloud，解决了之前遗留的一些问题，遂决定用这篇文章记录一下新的部署过程。开始之前还是不得不吐槽一下nextcloud这货。。。这么多年过去了，还是原来那熟悉的味道：各种小bug，性能拉跨。不知道现在最新的22版本上了php8后有没有性能方面的提升（本文还是使用的21版本）至于为啥要用21不用22，那是因为21依旧还是当前的stable版本。这种本来就bug满天飞的程序，我实在是不敢太冒进。。

比如我这次部署的时候就碰到了这个问题：https://help.nextcloud.com/t/redirect-to-non-existing-page-http-index-php-core-apps-recommended/113558

可以看到这个问题至少半年前就存在了，但到现在还没有修复，因为这个问题害我走了很多弯路，我总觉得是我配置哪里没配置好导致的，直到我搜到上面的网址后才知道这TM是这程序的bug。。然后这次我是用docker部署的，和我预料的一样，即便用的是docker依旧各种蛋疼。这里就不细说我碰到的各种蛋疼问题了。好在最终还是整出了一个自己非常满意的配置。

耳熟能详的步骤，首先装需要用到的软件：nginx/certbot/docker/docker-compose。

```
apt -y update
apt -y install curl nginx python3-certbot-nginx
curl -sSL https://get.docker.com/ | sh
systemctl enable --now nginx docker
curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

新建一个docker-compose配置文件：

```
mkdir /opt/nextcloud && cd /opt/nextcloud && nano docker-compose.yml
```

下面这个compose是目前我用在生产环境的配置：

```yaml
version: '3.5'

services:
  db:
    image: mariadb:10.5
    restart: unless-stopped
    environment:
      - MARIADB_ROOT_PASSWORD=password
      - MARIADB_DATABASE=nextcloud
      - MARIADB_USER=nextcloud
      - MARIADB_PASSWORD=password
    volumes:
      - nextcloud_db:/var/lib/mysql
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW

  redis:
    image: redis:alpine
    restart: unless-stopped

  app:
    image: nextcloud:stable-apache
    restart: unless-stopped
    depends_on:
      - db
      - redis
    environment:
      - NEXTCLOUD_TRUSTED_DOMAINS=cloud.example.com
      - OVERWRITEHOST=cloud.example.com
      - OVERWRITEPROTOCOL=https
      - PHP_MEMORY_LIMIT=2048M
      - PHP_UPLOAD_LIMIT=5120M
      - REDIS_HOST=redis
      - MYSQL_HOST=db
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=password
      - OBJECTSTORE_S3_HOST=121.46.238.135
      - OBJECTSTORE_S3_BUCKET=nextcloud
      - OBJECTSTORE_S3_KEY=wBh26XzE7Qq5rdP
      - OBJECTSTORE_S3_SECRET=wEVp8SZ225p5zVVMHQsXfHtUQGvJTx
      - OBJECTSTORE_S3_PORT=9000
      - OBJECTSTORE_S3_SSL=true
      - OBJECTSTORE_S3_REGION=hosthatch-uk
      - OBJECTSTORE_S3_USEPATH_STYLE=false
    ports:
      - 127.0.0.1:8080:80
    volumes:
      - nextcloud:/var/www/html

  cron:
    image: nextcloud:stable-apache
    restart: unless-stopped
    depends_on:
      - db
      - redis    
    volumes:
      - nextcloud:/var/www/html
    entrypoint: /cron.sh

volumes:
  nextcloud:
  nextcloud_db:
```

整这一套compose出来真不容易，可以说满满的全是细节（坑）。。

1.首先是这沙雕程序不兼容最新的mariadb10.6：https://github.com/nextcloud/docker/issues/1536。我踩坑后才知道，遂把容器的mariadb版本锁定到10.5。

2.反向代理后访问提示你正在从不信任的域名访问balabala，加上NEXTCLOUD_TRUSTED_DOMAINS变量解决。

3.能访问了，输入账号密码登录又不跳转：https://github.com/nextcloud/docker/issues/1447。加上OVERWRITEHOST以及OVERWRITEPROTOCOL变量解决。

4.更神奇的是加上OVERWRITEHOST以及OVERWRITEPROTOCOL变量后还能解决之前提到的问题：https://help.nextcloud.com/t/redirect-to-non-existing-page-http-index-php-core-apps-recommended/113558

5.增加php脚本占用内存，防止一些操作被中断：PHP_MEMORY_LIMIT=2048M

6.上传大文件：PHP_UPLOAD_LIMIT=5120M

7.增加redis缓存和cron计划任务的配置，提升性能。尤其是这个cron，玩过nextcloud的都知道默认的ajax计划任务非常拉跨。

注意事项：

1.我的minio对象存储bucket启用了virtual-host-style，如果你也是自建的minio，默认情况下minio对象存储的bucket是path-style，此时你需要修改这个变量OBJECTSTORE_S3_USEPATH_STYLE为true。

2.在compose内设置的所有这些环境变量，删除后不会从nextcloud的配置文件内删除，也就是说删除了变量实际上依旧生效。

3.在compose内设置环境变量只在第一次启动容器的时候有效，后续增加新的变量不会应用到nextcloud的配置文件内。也就是说新增变量不会生效。

在容器启动后要修改nextcloud的配置，可以编辑这个配置文件：

```
nano /var/lib/docker/volumes/nextcloud_nextcloud/_data/config/config.php
```

确认配置无误后启动：

```
docker-compose up -d
```

新建nginx反代配置文件：

```
nano /etc/nginx/conf.d/nextcloud.conf
```

写入如下配置：

```
upstream nextcloud {
  server 127.0.0.1:8080;
}

server {
  listen 80;
  server_name cloud.example.com;
  client_max_body_size 0;
  add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;" always;

  location / {
    proxy_pass http://nextcloud;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme; 
  }

  location /.well-known/carddav {
    return 301 $scheme://$host/remote.php/dav;
  }

  location /.well-known/caldav {
    return 301 $scheme://$host/remote.php/dav;
  }
}
```

签发ssl证书：

```
certbot --nginx
```

如果正常的话，现在访问你的域名即可创建你的nextcloud管理员账号。

登录后还有一些问题需要解决。在后台的安全设置和警告，提示：此实例中的 php-imagick 模块不支持 SVG。为了获得更好的兼容性，建议安装它。

具体问题：https://github.com/nextcloud/docker/issues/1414。临时的解决办法，在当前容器内直接安装libmagickcore-6.q16-6-extra：

```
docker-compose exec app apt update
docker-compose exec app apt install libmagickcore-6.q16-6-extra
```

永久解决办法，自己用dokcerfile构建私有镜像，这种步骤略复杂这里就不详细说了。

当nextcloud有新版本时，切记不要用后台设置里面的更新器更新，而是通过pull新的docker镜像来更新：

```
docker-compose pull
docker-compose up -d
```

日常维护nextcloud，使用自带的occ命令行工具：

```
docker-compose exec --user www-data app php occ
```

在nextcloud试着传一些文件，可以看到minio的bucket里面存的文件类型：

可以看到这些数据只能通过nextcloud来访问。

如果你不想把对象存储当作nextcloud的主存储来用的话可以直接去掉compose里面那些OBJECTSTORE_S3开头的变量，照样是一个完全可以用在生产环境的配置。

另外nextcloud也可以通过外部存储的方式来使用对象存储，只是通过外部存储的话在nextcloud里面就只有那一个目录可以用。

在最后其实我还有几个需求没有完成，一个是备份，这个compose里面用到的都是命名卷，备份起来略麻烦一些。再者就是我还是想装个ocdownloader上去配合aria2实现离线下载，但是这一套配置要想整合进官方的这个docker镜像的话也比较麻烦，等有空再折腾一下。可能未来几天会水一下这方面的配置。

`https://lala.im/8072.html`

