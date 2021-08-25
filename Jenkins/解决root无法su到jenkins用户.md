## yum安装jenkins后root无法su到jenkins用户
cat /etc/passwd发jenkins用户后被yum安装设置为了/bin/false

```
~]# cat /etc/passwd | grep jenkins
jenkins:x:992:988:Jenkins Automation Server:/var/lib/jenkins:/bin/false
```

sudo vim /etc/passwd改为/bin/bash

```
~]# cat /etc/passwd | grep jenkins
jenkins:x:992:988:Jenkins Automation Server:/var/lib/jenkins:/bin/bash
```

再次su jenkins发现用户变为了-bash-4.1#

```
~]# echo "export PS1='[\u@\h \W]\$'" >> ~/.bash_profile
~]# source ~/.bash_profile
```

再次su jenkins，发现正常

