---
title: lvm磁盘扩容
date: 2022-12-29 10:07:11
tags: lvm扩容
---

# lvm扩容

<!--more-->

**场景1：新增一个磁盘扩容**

```
[root@lb02 ~]# lsblk 
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0              11:0    1 1024M  0 rom  
vda             252:0    0   40G  0 disk 
├─vda1          252:1    0    1G  0 part /boot
└─vda2          252:2    0   39G  0 part 
  ├─centos-root 253:0    0 35.1G  0 lvm  /
  └─centos-swap 253:1    0  3.9G  0 lvm  [SWAP]
vdb             252:16   0  100G  0 disk

[root@lb02 ~]# fdisk -l

Disk /dev/vda: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000bfa4f

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048     2099199     1048576   83  Linux
/dev/vda2         2099200    83886079    40893440   8e  Linux LVM

Disk /dev/mapper/centos-root: 37.7 GB, 37706792960 bytes, 73646080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos-swap: 4160 MB, 4160749568 bytes, 8126464 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/vdb: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

新增的磁盘为/dev/vdb
将/dev/vdb创建物理分区
# pvcreate /dev/vdb

查看物理卷
# pvdisplay

查看卷组
# vgdisplay

将物理卷加入到卷组
# vgextend centos /dev/vdb

加入后，再次查看卷组
# vgdisplay

查看逻辑卷
# lvdisplay

将剩余百分百空间都添加都逻辑卷中
lvextend -l +100%free /dev/mapper/centos-root

重新识别一下分区大小，就可以通过df -h看到新增的容量了
# xfs_growfs /dev/mapper/centos-root
```



**场景2：在原有的磁盘上扩容**

```
[root@node01 ~]# df -h
Filesystem                                        Size  Used Avail Use% Mounted on
/dev/mapper/centos-root                           36G   30G  6.0G  83% /
devtmpfs                                          16G     0   16G   0% /dev
tmpfs                                             16G  4.0K   16G   1% /dev/shm
tmpfs                                             16G  100M   16G   1% /run
tmpfs                                             16G     0   16G   0% /sys/fs/cgroup
/dev/vda1                                         1014M  166M  849M  17% /boot
tmpfs                                             3.2G     0  3.2G   0% /run/user/0

[root@node01 ~]# lsblk 
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0              11:0    1 1024M  0 rom  
vda             252:0    0  100G  0 disk 
├─vda1          252:1    0    1G  0 part /boot
└─vda2          252:2    0   39G  0 part 
  ├─centos-root 253:0    0 35.1G  0 lvm  /
  └─centos-swap 253:1    0  3.9G  0 lvm 

[root@node01 ~]# fdisk -l
#还有空间没有使用先进行分区
Disk /dev/vda: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000bfa4f

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048     2099199     1048576   83  Linux
/dev/vda2         2099200    83886079    40893440   8e  Linux LVM

Disk /dev/mapper/centos-root: 37.7 GB, 37706792960 bytes, 73646080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos-swap: 4160 MB, 4160749568 bytes, 8126464 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

分区
# fdisk /dev/vda  ## /dev/sda为通过fdisk -l 查看到的物理磁盘(第一行)
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n  ## n为创建一个新的分区
Partition type:   ## 这里需要选择时创建主分区还是扩展分区&#xff0c;都可以&#xff0c;这里直接选了主分区
   p   primary (2 primary, 0 extended, 2 free)   
   e   extended
Select (default p): p  ## 选择创建一个主分区&#xff0c;主分区只能有4个&#xff0c;编号为1-4,下面的全部直接回车就好了&#xff0c;会自动将剩余所用空间都创建
Partition number (3,4, default 3):   
First sector (104857600-209715199, default 104857600): 
Using default value 104857600
Last sector, &#43;sectors or &#43;size{K,M,G} (104857600-209715199, default 209715199): 
Using default value 209715199
Partition 3 of type Linux and of size 50 GiB is set

Command (m for help): t   ## t为修改分区类型
Partition number (1-3, default 3): 3  ## 刚才创建的分区编号为3
Hex code (type L to list all codes): 8e  ## 8e就是 lvm格式的分区
Changed type of partition &#39;Linux&#39; to &#39;Linux LVM&#39;

Command (m for help): w  ## w保存并写入磁盘。
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.

这里，在保存后，会发现，可能会出现报错，显示繁忙，无法重新读取分区信息。下面有解决办法。可以用过重启或者执行 partprobe or kpartx。所以，这里直接执行partprobe
# partprobe

将/dev/vda3创建物理分区
# pvcreate /dev/vda3

查看物理卷
# pvdisplay

查看卷组
# vgdisplay

将物理卷加入到卷组
# vgextend centos /dev/vda3

加入后，再次查看卷组
# vgdisplay

查看逻辑卷
# lvdisplay

将剩余百分百空间都添加都逻辑卷中
# lvextend -l +100%free /dev/mapper/centos-root

重新识别一下分区大小，就可以通过df -h看到新增的容量了
# xfs_growfs /dev/mapper/centos-root
```



