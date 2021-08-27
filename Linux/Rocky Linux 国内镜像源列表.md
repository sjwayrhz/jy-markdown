# Rocky Linux 国内镜像源列表

上海交通大学示例：

```bash
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.sjtug.sjtu.edu.cn/rocky|g' \
    -i.bak \
    /etc/yum.repos.d/Rocky-*.repo
```

更换其他镜像，对应按照上面替换 Mirror Name 即可，注意路径“/rocky”。

官方列表：https://mirror.rockylinux.org/mirrormanager/mirrors

CN 开头的站点

| Country | Site Name                           | Mirror Name               | Categories                          | Bandwidth | Internet2 |
| ------- | ----------------------------------- | ------------------------- | ----------------------------------- | --------- | --------- |
| CN      | EuropeJing                          | mirrors.europejing.com    | Rocky Linux: http                   | 300       | No        |
| CN      | eScience Center, Nanjing University | mirrors.nju.edu.cn        | Rocky Linux: http \| https \| rsync | 10000     | No        |
| CN      | SDU-Mirrors                         | mirrors.sdu.edu.cn        | Rocky Linux: https                  | 1000      | No        |
| CN      | SJTUG                               | mirrors.sjtug.sjtu.edu.cn | Rocky Linux: https \| rsync         | 1000      | No        |
| CN      | SKYSHE                              | mirrors.skyshe.cn         | Rocky Linux: https \| http          | 1000      | Yes       |

Almalinux

```bash
sed -e 's|^baseurl=https://repo.almalinux.org|baseurl=https://mirrors.sjtug.sjtu.edu.cn|g' \
    -i.bak \
    /etc/yum.repos.d/almalinux*.repo
```

