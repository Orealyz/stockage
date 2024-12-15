
# II. SAN network


## 1. Storage machines

### A. Disks and RAID


🌞 **Configurer des RAID**

sto1
```
[root@sto1 ~]# mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
[root@sto1 ~]# mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/sdd /dev/sde
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
[root@sto1 ~]# mdadm --create /dev/md2 --level=1 --raid-devices=2 /dev/sdf /dev/sdg
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md2 started.

```

sto2
```
[root@sto2 ~]# mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
[root@sto2 ~]# mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/sdd /dev/sde
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
[root@sto2 ~]# mdadm --create /dev/md2 --level=1 --raid-devices=2 /dev/sdf /dev/sdg
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md2 started.

```

🌞 **Prouvez que vous avez 3 volumes RAID prêts à l'emploi**

sto1
```
[root@sto1 ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0   10G  0 disk  
├─sda1        8:1    0    1G  0 part  /boot
└─sda2        8:2    0    9G  0 part  
  ├─rl-root 253:0    0    8G  0 lvm   /
  └─rl-swap 253:1    0    1G  0 lvm   [SWAP]
sdb           8:16   0    1G  0 disk  
└─md0         9:0    0 1022M  0 raid1 
sdc           8:32   0    1G  0 disk  
└─md0         9:0    0 1022M  0 raid1 
sdd           8:48   0    1G  0 disk  
└─md1         9:1    0 1022M  0 raid1 
sde           8:64   0    1G  0 disk  
└─md1         9:1    0 1022M  0 raid1 
sdf           8:80   0    1G  0 disk  
└─md2         9:2    0 1022M  0 raid1 
sdg           8:96   0    1G  0 disk  
└─md2         9:2    0 1022M  0 raid1 
sr0          11:0    1 1024M  0 rom 
```

sto2
```
[root@sto2 ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0   10G  0 disk  
├─sda1        8:1    0    1G  0 part  /boot
└─sda2        8:2    0    9G  0 part  
  ├─rl-root 253:0    0    8G  0 lvm   /
  └─rl-swap 253:1    0    1G  0 lvm   [SWAP]
sdb           8:16   0    1G  0 disk  
└─md0         9:0    0 1022M  0 raid1 
sdc           8:32   0    1G  0 disk  
└─md0         9:0    0 1022M  0 raid1 
sdd           8:48   0    1G  0 disk  
└─md1         9:1    0 1022M  0 raid1 
sde           8:64   0    1G  0 disk  
└─md1         9:1    0 1022M  0 raid1 
sdf           8:80   0    1G  0 disk  
└─md2         9:2    0 1022M  0 raid1 
sdg           8:96   0    1G  0 disk  
└─md2         9:2    0 1022M  0 raid1 
sr0          11:0    1 1024M  0 rom 
```

### B. iSCSI target


🌞 **Installer `target`**

```
[root@sto1 ~]#  dnf install targetcli -y
[root@sto2 ~]# dnf install targetcli -y

```

🌞 **Démarrer le service `target`**

```
[root@sto1 ~]# systemctl enable --now target
[root@sto2 ~]# systemctl enable --now target
Created symlink /etc/systemd/system/multi-user.target.wants/target.service → /usr/lib/systemd/system/target.service.
```

🌞 **Configurer les *targets iSCSI***

