# LVM

![LVM](http://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/Lvm.svg/500px-Lvm.svg.png)

## Physical Volumes (PVs)

When Using LVM, its best practice to have a single PV per physical device. LVM can work directly with device files, but its best practice to have it work with partitions:

```
$ ls -l /dev | grep sd
brw-rw----. 1 root    disk      8,   0 May  1 20:39 sda
brw-rw----. 1 root    disk      8,   1 May  1 20:39 sda1
brw-rw----. 1 root    disk      8,   2 May  1 20:39 sda2
brw-rw----. 1 root    disk      8,  16 May  1 20:44 sdb
brw-rw----. 1 root    disk      8,  17 May  1 20:44 sdb1
brw-rw----. 1 root    disk      8,  32 May  1 20:44 sdc
brw-rw----. 1 root    disk      8,  33 May  1 20:44 sdc1
```

Before LVM can use partitions, they need to be initialized so that LVM recognizes them as PVs:

```
$ sudo pvcreate /dev/sdb1 /dev/sdc1
  Physical volume "/dev/sdb1" successfully created
  Physical volume "/dev/sdc1" successfully created
$ sudo pvs
  PV         VG       Fmt  Attr PSize    PFree
  /dev/sda2  VolGroup lvm2 a--    39.51g       0
  /dev/sdb1           lvm2 a--  1019.72m 1019.72m
  /dev/sdc1           lvm2 a--     2.00g    2.00g
```

By default `pvcreate` writes 1 metadata copy to the PV. VGs require at least 1 metadata copy within all of their PVs.

### Query LVM metadata on a PV

```
$ sudo pvdisplay /dev/sdb1
  "/dev/sdb1" is a new physical volume of "1019.72 MiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name
  PV Size               1019.72 MiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               9VpTjw-6hWi-J152-HbGY-x9xf-Wm7Q-lR8Raj
```

### Deleting a PV

```
$ sudo pvremove /dev/sdb1 /dev/sdc1
  Labels on physical volume "/dev/sdb1" successfully wiped
  Labels on physical volume "/dev/sdc1" successfully wiped
$ sudo pvs
  PV         VG       Fmt  Attr PSize  PFree
  /dev/sda2  VolGroup lvm2 a--  39.51g    0
```

## Volume Groups (VGs)

### Creating a VG

```
$ sudo vgs
  VG       #PV #LV #SN Attr   VSize  VFree
  VolGroup   1   2   0 wz--n- 39.51g    0
$ sudo vgcreate vg_test /dev/sdb1
  No physical volume label read from /dev/sdb1
  Physical volume /dev/sdb1 not found
  Physical volume "/dev/sdb1" successfully created
  Volume group "vg_test" successfully created
$ sudo vgs
  VG       #PV #LV #SN Attr   VSize    VFree
  VolGroup   1   2   0 wz--n-   39.51g       0
  vg_test    1   0   0 wz--n- 1016.00m 1016.00m
```

### Query a VG

```
$ sudo vgdisplay vg_test
  --- Volume group ---
  VG Name               vg_test
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
  VG Size               1016.00 MiB
  PE Size               4.00 MiB
  Total PE              254
  Alloc PE / Size       0 / 0
  Free  PE / Size       254 / 1016.00 MiB
  VG UUID               uIymMB-lJ1i-T0Z2-c7Eu-QBuJ-Ee1g-vfYKfp
```

### Add a PV to an existing VG

```
$ sudo vgextend vg_test /dev/sdc1
  Volume group "vg_test" successfully extended
$ sudo vgdisplay vg_test -v
    Using volume group(s) on command line
    Finding volume group "vg_test"
  --- Volume group ---
  VG Name               vg_test
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               2.99 GiB
  PE Size               4.00 MiB
  Total PE              765
  Alloc PE / Size       0 / 0
  Free  PE / Size       765 / 2.99 GiB
  VG UUID               uIymMB-lJ1i-T0Z2-c7Eu-QBuJ-Ee1g-vfYKfp

  --- Physical volumes ---
  PV Name               /dev/sdb1
  PV UUID               1q8vUv-RO2n-Yl7I-prOI-H3HS-9DU7-E6BYod
  PV Status             allocatable
  Total PE / Free PE    254 / 254

  PV Name               /dev/sdc1
  PV UUID               G4iqjV-dJc1-gJri-FaWd-fAct-7NvH-50QZmp
  PV Status             allocatable
  Total PE / Free PE    511 / 511

```

## Logical Volumes (LVs)

### Create a 2 GB LV and a 512 MB LV

```
$ sudo lvcreate -L 2G -n lv_test vg_test
  Logical volume "lv_test" created
$ sudo lvcreate -L 512M -n lv_test2 vg_test
  Logical volume "lv_test2" created
$ sudo lvs
  LV       VG       Attr       LSize   Pool Origin Data%  Move Log Cpy%Sync Convert
  lv_root  VolGroup -wi-ao----  38.60g
  lv_swap  VolGroup -wi-ao---- 928.00m
  lv_test  vg_test  -wi-a-----   2.00g
  lv_test2 vg_test  -wi-a----- 512.00m
$ ls -l /dev/vg_test
total 0
lrwxrwxrwx. 1 root root 7 May  1 22:15 lv_test -> ../dm-2
lrwxrwxrwx. 1 root root 7 May  1 22:16 lv_test2 -> ../dm-3
```

### Add a filesystem to an LV and mount it

```
$ sudo mke2fs -t ext4 /dev/vg_test/lv_test
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
131072 inodes, 524288 blocks
26214 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=536870912
16 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 36 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
$ sudo mkdir /lvm_testmount
$ sudo mount -t ext4 /dev/vg_test/lv_test /lvm_testmount
```

## Growing and Shrinking LVs

### Growing an LV

In this example we have an LV `jeffs_lv` in VG `vg_test` with 1.52 GB free:

```
$ sudo lvs
  LV       VG       Attr       LSize   Pool Origin Data%  Move Log Cpy%Sync Convert
  lv_root  VolGroup -wi-ao----  38.60g
  lv_swap  VolGroup -wi-ao---- 928.00m
  jeffs_lv vg_test  -wi-ao----   1.46g
$ sudo vgs
  VG       #PV #LV #SN Attr   VSize  VFree
  VolGroup   1   2   0 wz--n- 39.51g    0
  vg_test    2   1   0 wz--n-  2.99g 1.52g
$ sudo lvextend -L 2G /dev/vg_test/jeffs_lv
  Extending logical volume jeffs_lv to 2.00 GiB
  Logical volume jeffs_lv successfully resized
$ sudo lvs
  LV       VG       Attr       LSize   Pool Origin Data%  Move Log Cpy%Sync Convert
  lv_root  VolGroup -wi-ao----  38.60g
  lv_swap  VolGroup -wi-ao---- 928.00m
  jeffs_lv vg_test  -wi-ao----   2.00g
$ sudo vgs
  VG       #PV #LV #SN Attr   VSize  VFree
  VolGroup   1   2   0 wz--n- 39.51g       0
  vg_test    2   1   0 wz--n-  2.99g 1012.00m
```

Then the filesystem on that LV needs to be resized:

```
$ sudo df -h /lvm_testmount
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/vg_test-jeffs_lv  1.5G   36M  1.4G   3% /lvm_testmount
$ sudo resize2fs /dev/vg_test/jeffs_lv
resize2fs 1.41.12 (17-May-2010)
Filesystem at /dev/vg_test/jeffs_lv is mounted on /lvm_testmount; on-line resizing required
old desc_blocks = 1, new_desc_blocks = 1
Performing an on-line resize of /dev/vg_test/jeffs_lv to 524288 (4k) blocks.
The filesystem on /dev/vg_test/jeffs_lv is now 524288 blocks long.
$ sudo df -h /lvm_testmount
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/vg_test-jeffs_lv  2.0G   36M  1.9G   2% /lvm_testmount
```

### Shrinking an LV

1. Backup data
2. Unmount the filesystem
3. Check the filesystem
4. Reduce the filesystem
5. Reduce the LV
6. Re-check the filesystem
7. Re-mount the filesystem

TODO
