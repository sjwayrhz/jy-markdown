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

### 安装基础组件

```bash
~]# dnf install vim net-tools wget -y
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



### 修改dnf源

进入dnf源目录

```bash
~]# cd /etc/yum.repos.d
```

将dnf源里的centos地址换成阿里云

```bash
yum.repos.d]# sed -i 's/http:\/\/mirror.centos.org/https:\/\/mirrors.aliyun.com/g' *
```

修改配置文件，不使用镜像列表，而是固定使用阿里云

```bash
yum.repos.d]# sed -i 's/mirrorlist=/#mirrorlist=/g' *

yum.repos.d]# sed -i 's/# baseurl=/baseurl=/g' *
```

查看结果

```bash
~]# dnf repolist
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



