# TP1 : Single-machine storage

## Sommaire

- [TP1 : Single-machine storage](#tp1--single-machine-storage)
  - [Sommaire](#sommaire)
- [I. Fundamentals](#i-fundamentals)
- [II. Partitioning](#ii-partitioning)
- [III. RAID](#iii-raid)
  - [1. Simple RAID](#1-simple-raid)
  - [2. Break it](#2-break-it)
  - [3. Spare disk](#3-spare-disk)
  - [4. Grow](#4-grow)
- [IV. NFS](#iv-nfs)

# I. Fundamentals

🌞 **Lister tous les périphériques de stockage branchés à la VM**

```
[root@storage ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 10.3G  0 disk 
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0  9.3G  0 part 
  ├─rl-root 253:0    0  8.3G  0 lvm  /
  └─rl-swap 253:1    0    1G  0 lvm  [SWAP]
sdb           8:16   0   10G  0 disk 
sdc           8:32   0   10G  0 disk 
sdd           8:48   0   10G  0 disk 
sde           8:64   0   10G  0 disk 
sdf           8:80   0   10G  0 disk 
sdg           8:96   0   10G  0 disk 
sr0          11:0    1 1024M  0 rom  
```

🌞 **Lister toutes les partitions des périphériques de stockage**

```
[root@storage ~]# lsblk -a -f
NAME        FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                                          
├─sda1      xfs                        c25d6621-d27e-41d9-b6d3-f326ef0d8612    661.5M    31% /boot
└─sda2      LVM2_member LVM2 001       iD31S0-vBOT-zYV3-GiX0-z2bM-nURX-wLYZ0p                
  ├─rl-root xfs                        a3739e1c-10c4-4bd0-b7e6-ab0cc1843804      6.1G    26% /
  └─rl-swap swap        1              5aaad074-6c89-4041-ad41-df18904ac375                  [SWAP]
sdb                                                                                          
sdc                                                                                          
sdd                                                                                          
sde                                                                                          
sdf                                                                                          
sdg                                                                                          
sr0 
```

🌞 **Effectuer un test SMART sur le disque**

```
[root@storage ~]# smartctl -a /dev/sda
smartctl 7.2 2020-12-30 r5155 [x86_64-linux-5.14.0-427.42.1.el9_4.x86_64] (local build)
Copyright (C) 2002-20, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Device Model:     VBOX HARDDISK
Serial Number:    VB68b55215-42eb4507
Firmware Version: 1.0
User Capacity:    11,046,649,856 bytes [11.0 GB]
Sector Size:      512 bytes logical/physical
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ATA/ATAPI-6 published, ANSI INCITS 361-2002
Local Time is:    Thu Nov 14 13:50:19 2024 CET
SMART support is: Unavailable - device lacks SMART capability.

A mandatory SMART command failed: exiting. To continue, add one or more '-T permissive' options.

```

🌞 **Espace disque...**

```
[root@storage ~]# df -h /
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/rl-root  8.2G  2.2G  6.1G  26% /
```

🌞 **Inodes**

```
[root@storage ~]# df -i 
Filesystem           Inodes IUsed   IFree IUse% Mounted on
devtmpfs             463332   443  462889    1% /dev
tmpfs                468377     1  468376    1% /dev/shm
tmpfs                819200   661  818539    1% /run
/dev/mapper/rl-root 4327424 36040 4291384    1% /
/dev/sda1            524288   366  523922    1% /boot
tmpfs                 93675    15   93660    1% /run/user/0
```
🌞 **Latence disque**

```
[root@storage ~]#  ioping -c 10 /
4 KiB <<< / (xfs /dev/dm-0 8.19 GiB): request=1 time=964.9 us (warmup)
4 KiB <<< / (xfs /dev/dm-0 8.19 GiB): request=2 time=933.6 us
4 KiB <<< / (xfs /dev/dm-0 8.19 GiB): request=3 time=983.9 us
4 KiB <<< / (xfs /dev/dm-0 8.19 GiB): request=4 time=766.5 us
4 KiB <<< / (xfs /dev/dm-0 8.19 GiB): request=5 time=1.09 ms
4 KiB <<< / (xfs /dev/dm-0 8.19 GiB): request=6 time=985.6 us
4 KiB <<< / (xfs /dev/dm-0 8.19 GiB): request=7 time=913.5 us
4 KiB <<< / (xfs /dev/dm-0 8.19 GiB): request=8 time=435.2 us (fast)
4 KiB <<< / (xfs /dev/dm-0 8.19 GiB): request=9 time=1.38 ms
4 KiB <<< / (xfs /dev/dm-0 8.19 GiB): request=10 time=515.5 us

--- / (xfs /dev/dm-0 8.19 GiB) ioping statistics ---
9 requests completed in 8.00 ms, 36 KiB read, 1.12 k iops, 4.39 MiB/s
generated 10 requests in 9.02 s, 40 KiB, 1 iops, 4.43 KiB/s
min/avg/max/mdev = 435.2 us / 889.0 us / 1.38 ms / 270.7 us

```

🌞 **Déterminer la taille du cache *filesystem***

```
[root@storage ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           3.6Gi       383Mi       3.1Gi       8.0Mi       359Mi       3.2Gi
Swap:          1.0Gi          0B       1.0Gi

```

# II. Partitioning

🌞 **Ajouter `sdb` comme Physical Volume LVM**

```
[root@storage ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.

```

```
[root@storage ~]# pvdisplay /dev/sdb
  "/dev/sdb" is a new physical volume of "10.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb
  VG Name               
  PV Size               10.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               Qedqpe-zf7A-gopi-9azy-Jlfg-B5OL-O2n0ic

```

🌞 **Créer un Volume Group LVM nommé storage**

```
[root@storage ~]# vgcreate storage /dev/sdb
  Volume group "storage" successfully created
```

🌞 **Créer un Logical Volume LVM**

```
[root@storage ~]# lvcreate -L 2G -n smol_data storage
  Logical volume "smol_data" created.
```

🌞 **Créer un deuxième Logical Volume LVM**

```
[root@storage ~]# lvcreate -l 100%FREE -n big_data storage
  Logical volume "big_data" created.
```

🌞 **Créez un système de fichiers sur les deux LVs (Avec xfs problème avec ext4)**

```
[root@storage network-scripts]# mkfs.xfs -f /dev/storage/smol_data
meta-data=/dev/storage/smol_data isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@storage network-scripts]# mkfs.xfs -f /dev/storage/big_data
meta-data=/dev/storage/big_data  isize=512    agcount=4, agsize=524032 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=2096128, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

🌞 **Montez la partition**

```
[root@storage network-scripts]# mount -t xfs /dev/storage/big_data /mnt/lvm_storage
[root@storage network-scripts]# 

```

🌞 **Configurer un *automount***

```
[root@storage ~]# cat /etc/systemd/system/mnt-lvm_storage.automount
[Unit]
Description=Automount for big_data

[Automount]
Where=/mnt/lvm_storage
TimeoutSec=30

[Install]
WantedBy=multi-user.target

[root@storage ~]# cat /etc/systemd/system/mnt-lvm_storage.mount
[Unit]
Description=Mount for big_data
After=network.target

[Mount]
What=/dev/storage/big_data
Where=/mnt/lvm_storage
Type=xfs
Options=defaults
TimeoutSec=30

[Install]
WantedBy=multi-user.target


```

```
[root@storage ~]# systemctl daemon-reload
[root@storage ~]# systemctl enable  mnt-lvm_storage.automount
[root@storage ~]# systemctl enable  mnt-lvm_storage.mount

```

🌞 **Prouvez que l'*automount* est effectif**

```
[root@storage ~]# lsblk
NAME                MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                   8:0    0 10.3G  0 disk 
├─sda1                8:1    0    1G  0 part /boot
└─sda2                8:2    0  9.3G  0 part 
  ├─rl-root         253:0    0  8.3G  0 lvm  /
  └─rl-swap         253:1    0    1G  0 lvm  [SWAP]
sdb                   8:16   0   10G  0 disk 
├─storage-smol_data 253:2    0    2G  0 lvm  
└─storage-big_data  253:3    0    8G  0 lvm  
sdc                   8:32   0   10G  0 disk 
sdd                   8:48   0   10G  0 disk 
sde                   8:64   0   10G  0 disk 
sdf                   8:80   0   10G  0 disk 
sdg                   8:96   0   10G  0 disk 
sr0                  11:0    1 1024M  0 rom  

[root@storage ~]# ls /mnt/lvm_storage
[root@storage ~]# lsblk
NAME                MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                   8:0    0 10.3G  0 disk 
├─sda1                8:1    0    1G  0 part /boot
└─sda2                8:2    0  9.3G  0 part 
  ├─rl-root         253:0    0  8.3G  0 lvm  /
  └─rl-swap         253:1    0    1G  0 lvm  [SWAP]
sdb                   8:16   0   10G  0 disk 
├─storage-smol_data 253:2    0    2G  0 lvm  
└─storage-big_data  253:3    0    8G  0 lvm  /mnt/lvm_storage
sdc                   8:32   0   10G  0 disk 
sdd                   8:48   0   10G  0 disk 
sde                   8:64   0   10G  0 disk 
sdf                   8:80   0   10G  0 disk 
sdg                   8:96   0   10G  0 disk 
sr0                  11:0    1 1024M  0 rom  

[root@storage ~]# lsblk
NAME                MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                   8:0    0 10.3G  0 disk 
├─sda1                8:1    0    1G  0 part /boot
└─sda2                8:2    0  9.3G  0 part 
  ├─rl-root         253:0    0  8.3G  0 lvm  /
  └─rl-swap         253:1    0    1G  0 lvm  [SWAP]
sdb                   8:16   0   10G  0 disk 
├─storage-smol_data 253:2    0    2G  0 lvm  
└─storage-big_data  253:3    0    8G  0 lvm  
sdc                   8:32   0   10G  0 disk 
sdd                   8:48   0   10G  0 disk 
sde                   8:64   0   10G  0 disk 
sdf                   8:80   0   10G  0 disk 
sdg                   8:96   0   10G  0 disk 
sr0                  11:0    1 1024M  0 rom  
```

# III. RAID

## 1. Simple RAID

🌞 **Mettre en place un RAID 5  **

```
[root@storage ~]# mdadm --create  /dev/md0 --level=5 --raid-devices=3 /dev/sdc /dev/sdd /dev/sde
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

```

🌞 **Prouvez que le RAID5 est en place**, on doit voir :

```
[root@storage ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sde[3] sdd[1] sdc[0]
      20953088 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/2] [UU_]
      [==========>..........]  recovery = 52.9% (5543800/10476544) finish=0.7min speed=109673K/sec
      
unused devices: <none>
[root@storage ~]# mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Dec  4 09:21:44 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 3
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Wed Dec  4 09:23:18 2024
             State : clean 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 211fa87d:7d68597b:89dbf086:5ff41773
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       32        0      active sync   /dev/sdc
       1       8       48        1      active sync   /dev/sdd
       3       8       64        2      active sync   /dev/sde
[root@storage ~]# df -h /mnt/raid5
Filesystem      Size  Used Avail Use% Mounted on
/dev/md0         20G   24K   19G   1% /mnt/raid5

```

🌞 **Rendre la configuration automatique au boot de la machine**

```
[root@storage ~]# cat /etc/mdadm.conf
ARRAY /dev/md0 metadata=1.2 UUID=211fa87d:7d68597b:89dbf086:5ff41773
```

```
[root@storage ~]# cat  /etc/fstab

#
# /etc/fstab
# Created by anaconda on Thu Nov 14 15:27:56 2024
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rl-root     /                       xfs     defaults        0 0
UUID=c93b1aaa-dc60-4efe-a460-91e7c7b714c9 /boot                   xfs     defaults        0 0
/dev/mapper/rl-swap     none                    swap    defaults        0 0
UUID="abcd1234-5678-90ef-ghij-klmnopqrstuv" /mnt/raid5  ext4  defaults  0  0

```

🌞 **Créez un système de fichiers sur la partition proposé par le RAID**

```
[root@storage ~]# blkid /dev/md0
/dev/md0: UUID="13646d91-6447-4594-ae18-0cdfe01b14be" TYPE="ext4"

```

🌞 **Monter la partition sur `/mnt/raid_storage`**

```
[root@storage ~]# mkdir -p /mnt/raid_storage
[root@storage ~]# mount /dev/md0 /mnt/raid_storage
[root@storage ~]# df -h /mnt/raid_storage
Filesystem      Size  Used Avail Use% Mounted on
/dev/md0         20G   24K   19G   1% /mnt/raid_storage
```

🌞 **Prouvez que...**

```
[root@storage ~]# df -h /mnt/raid_storage
Filesystem      Size  Used Avail Use% Mounted on
/dev/md0         20G   24K   19G   1% /mnt/raid_storage
[root@storage ~]# touch /mnt/raid_storage/testfile
[root@storage ~]# ls -l /mnt/raid_storage
total 16
drwx------. 2 root root 16384 Dec  4 09:25 lost+found
-rw-r--r--. 1 root root     0 Dec  4 09:32 testfile
[root@storage ~]# rm /mnt/raid_storage/testfile
rm: remove regular empty file '/mnt/raid_storage/testfile'? y
[root@storage ~]# ls -l /mnt/raid_storage
total 16
drwx------. 2 root root 16384 Dec  4 09:25 lost+found

```

🌞 **Mini benchmark**

```
[root@storage ~]# time sudo dd if=/dev/zero of=/mnt/raid_storage/testfile bs=1G count=1 oflag=direct
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 6.5723 s, 163 MB/s

real    0m6.634s
user    0m0.010s
sys     0m0.887s
```

- idem sur la partition créée avec LVM à la partie précédente : `/mnt/lvm_storage`

problème de vm la dernière fois je n'ai plus ce que j'avais fait avant.

## 2. Break it

🌞 **Simule une panne**

```
[root@storage ~]# mdadm --manage /dev/md0 --fail /dev/sdc
mdadm: set /dev/sdc faulty in /dev/md0
```

🌞 **Montre l'état du RAID dégradé**

```
[root@storage ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sde[3] sdd[1] sdc[0](F)
      20953088 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/2] [_UU]
```

🌞 **Remonte le disque dur**

```
[root@storage ~]# mdadm --manage /dev/md0 --add /dev/sdf
mdadm: added /dev/sdf
[root@storage ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sdf[4] sde[3] sdd[1] sdc[0](F)
      20953088 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/2] [_UU]
      [>....................]  recovery =  3.7% (392476/10476544) finish=0.8min speed=196238K/sec

```

## 3. Spare disk

🌞 **Ajoutez un disque encore inutilisé au RAID5 comme disque de *spare***

```
[root@storage ~]# mdadm --manage /dev/md0 --add /dev/sdg
mdadm: added /dev/sdg
[root@storage ~]# mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Dec  4 09:21:44 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 3
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Wed Dec  4 09:42:00 2024
             State : clean, degraded, recovering 
    Active Devices : 2
   Working Devices : 4
    Failed Devices : 1
     Spare Devices : 2

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

    Rebuild Status : 70% complete

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 211fa87d:7d68597b:89dbf086:5ff41773
            Events : 35

    Number   Major   Minor   RaidDevice State
       4       8       80        0      spare rebuilding   /dev/sdf
       1       8       48        1      active sync   /dev/sdd
       3       8       64        2      active sync   /dev/sde

       0       8       32        -      faulty   /dev/sdc
       5       8       96        -      spare   /dev/sdg

```

🌞 **Simuler une panne**

```
[root@storage ~]# mdadm --manage /dev/md0 --fail /dev/sdc
mdadm: set /dev/sdc faulty in /dev/md0
[root@storage ~]# mdadm --manage /dev/md0 --re-add /dev/sdc
```
🌞 **Remonter le disque débranché**

```
[root@storage ~]# mdadm --manage /dev/md0 --add /dev/sdg
mdadm: added /dev/sdg
[root@storage ~]# mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Dec  4 09:21:44 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 3
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Wed Dec  4 09:42:00 2024
             State : clean, degraded, recovering 
    Active Devices : 2
   Working Devices : 4
    Failed Devices : 1
     Spare Devices : 2

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

    Rebuild Status : 70% complete

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 211fa87d:7d68597b:89dbf086:5ff41773
            Events : 35

    Number   Major   Minor   RaidDevice State
       4       8       80        0      spare rebuilding   /dev/sdf
       1       8       48        1      active sync   /dev/sdd
       3       8       64        2      active sync   /dev/sde

       0       8       32        -      faulty   /dev/sdc
       5       8       96        -      spare   /dev/sdg

```

## 4. Grow

🌞 **Ajoutez un disque encore inutilisé au RAID5 comme disque de *spare***

```
[root@storage ~]# mdadm --manage /dev/md0 --add /dev/sdb
mdadm: added /dev/sdb
[root@storage ~]# mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Dec  4 09:21:44 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 3
     Total Devices : 6
       Persistence : Superblock is persistent

       Update Time : Wed Dec  4 09:50:32 2024
             State : clean 
    Active Devices : 3
   Working Devices : 5
    Failed Devices : 1
     Spare Devices : 2

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 211fa87d:7d68597b:89dbf086:5ff41773
            Events : 43

    Number   Major   Minor   RaidDevice State
       4       8       80        0      active sync   /dev/sdf
       1       8       48        1      active sync   /dev/sdd
       3       8       64        2      active sync   /dev/sde

       5       8       96        -      spare   /dev/sdg
       6       8       16        -      spare   /dev/sdb

```

🌞 **Grow !**

```
[root@storage ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sdb[6](S) sdg[5] sdf[4] sde[3] sdd[1]
      20953088 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/4] [UUU]

```
🌞 **Prouvez que le RAID5 propose désormais 4 disques actifs**

```
[root@storage ~]# cat /proc/mdstat 
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sdg[6] sdd[5](S) sdf[4] sde[3] sdc[0]
      15713280 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/4] [UUUU]
      
unused devices: <none>
[root@storage ~]# lsblk
NAME                MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
[...]
sdc                   8:32   0    5G  0 disk  
└─md0                 9:0    0   15G  0 raid5 /mnt/raid_storage
sdd                   8:48   0    5G  0 disk  
└─md0                 9:0    0   15G  0 raid5 /mnt/raid_storage
sde                   8:64   0    5G  0 disk  
└─md0                 9:0    0   15G  0 raid5 /mnt/raid_storage
sdf                   8:80   0    5G  0 disk  
└─md0                 9:0    0   15G  0 raid5 /mnt/raid_storage
sdg                   8:96   0    5G  0 disk  
└─md0                 9:0    0   15G  0 raid5 /mnt/raid_storage
sr0                  11:0    1  1.7G  0 rom 
```

🌞 **Euuuh wait a sec... `/mnt/raid_storage` ???**

```
[root@storage ~]# sudo resize2fs /dev/md0
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/md0 is mounted on /mnt/raid_storage; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 2
The filesystem on /dev/md0 is now 3928320 (4k) blocks long.

[root@storage ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                385M     0  385M   0% /dev/shm
tmpfs                154M  3.9M  150M   3% /run
/dev/mapper/rl-root  8.0G  1.6G  6.4G  20% /
/dev/sda1            960M  395M  566M  42% /boot
/dev/md0              15G   24K   14G   1% /mnt/raid_storage
tmpfs                 77M     0   77M   0% /run/user/0
```

# IV. NFS

🌞 **Installer un serveur NFS**

```
[root@storage lvm_storage]
sudo chmod -R 755 /mnt/raid_storage
sudo chmod -R 755 /mnt/lvm_storage
[root@storage lvm_storage]# sudo nano /etc/exports
[root@storage lvm_storage]# sudo systemctl enable --now nfs-server
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
[root@storage lvm_storage]# sudo exportfs -rav
exporting 192.168.10.0/24:/mnt/lvm_storage
exporting 192.168.10.0/24:/mnt/raid_storage
[root@storage lvm_storage]# sudo exportfs -v
/mnt/raid_storage
                192.168.10.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
/mnt/lvm_storage
                192.168.10.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
[root@storage lvm_storage]# sudo firewall-cmd --permanent --zone=public --add-service=nfs
sudo firewall-cmd --permanent --zone=public --add-service=rpc-bind
sudo firewall-cmd --permanent --zone=public --add-service=mountd
sudo firewall-cmd --reload
success
success
success
success
[root@storage lvm_storage]# sudo firewall-cmd --zone=trusted --add-interface=ens18 --permanent
sudo firewall-cmd --reload
The interface is under control of NetworkManager, setting zone to 'trusted'.
success
success
```

🌞 **Pop une deuxième VM en vif**

```
[root@raid2 test_raid]# sudo mount -t nfs 10.0.1.20:/mnt/lvm_storage /mnt/test_big/
[root@raid2 test_raid]# sudo mount -t nfs 10.0.1.20:/mnt/raid_storage /mnt/test_raid/
[root@raid2 test_raid]# ls
lost+found
[root@raid2 test_raid]# cd ..
[root@raid2 mnt]# ls
test_big  test_raid
[root@raid2 mnt]# cd test_big/
[root@raid2 test_big]# ls
lost+found  testfile
```

🌞 **Benchmarkz**

```
[root@raid2 mnt]# dd if=/dev/zero of=/mnt/test_raid/testfilentfs bs=1G count=1 oflag=direct
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 3.35611 s, 320 MB/s
[root@git mnt]# dd if=/dev/zero of=/mnt/test_big/testfilentfs bs=1G count=1 oflag=direct
dd: error writing '/mnt/test_big/testfilentfs': No space left on device
1+0 records in
0+0 records out
937689088 bytes (938 MB, 894 MiB) copied, 6.0969 s, 154 MB/s
#oups y a plus de place 


[root@storage lvm_storage]# ls
lost+found  testfile  testfilentfs
[root@storage lvm_storage]# cd ..
[root@storage mnt]# ls
lvm_storage  raid_storage
[root@storage mnt]# cd raid_storage/
[root@storage raid_storage]# ls
lost+found  testfilentfs
[root@storage raid_storage]# df -h
Filesystem                    Size  Used Avail Use% Mounted on
devtmpfs                      4.0M     0  4.0M   0% /dev
tmpfs                         385M     0  385M   0% /dev/shm
tmpfs                         154M  3.1M  151M   3% /run
/dev/mapper/rl-root           8.0G  1.6G  6.4G  20% /
/dev/sda1                     960M  395M  566M  42% /boot
/dev/md0                       15G  1.1G   13G   8% /mnt/raid_storage
/dev/mapper/storage-big_data  2.0G  1.9G     0 100% /mnt/lvm_storage
```