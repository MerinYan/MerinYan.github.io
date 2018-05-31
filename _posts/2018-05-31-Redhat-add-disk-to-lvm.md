---
layout: default
title:  "Redhat 添加新硬盘到LVM"
date:   2018-05-25 22:06:35 +0800
---

Redhat添加新的硬盘到卷组中

1. 创建物理卷
```
[root@master ~]# pvcreate /dev/sda  #将整个磁盘创建为物理卷的命令
Physical volume "/dev/sda" successfully created.
[root@master ~]# pvcreate /dev/sdc1  #将单个分区创建为物理卷的命令为：
Physical volume "/dev/sdc1" successfully created
```
2. 添加新的物理卷到卷组中
```
[root@master ~]# vgdisplay  #查看现有的卷组
  --- Volume group ---
  VG Name               rhel
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
  VG Size               <29.00 GiB
  PE Size               4.00 MiB
  Total PE              7423
  Alloc PE / Size       7423 / <29.00 GiB
  Free  PE / Size       0 / 0   
  VG UUID               zp9oyN-VFhi-Elkj-e7Q5-P6Qd-Udc6-zRO4YA
[root@master ~]# vgextend rhel /dev/sda  #将物理卷添加到卷组中
  Volume group "rhel" successfully extended
[root@master ~]# vgs   #显示卷组空间
  VG   #PV #LV #SN Attr   VSize  VFree  
  rhel   2   2   0 wz--n- 58.99g <30.00g
```
3. 扩展现有的逻辑卷
```
[root@master ~]# lvextend /dev/rhel/root -L +26G    #空间增加26GB
  Rounding size to boundary between physical extents: 26.00 GiB.
  Size of logical volume rhel/root changed from <52.00 GiB (13311 extents) to <53.00 GiB (13567 extents).
  Logical volume rhel/root successfully resized.
[root@master ~]# xfs_growfs /dev/rhel/root    #使得空间生效
meta-data=/dev/mapper/rhel-root  isize=512    agcount=4, agsize=1703680 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=6814720, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=3327, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 6814720 to 13630464
```