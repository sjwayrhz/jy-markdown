## Install

```
git clone https://github.com/acmesh-official/acme.sh.git
cd acme.sh
./acme.sh --install  \
--home ~/myacme \
--config-home ~/myacme/data \
--cert-home  ~/mycerts \
--accountemail  "my@example.com" \
--accountkey  ~/myaccount.key \
--accountconf ~/myaccount.conf \
--useragent  "this is my client."
```

You don't need to set them all, just set those ones you care about.

Explanations :

- `--home` is a customized dir to install `acme.sh` in. By default, it installs into `~/.acme.sh`
- `--config-home` is a writable folder, acme.sh will write all the files(including cert/keys, configs) there. By default, it's in `--home`
- `--cert-home` is a customized dir to save the certs you issue. By default, it's saved in `--config-home`.
- `--accountemail `is the email used to register an account to Let's Encrypt, you will receive a renewal notice email here.
- `--accountkey` is the file saving your account private key. By default, it's saved in `--config-home`.
- `--user-agent` is the user-agent header value used to send to Let's Encrypt.
- `--nocron` install acme.sh without cronjob



```bash
$ git clone https://gitlab.sjhz.tk/caobo/acme.sh.git 
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

单点签证

```bash
$ acme.sh --issue --dns dns_dp -d test.sjhz.tk
```

多点签证

```bash
$ acme.sh --issue --dns dns_dp -d test1.sjhz.tk -d test2.sjhz.tk
```

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

为Nginx安装

```bash
$ acme.sh --install-cert -d gitlab.sjhz.tk \
  --key-file       /etc/nginx/ssl/gitlab.sjhz.tk.key  \
  --fullchain-file /etc/nginx/ssl/fullchain.cer \
  --reloadcmd     "service nginx force-reload"
```







