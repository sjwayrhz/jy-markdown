## docker安装

### prerequisite

1. 安装docker

   ```bash
   $ curl sh.rainbond.com/install_docker | bash
   ```

2. 配置EIP 

   ```bash
   $ export EIP=IPAddress
   ```

   可以是内网网卡的IP地址，用于WEB访问

3. 加载iptable内核模块

   ```bash
   $ modprobe ip_tables
   ```

### 使用 docker启动

   ```bash
   $ docker run --privileged -d  -p 7070:7070 -p 80:80 -p 443:443 -p 6060:6060 -p 8443:8443 \
   --name=rainbond-allinone --restart=unless-stopped \
   -v ~/.ssh:/root/.ssh \
   -v ~/rainbonddata:/app/data \
   -v /opt/rainbond:/opt/rainbond \
   -v ~/dockerdata:/var/lib/docker \
   -e ENABLE_CLUSTER=true \
   -e EIP=$EIP \
   registry.cn-hangzhou.aliyuncs.com/goodrain/rainbond:v5.6.0-dind-allinone \
   && docker logs -f rainbond-allinone
   ```

   

   

