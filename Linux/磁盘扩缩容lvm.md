# 磁盘扩缩容

[TOC]

## 先决条件

操作系统：rocky linux 8.5

系统磁盘：/dev/sda  40GB，其中 /dev/mapper/rl-root: 35 GiB  ,另外 /dev/mapper/rl-swap: 4 GiB

添加磁盘1：/dev/sdb:  50 GiB

添加磁盘2：/dev/sdc:  100 GiB

需求将添加到磁盘扩容到 /dev/mapper/rl-root中

## 概念

###  物理存储介质（The physical media）

这里指系统的存储设备：硬盘，如：/dev/hda、/dev/sda 等等，是存储系统最低层的存储单元。

### 物理卷（physicalvolume）

物理卷就是指硬盘分区或从逻辑上与磁盘分区具有同样功能的设备(如 RAID)，是 LVM 的基本存储逻辑块，但和基本的物理存储介质（如分区、磁盘等）比较，却包含有与 LVM 相关的管理参数。

### 卷组（Volume Group）

LVM 卷组类似于非 LVM 系统中的物理硬盘，其由物理卷组成。可以在卷组上创建一个或多个“LVM 分区”（逻辑卷），LVM 卷组由一个或多个物理卷组成。

### 逻辑卷（logicalvolume）

LVM 的逻辑卷类似于非 LVM 系统中的硬盘分区，在逻辑卷之上可以建立文件系统(比如/home 或者/usr 等)。

### PE（physical extent）

每一个物理卷被划分为称为 PE(Physical Extents)的基本单元，具有唯一编号的 PE 是可以被 LVM 寻址的最小单元。PE 的大小是可配置的，默认为 4MB。

### LE（logical extent）

逻辑卷也被划分为被称为 LE(Logical Extents) 的可被寻址的基本单位。在同一个卷组中，LE 的大小和 PE 是相同的，并且一一对应。

## 操作步骤

从 esxi/vmware 添加磁盘，新加磁盘默认用`fdisk -l` 看不到，要么重启，要么 `echo "scsi add-single-device 32 0 2 0">/proc/scsi/scsi` 后再`fdisk -l`

### 磁盘1创建物理卷，逻辑卷组，逻辑卷

基于磁盘创建物理卷

```bash
~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
```

查看已有物理卷

```
~]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               rl
  PV Size               <39.00 GiB / not usable 3.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              9983
  Free PE               0
  Allocated PE          9983
  PV UUID               Xlx9Av-Fsb6-yscd-PSF8-CPCI-iSqJ-lsxI2U

  "/dev/sdb" is a new physical volume of "50.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb
  VG Name
  PV Size               50.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               toNveo-osl1-qlrH-QOaq-gcU8-pVeh-0jEFef
```

创建逻辑卷组

```bash
~]#  vgcreate vg_data /dev/sdb
  Volume group "vg_data" successfully created
```

查看逻辑卷组

```bash
~]# vgdisplay
  --- Volume group ---
  VG Name               vg_data
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <50.00 GiB
  PE Size               4.00 MiB
  Total PE              12799
  Alloc PE / Size       0 / 0
  Free  PE / Size       12799 / <50.00 GiB
  VG UUID               hjzrNQ-zMb7-fQwx-4PJZ-q5zr-nL2J-5nVcUm

  --- Volume group ---
  VG Name               rl
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <39.00 GiB
  PE Size               4.00 MiB
  Total PE              9983
  Alloc PE / Size       9983 / <39.00 GiB
  Free  PE / Size       0 / 0
  VG UUID               iok5u7-vcg3-EXCb-3sz0-9wno-Gkvl-aL6b7p
```

创建逻辑卷并分配所有空间

```bash
~]# lvcreate -l 100%VG -n lv_data vg_data
  Logical volume "lv_data" created. 
```

格式化 , 使用ext4格式成功，测试使用xfs格式失败

```bash
~]# mkfs.ext4 /dev/mapper/vg_data-lv_data
```

创建文件夹

```bash
~]# mkdir -p /data
```

备份

```bash
~]# cp /etc/fstab /etc/fstab.bak 
```

弄成开机自动挂载

```bash
~]# echo `blkid /dev/mapper/vg_data-lv_data | awk '{print $2}' | sed 's/\"//g'` /data ext4 defaults 0 0 >> /etc/fstab
```

现在挂载

```bash
~]# mount /dev/mapper/vg_data-lv_data /data/
```

发现已经挂载了一块50G的卷到了/data目录

### 将磁盘2扩容到磁盘1

向 50G 的 `/dev/mapper/vg_data-lv_data` 逻辑卷上增加一块 100G 盘(/dev/sdc)扩成 150G 空间

卸载/data

```bash
~]# umount /data/
```

创建物理卷

```bash
~]# pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
```

将磁盘加到 vg_data 逻辑组中

```bash
~]# vgextend vg_data /dev/sdc
  Volume group "vg_data" successfully extended
```

查询vg_data 显示为150G

```bash
~]# vgdisplay vg_data
  --- Volume group ---
  VG Name               vg_data
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               149.99 GiB
  PE Size               4.00 MiB
  Total PE              38398
  Alloc PE / Size       12799 / <50.00 GiB
  Free  PE / Size       25599 / <100.00 GiB
  VG UUID               Q7ooph-AhsH-nYws-k7wX-xvqQ-as3A-bky4hm
```