sto1
```
[root@sto1 ~]# targetcli
targetcli shell version 2.1.57
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/> 
/> ls
o- / ................................................................................. [...]
  o- backstores ...................................................................... [...]
  | o- block .......................................................... [Storage Objects: 0]
  | o- fileio ......................................................... [Storage Objects: 3]
  | | o- data-chunk1 ........................... [/dev/md0 (1022.0MiB) write-back activated]
  | | | o- alua ........................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ............................... [ALUA state: Active/optimized]
  | | o- data-chunk2 ........................... [/dev/md1 (1022.0MiB) write-back activated]
  | | | o- alua ........................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ............................... [ALUA state: Active/optimized]
  | | o- data-chunk3 ........................... [/dev/md2 (1022.0MiB) write-back activated]
  | |   o- alua ........................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ............................... [ALUA state: Active/optimized]
  | o- pscsi .......................................................... [Storage Objects: 0]
  | o- ramdisk ........................................................ [Storage Objects: 0]
  o- iscsi .................................................................... [Targets: 3]
  | o- iqn.2024-12.tp2.b3:data-chunk1 ............................................ [TPGs: 1]
  | | o- tpg1 ....................................................... [no-gen-acls, no-auth]
  | |   o- acls .................................................................. [ACLs: 1]
  | |   | o- iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator .............. [Mapped LUNs: 1]
  | |   |   o- mapped_lun0 .................................. [lun0 fileio/data-chunk1 (rw)]
  | |   o- luns .................................................................. [LUNs: 1]
  | |   | o- lun0 ....................... [fileio/data-chunk1 (/dev/md0) (default_tg_pt_gp)]
  | |   o- portals ............................................................ [Portals: 1]
  | |     o- 0.0.0.0:3260 ............................................................. [OK]
  | o- iqn.2024-12.tp2.b3:data-chunk2 ............................................ [TPGs: 1]
  | | o- tpg1 ....................................................... [no-gen-acls, no-auth]
  | |   o- acls .................................................................. [ACLs: 1]
  | |   | o- iqn.2024-12.tp2.b3:data-chunk2:chunk2-initiator .............. [Mapped LUNs: 1]
  | |   |   o- mapped_lun0 .................................. [lun0 fileio/data-chunk2 (rw)]
  | |   o- luns .................................................................. [LUNs: 1]
  | |   | o- lun0 ....................... [fileio/data-chunk2 (/dev/md1) (default_tg_pt_gp)]
  | |   o- portals ............................................................ [Portals: 1]
  | |     o- 0.0.0.0:3260 ............................................................. [OK]
  | o- iqn.2024-12.tp2.b3:data-chunk3 ............................................ [TPGs: 1]
  |   o- tpg1 ....................................................... [no-gen-acls, no-auth]
  |     o- acls .................................................................. [ACLs: 1]
  |     | o- iqn.2024-12.tp2.b3:data-chunk3:chunk3-initiator .............. [Mapped LUNs: 1]
  |     |   o- mapped_lun0 .................................. [lun0 fileio/data-chunk3 (rw)]
  |     o- luns .................................................................. [LUNs: 1]
  |     | o- lun0 ....................... [fileio/data-chunk3 (/dev/md2) (default_tg_pt_gp)]
  |     o- portals ............................................................ [Portals: 1]
  |       o- 0.0.0.0:3260 ............................................................. [OK]
  o- loopback ................................................................. [Targets: 0]
/> 

```


sto2
```
[root@sto2 ~]# targetcli
targetcli shell version 2.1.57
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/> ls
o- / ................................................................................. [...]
  o- backstores ...................................................................... [...]
  | o- block .......................................................... [Storage Objects: 0]
  | o- fileio ......................................................... [Storage Objects: 3]
  | | o- data-chunk1 ........................... [/dev/md0 (1022.0MiB) write-back activated]
  | | | o- alua ........................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ............................... [ALUA state: Active/optimized]
  | | o- data-chunk2 ........................... [/dev/md1 (1022.0MiB) write-back activated]
  | | | o- alua ........................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ............................... [ALUA state: Active/optimized]
  | | o- data-chunk3 ........................... [/dev/md2 (1022.0MiB) write-back activated]
  | |   o- alua ........................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ............................... [ALUA state: Active/optimized]
  | o- pscsi .......................................................... [Storage Objects: 0]
  | o- ramdisk ........................................................ [Storage Objects: 0]
  o- iscsi .................................................................... [Targets: 3]
  | o- iqn.2024-12.tp2.b3:data-chunk1 ............................................ [TPGs: 1]
  | | o- tpg1 ....................................................... [no-gen-acls, no-auth]
  | |   o- acls .................................................................. [ACLs: 1]
  | |   | o- iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator .............. [Mapped LUNs: 1]
  | |   |   o- mapped_lun0 .................................. [lun0 fileio/data-chunk1 (rw)]
  | |   o- luns .................................................................. [LUNs: 1]
  | |   | o- lun0 ....................... [fileio/data-chunk1 (/dev/md0) (default_tg_pt_gp)]
  | |   o- portals ............................................................ [Portals: 1]
  | |     o- 0.0.0.0:3260 ............................................................. [OK]
  | o- iqn.2024-12.tp2.b3:data-chunk2 ............................................ [TPGs: 1]
  | | o- tpg1 ....................................................... [no-gen-acls, no-auth]
  | |   o- acls .................................................................. [ACLs: 1]
  | |   | o- iqn.2024-12.tp2.b3:data-chunk2:chunk2-initiator .............. [Mapped LUNs: 1]
  | |   |   o- mapped_lun0 .................................. [lun0 fileio/data-chunk2 (rw)]
  | |   o- luns .................................................................. [LUNs: 1]
  | |   | o- lun0 ....................... [fileio/data-chunk2 (/dev/md1) (default_tg_pt_gp)]
  | |   o- portals ............................................................ [Portals: 1]
  | |     o- 0.0.0.0:3260 ............................................................. [OK]
  | o- iqn.2024-12.tp2.b3:data-chunk3 ............................................ [TPGs: 1]
  |   o- tpg1 ....................................................... [no-gen-acls, no-auth]
  |     o- acls .................................................................. [ACLs: 1]
  |     | o- iqn.2024-12.tp2.b3:data-chunk3:chunk3-initiator .............. [Mapped LUNs: 1]
  |     |   o- mapped_lun0 .................................. [lun0 fileio/data-chunk3 (rw)]
  |     o- luns .................................................................. [LUNs: 1]
  |     | o- lun0 ....................... [fileio/data-chunk3 (/dev/md2) (default_tg_pt_gp)]
  |     o- portals ............................................................ [Portals: 1]
  |       o- 0.0.0.0:3260 ............................................................. [OK]
  o- loopback ................................................................. [Targets: 0]
/> 
```

