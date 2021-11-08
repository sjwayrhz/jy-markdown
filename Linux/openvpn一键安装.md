# 一键安装openvpn并配置使用账号密码登陆

下载脚本：

```
wget https://raw.githubusercontent.com/Nyr/openvpn-install/master/openvpn-install.sh
```

一键安装：

```
sh openvpn-install.sh
```

假设在此创建了soul账户

soul用户的客户端配置文件路径：/root/soul.ovpn

将此文件导入到openvpn中，连接即可。

## 配置使用账号密码验证

### 创建脚本:vim /etc/openvpn/checkpsw.sh

```
#!/bin/sh
###########################################################
# checkpsw.sh (C) 2004 Mathias Sundman <mathias@openvpn.se>
#
# This script will authenticate OpenVPN users against
# a plain text file. The passfile should simply contain
# one row per user with the username first followed by
# one or more space(s) or tab(s) and then the password.

PASSFILE="/etc/openvpn/psw-file"
LOG_FILE="/etc/openvpn/openvpn-password.log"
TIME_STAMP=`date "+%Y-%m-%d %T"`

###########################################################

if [ ! -r "${PASSFILE}" ]; then
  echo "${TIME_STAMP}: Could not open password file \"${PASSFILE}\" for reading." >> ${LOG_FILE}
  exit 1
fi

CORRECT_PASSWORD=`awk '!/^;/&&!/^#/&&$1=="'${username}'"{print $2;exit}' ${PASSFILE}`

if [ "${CORRECT_PASSWORD}" = "" ]; then 
  echo "${TIME_STAMP}: User does not exist: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
  exit 1
fi

if [ "${password}" = "${CORRECT_PASSWORD}" ]; then 
  echo "${TIME_STAMP}: Successful authentication: username=\"${username}\"." >> ${LOG_FILE}
  exit 0
fi

echo "${TIME_STAMP}: Incorrect password: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
exit 1
```

### 添加权限

```
chmod 755 /etc/openvpn/checkpsw.sh
```

### 添加账号密码

```
echo 'username1 password1' >> /etc/openvpn/psw-file
```

### 修改/etc/openvpn/server/server.conf

```
# 追加以下内容
script-security 3
auth-user-pass-verify /etc/openvpn/checkpsw.sh via-env
username-as-common-name
verify-client-cert none
log /var/log/openvpn.log
```

### 重启服务

```
systemctl restart openvpn-server@server
```

### 修改客户端文件soul.ovpn

```
# 追加以下内容,<cert>和<key>部分可以删掉
auth-user-pass
```

客户端填写用户名密码,即可使用

### 添加用户

```
vi /etc/openvpn/psw-file
```

在这个文件下添加即可

### 设置局部路由

主要由 route-nopull、vpn_gateway、net_gateway 三个参数决定

> route-nopull

当客户端加入这个参数后,openvpn 连接后不会添加路由,也就是不会有任何网络请求走 
> vpn_gateway

当客户端加入 route-nopull 后,所有出去的访问都不从 Openvpn 出去,但可通过添加 vpn_gateway 参数使部分IP访问走 Openvpn 出去
```
route 192.168.1.0 255.255.0.0 vpn_gateway
route 172.121.0.0 255.255.0.0 vpn_gateway
```
> net_gateway

这个参数和 vpn_gateway 相反,表示在默认出去的访问全部走 Openvpn 时,强行指定部分IP访问不通过 Openvpn 出去. max-routes 参数表示可以添加路由的条数,默认只允许添加100条路由,如果少于100条路由可不加这个参数.
```
max-routes 1000
route 172.121.0.0 255.255.0.0 net_gateway
```
比较常用做法是在客户端配置文件中加上 route-nopull 再使用 vpn-gateway 逐条添加需要走 Openvpn 的 ip。

设置如下：
```
#block-outside-dns
route-nopull
#以下路由根据自己实际情况进行添加调整
route 172.16.0.0 255.255.0.0 vpn_gateway
route 172.17.0.0 255.255.0.0 vpn_gateway
```
遇到的问题

Linux 上没有问题，但是在 Win 10 上 OpenVPN，配置中写入了route-nopull，发现没有用，因为发现所有流量都走了 VPN。

可以通过route 命令进行查看

以管理员的身份运行CMD,打开CMD运行界面。首先分析路由情况，打印路由表。输入如下的命令：route print -4解决：

如果设置了 block-outside-dns

这样 OpenVPN 会添加 Windows 防火墙记录，拦掉除 tap 以外的所有网络接口上的 DNS 请求。需要把这行从你配置文件中删掉