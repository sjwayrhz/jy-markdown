# How to Install Jenkins on CentOS 7

Published on: Thu, May 19, 2016 at 4:54 am EST

[CentOS](https://www.vultr.com/docs/category/centos/) [DevOps](https://www.vultr.com/docs/category/devops/) [Linux Guides](https://www.vultr.com/docs/category/linux-guides/) [Server Apps](https://www.vultr.com/docs/category/server-apps/)

Jenkins is a popular open source CI (Continuous Integration) tool which is widely used for project development, deployment, and automation.

This article will guide you through the process of installing Jenkins on a Vultr CentOS 7 server instance. In order to facilitate visitors' access, Nginx will also be installed as the reverse proxy for Jenkins.

### Prerequisites

Before proceeding, you must have:

- Deployed a Vultr CentOS 7 server instance from scratch.
- Logged into your machine as a non-root user with sudo privileges.

### Step 1: Update your CentOS 7 system

One of the Linux system administrator's best practices is keeping a system up to date. Install the latest stable packages, then reboot.

```
# yum install epel-release
# yum update
# reboot
```

When the reboot finishes, login with the same sudo user.

### Step 2: Install Java

Before you can install Jenkins, you need to setup a Java virtual machine on your system. Here, let's install the latest OpenJDK Runtime Environment 1.8.0 using YUM:

```
# yum install java-1.8.0-openjdk.x86_64
```

After the installation, you can confirm it by running the following command:

```
# java -version
```

This command will tell you about the Java runtime environment that you have installed:

```
openjdk version "1.8.0_91"
OpenJDK Runtime Environment (build 1.8.0_91-b14)
OpenJDK 64-Bit Server VM (build 25.91-b14, mixed mode)
```

In order to help Java-based applications locate the Java virtual machine properly, you need to set two environment variables: "JAVA_HOME" and "JRE_HOME".

```
# cp /etc/profile /etc/profile_backup
# echo 'export JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk' | sudo tee -a /etc/profile
# echo 'export JRE_HOME=/usr/lib/jvm/jre' | sudo tee -a /etc/profile
# source /etc/profile
```

Finally, you can print them for review:

```
# echo $JAVA_HOME
# echo $JRE_HOME
```

### Step 3: Install Jenkins

Use the official YUM repo to install the latest stable version of Jenkins, which is `2.138.2-1.1` at the time of writing:

```
# cd ~ 
# wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
# rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
# yum install jenkins
```

Start the Jenkins service and set it to run at boot time:

```
# systemctl start jenkins.service
# systemctl enable jenkins.service
```

In order to allow visitors access to Jenkins, you need to allow inbound traffic on port 8080:

```
# firewall-cmd --zone=public --permanent --add-port=8080/tcp
# firewall-cmd --reload
OR
# systemctl disable --now firewalld
```

Now, test Jenkins by visiting the following address from your web browser:

```
http://<your-Vultr-server-IP>:8080
```

### Step 4: Install Nginx (optional)

In order to facilitate visitors' access to Jenkins, you can setup an Nginx reverse proxy for Jenkins, so visitors will no longer need to key in the port number 8080 when accessing your Jenkins application.

Install Nginx using YUM:

```
# rpm -ivh http://nginx.org/packages/centos/7/x86_64/RPMS/nginx-1.16.0-1.el7.ngx.x86_64.rpm
```

Modify the configuration of Nginx:

```
# cp /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.bak
# mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/jenkins.conf

# vi /etc/nginx/conf.d/jenkins.conf
```

Find the two lines below:

```
location / {
}
```

Insert the six lines below into the { } segment:

```
proxy_pass http://127.0.0.1:8080;
proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

The final result should be:

```
location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_redirect off;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

Save and quit:

```
:wq
```

Start and enable the Nginx service:

```
# systemctl enable --now nginx.service
```

Allow traffic on port 80:

```
# firewall-cmd --zone=public --permanent --add-service=http
# firewall-cmd --reload
```

Finally, visit the following address from your web browser to confirm your installation:

```
http://<your-Vultr-server-IP>
```

排错1：

如果此时开启了SELinux，nginx将会无法代理jenkins导致出现Permission denied错误。

排错2：

```
~] # systemctl status jenkins
● jenkins.service - LSB: Jenkins Automation Server
   Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
   Active: failed (Result: exit-code) since Wed 2019-04-03 23:51:34 CST; 6s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 19988 ExecStart=/etc/rc.d/init.d/jenkins start (code=exited, status=1/FAILURE)

Apr 03 23:51:34 test systemd[1]: Starting LSB: Jenkins Automation Server...
Apr 03 23:51:34 test runuser[19993]: pam_unix(runuser:session): session opened for user jenkins by (uid=0)
Apr 03 23:51:34 test jenkins[19988]: Starting Jenkins bash: /usr/bin/java: No such file or directory
Apr 03 23:51:34 test runuser[19993]: pam_unix(runuser:session): session closed for user jenkins
Apr 03 23:51:34 test jenkins[19988]: [FAILED]
Apr 03 23:51:34 test systemd[1]: jenkins.service: control process exited, code=exited status=1
Apr 03 23:51:34 test systemd[1]: Failed to start LSB: Jenkins Automation Server.
Apr 03 23:51:34 test systemd[1]: Unit jenkins.service entered failed state.
Apr 03 23:51:34 test systemd[1]: jenkins.service failed.
```

标红的为 `Failed to start LSB: Jenkins Automation Server.`

这是因为jenkins没有配置java的环境变量

在`/usr/bin/java`的下一行添加`/usr/local/jdk` 即可

修改之后

```
~]# systemctl daemon-reload
~]# systemctl start jenkins
~]# systemctl status jenkins
```