## 2. Chunks machine


### A. Simple iSCSI

🌞 **Installer les tools iSCSI sur `chunk1.tp2.b3`**

```
[root@chunk1 ~]# dnf install -y iscsi-initiator-utils

```

🌞 **Configurer un iSCSI initiator**

```
[root@chunk1 ~]# cat /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator

```

```
[root@chunk1 ~]# iscsiadm -m discoverydb -t st -p 10.3.1.1:3260 --discover
10.3.1.1:3260,1 iqn.2024-12.tp2.b3:data-chunk1
10.3.1.1:3260,1 iqn.2024-12.tp2.b3:data-chunk2
10.3.1.1:3260,1 iqn.2024-12.tp2.b3:data-chunk3

```

```
[root@chunk1 ~]# iscsiadm -m node
10.3.1.1:3260,1 iqn.2024-12.tp2.b3:data-chunk1
10.3.1.2:3260,1 iqn.2024-12.tp2.b3:data-chunk1
10.3.2.1:3260,1 iqn.2024-12.tp2.b3:data-chunk1
10.3.2.2:3260,1 iqn.2024-12.tp2.b3:data-chunk1
10.3.1.1:3260,1 iqn.2024-12.tp2.b3:data-chunk2
10.3.1.2:3260,1 iqn.2024-12.tp2.b3:data-chunk2
10.3.2.1:3260,1 iqn.2024-12.tp2.b3:data-chunk2
10.3.2.2:3260,1 iqn.2024-12.tp2.b3:data-chunk2
10.3.1.1:3260,1 iqn.2024-12.tp2.b3:data-chunk3
10.3.1.2:3260,1 iqn.2024-12.tp2.b3:data-chunk3
10.3.2.1:3260,1 iqn.2024-12.tp2.b3:data-chunk3
10.3.2.2:3260,1 iqn.2024-12.tp2.b3:data-chunk3
```

```
[root@chunk1 ~]# cat /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
[root@chunk1 ~]# sudo systemctl restart iscsid
[root@chunk1 ~]# sudo systemctl restart iscsi
[root@chunk1 ~]# iscsiadm -m node -T iqn.2024-12.tp2.b3:data-chunk1 -p 10.3.1.1 --login
[root@chunk1 ~]# iscsiadm -m node -T iqn.2024-12.tp2.b3:data-chunk1 -p 10.3.1.2 --login
[root@chunk1 ~]# iscsiadm -m node -T iqn.2024-12.tp2.b3:data-chunk1 -p 10.3.2.1 --login
[root@chunk1 ~]# iscsiadm -m node -T iqn.2024-12.tp2.b3:data-chunk1 -p 10.3.2.2 --login
[root@chunk1 ~]# iscsiadm -m session
tcp: [18] 10.3.1.1:3260,1 iqn.2024-12.tp2.b3:data-chunk1 (non-flash)
tcp: [21] 10.3.2.1:3260,1 iqn.2024-12.tp2.b3:data-chunk1 (non-flash)
tcp: [22] 10.3.1.2:3260,1 iqn.2024-12.tp2.b3:data-chunk1 (non-flash)
tcp: [23] 10.3.2.2:3260,1 iqn.2024-12.tp2.b3:data-chunk1 (non-flash)

```
🌞 **Modifier la configuration du démon iSCSI**

```
[root@chunk1 ~]# cat /etc/iscsi/iscsid.conf | grep node.session.timeo.replacement_timeout
node.session.timeo.replacement_timeout = 0
[root@chunk1 ~]# systemctl restart iscsi
[root@chunk1 ~]# systemctl restart iscsid
```

🌞 **Prouvez que la configuration est prête**

