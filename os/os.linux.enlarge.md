
# 具体过程演示

以下内容又臭又长，可以略过：

```bash
lsblk
```

```
NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sr0              11:0    1   4.3G  0 rom.
vda             252:0    0   200G  0 dis.
├─vda1          252:1    0     1G  0 part /boot
└─vda2          252:2    0 194.2G  0 par.
  ├─centos-root 253:0    0 186.3G  0 lvm  /
  └─centos-swap 253:1    0   7.9G  0 lvm  [SWAP
```

```bash
df -h
```

```
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root  187G  1.4G  185G   1% /
devtmpfs                 7.8G     0  7.8G   0% /dev
tmpfs                    7.8G     0  7.8G   0% /dev/shm
tmpfs                    7.8G   11M  7.8G   1% /run
tmpfs                    7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/vda1               1014M  148M  867M  15% /boot
tmpfs                    1.6G     0  1.6G   0% /run/user/0
```

```bash
fdisk -l
```

```
Disk /dev/vda: 214.7 GB, 214748364800 bytes, 419430400 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000a4b12
   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048     2099199     1048576   83  Linux
/dev/vda2         2099200   409255935   203578368   8e  Linux LVM
Disk /dev/mapper/centos-root: 200.0 GB, 199996997632 bytes, 390619136 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk /dev/mapper/centos-swap: 8455 MB, 8455716864 bytes, 16515072 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

```bash
vgdisplay
```

```
--- Volume group ---
  VG Name               centos
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
  VG Size               194.14 GiB
  PE Size               4.00 MiB
  Total PE              49701
  Alloc PE / Size       49699 / <194.14 GiB
  Free  PE / Size       2 / 8.00 MiB
  VG UUID               sH16CU-JTk0-5yv5-mwUI-FQ4f-xyVA-9z6UO1
```

#### 

## 物理盘

查看磁盘，在超融合中挂载上新的硬盘后，用`fdisk -l`查看（不需要重启）。

```bash
# 查看磁盘
# 可以查看到新增的/dev/vdb, 大小为40G
fdisk -l
```
```
Disk /dev/vda: 214.7 GB, 214748364800 bytes, 419430400 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000a4b12
   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048     2099199     1048576   83  Linux
/dev/vda2         2099200   409255935   203578368   8e  Linux LVM
Disk /dev/vdb: 42.9 GB, 42949672960 bytes, 83886080 sectors                    <== 新加的40G硬盘
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk /dev/mapper/centos-root: 200.0 GB, 199996997632 bytes, 390619136 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk /dev/mapper/centos-swap: 8455 MB, 8455716864 bytes, 16515072 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

如果需要分区的话，**根据提示操作**

```bash
fdisk /dev/vdb
```

```
Welcome to fdisk (util-linux 2.23.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
Command (m for help): m                                                     <== 显示帮助信息
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)
Command (m for help): n                                                      <== add a new partition 新建分区
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p                                                        <== 选择primary
Partition number (1-4, default 1): 1                                         <== 选择分区数, 默认1, 范围1-4
First sector (2048-83886079, default 2048): 2048                             <== 设置分区开始偏移量, 默认2048
Last sector, +sectors or +size{K,M,G} (2048-83886079, default 83886079): 20971520  <== 分出10G空间给分区1, 20971520 = 2 * 1024 * 1024
Partition 1 of type Linux and of size 10 GiB is set
Command (m for help): p                                                      <== print the partition table 显示分区
Disk /dev/vdb: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0005c05b
   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048    20971520    10484736+  83  Linux                <== 已经有了分区1 /dev/vdb1
Command (m for help): n                                                      <== add a new partition 新建分区
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p                                                        <== 选择primary
Partition number (2-4, default 2): 2                                         <== 选择分区数, 默认2(分区1已经被占用了), 范围2-4
First sector (20971521-83886079, default 20973568):                          <== 设置分区开始偏移量
Using default value 20973568
Last sector, +sectors or +size{K,M,G} (20973568-83886079, default 83886079): <== 剩余分区全部给分区2
Using default value 83886079
Partition 2 of type Linux and of size 30 GiB is set
Command (m for help): p                                                      <== print the partition table 显示分区
Disk /dev/vdb: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0005c05b
   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048    20971520    10484736+  83  Linux                <== 已经有了分区1 /dev/vdb1
/dev/vdb2        20973568    83886079    31456256   83  Linux                <== 已经有了分区2 /dev/vdb2
Command (m for help): w                                                      <== 将变更写入磁盘
The partition table has been altered!
Calling ioctl() to re-read partition table.
Syncing disks.
```

