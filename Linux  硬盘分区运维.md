Linux  硬盘分区运维  

VMware调整硬盘大小的逻辑    原来的硬盘大小sda是50G     直接把sd1变成100G 

```

[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0  100G  0 disk
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   15G  0 part
  ├─zstack-root 253:0    0 13.4G  0 lvm  /
  └─zstack-swap 253:1    0  1.6G  0 lvm  [SWAP]
sr0              11:0    1 1024M  0 rom
[root@localhost ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 3.8G     0  3.8G   0% /dev
tmpfs                    3.8G     0  3.8G   0% /dev/shm
tmpfs                    3.8G  9.1M  3.8G   1% /run
tmpfs                    3.8G     0  3.8G   0% /sys/fs/cgroup
/dev/mapper/zstack-root   14G   14G   28M 100% /
/dev/sda1               1014M  205M  810M  21% /boot
tmpfs                    777M     0  777M   0% /run/user/0
[root@localhost ~]#


```

？  那现在我们想要把/dev/mapper/zstack-root      这个根文件路径变大    应该怎么做呢？   

```bsah

[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0  100G  0 disk
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   15G  0 part
  ├─zstack-root 253:0    0 13.4G  0 lvm  /
  └─zstack-swap 253:1    0  1.6G  0 lvm  [SWAP]
sr0              11:0    1 1024M  0 rom
[root@localhost ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 3.8G     0  3.8G   0% /dev
tmpfs                    3.8G     0  3.8G   0% /dev/shm
tmpfs                    3.8G  9.1M  3.8G   1% /run
tmpfs                    3.8G     0  3.8G   0% /sys/fs/cgroup
/dev/mapper/zstack-root   14G   14G   28M 100% /
/dev/sda1               1014M  205M  810M  21% /boot
tmpfs                    777M     0  777M   0% /run/user/0
[root@localhost ~]#
[root@localhost ~]# fdisk -l
Disk /dev/sda: 100 GiB, 107374182400 bytes, 209715200 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa53eaa93

Device     Boot   Start      End  Sectors Size Id Type
/dev/sda1  *       2048  2099199  2097152   1G 83 Linux
/dev/sda2       2099200 33554431 31455232  15G 8e Linux LVM




Disk /dev/mapper/zstack-root: 13.4 GiB, 14382268416 bytes, 28090368 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/zstack-swap: 1.6 GiB, 1719664640 bytes, 3358720 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
[root@localhost ~]# lv
lvchange        lvmconfig       lvmlockd        lvrename
lvconvert       lvmdevices      lvmpolld        lvresize
lvcreate        lvmdiskscan     lvmsadc         lvs
lvdisplay       lvmdump         lvmsar          lvscan
lvextend        lvm_import_vdo  lvreduce
lvm             lvmlockctl      lvremove
[root@localhost ~]# lvextend   -1   +100%free   /dev/sda2
lvextend: invalid option -- '1'
  Error during parsing of command line.
[root@localhost ~]# lvextend   -l   +100%free   /dev/sda2
  "/dev/sda2": Invalid path for Logical Volume.
  Run `lvextend --help' for more information.
[root@localhost ~]# lvs
  LV   VG     Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root zstack -wi-ao---- 13.39g                                                 
  swap zstack -wi-ao----  1.60g                                                 
[root@localhost ~]# fdisk /dev/sda

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (3,4, default 3):
First sector (33554432-209715199, default 33554432):
Last sector, +sectors or +size{K,M,G,T,P} (33554432-209715199, default 209715199): +50G

Created a new partition 3 of type 'Linux' and of size 50 GiB.

Command (m for help): p
Disk /dev/sda: 100 GiB, 107374182400 bytes, 209715200 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa53eaa93

Device     Boot    Start       End   Sectors Size Id Type
/dev/sda1  *        2048   2099199   2097152   1G 83 Linux
/dev/sda2        2099200  33554431  31455232  15G 8e Linux LVM
/dev/sda3       33554432 138412031 104857600  50G 83 Linux

