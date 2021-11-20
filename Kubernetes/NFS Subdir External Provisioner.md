* 配置nfs-storage

  登陆nfs-server，创建nfs共享存储

  ### 配置nfs-server

  ```bash
  ~]# dnf -y install nfs-utils rpcbind
  ```

  配置 nfs，nfs 的默认配置文件在 /etc/exports 文件下，在该文件中添加下面的配置信息：

  ```
  ~]# vi /etc/exports
  /data  *(rw,sync,no_root_squash)
  ```

  配置说明：

  ```
  /nfs：是共享的数据目录
  *：表示任何人都有权限连接，当然也可以是一个网段，一个 IP，也可以是域名
  rw：读写的权限
  sync：表示文件同时写入硬盘和内存
  no_root_squash：当登录 NFS 主机使用共享目录的使用者是 root 时，其权限将被转换成为匿名使用者，通常它的 UID 与 GID，都会变成 nobody 身份
  ```

  启动rpcbind 、nfs

  ```bash
  ~]# systemctl enable --now rpcbind
  ~]# systemctl enable --now nfs-server
  ```

  确认nfs开启

  ```bash
  ~]# rpcinfo -p|grep nfs
      100003    3   tcp   2049  nfs
      100003    4   tcp   2049  nfs
      100227    3   tcp   2049  nfs_acl
  ```

  创建并查看nfs共享文件夹

  ```bash
  ~]# mkdir /data
  ~]# showmount -e
  Export list for nfs-server:
  /data *
  ```

  ### 配置nfs-client

  使用ansible为k8s集群安装nfs-utils

  ```bash
  ~]# ansible all -m yum -a "name=nfs-utils state=installed"
  ```

  创建挂载的/data目录

  ```bash
  ~]# ansible all -m shell -a "mkdir /data"
  ```

  假设nfs-server的ip地址为 192.168.177.206

  抽取k8s-node-10作为实验，在k8s-node-10虚拟机中查看开启的 nfs目录

  ```bash
  ~]# showmount -e 192.168.177.206
  Export list for 192.168.177.206:
  /data *
  ```

  创建data目录，并挂载nfs-server中的/data目录

  ```bash
  ~]# mount 192.168.177.206:/data /data
  ```

  测试完成后，卸载挂载

  ```bash
  ~]# umount 192.168.177.206:/data
  ```

  登陆到ansible虚拟机，修改集群的/etc/fstab

  ```bash
  ~]# ansible all -m shell -a "echo '192.168.177.206:/data /data nfs defaults 0 0' >> /etc/fstab"
  ```

  先尝试重启k8s-node-10，观测是否可以正常启动，如果可以，则重启k8s集群所有虚拟机

  ```bash
  ~]# ansible all -m shell -a "reboot"
  ```

  等待启动完成后，使用如下命令检测挂载成功

  ```bash
  ~]# ansible all -m shell -a "df -h | grep data" 
  ```
