# How to Install NextCloud 14 on CentOS 7

[TOC]

## Prerequisite

- CentOS 7
- SSH access with root privileges
- PHP 7 or above

## Step 1: Log in to your server via SSH:

```
# ssh root@IP_Address -p Port_number
```

Before starting, enter the command below to check whether you have the proper version of CentOS installed on your machine:

```
# cat /etc/redhat-release
```

which should give you the underneath output:

CentOS Linux release 7.5.1804 (Core)

## Step 2: Update the system

Make sure your server is fully up to date:

```
# yum update
```

If your system has not been updated for a while, it will take a few minutes to completed.

```
# yum install -y epel-release
```

## Step 3: Install Web Server

In this step, you can choose whether you want to install Apache or nginx.

### Install Nginx web server

```
# yum install yum-utils
```

create repo files for nginx

```
# vu /etc/yum.repos.d/nginx.repo 
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
```

enable mainline nginx

```
# yum-config-manager --enable nginx-mainline
```

install nginx

```
# yum install nginx
```



Enable nginx to start on boot and start the service using:

```
# systemctl enable --now nginx
```

### Install Apache web server

If you would like to choose Apache instead of nginx, you can skip nginx installation above then install Apache.

```
# yum install httpd
```

Enable Apache to start on boot and start the service using:

```
# systemctl enable httpd

# systemctl start httpd
```

## Step 4: Install PHP 7

The default PHP version on CentOS 7 is PHP 5.4 and Nextcloud 14 requires PHP 7 or above, in this step we will install PHP version 7.

### Install Remi and EPEL repository packages:

remove php in old version

```
# yum remove php*
```

```
# rpm -Uvh http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

install  yum-utils

```
# yum install yum-utils -y
```

### Enable Remi PHP 7 repo:

```
# yum-config-manager --enable remi-php72
```

and install PHP 7 and several PHP modules required by Nextcloud by executing the following command:

```
# yum install php php-mysql php-pecl-zip php-xml php-mbstring php-gd php-fpm php-intl
```

Now, let’s find the following strings in /etc/php-fpm.d/www.conf

```
# vi /etc/php-fpm.d/www.conf
```

```
user = apache
group = apache
```

Replace the values with

```
user = nginx
group = nginx
```

Then, change the permission for PHP session directory, you need to skip this step if you want to use Apache instead of nginx.

```
# ll /var/lib/php/
total 0
drwxrwx---. 2 root apache 6 Apr 13  2018 session

# chown -R root:nginx /var/lib/php/session/
```

Finally, restart php-fpm

```
# systemctl restart php-fpm
```



## Step 5: Install MariaDB database server

```
# vi /etc/yum.repos.d/MariaDB.repo

# MariaDB 10.3 CentOS repository list - created 2018-10-08 02:47 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.3/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1

# yum install MariaDB-server MariaDB-client
# systemctl enable --now mariadb
# systemctl status mariadb
```

At this point, MariaDB is running and we are now going to create a password for the root user. Run the following command to create a root password, remove the test database, remove the anonymous user then reload the privileges.

```
# mysql_secure_installation
```

Once created, you can test the password by invoking this command, you will be asked for the password:

```
# mysql -u root -p
```

## Step 6: Create a database

```
# mysql -uroot -p -e "CREATE DATABASE nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci"
# mysql -uroot -p -e "GRANT ALL on nextcloud.* to sjwayrhz@localhost identified by 'vwv56ty7'"
# mysql -uroot -p -e "FLUSH privileges"
```

## Step 7: Configure Web Server

In the previous step, you chose a web server to install, now you will need to configure it.

#### Nginx configuration

If you want to use nginx, please create a configuration file for the nginx server block

```
# vi /etc/nginx/conf.d/yourdomain.com.conf

upstream php {
	server 127.0.0.1:9000;
	}

server {
	server_name yourdomain.com;

	add_header X-Content-Type-Options nosniff;
	add_header X-XSS-Protection "1; mode=block";
	add_header X-Robots-Tag none;
	add_header X-Download-Options noopen;
	add_header X-Permitted-Cross-Domain-Policies none;

	# Path to the root of your installation
	root /var/www/nextcloud/;

		location = /robots.txt {
		allow all;
		log_not_found off;
		access_log off;
	}

	location = /.well-known/carddav {
		return 301 $scheme://$host/remote.php/dav;
	}

	location = /.well-known/caldav {
		return 301 $scheme://$host/remote.php/dav;
	}

	# set max upload size
	client_max_body_size 512M;
	fastcgi_buffers 64 4K;

	# Enable gzip but do not remove ETag headers
	gzip on;
	gzip_vary on;
	gzip_comp_level 4;
	gzip_min_length 256;
	gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
	gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

	location / {
		rewrite ^ /index.php$request_uri;
	}

	location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)/ {
		deny all;
	}
	location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) {
		deny all;
	}

	location ~ ^/(?:index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|ocs-provider/.+)\.php(?:$|/) {
		fastcgi_split_path_info ^(.+?\.php)(/.*)$;
		include fastcgi_params;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		fastcgi_param PATH_INFO $fastcgi_path_info;
		fastcgi_param HTTPS on;
		#Avoid sending the security headers twice
		fastcgi_param modHeadersAvailable true;
		fastcgi_param front_controller_active true;
		fastcgi_pass php;
	
		fastcgi_intercept_errors on;
		fastcgi_request_buffering off;
	}

	location ~ ^/(?:updater|ocs-provider)(?:$|/) {
		try_files $uri/ =404;
		index index.php;
	}

	# Adding the cache control header for js and css files
	# Make sure it is BELOW the PHP block
	location ~ \.(?:css|js|woff|svg|gif)$ {
		try_files $uri /index.php$request_uri;
		add_header Cache-Control "public, max-age=15778463";

		add_header X-Content-Type-Options nosniff;
		add_header X-XSS-Protection "1; mode=block";
		add_header X-Robots-Tag none;
		add_header X-Download-Options noopen;
		add_header X-Permitted-Cross-Domain-Policies none;
		# Optional: Don't log access to assets
		access_log off;
	}

	location ~ \.(?:png|html|ttf|ico|jpg|jpeg)$ {
		try_files $uri /index.php$request_uri;
		# Optional: Don't log access to other assets
		access_log off;
	}
}
```

Test nginx configuration file, then restart the service

```
# nginx -t
# systemctl restart nginx
```

#### Apache configuration

Create a virtual host configuration file for the domain you want to use to host Nextcloud.

```
# vi /etc/httpd/conf.d/yourdomain.com.conf