查看可用块，`vdb`下面已经有了两个 10G 和 30G 的分区了。

```bash
lsblk
```

```
NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sr0              11:0    1   4.3G  0 rom.
vda             252:0    0   200G  0 dis.
├─vda1          252:1    0     1G  0 part /boot
└─vda2          252:2    0 194.2G  0 par.
  ├─centos-root 253:0    0 186.3G  0 lvm  /
  └─centos-swap 253:1    0   7.9G  0 lvm  [SWAP]
vdb             252:16   0    40G  0 dis.
├─vdb1          252:17   0    10G  0 par.
└─vdb2          252:18   0    30G  0 part
```


## 物理卷 PV

创建物理卷：
```bash
# vdb分区后不能使用/dev/vdb创建pv
pvcreate /dev/vdb1
pvcreate /dev/vdb2
```

```
Physical volume "/dev/vdb1" successfully created.
  Physical volume "/dev/vdb2" successfully created.
```

查看物理卷，可以看到新增的物理卷`/dev/vdb1`

```bash
pvdisplay
```

```
--- Physical volume ---
  PV Name               /dev/vda2
  VG Name               centos
  PV Size               <194.15 GiB / not usable 3.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              49701
  Free PE               2
  Allocated PE          49699
  PV UUID               MeaN4C-EeCH-sBzL-UWoE-alSV-uf1x-ORyzHC
  "/dev/vdb1" is a new physical volume of "<10.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/vdb1
  VG Name
  PV Size               <10.00 GiB                                 <== 10G空间
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               eHA0g7-Zcsf-vDZ0-KoFR-M4VR-KPal-EQxnNJ
  "/dev/vdb2" is a new physical volume of "<30.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/vdb2
  VG Name
  PV Size               <30.00 GiB                                 <== 30G空间
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               LYeEb1-jsTc-NSUe-Mj2D-AosK-iGsC-fvvAQl
```


## 卷组 VG

朝已有的虚拟卷组里添加新的卷，不可以把已经隶属某虚拟卷组的卷利用`vgcreate`建立为新的卷组，会提示它已经被占用。

```bash
# 扩展虚拟卷组, 朝已有卷组增加卷
vgextend centos /dev/vdb1
```

```
Volume group "centos" successfully extended
```

查看虚拟卷组。

```bash
# 查看虚拟卷组
vgdisplay
```

```
--- Volume group ---
  VG Name               centos
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               204.14 GiB                    <== VG增加了10G
  PE Size               4.00 MiB
  Total PE              52260
  Alloc PE / Size       49699 / <194.14 GiB
  Free  PE / Size       2561 / 10.00 GiB
  VG UUID               sH16CU-JTk0-5yv5-mwUI-FQ4f-xyVA-9z6UO1
```

对`/dev/vdb2`同理

```bash
vgextend centos /dev/vdb2
```

```
Volume group "centos" successfully extended
```

```
--- Volume group ---
  VG Name               centos
  System ID
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <234.14 GiB                    <== VG又增加了30G
  PE Size               4.00 MiB
  Total PE              59939
  Alloc PE / Size       49699 / <194.14 GiB
  Free  PE / Size       10240 / 40.00 GiB
  VG UUID               sH16CU-JTk0-5yv5-mwUI-FQ4f-xyVA-9z6UO1
```


## 逻辑卷 LV

### 默认逻辑卷扩容