**场景3：在同一个vg卷组下，不同逻辑卷，将空间的容量加到root卷组下**

```
# 列出所有可用块设备的信息
[root@master01 software]# lsblk
NAME                                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0                                    11:0    1 1024M  0 rom  
vda                                   253:0    0  100G  0 disk 
├─vda1                                253:1    0  200M  0 part /boot/efi
├─vda2                                253:2    0    1G  0 part /boot
└─vda3                                253:3    0 98.8G  0 part 
  ├─klas_host--10--200--91--54-root   252:0    0 63.7G  0 lvm  /
  ├─klas_host--10--200--91--54-swap   252:1    0    4G  0 lvm  
  └─klas_host--10--200--91--54-backup 252:2    0 31.1G  0 lvm  
```

```
# 查看卷组
[root@master01 software]# vgdisplay
  --- Volume group ---
  VG Name               klas_host-10-200-91-54
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               98.80 GiB
  PE Size               4.00 MiB
  Total PE              25293
  Alloc PE / Size       25293 / 98.80 GiB
  Free  PE / Size       0 / 0   
  VG UUID               kkFp7N-fdS3-lf3n-Fylz-Q7LG-h6Ex-T7bvph
```

```
# 查看逻辑卷
[root@master01 software]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/klas_host-10-200-91-54/swap
  LV Name                swap
  VG Name                klas_host-10-200-91-54
  LV UUID                mJCfg8-LLgS-NXuL-WtJ1-4T1X-a9bV-LcLrdO
  LV Write Access        read/write
  LV Creation host, time host-10-200-91-54, 2022-01-18 17:41:43 +0800
  LV Status              available
  # open                 0
  LV Size                <4.03 GiB
  Current LE             1031
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     4096
  Block device           252:1
   
  --- Logical volume ---
  LV Path                /dev/klas_host-10-200-91-54/backup
  LV Name                backup
  VG Name                klas_host-10-200-91-54
  LV UUID                yul3Nj-Rtyd-YGu0-nWny-sIHF-gj21-3LARG6
  LV Write Access        read/write
  LV Creation host, time host-10-200-91-54, 2022-01-18 17:41:43 +0800
  LV Status              available
  # open                 0
  LV Size                <31.09 GiB
  Current LE             7959
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     4096
  Block device           252:2
   
  --- Logical volume ---
  LV Path                /dev/klas_host-10-200-91-54/root
  LV Name                root
  VG Name                klas_host-10-200-91-54
  LV UUID                NwRken-eyAM-njwk-l2IU-jz0N-YfCF-0mvzcJ
  LV Write Access        read/write
  LV Creation host, time host-10-200-91-54, 2022-01-18 17:41:44 +0800
  LV Status              available
  # open                 1
  LV Size                63.68 GiB
  Current LE             16303
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     4096
  Block device           252:0
```

```
# 查看挂的云硬盘的名称
[root@master01 software]# fdisk -l 
Disk /dev/vda: 100 GiB, 107374182400 bytes, 209715200 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 26046881-8F99-4CA0-B01F-0473C537AC20

Device       Start       End   Sectors  Size Type
/dev/vda1     2048    411647    409600  200M EFI System
/dev/vda2   411648   2508799   2097152    1G Linux filesystem
/dev/vda3  2508800 209713151 207204352 98.8G Linux LVM




Disk /dev/mapper/klas_host--10--200--91--54-root: 63.7 GiB, 68379738112 bytes, 133554176 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/klas_host--10--200--91--54-swap: 4.3 GiB, 4324327424 bytes, 8445952 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/klas_host--10--200--91--54-backup: 31.9 GiB, 33382465536 bytes, 65200128 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

```
# 从上面看到卷组有100G，逻辑卷有一个backup的空闲，把空闲的加到 /dev/klas_host-10-200-91-54/root
# 删除卷组/dev/mapper/klas_host--10--200--91--54-backup
lvremove /dev/mapper/klas_host--10--200--91--54-backup
```

```
# 将剩余百分百空间都添加到逻辑卷中
lvextend -l +100%free /dev/mapper/klas_host--10--200--91--54-root
```

```
# 然后重新识别一下分区大小
xfs_growfs /dev/mapper/klas_host--10--200--91--54-root  ## 命令，后面跟的是逻辑卷的path
```