扩容逻辑卷

```bash
~]# lvextend -l +100%FREE /dev/mapper/vg_data-lv_data
  Size of logical volume vg_data/lv_data changed from <50.00 GiB (12799 extents) to 149.99 GiB (38398 extents).
  Logical volume vg_data/lv_data successfully resized.
```

查看逻辑卷容量

```bash
~]# lvdisplay /dev/mapper/vg_data-lv_data
  --- Logical volume ---
  LV Path                /dev/vg_data/lv_data
  LV Name                lv_data
  VG Name                vg_data
  LV UUID                lPbis4-aDKx-qe28-jlbU-lKt6-FKd4-zffUOe
  LV Write Access        read/write
  LV Creation host, time lvm, 2022-02-15 10:44:22 +0800
  LV Status              available
  # open                 0
  LV Size                149.99 GiB
  Current LE             38398
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
```

重新计算逻辑卷大小

```bash
~]# e2fsck -f /dev/mapper/vg_data-lv_data
.e2fsck 1.45.6 (20-Mar-2020)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/mapper/vg_data-lv_data: 11/3276800 files (0.0% non-contiguous), 284558/13106176 blocks
~]# resize2fs  /dev/mapper/vg_data-lv_data
resize2fs 1.45.6 (20-Mar-2020)
Resizing the filesystem on /dev/mapper/vg_data-lv_data to 39319552 (4k) blocks.
The filesystem on /dev/mapper/vg_data-lv_data is now 39319552 (4k) blocks long.
```

检查文件完整性

```bash
~]# e2fsck -f /dev/mapper/vg_data-lv_data
e2fsck 1.42.13 (17-May-2015)
第一步: 检查inode,块,和大小
第二步: 检查目录结构
第3步: 检查目录连接性
Pass 4: Checking reference counts
第5步: 检查簇概要信息
/dev/mapper/vg_data-lv_data: 11/9830400 files (0.0% non-contiguous), 664949/39319552 blocks
```

挂载

```bash
~]# mount  /dev/mapper/vg_data-lv_data /data/
```

查看现在磁盘

```bash
~]# df -h
Filesystem                   Size  Used Avail Use% Mounted on
devtmpfs                     373M     0  373M   0% /dev
tmpfs                        392M     0  392M   0% /dev/shm
tmpfs                        392M  5.5M  386M   2% /run
tmpfs                        392M     0  392M   0% /sys/fs/cgroup
/dev/mapper/rl-root           35G  2.0G   34G   6% /
/dev/sda1                   1014M  192M  823M  19% /boot
tmpfs                         79M     0   79M   0% /run/user/0
/dev/mapper/vg_data-lv_data  148G   60M  140G   1% /data
```

### 卷缩容

一般不会缩容，除非是，实在没剩余空间了，而另外一个文件夹更需要磁盘，将 A 缩容省出的空间给 B。

卸载/data

```bash
~]# umount /data/
```

检查文件完整性

```bash
~]# e2fsck -f /dev/mapper/vg_data-lv_data
e2fsck 1.42.13 (17-May-2015)
第一步: 检查inode,块,和大小
第二步: 检查目录结构
第3步: 检查目录连接性
Pass 4: Checking reference counts
第5步: 检查簇概要信息
/dev/mapper/vg_data-lv_data: 11/9830400 files (0.0% non-contiguous), 664949/39319552 blocks
```

缩容到50G

```bash
~]# resize2fs  /dev/mapper/vg_data-lv_data 49G
resize2fs 1.45.6 (20-Mar-2020)
Resizing the filesystem on /dev/mapper/vg_data-lv_data to 12845056 (4k) blocks.
The filesystem on /dev/mapper/vg_data-lv_data is now 12845056 (4k) blocks long.
```

逻辑卷缩容到49G

```bash
~]# lvreduce -L 50G /dev/mapper/vg_data-lv_data
  WARNING: Reducing active logical volume to 50.00 GiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce vg_data/lv_data? [y/n]: y
  Size of logical volume vg_data/lv_data changed from 149.99 GiB (38398 extents) to 50.00 GiB (12800 extents).
  Logical volume vg_data/lv_data successfully resized.
```

查看逻辑卷信息

```bash
~]# lvdisplay /dev/mapper/vg_data-lv_data
  --- Logical volume ---
  LV Path                /dev/vg_data/lv_data
  LV Name                lv_data
  VG Name                vg_data
  LV UUID                lPbis4-aDKx-qe28-jlbU-lKt6-FKd4-zffUOe
  LV Write Access        read/write
  LV Creation host, time lvm, 2022-02-15 10:44:22 +0800
  LV Status              available
  # open                 0
  LV Size                50.00 GiB
  Current LE             12800
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
```

将/dev/sdc硬盘从逻辑卷组中移除

```bash
~]# vgreduce vg_data /dev/sdc
  Removed "/dev/sdc" from volume group "vg_data"
```

查看地址

```url
https://anjia0532.github.io/2021/04/27/ubuntu-lvm/
```