逻辑卷中扩展大小，**这种操作没有指定挂载目录, 需要则查看新增逻辑卷扩容**。

```bash
# 扩展逻辑卷
# +号表示增加容量  无+号表示扩展到的容量
# lvresize -L +10G /dev/mapper/centos-root(舍弃.
lvextend -l +100%free /dev/mapper/centos-root
```

```
Size of logical volume centos/root changed from 186.26 GiB (47683 extents) to 196.26 GiB (50243 extents).
  Logical volume centos/root successfully resized.
```

查看可用块。

```bash
lsblk
```

```
NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sr0              11:0    1   4.3G  0 rom
vda             252:0    0   200G  0 disk
├─vda1          252:1    0     1G  0 part /boot
└─vda2          252:2    0 194.2G  0 part
  ├─centos-root 253:0    0 197.3G  0 lvm  /
  └─centos-swap 253:1    0   7.9G  0 lvm  [SWAP]
vdb             252:16   0    40G  0 disk
└─centos-root   253:0    0 197.3G  0 lvm  /                  <== 已经增加了10G, 从186G增加到197GG
```

现在卷加载了但`df -h`的大小仍然没有变化。**调整文件系统**的大小，这步报错了。

```bash
# 调整文件系统
resize2fs /dev/mapper/centos-root
```

```
resize2fs 1.42.9 (28-Dec-2013)
resize2fs: Bad magic number in super-block while trying to open /dev/mapper/centos-root
Couldn't find valid filesystem superblock.
```

查看挂载类型, 发现为原类型为`xfs`。

```bash
mount | grep /dev/mapper/centos-root
```

```
/dev/mapper/centos-root on / type xfs (rw,relatime,attr2,inode64,noquota)
```

`xfs`更新需要使用`xfs_growfs`。

```bash
# 调整文件系统
xfs_growfs /dev/mapper/centos-root
```

```
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=12206848 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=48827392, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=23841, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 48827392 to 51448832
```

查看文件系统

```bash
df -h
```

```
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root  197G  1.4G  195G   1% /                    <== 已经增加了10G, 从186G增加到197GG
devtmpfs                 7.8G     0  7.8G   0% /dev
tmpfs                    7.8G     0  7.8G   0% /dev/shm
tmpfs                    7.8G   11M  7.8G   1% /run
tmpfs                    7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/vda1               1014M  148M  867M  15% /boot
tmpfs                    1.6G     0  1.6G   0% /run/user/0
```


### 新增逻辑卷扩容

查看逻辑卷。

```bash
lvdisplay
```

```
--- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                ZrxK5j-jcMc-7HSO-hEav-weJQ-odly-ltWQlp
  LV Write Access        read/write
  LV Creation host, time localhost, 2021-03-19 08:52:34 +0800
  LV Status              available
  # open                 1
  LV Size                196.26 GiB                                 <== 已经增加了10G, 从186G增加到197GG
  Current LE             50243
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
  --- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                93Y9wh-Iueg-Hn03-lA59-h54X-qd8s-dxz7lz
  LV Write Access        read/write
  LV Creation host, time localhost, 2021-03-19 08:52:34 +0800
  LV Status              available
  # open                 2
  LV Size                <7.88 GiB
  Current LE             2016
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1
```

创建一个新的逻辑卷。

```bash
lvcreate -L 20G -n /dev/centos/data
```

```
Logical volume "data" created.
```

查看逻辑卷。

```bash
lvdisplay
```

```
--- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                ZrxK5j-jcMc-7HSO-hEav-weJQ-odly-ltWQlp
  LV Write Access        read/write
  LV Creation host, time localhost, 2021-03-19 08:52:34 +0800
  LV Status              available
  # open                 1
  LV Size                196.26 GiB
  Current LE             50243
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
  --- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                93Y9wh-Iueg-Hn03-lA59-h54X-qd8s-dxz7lz
  LV Write Access        read/write
  LV Creation host, time localhost, 2021-03-19 08:52:34 +0800
  LV Status              available
  # open                 2
  LV Size                <7.88 GiB
  Current LE             2016
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1
  --- Logical volume ---
  LV Path                /dev/centos/data
  LV Name                data
  VG Name                centos
  LV UUID                xSlCic-P8m1-ewy8-X7Kl-Y5mp-26Xa-mpy1Ht
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2021-05-25 17:42:01 +0800
  LV Status              available
  # open                 0
  LV Size                20.00 GiB                                <== 分配20G给/dev/centos/data
  Current LE             5120
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
```

