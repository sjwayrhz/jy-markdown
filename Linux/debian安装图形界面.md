# debian安装图形界面，使用mstsc远程登录

    环境：debian 10.2

更新软件列表
```
apt-get update
```
安装最基本的gnome图形相关软件
```
apt install x-window-system-core gnome-core
```
安装xrdp（可以使用windows远程桌面登录）
```
apt-get install xrdp
```
设置系统可以使用root登录
```
修改/etc/gdm3/daemon.conf文件，在[security]下增加一行AllowRoot = true
修改/etc/pam.d/gdm-password文件，注释掉auth required pam_succeed_if.so user != root quiet_success
```
以图形界面启动debian
```
init 6
```
重启后在windows机器上使用远程桌面登录debian系统