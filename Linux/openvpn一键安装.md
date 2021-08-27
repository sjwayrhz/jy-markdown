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

1\.创建脚本:vim /etc/openvpn/checkpsw.sh

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

2\.添加权限

```
chmod 755 /etc/openvpn/checkpsw.sh
```

3\.添加账号密码

```
echo 'username1 password1' >> /etc/openvpn/psw-file
```

4\.修改/etc/openvpn/server/server.conf

```
# 追加以下内容
script-security 3
auth-user-pass-verify /etc/openvpn/checkpsw.sh via-env
username-as-common-name
verify-client-cert none
log /var/log/openvpn.log
```

5\.重启服务

```
systemctl restart openvpn-server@server
```

6\.修改客户端文件soul.ovpn

```
# 追加以下内容,<cert>和<key>部分可以删掉
auth-user-pass
```

客户端填写用户名密码,即可使用

7\.添加用户

```
vi /etc/openvpn/psw-file
```

在这个文件下添加即可