```
[root@chunk1 ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   10G  0 disk 
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0    9G  0 part 
  ├─rl-root 253:0    0    8G  0 lvm  /
  └─rl-swap 253:1    0    1G  0 lvm  [SWAP]
sdb           8:16   0 1022M  0 disk 
sdc           8:32   0 1022M  0 disk 
sdd           8:48   0 1022M  0 disk 
sde           8:64   0 1022M  0 disk 
sr0          11:0    1 1024M  0 rom  

```

```
[root@chunk1 ~]# iscsiadm -m session -P 3
iSCSI Transport Class version 2.0-870
version 6.2.1.9
Target: iqn.2024-12.tp2.b3:data-chunk1 (non-flash)
        Current Portal: 10.3.1.1:3260,1
        Persistent Portal: 10.3.1.1:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
                Iface IPaddress: 10.3.1.101
                Iface HWaddress: default
                Iface Netdev: default
                SID: 18
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 120
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 3  State: running
                scsi3 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdb          State: running
        Current Portal: 10.3.2.1:3260,1
        Persistent Portal: 10.3.2.1:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
                Iface IPaddress: 10.3.2.101
                Iface HWaddress: default
                Iface Netdev: default
                SID: 21
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 120
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 6  State: running
                scsi6 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sde          State: running
        Current Portal: 10.3.1.2:3260,1
        Persistent Portal: 10.3.1.2:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
                Iface IPaddress: 10.3.1.101
                Iface HWaddress: default
                Iface Netdev: default
                SID: 22
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 120
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 7  State: running
                scsi7 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdd          State: running
        Current Portal: 10.3.2.2:3260,1
        Persistent Portal: 10.3.2.2:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
                Iface IPaddress: 10.3.2.101
                Iface HWaddress: default
                Iface Netdev: default
                SID: 23
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 120
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 8  State: running
                scsi8 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdc          State: running
```

### B. Multipathing

🌞 **Installer les outils multipath sur `chunk1.tp2.b3`**

```
[root@chunk1 ~]# dnf install -y device-mapper-multipath
```

🌞 **Configurer le fichier `/etc/multipath.conf`**


```
[root@chunk1 etc]# cat /etc/multipath.conf
# This is a basic configuration file with some examples, for device mapper
# multipath.
#
# For a complete list of the default configuration values, run either
# multipath -t
# or
# multipathd show config
#
# For a list of configuration options with descriptions, see the multipath.conf
# man page

## By default, devices with vendor = "IBM" and product = "S/390.*" are
## blacklisted. To enable mulitpathing on these devies, uncomment the
## following lines.
#blacklist_exceptions {
#       device {
#               vendor  "IBM"
#               product "S/390.*"
#       }
#}

## Use user friendly names, instead of using WWIDs as names.
defaults {
        user_friendly_names yes
        find_multipaths yes
}
##
## Here is an example of how to configure some standard options.
##
#
defaults {
  user_friendly_names yes
  find_multipaths yes
  path_grouping_policy failover
  features "1 queue_if_no_path"
  no_path_retry 100
}
##
## The wwid line in the following blacklist section is shown as an example
## of how to blacklist devices by wwid.  The 2 devnode lines are the
## compiled in default blacklist. If you want to blacklist entire types
## of devices, such as all scsi devices, you should use a devnode line.
## However, if you want to blacklist specific devices, you should use
## a wwid line.  Since there is no guarantee that a specific device will
## not change names on reboot (from /dev/sda to /dev/sdb for example)
## devnode lines are not recommended for blacklisting specific devices.
##
#blacklist {
#       wwid 26353900f02796769
#       devnode "^(ram|raw|loop|fd|md|dm-|sr|scd|st)[0-9]*"
#       devnode "^hd[a-z]"
#}
#multipaths {
#       multipath {
#               wwid                    3600508b4000156d700012000000b0000
#               alias                   yellow
#               path_grouping_policy    multibus
#               path_checker            readsector0
#               path_selector           "round-robin 0"
#               failback                manual
#               rr_weight               priorities
#               no_path_retry           5
#       }
#       multipath {
#               wwid                    1DEC_____321816758474
#               alias                   red
#       }
#}
#devices {
#       device {
#               vendor                  "COMPAQ  "
#               product                 "HSV110 (C)COMPAQ"
#               path_grouping_policy    multibus
#               path_checker            readsector0
#               path_selector           "round-robin 0"
#               hardware_handler        "0"
#               failback                15
#               rr_weight               priorities
#               no_path_retry           queue
#       }
#       device {
#               vendor                  "COMPAQ  "
#               product                 "MSA1000         "
#               path_grouping_policy    multibus
#       }
#}

```

