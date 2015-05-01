# Filesystems

### List supported filesystems

Right columns are supported, those with "nodev" on the left are not currently being used:

```
$ cat /proc/filesystems
nodev   sysfs
nodev   rootfs
nodev   bdev
nodev   proc
nodev   cgroup
nodev   cpuset
nodev   tmpfs
nodev   devtmpfs
nodev   binfmt_misc
nodev   debugfs
nodev   securityfs
nodev   sockfs
nodev   usbfs
nodev   pipefs
nodev   anon_inodefs
nodev   inotifyfs
nodev   devpts
nodev   ramfs
nodev   hugetlbfs
        iso9660
nodev   pstore
nodev   mqueue
nodev   selinuxfs
        ext4
nodev   vboxsf
```

### Create a filesystem (use `-n` to do a dry run)

```
$ sudo mke2fs -t ext4 /dev/sdb1
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
65280 inodes, 261048 blocks
13052 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=268435456
8 block groups
32768 blocks per group, 32768 fragments per group
8160 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 23 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
```

### Manually mounting a filesystem

```
$ sudo mkdir /testmount
$ sudo mount -t ext4 /dev/sdb1 /testmount
```

### List mounted filesystems

Using `mount`:

```
$ mount -l -t ext4
/dev/mapper/VolGroup-lv_root on / type ext4 (rw)
/dev/sda1 on /boot type ext4 (rw)
/dev/sdb1 on /testmount type ext4 (rw)
```

Using `lsblk`:

```
$ lsblk
NAME                        MAJ:MIN RM    SIZE RO TYPE MOUNTPOINT
sda                           8:0    0     40G  0 disk
├─sda1                        8:1    0    500M  0 part /boot
└─sda2                        8:2    0   39.5G  0 part
  ├─VolGroup-lv_root (dm-0) 253:0    0   38.6G  0 lvm  /
  └─VolGroup-lv_swap (dm-1) 253:1    0    928M  0 lvm  [SWAP]
sdb                           8:16   0      1G  0 disk
└─sdb1                        8:17   0 1019.7M  0 part /testmount
sdc                           8:32   0      2G  0 disk
```

### Automatically mount filesystems on boot

Using the device file:

```
$ sudo echo '/dev/sdb1 /testmount ext4 defaults 0 0' >> /etc/fstab
```

Using the UUID (this is preferable since device file names can change):

```
$ sudo echo 'UUID=b72eb8b9-ce69-4e71-8a53-8ac81ccc7516 /testmount ext4 defaults 0 0' >> /etc/fstab
```

Mount all filesystems defined in `/etc/fstab` (happens on boot):

```
$ sudo mount -a
```

### Get UUIDs of devices:

Using `blkid`:

```
$ sudo blkid
/dev/sda1: UUID="6eb6217d-f881-47c9-89b7-6eb9f7507295" TYPE="ext4"
/dev/sda2: UUID="TM0qy8-z9HK-c7B5-xGSk-Q11N-oF7S-ysS2MT" TYPE="LVM2_member"
/dev/sdb1: UUID="b72eb8b9-ce69-4e71-8a53-8ac81ccc7516" TYPE="ext4"
/dev/mapper/VolGroup-lv_root: UUID="097a9d08-afad-401d-af65-92a4c587a2e5" TYPE="ext4"
/dev/mapper/VolGroup-lv_swap: UUID="b7e866d5-831b-4ee8-b65d-be2589214a35" TYPE="swap"
```

Using `lsblk`:

```
$ sudo lsblk -f
NAME                        FSTYPE      LABEL UUID                                   MOUNTPOINT
sda
├─sda1                      ext4              6eb6217d-f881-47c9-89b7-6eb9f7507295   /boot
└─sda2                      LVM2_member       TM0qy8-z9HK-c7B5-xGSk-Q11N-oF7S-ysS2MT
  ├─VolGroup-lv_root (dm-0) ext4              097a9d08-afad-401d-af65-92a4c587a2e5   /
  └─VolGroup-lv_swap (dm-1) swap              b7e866d5-831b-4ee8-b65d-be2589214a35   [SWAP]
sdb
└─sdb1                      ext4              b72eb8b9-ce69-4e71-8a53-8ac81ccc7516   /testmount
```

### Unmount filesystems

Specify device file:

```
$ sudo umount /dev/sdb1
```

Or specify the mountpoint:

```
$ sudo umount /testmount
```
