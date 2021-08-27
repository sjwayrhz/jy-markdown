# Linux系统优化

[TOC]

## 网络环境

修改ifcfg-xxx文件，手动指定ip地址如下

```bash
~]# cat /etc/sysconfig/network-scripts/ifcfg-ens192
BOOTPROTO=static
NAME=ens192
DEVICE=ens192
ONBOOT=yes
IPADDR=10.230.7.38
NETMASK=255.255.255.0
GATEWAY=10.230.7.1
DNS1=114.114.114.114
DNS2=8.8.8.8
```

改完之后，重新加载网卡，使之生效

```bash
~]# nmcli c reload
```



## 基础组件

### 修改主机名

```bash
~]# echo 'almalinux' >/etc/hostname

~]# hostname `cat /etc/hostname`

~]# bash
```

### 换源

```bash
~]# sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.sjtug.sjtu.edu.cn/rocky|g' \
    -i.bak \
    /etc/yum.repos.d/Rocky-*.repo
```

### 安装基础组件

```bash
~]# dnf install vim net-tools wget telnet epel-release -y
```

### 关闭selinux

```bash
~]# setenforce=0
~]# sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config
```

### 时间同步

```bash
~]# dnf install chrony
~]# systemctl enable --now chronyd
```

### 关闭防火墙

```bash
~]# systemctl disable --now firewalld
```



### 提升ssh访问速度

```bash
~]# sed -ri '/UseDNS/cUseDNS no' /etc/ssh/sshd_config
~]# sed -ri '/GSSAPIAuthentication/cGSSAPIAuthentication no' /etc/ssh/sshd_config
~]# systemctl restart sshd
```



## 可选配置

### disable swap

```bash
~]# sed -ri 's/.*swap.*/#&/' /etc/fstab  
```

it is better to restart.

### Letting iptables see bridged traffic

```bash
~]# cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
~]# sudo sysctl --system
```