🌞 **Démarrer le service `multipathd`**

```
[root@chunk1 etc]# systemctl restart multipathd
```

🌞 **Et euh c'est tout, il est smart enough**

```
[root@chunk1 ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0   10G  0 disk  
├─sda1        8:1    0    1G  0 part  /boot
└─sda2        8:2    0    9G  0 part  
  ├─rl-root 253:0    0    8G  0 lvm   /
  └─rl-swap 253:1    0    1G  0 lvm   [SWAP]
sdb           8:16   0 1022M  0 disk  
└─mpatha    253:2    0 1022M  0 mpath 
sdc           8:32   0 1022M  0 disk  
└─mpathb    253:3    0 1022M  0 mpath 
sdd           8:48   0 1022M  0 disk  
└─mpathb    253:3    0 1022M  0 mpath 
sde           8:64   0 1022M  0 disk  
└─mpatha    253:2    0 1022M  0 mpath 
sr0          11:0    1 1024M  0 rom  
```

```
[root@chunk1 ~]# multipath -ll
4735.224337 | /etc/multipath.conf line 31, duplicate keyword: defaults
mpatha (36001405a1a29add738a4bf5aaf01b3b2) dm-2 LIO-ORG,data-chunk1
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 3:0:0:0 sdb 8:16 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 6:0:0:0 sde 8:64 active ready running
mpathb (36001405a1cc77d56b1b43029b9df2a9c) dm-3 LIO-ORG,data-chunk1
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 7:0:0:0 sdd 8:48 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 8:0:0:0 sdc 8:32 active ready running

```

➜ **On aboutit sur un truc comme ça :**


## 3. Formatage et montage

🌞 **Créez une partition sur les devices `mpatha` et `mpathb`**


```
[root@chunk1 ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0   10G  0 disk  
├─sda1        8:1    0    1G  0 part  /boot
└─sda2        8:2    0    9G  0 part  
  ├─rl-root 253:0    0    8G  0 lvm   /
  └─rl-swap 253:1    0    1G  0 lvm   [SWAP]
sdb           8:16   0 1022M  0 disk  
└─mpatha    253:2    0 1022M  0 mpath 
  └─mpatha1 253:4    0 1014M  0 part  
sdc           8:32   0 1022M  0 disk  
└─mpathb    253:3    0 1022M  0 mpath 
  └─mpathb1 253:5    0 1014M  0 part  
sdd           8:48   0 1022M  0 disk  
└─mpathb    253:3    0 1022M  0 mpath 
  └─mpathb1 253:5    0 1014M  0 part  
sde           8:64   0 1022M  0 disk  
└─mpatha    253:2    0 1022M  0 mpath 
  └─mpatha1 253:4    0 1014M  0 part  
sr0          11:0    1 1024M  0 rom  
```

🌞 **Formatez en `xfs` les partitions**

```
[root@chunk1 ~]# mkfs.xfs /dev/mapper/mpatha1
meta-data=/dev/mapper/mpatha1    isize=512    agcount=4, agsize=64896 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=259584, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@chunk1 ~]# mkfs.xfs /dev/mapper/mpathb1
meta-data=/dev/mapper/mpathb1    isize=512    agcount=4, agsize=64896 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=259584, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

[root@chunk1 ~]# lsblk -f
NAME        FSTYPE       FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                                           
├─sda1      xfs                         c93b1aaa-dc60-4efe-a460-91e7c7b714c9    646.4M    33% /boot
└─sda2      LVM2_member  LVM2 001       GJ1Y2g-dZIl-Mf0j-22bc-xNmq-IJZ9-L2lXFx                
  ├─rl-root xfs                         507b564a-9344-4014-95e3-cdb69709aee3      6.6G    17% /
  └─rl-swap swap         1              835bf84c-2643-4e3f-b270-d93194c68179                  [SWAP]
sdb         mpath_member                                                                      
└─mpatha                                                                                      
  └─mpatha1 xfs                         5e9b537f-ef74-4890-8e13-35e5a6ae4a05                  
sdc         mpath_member                                                                      
└─mpathb                                                                                      
  └─mpathb1                                                                                   
sdd         mpath_member                                                                      
└─mpathb                                                                                      
  └─mpathb1                                                                                   
sde         mpath_member                                                                      
└─mpatha                                                                                      
  └─mpatha1 xfs                         5e9b537f-ef74-4890-8e13-35e5a6ae4a05                  
sr0 
```

🌞 **Point de montage `/mnt/data_chunk1`**


