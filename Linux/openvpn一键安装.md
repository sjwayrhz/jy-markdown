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
```
route-nopull #不添加路由， 也就是不会有任何网络请求走OpenVPN 

route 192.168.2.0 255.255.255.0 vpn_gateway #指定此網段才走VPN代理

#max-routes 参数表示可以添加路由的条数
max-routes 1000

#net_gateway則與vpn_gateway 相反，它是指定哪些IP不走VPN代理
route 172.121.0.0 255.255.0.0 net_gateway

##請注意，你可能需要註釋或者刪除以下配置，否則可能導致無法DNS(域名)解析
#setenv opt block-outside-dns
```

至此，我們就可以重新連接一下VPN，測試一下配置是否達到預期效果