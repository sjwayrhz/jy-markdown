## 挂载大于2TB磁盘

Try to create a partition start GNU parted as follows:

```
sudo -i
parted /dev/sda
```

Output:

```
GNU Parted 2.3
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)
```

Creates a new GPT disklabel i.e. partition table:

```
mklabel gpt
```

Sample outputs:

```
Warning: The existing disk label on /dev/sda will be destroyed and all data on this disk will be lost. Do you want to continue?
Yes/No? yes
```

Next, set the default unit to TB, enter:

```
unit TB
```

To create a 2TB partition size, enter:

```
mkpart primary 0.00TB 2.00TB
```

To print the current partitions, enter:

```
print
```

Sample outputs:

```
Model: ATA ST33000651AS (scsi)
Disk /dev/sda: 2.00TB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Number  Start   End     Size    File system  Name     Flags
 1      0.00TB  2.00TB  2.00TB  ext4         primary
```

Quit and save the changes, enter:

```
quit
```

Use the mkfs.ext4 command to format the file system, enter:

```
mkfs.ext4 /dev/sdb1
```

Create a new dir /data

```
mkdir /data
```

Mount the data dir 

```
mount /dev/sdb1 /data
```

Show the disk 

```
df -h
```

