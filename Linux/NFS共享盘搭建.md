# Quick NFS Server configuration on Redhat 7 Linux System 

[TOC]

## Basic NFS Configuration

In  this config will guide you trough a quick and basic configuration of  NFS server on RHEL7 Linux system. We do not take any security concerns  into the consideration, nor we will be concerned with fine tuning and  access control. In our scenario we define two hosts: 

- NFS Server, IP 192.168.0.72
- NFS Client, IP 192.168.0.73

  Assuming your already have a running Redhat 7 Linux system in order to  setup NFS server you will need to install few additional packages: 

### NFS Server configuration

Run the below commands to begin the NFS Server installation: 

```
[nfs-server ]# yum install nfs-utils rpcbind
```

 Next we export some arbitrary directory called `/opt/nfs``/opt/nfs`

```
[nfs-server ]# mkdir -p /opt/nfs
```

and edit `/etc/exports` NFS exports file to add the below line while replacing the IP address `192.168.0.73` with the IP address of your client: 

```
[nfs-server ]# echo "/opt/nfs 192.168.0.73(no_root_squash,rw,sync)" >> /etc/exports
```

 Next make sure to enable `2049`

```
[nfs-server ]# firewall-cmd --zone=public --add-port=2049/tcp --permanent
[nfs-server ]# firewall-cmd --reload
```

 Start `rpcbind`

```
[nfs-server ]# systemctl enable --now nfs
```

 Check the NFS server status: 

```
[nfs-server ]# systemctl status nfs
● nfs-server.service - NFS server and services
   Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; vendor preset: disabled)
  Drop-In: /run/systemd/generator/nfs-server.service.d
           └─order-with-mounts.conf
   Active: active (exited) since Mon 2019-07-22 10:42:10 CST; 842ms ago
  Process: 6603 ExecStartPost=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl restart gssproxy ; fi (code=exited, status=0/SUCCESS)
  Process: 6586 ExecStart=/usr/sbin/rpc.nfsd $RPCNFSDARGS (code=exited, status=0/SUCCESS)
  Process: 6585 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
 Main PID: 6586 (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/nfs-server.service

Jul 22 10:42:10 nfs-server systemd[1]: Starting NFS server and services...
Jul 22 10:42:10 nfs-server systemd[1]: Started NFS server and services.
```

### NFS Client configuration

To  be able to mount NFS exported directories on your client the following  packages needs to be installed. Depending on your client's Linux  distribution the installation procedure may be different. On Redhat 7  Linux the installation steps are as follows: 

```
[nfs-client ]# yum install nfs-utils rpcbind
[nfs-client ]# systemctl enable --now rpcbind
```

What remains is to create a mount point directory eg. `/mnt/nfs` and mount previously NFS exported `/opt/nfs` directory: 

```
[nfs-client ]# mkdir -p /mnt/nfs
[nfs-client ]# mount 192.168.0.72:/opt/nfs /mnt/nfs/
```

 Test correctness of our setup between NFS Server and NFS client.  Create an arbitrary file within NFS mounted directory on the client  side: 

```
[nfs-client ]# cd /mnt/nfs/
[nfs-client ]# touch NFS.test
[nfs-client ]# ls -l
total 0
-rw-r--r--. 1 root root 0 Dec 11 08:13 NFS.test
```

 Move the the server side and check whether our newly `NFS.test`

```
[nfs-server ]# cd /opt/nfs/
[nfs-server ]# ls -l
total 0
-rw-r--r--. 1 root root 0 Dec 11 08:13 NFS.test
```

## Configuring permanent NFS mount

Now that we have a basic NFS configuration on RHEL7 Linux system done, 
next we can add additional settings such as server persistence and 
permanent client mount using `/etc/fstab`. In order to have our NFS exports permanently available after the NFS server system reboot we need to make sure that `nfs` service starts after reboot: 

```
[nfs-server ]# systemctl enable nfs-server
[nfs-server ]# ln -s '/usr/lib/systemd/system/nfs-server.service' '/etc/systemd/system/multi-user.target.wants/nfs-server.service'
```

To allow client to mount NFS exported directory permanently after reboot we need to define a mount procedure within `/etc/fstab` config file. Open `/etc/fstab` file and add the following line:

```
[nfs-client ]# echo "192.168.0.72:/opt/nfs/  mnt/nfs                 nfs     defaults        0 0 " >> /etc/fstab
```

## Mount User Home Directory

In the following steps we will export a user home directory `/home/rhel7`. Since NFS needs full access privileges to access `/home/rhel7`: 

```
[nfs-server ]# ls -ld /home/rhel7/
drwx------. 2 rhel7 rhel7 59 Jul 17 14:22 /home/rhel7/
```

 we will bind it to a new directory: 

```
[nfs-server ]# mkdir -p /exports/rhel7
[nfs-server ]# mount --bind /home/rhel7/ /exports/rhel7/
```

 To make the above permanent add the following line into your `/etc/fstab`

```
/home/rhel7    /exports/rhel7   none    bind  0  0
```

 Next, add another export line into `/etc/exports`

```
/exports/rhel7 192.168.0.73(no_root_squash,rw,sync)
```

 Re-export all NFS directories: 

```
[nfs-server ]# exportfs -ra
```

 What has left is to mount the above user directory using our client host: 

```
[nfs-client ]# mount 192.168.0.72:/exports/rhel7 /mnt/rhel7/
[nfs-client ]# cd /mnt/rhel7/
[nfs-client ]# ls
[nfs-client ]# touch RHEL7-test-nfs
[nfs-client ]# ls
RHEL7-test-nfs
```

 Confirm that the file `RHEL7-test-nfs`

```
# ls -l /home/rhel7/
total 0
-rw-r--r--. 1 root root 0 Dec 11 09:13 RHEL7-test-nfs
```