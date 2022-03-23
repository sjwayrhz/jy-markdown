# Rocky Linux 国内镜像源列表

[TOC]

### 恢复官方源

```bash
sed -e 's|^#mirrorlist=|mirrorlist=|g' \
    -e 's|^baseurl=https://mirrors.sjtug.sjtu.edu.cn/rocky|#baseurl=http://dl.rockylinux.org/$contentdir|g' \
    -i.bak \
    /etc/yum.repos.d/Rocky-*.repo
```

### Rockylinux 中华电信

中华电信示例：

```bash
$ sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirror01.idc.hinet.net/rocky|g' \
    -i.bak \
    /etc/yum.repos.d/Rocky-*.repo
```

安装epel

```bash
$ dnf install -y epel-release
```

中华电信epel示例：

````bash
$ sed -e 's|^metalink=|#metalink=|g' \
    -e 's|^#baseurl=https://download.example/pub|baseurl=https://mirror01.idc.hinet.net|g' \
    -i.bak \
    /etc/yum.repos.d/epel*.repo
````



### Rockylinux-8.4

阿里云示例：

```bash
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.aliyun.com/rocky|g' \
    -i.bak \
    /etc/yum.repos.d/Rocky-*.repo
```

上海交通大学示例：

```bash
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.sjtug.sjtu.edu.cn/rocky|g' \
    -i.bak \
    /etc/yum.repos.d/Rocky-*.repo
```

#### 替换

上海交通大学替换到阿里云

```bash
sed -e 's|^baseurl=https://mirrors.sjtug.sjtu.edu.cn|baseurl=https://mirrors.aliyun.com|g' \
    -i.bak \
    /etc/yum.repos.d/Rocky-*.repo
```

阿里云更换南京大学

```bash
sed -e 's|^baseurl=https://mirrors.aliyun.com|baseurl=https://mirrors.nju.edu.cn|g' \
    -i.bak \
    /etc/yum.repos.d/Rocky-*.repo
```

上海大学换中华电信

````bash
sed -e 's|^baseurl=https://mirrors.sjtug.sjtu.edu.cn|baseurl=https://mirror01.idc.hinet.net|g' \
    -i.bak \
    /etc/yum.repos.d/Rocky-*.repo
````



更换其他镜像，对应按照上面替换 Mirror Name 即可，注意路径“/rocky”。

#### epel

```bash
sed -e 's|^metalink=|#metalink=|g' \
    -e 's|^#baseurl=https://download.example/pub|baseurl=https://mirrors.aliyun.com|g' \
    -i.bak \
    /etc/yum.repos.d/epel*.repo
# 注意：
# Rocky Linux 中 #baseurl=https://download.example/pub
# 与 CentOS 相同，而 Alma Linux #baseurl=https://download.fedoraproject.org/pub
```

官方列表：https://mirror.rockylinux.org/mirrormanager/mirrors

CN 开头的站点

| Country | Site Name                           | Mirror Name               | Categories                          | Bandwidth | Internet2 |
| ------- | ----------------------------------- | ------------------------- | ----------------------------------- | --------- | --------- |
| CN      | EuropeJing                          | mirrors.europejing.com    | Rocky Linux: http                   | 300       | No        |
| CN      | eScience Center, Nanjing University | mirrors.nju.edu.cn        | Rocky Linux: http \| https \| rsync | 10000     | No        |
| CN      | SDU-Mirrors                         | mirrors.sdu.edu.cn        | Rocky Linux: https                  | 1000      | No        |
| CN      | SJTUG                               | mirrors.sjtug.sjtu.edu.cn | Rocky Linux: https \| rsync         | 1000      | No        |
| CN      | SKYSHE                              | mirrors.skyshe.cn         | Rocky Linux: https \| http          | 1000      | Yes       |



### Rockylinux-8.5

### Almalinux

```bash
sed -e 's|^baseurl=https://repo.almalinux.org|baseurl=https://mirrors.sjtug.sjtu.edu.cn|g' \
    -i.bak \
    /etc/yum.repos.d/almalinux*.repo
```

###  Epel源

更改原先的epel源为清华源

```bash
sed -e 's!^metalink=!#metalink=!g' \
    -e 's!^#baseurl=!baseurl=!g' \
    -e 's!//download\.fedoraproject\.org/pub!//mirrors.tuna.tsinghua.edu.cn!g' \
    -e 's!//download\.example/pub!//mirrors.tuna.tsinghua.edu.cn!g' \
    -e 's!http://mirrors!https://mirrors!g' \
    -i /etc/yum.repos.d/epel*.repo
```