格式化这个逻辑卷。

```bash
mkfs -t ext4 /dev/centos/data
```

```
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
1310720 inodes, 5242880 blocks
262144 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2153775104
160 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000
Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

创建挂载目录并挂载到其上.

```bash
# 创建目录
mkdir /data
# 挂载目录
mount /dev/centos/data /data
# 查看文件系统
df -h
```

```
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root  197G  1.4G  195G   1% /
devtmpfs                 7.8G     0  7.8G   0% /dev
tmpfs                    7.8G     0  7.8G   0% /dev/shm
tmpfs                    7.8G   11M  7.8G   1% /run
tmpfs                    7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/vda1               1014M  148M  867M  15% /boot
tmpfs                    1.6G     0  1.6G   0% /run/user/0
/dev/mapper/centos-data   20G   45M   19G   1% /data            <== 挂载20G逻辑卷到/data目录下
```

检查可用块。

```bash
lsblk
```

```
NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sr0              11:0    1   4.3G  0 rom
vda             252:0    0   200G  0 disk
├─vda1          252:1    0     1G  0 part /boot
└─vda2          252:2    0 194.2G  0 part
  ├─centos-root 253:0    0 196.3G  0 lvm  /
  └─centos-swap 253:1    0   7.9G  0 lvm  [SWAP]
vdb             252:16   0    40G  0 disk
├─vdb1          252:17   0    10G  0 part
│ └─centos-root 253:0    0 196.3G  0 lvm  /
└─vdb2          252:18   0    30G  0 part
  └─centos-data 254:2    0    20G  0 lvm  /data            <== 20G可用块
```

配置开机自动挂载，在`/etc/fstab`文件下写入. 如果不这么做, 重启后将不会出现挂载的目录。

```bash
vim /etc/fstab
```

```
/dev/mapper/centos-data /data     ext4    defaults   0    0
```

`/etc/fstab`中各个域的解释（建议不要改动最后两个域，容易导致**内核启动失败**）。

| 域 | 解释 |
| --- | --- |
| file system | 挂载的文件系统的设备名称或块信息，也可以是远程的文件系统。 |
| mount point | 挂载点: 挂到这个目录上，然后就可以从这个目录中访问要挂载文件系统。<br />对于`swap`<br />分区填：`none`，表示没有挂载点 |
| type | 指定文件系统的类型：<br />`ext3``ext2``ext``swap``nfs``hpfs``ncpfs``ntfs``proc``cifs``iso9660`等等 |
| options | 这里用来填写设置选项, 各个选项用逗号隔开；<br />选项非常多, 不作详细介绍, 如需了解, 请用 命令 `man mount`<br />来查看；<br />可以使用`defaults`<br />关键字：它代表包含了选项`rw``suid``dev``exec``auto``nouser``async` |
| dump | 为 1 的话，表示要将整个里的内容备份；<br />为 0 的话。表示不备份。<br />现在很少用到 dump 这个工具，在这里一般选 0。 |
| pass | 使用`fsck`<br />来检查硬盘：<br />如果这里填 0，则不检查；<br />挂载点为`/`的（即根分区），必须在这里填写 1，其他的都不能填写 1；<br />如果有分区填写大于 1 的话，则在检查完根分区后，接着按填写的数字从小到大依次检查下去；<br />同数字的同时检查。比如第一和第二个分区填写 2，第三和第四个分区填写 3，则系统在检查完根分区后，接着同时检查第一和第二个分区，然后再同时检查第三和第四个分区。 |