```
[root@chunk1 ~]# mkdir /mnt/data_chunk1
[root@chunk1 ~]# mount /dev/mapper/mpatha1 /mnt/data_chunk1
[root@chunk1 ~]# cat /etc/systemd/system/mnt-data_chunk1.mount
[Unit]
Description=Mount mpatha1 on /mnt/data_chunk1
After=network-online.target
Wants=network-online.target

[Mount]
What=/dev/mapper/mpatha1
Where=/mnt/data_chunk1
Type=xfs
Options=_netdev

[Install]
WantedBy=multi-user.target

[root@chunk1 ~]# systemctl daemon-reload
[root@chunk1 ~]# systemctl enable --now  mnt-data_chunk1.mount
Created symlink /etc/systemd/system/multi-user.target.wants/mnt-data_chunk1.mount → /etc/systemd/system/mnt-data_chunk1.mount.

```

🌞 **Point de montage `/mnt/data_chunk2`**

```

[root@chunk2 ~]# cat /etc/systemd/system/mnt-data_chunk2.mount 
[Unit]
Description=Mount for /mnt/data_chunk2
After=network-online.target
Wants=network-online.target

[Mount]
What=/dev/mapper/mpathb
Where=/mnt/data_chunk2
Type=xfs
Options=defaults

[Install]
WantedBy=multi-user.target
[root@chunk2 ~]# lsblk -f
NAME             FSTYPE       FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
[...]
sdb              mpath_member                                                                      
└─mpathb                                                                                           
  └─mpathb1      xfs                         f6674e0b-9fd3-4c34-a751-91ee610d4152    911.1M     4% /mnt/data_chunk2
sdc              mpath_member                                                                      
└─mpathb                                                                                           
  └─mpathb1      xfs                         f6674e0b-9fd3-4c34-a751-91ee610d4152    911.1M     4% /mnt/data_chunk2
sr0
[root@chunk2 ~]# df -h
Filesystem                Size  Used Avail Use% Mounted on
/dev/mapper/mpathb1       950M   39M  912M   5% /mnt/data_chunk2
```

## 4. Tests

### A. Simulation de panne

🌞 **Simuler une coupure réseau**

```
[root@chunk2 ~]# date
Fri Dec  6 10:23:25 CET 2024

mpatha (3600140557c66dd047214999a651eb037) dm-2 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=0 status=active
| `- 5:0:0:0 sdd 8:48 failed faulty running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 6:0:0:0 sde 8:64 active ready running
mpathb (3600140532c8eec0828442df8df844bb1) dm-3 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=0 status=enabled
| `- 7:0:0:0 sdb 8:16 failed faulty running
`-+- policy='service-time 0' prio=50 status=active
  `- 8:0:0:0 sdc 8:32 active ready running

[root@chunk2 ~]# date
Fri Dec  6 10:24:43 CET 2024

mpatha (3600140557c66dd047214999a651eb037) dm-2 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 5:0:0:0 sdd 8:48 failed ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 6:0:0:0 sde 8:64 active ready running
mpathb (3600140532c8eec0828442df8df844bb1) dm-3 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=enabled
| `- 7:0:0:0 sdb 8:16 failed ready running
`-+- policy='service-time 0' prio=50 status=active
  `- 8:0:0:0 sdc 8:32 active ready running

```

### B. Jouer avec les paramètres



🌞 **Resimuler une panne**
 
```
[root@chunk2 ~]# date
Fri Dec  6 10:50:3 CET 2024

mpatha (3600140557c66dd047214999a651eb037) dm-2 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=0 status=active
| `- 5:0:0:0 sdd 8:48 failed faulty running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 6:0:0:0 sde 8:64 active ready running
mpathb (3600140532c8eec0828442df8df844bb1) dm-3 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=0 status=enabled
| `- 7:0:0:0 sdb 8:16 failed faulty running
`-+- policy='service-time 0' prio=50 status=active
  `- 8:0:0:0 sdc 8:32 active ready running

[root@chunk2 ~]# date
Fri Dec  6 10:50:27 CET 2024

mpatha (3600140557c66dd047214999a651eb037) dm-2 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 5:0:0:0 sdd 8:48 failed ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 6:0:0:0 sde 8:64 active ready running
mpathb (3600140532c8eec0828442df8df844bb1) dm-3 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=enabled
| `- 7:0:0:0 sdb 8:16 failed ready running
`-+- policy='service-time 0' prio=50 status=active
  `- 8:0:0:0 sdc 8:32 active ready running
```

## 5. Replicate

🌞 **Preuve du setup**

