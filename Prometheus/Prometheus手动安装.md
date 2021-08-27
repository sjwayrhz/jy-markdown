# How To Install and Configure Prometheus On a Linux Server

[TOC]

Prometheus is an open-source monitoring system which is very lightweight and has a good alerting mechanism.

## Install and Configure Prometheus

This guide explains how to install and configure the latest Prometheus on a Linux VM.

If you would like to install Prometheus on a Kubernetes cluster, please see the [Prometheus on kubernetes](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/) guide.

### Before You Begin

1. Ensure that you have sudo access to the Linux server because the commands used in this guide require elevated privileges.
2. The server has access to the internet for downloading the Prometheus binary.
3. Most importantly, firewall rules opened for accessing Prometheus port 9090 on the server.

### Setup Prometheus Binaries

**Step 1:** Update the yum package repositories.

```
sudo yum update -y
```

**Step 2:** Go to the official Prometheus [downloads page](https://prometheus.io/download/) and get the latest download link for the Linux binary.

**Step 3:** Download the source using curl, untar it, and rename the extracted folder to prometheus-files.

```
curl -LO url -LO https://github.com/prometheus/prometheus/releases/download/v2.22.0/prometheus-2.22.0.linux-amd64.tar.gz
tar -xvf prometheus-2.22.0.linux-amd64.tar.gz
mv prometheus-2.22.0.linux-amd64 prometheus-files
```

**Step 4:** Create a Prometheus user, required directories, and make Prometheus the user as the owner of those directories.

```
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

**Step 5:** Copy prometheus and promtool binary from prometheus-files folder to /usr/local/bin and change the ownership to prometheus user.

```
sudo cp prometheus-files/prometheus /usr/local/bin/
sudo cp prometheus-files/promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

**Step 6:** Move the consoles and console_libraries directories from prometheus-files to /etc/prometheus folder and change the ownership to prometheus user.

```
sudo cp -r prometheus-files/consoles /etc/prometheus
sudo cp -r prometheus-files/console_libraries /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

### Setup Prometheus Configuration

All the prometheus configurations should be present in /etc/prometheus/prometheus.yml file.

**Step 1:** Create the prometheus.yml file.

```
sudo vi /etc/prometheus/prometheus.yml
```

**Step 2:** Copy the following contents to the prometheus.yml file.  "localhost"must be your global ip.

```
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
```

**Step 3:** Change the ownership of the file to prometheus user.

```
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

### Setup Prometheus Service File

**Step 1:** Create a prometheus service file.

```
sudo vi /etc/systemd/system/prometheus.service
```

**Step 2:** Copy the following content to the file.

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

**Step 3:** Reload the systemd service to register the prometheus service and start the prometheus service.

```
sudo systemctl daemon-reload
sudo systemctl start prometheus
```

Check the prometheus service status using the following command.

```
sudo systemctl status prometheus
```

The status should show the active state as shown below.

### Access Prometheus Web UI

Now you will be able to access the prometheus UI on 9090 port of the prometheus server.

```
http://<prometheus-ip>:9090/graph
```

You should be able to see the following UI as shown below.



## Install grapfana

### Installing Grafana via YUM Repository

Create a repo file.

```
vim /etc/yum.repos.d/grafana.repo
```

Add the following contents to file:

```
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```

### Install Grafana

```
yum install grafana
```

The Package does the following things:

- Installs binary to /usr/sbin/grafana-server
- Copies init.d script to /etc/init.d/grafana-server
- Installs default file to /etc/sysconfig/grafana-server
- Copies configuration file to /etc/grafana/grafana.ini
- Installs systemd service (if systemd is available) name grafana-server.service
- The default configuration uses a log file at /var/log/grafana/grafana.log

### Install additional font packages

Continue with following commands to install the free type and urw fonts.

```
yum install fontconfig
yum install freetype*
yum install urw-fonts
```

### Enable Grafana Service

Check the status of the service.

```
systemctl status grafana-server
```

If service is not active, start it using the following command:

```
systemctl start grafana-server
```

Enable Grafana service on system boot

```
systemctl enable grafana-server.service
```

### Browse Grafana

Use the following URL to access the Grafana web interface.

```
http://Your Server IP or Host Name:3000/
```

Enter “admin” in the login and password fields for first-time use; then it should ask you to change the password.

It should redirect to the Dashboard.

## Install node_exporter

Paste the copied URL after wget in the following command:

```
wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
```

Extract the downloaded package.

```
tar -xvzf node_exporter-0.18.1.linux-amd64.tar.gz
```

Create a user for the node exporter.

```
useradd -rs /bin/false nodeusr
```

Move binary to “/usr/local/bin” from the downloaded extracted package.

```
mv node_exporter-0.18.1.linux-amd64/node_exporter /usr/local/bin/
```

Create a service file for the node exporter.

```
vim /etc/systemd/system/node_exporter.service
```

Add the following content to the file.

```
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=nodeusr
Group=nodeusr
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Save and exit the file.

Reload the system daemon.

```
systemctl daemon-reload
```

Start node exporter service.

```
systemctl start node_exporter
```

Enable node exporter on system boot.

```
systemctl enable node_exporter
```

View the metrics browsing node exporter URL.

```
http://IP-Address:9100/metrics
```

## Configured node_exporter 

### Monitor Linux Server Using Prometheus

Login to Prometheus server and modify the prometheus.yml file

Edit the file

```
vim /etc/prometheus/prometheus.yml
```

Add the following configurations under the scrape config.

```
 - job_name: 'node_exporter_centos'
    scrape_interval: 5s
    static_configs:
      - targets: ['172.16.34.12:9100']
```

The file

```
~]# cat /etc/prometheus/prometheus.yml
```

 should look like as follows.

```
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['172.16.34.11:9090']
  - job_name: 'node_exporter_centos'
    scrape_interval: 5s
    static_configs:
      - targets: ['172.16.34.12:9100']
