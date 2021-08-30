1. 安装epel-release
```
dnf install epel-release
```

2. 安装certbot相关组件
```
dnf install certbot python3-certbot-nginx
```

3. 根据需求生成nginx证书
```
certbot --nginx
```

4. 自动更新
```
crontab -e

0 0 1 * * /usr/bin/certbot renew --force-renewal
```