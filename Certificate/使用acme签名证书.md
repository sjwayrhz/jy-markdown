# acme使用手册

[TOC]

## Install

```bash
$ git clone https://gitee.com/sjwayrhz/acme.sh.git
$ cd acme.sh
```

excute

```bash
$ sh acme.sh --install  \
  --home /usr/local/bin \
  --config-home /usr/local/acme/data \
  --cert-home  /usr/local/acme/certs \
  --accountemail  "i@sjhz.cf" \
  --accountkey  /usr/local/acme/account.key \
  --accountconf /usr/local/acme/account.conf \
  --useragent  "this is my client."
```

## Useful

注册

```bash
$ acme.sh --register-account -m i@sjhz.cf
```

## cloudflare API

Cloudflare Domain API offers two methods to automatically issue certs.

### Using the global API key

```bash
export CF_Key="95989177cdf5347b93bc13a550d049760653f"
export CF_Email="i@sjhz.cf"
```

### Using the new cloudflare api token

Create a token that can read all resources

#### juneyao.cf

export vars

```bash
export CF_Token="oH7tmqW1m8TAKxwebYCo0btP5AUO2b4t8vXhLgCO"
export CF_Account_ID="e1946178dc74948cd7d25ba573a577c0"
export CF_Zone_ID="b19966bec833112dfccbc602c923cfcf"
```

bind dns

```
*.juneyao.cf.	1	IN	A	10.220.62.52
```

then issue a cert

```bash
$ acme.sh --issue --dns dns_cf -d *.juneyao.cf
```

#### caobo.cf

export vars

```bash
export CF_Token="BXR35WxyKcLVhgbUkkEv9d3T8t1j-AcrFEBACmEt"
export CF_Account_ID="93ff417fdc3e39e117ed1ff99f0c0a56"
export CF_Zone_ID="b19966bec833112dfccbc602c923cfcf"
```

bind dns

then , issue a cert

```bash
$ acme.sh --issue --dns dns_cf -d *.caobo.cf
```



## DNSpod API

以下是2022年3月22日申请的API

| 名称 |   ID   |              token               |
| :--: | :----: | :------------------------------: |
| demo | 300838 | d250f22a8fa48a48edc2ad02900af8d2 |

于是使用api签证为

```bash
$ export DP_Id="300838"
$ export DP_Key="d250f22a8fa48a48edc2ad02900af8d2"
```

在dnspod中绑定泛域名到一个acme.sh云主机的内网ip上

例如 *.sjhz.tk绑定到 10.220.62.52

然后，可以使用泛域名签证

```bash
$ acme.sh --issue --dns dns_dp -d *.sjhz.tk
```

## 使用证书

查看证书

```bash
$ ll  /root/.acme.sh/test.sjhz.tk/
total 36
-rw-r--r-- 1 root root 4399 Mar 22 10:26 ca.cer
-rw-r--r-- 1 root root 6680 Mar 22 10:26 fullchain.cer
-rw-r--r-- 1 root root 2281 Mar 22 10:26 test.sjhz.tk.cer
-rw-r--r-- 1 root root  566 Mar 22 10:26 test.sjhz.tk.conf
-rw-r--r-- 1 root root  952 Mar 22 10:23 test.sjhz.tk.csr
-rw-r--r-- 1 root root  147 Mar 22 10:23 test.sjhz.tk.csr.conf
-rw------- 1 root root 1675 Mar 22 10:23 test.sjhz.tk.key
```

尝试的nginx中的defaults.conf

```nginx
server {
    listen 80;
    server_name test.sjhz.tk;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name test.sjhz.tk;

    ssl_certificate /etc/nginx/ssl/fullchain.cer;
    ssl_certificate_key /etc/nginx/ssl/test.sjhz.tk.key;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        # proxy_pass http://127.0.0.1:8080;
        # proxy_http_version 1.1;
        # proxy_set_header Host $proxy_host; 
        # proxy_set_header X-Real-IP $remote_addr; 
        # proxy_set_header X-Forwarded-For $remote_addr;      
    }
}
```

为Nginx安装单域名

```bash
$ acme.sh --install-cert -d test.sjhz.tk \
  --key-file       /etc/nginx/ssl/test.sjhz.tk.key  \
  --fullchain-file /etc/nginx/ssl/fullchain.cer \
  --reloadcmd     "service nginx force-reload"
```

为Nginx安装泛域名

test1

```bash
$ acme.sh --install-cert -d *.sjhz.tk \
  --key-file       /etc/nginx/ssl/test1.sjhz.tk.key  \
  --fullchain-file /etc/nginx/ssl/fullchain.cer \
  --reloadcmd     "service nginx force-reload"
```

test2

```bash
$ acme.sh --install-cert -d *.sjhz.tk \
  --key-file       /etc/nginx/ssl/test2.sjhz.tk.key  \
  --fullchain-file /etc/nginx/ssl/fullchain.cer \
  --reloadcmd     "service nginx force-reload"
```