Command (m for help): t
Partition number (1-3, default 3):
Hex code (type L to list all codes): 8e

Changed type of partition 'Linux' to 'Linux LVM'.

Command (m for help): p
Disk /dev/sda: 100 GiB, 107374182400 bytes, 209715200 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa53eaa93

Device     Boot    Start       End   Sectors Size Id Type
/dev/sda1  *        2048   2099199   2097152   1G 83 Linux
/dev/sda2        2099200  33554431  31455232  15G 8e Linux LVM
/dev/sda3       33554432 138412031 104857600  50G 8e Linux LVM

Command (m for help): w
The partition table has been altered.
Syncing disks.

[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0  100G  0 disk
├─sda1            8:1    0    1G  0 part /boot
├─sda2            8:2    0   15G  0 part
│ ├─zstack-root 253:0    0 13.4G  0 lvm  /
│ └─zstack-swap 253:1    0  1.6G  0 lvm  [SWAP]
└─sda3            8:3    0   50G  0 part
sr0              11:0    1 1024M  0 rom
[root@localhost ~]# mkfs -t ext4 /dev/sda3
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 13107200 4k blocks and 3276800 inodes
Filesystem UUID: b94b8596-748b-4753-a663-5eae07eda7a6
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424

Allocating group tables: done
Writing inode tables: done
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done

[root@localhost ~]# pvcreate /dev/sda3
WARNING: ext4 signature detected on /dev/sda3 at offset 1080. Wipe it? [y/n]: y
  Wiping ext4 signature on /dev/sda3.
  Physical volume "/dev/sda3" successfully created.
[root@localhost ~]# gextend root  /dev/sda3
-bash: gextend: command not found
[root@localhost ~]# vgextend root  /dev/sda3
  Volume group "root" not found
  Cannot process volume group root
[root@localhost ~]# vgdisplay
  --- Volume group ---
  VG Name               zstack
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <15.00 GiB
  PE Size               4.00 MiB
  Total PE              3839
  Alloc PE / Size       3839 / <15.00 GiB
  Free  PE / Size       0 / 0
  VG UUID               C5LqDW-IVQG-RJ9W-1xWS-0Mu2-RT95-y45q8L

[root@localhost ~]# vgextend zstack  /dev/sda3
  Volume group "zstack" successfully extended
[root@localhost ~]# lvextend -L +50G /dev/zstack/root
  Insufficient free space: 12800 extents needed, but only 12799 available
[root@localhost ~]# lvextend -L +49G /dev/zstack/root
  Size of logical volume zstack/root changed from 13.39 GiB (3429 extents) to 62.39 GiB (15973 extents).
  Logical volume zstack/root successfully resized.
[root@localhost ~]#  xfs_growfs /dev/zstack/root
meta-data=/dev/mapper/zstack-root isize=512    agcount=4, agsize=877824 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=3511296, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 3511296 to 16356352
[root@localhost ~]# df -th
df: no file systems processed
[root@localhost ~]# df -Th
Filesystem              Type      Size  Used Avail Use% Mounted on
devtmpfs                devtmpfs  3.8G     0  3.8G   0% /dev
tmpfs                   tmpfs     3.8G     0  3.8G   0% /dev/shm
tmpfs                   tmpfs     3.8G  9.2M  3.8G   1% /run
tmpfs                   tmpfs     3.8G     0  3.8G   0% /sys/fs/cgroup
/dev/mapper/zstack-root xfs        63G   14G   49G  22% /
/dev/sda1               xfs      1014M  205M  810M  21% /boot
tmpfs                   tmpfs     777M     0  777M   0% /run/user/0
[root@localhost ~]# vgdisplay
  --- Volume group ---
  VG Name               zstack
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               64.99 GiB
  PE Size               4.00 MiB
  Total PE              16638
  Alloc PE / Size       16383 / <64.00 GiB
  Free  PE / Size       255 / 1020.00 MiB
  VG UUID               C5LqDW-IVQG-RJ9W-1xWS-0Mu2-RT95-y45q8L

[root@localhost ~]#

```



