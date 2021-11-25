

国内安装debian，当然要用国内的源，优点只有一个：速度快。国内的源比较多，下面就把更改源的方法说一下。

登录到debian 10，如果是root用户，直接输入命令，如果是非root用户，需要在命令前面加上sudo。

一，先备份一下原始的源
```
$ mv /etc/apt/sources.list /etc/apt/sources.list.bak
```
这样就把原始的源文件改了一个名字，备份一下，不好删掉。

二，更新为国内的源

然后把下面的源地址复制进去sources.list文件
```
deb http://mirrors.ustc.edu.cn/debian/ buster main
deb-src http://mirrors.ustc.edu.cn/debian/ buster main$ 
deb http://security.debian.org/debian-security buster/updates main
deb-src http://security.debian.org/debian-security buster/updates$ main$ 
# buster-updates, previously known as 'volatile'
deb http://mirrors.ustc.edu.cn/debian/ buster-updates main
deb-src http://mirrors.ustc.edu.cn/debian/ buster-updates main$ 
deb http://mirrors.ustc.edu.cn/debian/ buster-backports main$ non-free contrib
deb-src http://mirrors.ustc.edu.cn/debian/ buster-backports main$ non-free contrib
```
保存

三，更新一下源
```
$ apt-get update
```