```
[root@chunk2 ~]# lsblk
NAME             MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda                8:0    0   20G  0 disk  
├─sda1             8:1    0    1G  0 part  /boot
└─sda2             8:2    0   19G  0 part  
  ├─rl_vbox-root 253:0    0   17G  0 lvm   /
  └─rl_vbox-swap 253:1    0    2G  0 lvm   [SWAP]
sdb                8:16   0 1022M  0 disk  
└─mpathb         253:3    0 1022M  0 mpath 
  └─mpathb1      253:5    0 1014M  0 part  /mnt/data_chunk2
sdc                8:32   0 1022M  0 disk  
└─mpathb         253:3    0 1022M  0 mpath 
  └─mpathb1      253:5    0 1014M  0 part  /mnt/data_chunk2
sdd                8:48   0 1022M  0 disk  
└─mpatha         253:2    0 1022M  0 mpath 
  └─mpatha1      253:4    0 1014M  0 part  /mnt/data_chunk1
sde                8:64   0 1022M  0 disk  
└─mpatha         253:2    0 1022M  0 mpath 
  └─mpatha1      253:4    0 1014M  0 part  /mnt/data_chunk1
sr0               11:0    1 1024M  0 rom   
[root@chunk2 ~]# df -h
Filesystem                Size  Used Avail Use% Mounted on
devtmpfs                  4.0M     0  4.0M   0% /dev
tmpfs                     882M     0  882M   0% /dev/shm
tmpfs                     353M  5.0M  348M   2% /run
/dev/mapper/rl_vbox-root   17G  1.2G   16G   8% /
/dev/sda1                 960M  223M  738M  24% /boot
tmpfs                     177M     0  177M   0% /run/user/0
/dev/mapper/mpatha1       950M   39M  912M   5% /mnt/data_chunk1
/dev/mapper/mpathb1       950M   42M  909M   5% /mnt/data_chunk2
[root@chunk2 ~]# iscsiadm -m session -P 3
iSCSI Transport Class version 2.0-870
version 6.2.1.9
Target: iqn.2024-12.tp2.b3:data-chunk2 (non-flash)
	Current Portal: 10.3.2.2:3260,1
	Persistent Portal: 10.3.2.2:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk2:chunk1-initiator
		Iface IPaddress: 10.3.2.102
		Iface HWaddress: default
		Iface Netdev: default
		SID: 10
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 5
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: <empty>
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 262144
		FirstBurstLength: 65536
		MaxBurstLength: 262144
		ImmediateData: Yes
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 6	State: running
		scsi6 Channel 00 Id 0 Lun: 0
			Attached scsi disk sde		State: running
	Current Portal: 10.3.1.1:3260,1
	Persistent Portal: 10.3.1.1:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk2:chunk1-initiator
		Iface IPaddress: 10.3.1.102
		Iface HWaddress: default
		Iface Netdev: default
		SID: 5
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 5
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: <empty>
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 262144
		FirstBurstLength: 65536
		MaxBurstLength: 262144
		ImmediateData: Yes
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 7	State: running
		scsi7 Channel 00 Id 0 Lun: 0
			Attached scsi disk sdb		State: running
	Current Portal: 10.3.2.1:3260,1
	Persistent Portal: 10.3.2.1:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk2:chunk1-initiator
		Iface IPaddress: 10.3.2.102
		Iface HWaddress: default
		Iface Netdev: default
		SID: 6
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 5
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: <empty>
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 262144
		FirstBurstLength: 65536
		MaxBurstLength: 262144
		ImmediateData: Yes
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 8	State: running
		scsi8 Channel 00 Id 0 Lun: 0
			Attached scsi disk sdc		State: running
	Current Portal: 10.3.1.2:3260,1
	Persistent Portal: 10.3.1.2:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk2:chunk1-initiator
		Iface IPaddress: 10.3.1.102
		Iface HWaddress: default
		Iface Netdev: default
		SID: 9
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 5
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: <empty>
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 262144
		FirstBurstLength: 65536
		MaxBurstLength: 262144
		ImmediateData: Yes
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 5	State: running
		scsi5 Channel 00 Id 0 Lun: 0
			Attached scsi disk sdd		State: running
[root@chunk2 ~]# multipath -ll
6996.373607 | /etc/multipath.conf line 26, invalid keyword in the multipath section: path_checker
mpatha (3600140557c66dd047214999a651eb037) dm-2 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 5:0:0:0 sdd 8:48 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 6:0:0:0 sde 8:64 active ready running
mpathb (3600140532c8eec0828442df8df844bb1) dm-3 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=enabled
| `- 7:0:0:0 sdb 8:16 active ready running
`-+- policy='service-time 0' prio=50 status=active
  `- 8:0:0:0 sdc 8:32 active ready running

