# prometheus + influxdb + grafana + mysql

# 前言

本文介绍使用influxdb 作为prometheus持久化存储和使用mysql 作为grafana 持久化存储的安装方法

### 一 安装go环境

```
~]# wget https://dl.google.com/go/go1.15.7.linux-amd64.tar.gz
~]# tar -C /usr/local -xzf go1.15.7.linux-amd64.tar.gz

~]# echo "export PATH=$PATH:/usr/local/go/bin" >> /etc/profile

~]# go version
```



### 安装influxdb

在influxdb的官网下载链接为`https://portal.influxdata.com/downloads/`

当前最新版为2.0.3

```
~]# wget https://dl.influxdata.com/influxdb/releases/influxdb2-2.0.3.x86_64.rpm
~]# sudo yum localinstall influxdb2-2.0.3.x86_64.rpm
```
启动influxdb

```
~]# systemctl enable --now influxdb
```
设置influxdb CLI

```
~]# influx setup \
--username caobo \
--password 2wsx#EDC \
--org cls \
--bucket clsBucket \
--retention 168 \
--force
```

也可以通过浏览器输入 `$ip:8086`进行设置

### 安装Telegraf

Telegraf可以和indfluxdb安装在同一台服务器中

下载yum源

```
~]# cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
EOF
```

安装并启动telegraf

```
~]# yum install telegraf
~]# systemctl start telegraf
```

创建telegraf配置文件

```
telegraf --input-filter <pluginname>[:<pluginname>] --output-filter <outputname>[:<outputname>] config > telegraf.conf
```



### 安装prometheus

安装步骤如下：

```
~]# curl -LO url -LO https://github.com/prometheus/prometheus/releases/download/v2.22.0/prometheus-2.22.0.linux-amd64.tar.gz
~]# tar -xvf prometheus-2.22.0.linux-amd64.tar.gz
~]# mv prometheus-2.22.0.linux-amd64 /opt/prometheus
```