<VirtualHost *:80>

ServerAdmin admin@yourdomain.com
DocumentRoot /var/www/nextcloud
ServerName yourdomain.com
ServerAlias www.yourdomain.com

<Directory /var/www/html/nextcloud>
Options +FollowSymlinks
AllowOverride All

<IfModule mod_dav.c>
Dav off
</IfModule>

SetEnv HOME /var/www/nextcloud
SetEnv HTTP_HOME /var/www/nextcloud
</Directory>

ErrorLog /var/log/httpd/nextcloud-error_log
CustomLog /var/log/httpd/nextcloud-access_log common

</VirtualHost>
```

Go to Nextcloud’s official website and download the latest stable release of the application

```
wget https://download.nextcloud.com/server/releases/nextcloud-21.0.0.zip
```

unpack the downloaded zip archive to the document root directory on your server

```
# unzip nextcloud-21.0.0.zip -d /var/www/
# mkdir /var/www/nextcloud/data
# chown -R nginx: /var/www/nextcloud

If you chose Apache, then you need to set the permission for Apache user
# chown -R apache: /var/www/nextcloud

You can now proceed with Nextcloud 14 installation via web installer at http://yourdomain.com, fill the blank as required, then click on the “Finish setup” button to finish it.
```

It is recommended to run the Nextcloud 14 in https mode. We will need to install an SSL certificate for this. In this step, we will show you how to install an SSL certificate from Letsencrypt.

```
# yum install epel-release
# yum install certbot-nginx certbot-apache
# certbot
```

You will be asked for your email address then you need to agree with the ToS to proceed with the certificate installation.

If there is no issue when requesting the certificate, Certbot will automatically edit your existing nginx server block to install the certificate.

At this point, you can access your Nextcloud 14 installation on https://yourdomain.com

And that’s it, with the last step of this tutorial we have successfully installed Nextcloud 14 on your CentOs 7 and you can log in with the login credentials of your admin user. For more information, you can [visit the official documentation of Nextcloud 14](https://nextcloud.com/support).

RoseHosting has been listed as a [recommended ](https://nextcloud.com/providers/)Nextcloud hosting provider on the Nextcloud.com. If you want to try our [fully managed ](https://www.rosehosting.com/nextcloud-hosting.html)Nextcloud VPS hosting, use the coupon code: **50FIRST** to get 50% off your first-month invoice. We have 7 days money-back guarantee. If you are one of our clients, you don’t have to install Nextcloud 14 on CentOS 7, you can simply ask our system administrators to install and configure your Nextcloud instance on CentOS or any other Linux OS. They are available 24×7 and will take care of your request immediately.

------

**PS**. If you liked this post, on How To Install Nextcloud 14 on CentOS 7, please share it with your friends on the social networks using the buttons on the left or simply leave a reply below. Thanks.





### 编辑个人云盘文件自动发送到邮箱

- 安装sendmail ,mailx ，做出自动发邮件。
- 搭建nextcloud云服务

压缩打包，带时间戳

```
# zip -r /opt/nextcloud_data_$(date +%Y%m%d).zip /var/www/nextcloud/data/sjwayrhz/files/
```

脚本

```
# vi /opt/nxsuccess.txt

If you have recieved this message ,that means the backup_sever is still work on. 
	
	
	
	
											Thank you ! 

# vi /opt/nxfailed.txt

failed

```



```
#!/bin/bash

cd /var/www/nextcloud/data/sjwayrhz/files/ && zip -r /opt/note/nextcloud_data_$(date +%Y%m%d).zip *

if [ -f "/opt/note/nextcloud_data_$(date +%Y%m%d).zip" ];then
    mail -s nextcloud_bakup_$(date +%Y%m%d) -a /opt/note/nextcloud_data_$(date +%Y%m%d).zip taoistmonk@163.com < /opt/nxsuccess.txt
else
    mail -s nextcloud taoistmonk@163.com < /opt/nxfailed.txt
fi

find /opt/note/ -ctime +40 -type f | xargs rm -rf
```



定时任务

```
# crontab -l 

0 17 * * * /opt/send.sh
```





opt文件夹下的文件

```
# ls /opt
note  nxfailed.txt  nxsuccess.txt  send.sh
```