[root@chunk3 ~]# lsblk
NAME             MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda                8:0    0   20G  0 disk  
├─sda1             8:1    0    1G  0 part  /boot
└─sda2             8:2    0   19G  0 part  
  ├─rl_vbox-root 253:0    0   17G  0 lvm   /
  └─rl_vbox-swap 253:1    0    2G  0 lvm   [SWAP]
sdb                8:16   0 1022M  0 disk  
└─mpathb         253:3    0 1022M  0 mpath 
  └─mpathb1      253:5    0 1014M  0 part  
sdc                8:32   0 1022M  0 disk  
└─mpathb         253:3    0 1022M  0 mpath 
  └─mpathb1      253:5    0 1014M  0 part  
sdd                8:48   0 1022M  0 disk  
└─mpatha         253:2    0 1022M  0 mpath 
  └─mpatha1      253:4    0 1014M  0 part  
sde                8:64   0 1022M  0 disk  
└─mpatha         253:2    0 1022M  0 mpath 
  └─mpatha1      253:4    0 1014M  0 part  
sr0               11:0    1 1024M  0 rom   
[root@chunk3 ~]# df -h
Filesystem                Size  Used Avail Use% Mounted on
devtmpfs                  4.0M     0  4.0M   0% /dev
tmpfs                     882M     0  882M   0% /dev/shm
tmpfs                     353M  5.0M  348M   2% /run
/dev/mapper/rl_vbox-root   17G  1.2G   16G   8% /
/dev/sda1                 960M  223M  738M  24% /boot
tmpfs                     177M     0  177M   0% /run/user/0
[root@chunk3 ~]# multipath -ll
7073.694832 | /etc/multipath.conf line 26, invalid keyword in the multipath section: path_checker
mpatha (360014054dab0c38acbe481b985650efa) dm-2 LIO-ORG,data-chunk3
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 13:0:0:0 sdd 8:48 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 14:0:0:0 sde 8:64 active ready running
mpathb (3600140568639b32dcf64b85992c43877) dm-3 LIO-ORG,data-chunk3
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 5:0:0:0  sdc 8:32 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 6:0:0:0  sdb 8:16 active ready running
[root@chunk3 ~]# iscsiadm -m session -P 3
iSCSI Transport Class version 2.0-870
version 6.2.1.9
Target: iqn.2024-12.tp2.b3:data-chunk3 (non-flash)
	Current Portal: 10.3.1.2:3260,1
	Persistent Portal: 10.3.1.2:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk3:chunk1-initiator
		Iface IPaddress: 10.3.1.103
		Iface HWaddress: default
		Iface Netdev: default
		SID: 11
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 5
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: <empty>
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 262144
		FirstBurstLength: 65536
		MaxBurstLength: 262144
		ImmediateData: Yes
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 13	State: running
		scsi13 Channel 00 Id 0 Lun: 0
			Attached scsi disk sdd		State: running
	Current Portal: 10.3.2.2:3260,1
	Persistent Portal: 10.3.2.2:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk3:chunk1-initiator
		Iface IPaddress: 10.3.2.103
		Iface HWaddress: default
		Iface Netdev: default
		SID: 12
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 5
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: <empty>
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 262144
		FirstBurstLength: 65536
		MaxBurstLength: 262144
		ImmediateData: Yes
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 14	State: running
		scsi14 Channel 00 Id 0 Lun: 0
			Attached scsi disk sde		State: running
	Current Portal: 10.3.1.1:3260,1
	Persistent Portal: 10.3.1.1:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk3:chunk1-initiator
		Iface IPaddress: 10.3.1.103
		Iface HWaddress: default
		Iface Netdev: default
		SID: 3
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 5
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: <empty>
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 262144
		FirstBurstLength: 65536
		MaxBurstLength: 262144
		ImmediateData: Yes
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 5	State: running
		scsi5 Channel 00 Id 0 Lun: 0
			Attached scsi disk sdc		State: running
	Current Portal: 10.3.2.1:3260,1
	Persistent Portal: 10.3.2.1:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk3:chunk1-initiator
		Iface IPaddress: 10.3.2.103
		Iface HWaddress: default
		Iface Netdev: default
		SID: 4
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 5
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: <empty>
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 262144
		FirstBurstLength: 65536
		MaxBurstLength: 262144
		ImmediateData: Yes
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 6	State: running
		scsi6 Channel 00 Id 0 Lun: 0
			Attached scsi disk sdb		State: running

```