```

Restart Prometheus service

```
systemctl restart prometheus
```

Login to Prometheus server web interface, and check targets.

```
http://Prometheus-Server-IP:9090/targets
```

You can click the graph and query any server metrics and click execute to show output. It will show the console output.

### Monitor MySQL Server Using Prometheus

Login to MySQL and execute the following queries.

```
CREATE USER 'mysqlexporter'@'localhost' IDENTIFIED BY 's56fsg#4W2126&dfk' WITH max_user_connections 2;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'mysqlexporter'@'localhost';
FLUSH PRIVILEGES;
```

Download mysqld_exporter from the official download page.`https://prometheus.io/download`


Mysqld Exporter

```
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.11.0/mysqld_exporter-0.11.0.linux-amd64.tar.gz
```

Extract the Downloaded file.

```
tar -xvzf mysqld_exporter-0.11.0.linux-amd64.tar.gz
```

Add a user for mysqld_exporter.

```
useradd -rs /bin/false mysqld_exporter
```

Copy mysqld_exporter file to /usr/bin.

```
 mv mysqld_exporter-0.11.0.linux-amd64/mysqld_exporter /usr/bin
```

Change ownership of the file.

```
chown mysqld_exporter:mysqld_exporter /usr/bin/mysqld_exporter
```

Create needed folders.

```
mkdir -p /etc/mysql_exporter
```

Create a MySQL password file for mysqld_exporter.

```
vim /etc/mysql_exporter/.my.cnf
```

Add the following configurations to the file.

```
[client]
user=mysqlexporter
password=sdfsg#4W2126&gh
```

Save and exit the file.

Change ownership.

```
chown -R mysqld_exporter:mysqld_exporter /etc/mysql_exporter
```

Grant needed permission.

```
chmod 600 /etc/mysql_exporter/.my.cnf
```

Create a service file.

```
vim /etc/systemd/system/mysql_exporter.service
```

Add the following content to the file.

```
[Unit]
Description=MySQL Server fosslinux
After=network.target

[Service]
User=mysqld_exporter
Group=mysqld_exporter
Type=simple
ExecStart=/usr/bin/mysqld_exporter \
--config.my-cnf="/etc/mysql_exporter/.my.cnf"
Restart=always

[Install]
WantedBy=multi-user.target
```

Reload the system daemon.

```
 systemctl daemon-reload
```

Enable mysql_exporter on system boot.

```
systemctl enable mysql_exporter
```

Start service.

```
systemctl start mysql_exporter
```

View the metrics using the following URL.

```
http://Server_IP:9104/metrics
```

Now go to Prometheus server and modify the prometheus.yml file.

```
 vim /etc/prometheus/prometheus.yml
```

Add the following content to the file.

```
- job_name: 'mysql_exporter_fosslinux'
    scrape_interval: 5s
    static_configs:
      - targets: ['172.16.34.15:9104']
```

Restart Prometheus.

```
 systemctl restart prometheus
```

You can see added targets by clicking targets under the status.

```
http://IP:9090/targets
```

Now you can select query using query browser and get the result of MySQL server.

### Link

```
https://www.fosslinux.com/10398/how-to-install-and-configure-prometheus-on-centos-7.htm
```