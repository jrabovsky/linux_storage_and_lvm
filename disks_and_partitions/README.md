### Finding disk free space / disk usage

```
$ du -h ~
8.0K    /home/vagrant/.ssh
28K     /home/vagrant
28K     total
$ df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root   38G  765M   36G   3% /
tmpfs                         230M     0  230M   0% /dev/shm
/dev/sda1                     485M   32M  429M   7% /boot
vagrant                       105G   86G   19G  83% /vagrant
/dev/sdb1                    1004M   18M  936M   2% /testmount
```

### Everything in Linux is a file, even your terminal

```
$ tty
/dev/pts/0
$ ls -l /dev/pts
total 0
crw--w----. 1 vagrant tty  136, 0 Apr 30 18:44 0
c---------. 1 root    root   5, 2 Apr 30 18:06 ptmx
$ echo "this goes to your terminal" > `tty`
this goes to your terminal
```

### Get disk device files

```
$ ls -l /dev | grep sd
brw-rw----. 1 root    disk      8,   0 Apr 30 18:07 sda
brw-rw----. 1 root    disk      8,   1 Apr 30 18:07 sda1
brw-rw----. 1 root    disk      8,   2 Apr 30 18:07 sda2
```

* In this case, SCSI drive 'a' has partitions 1 and 2
* File types (indicated by the first letter of each row in the output of `ls -l`)
  - Regular file(-)
  - Directory files(d)
  - Block file(b)
  - Character device file(c)
  - Named pipe file or just a pipe file(p)
  - Symbolic link file(l)
  - Socket file(s)

### Show partitions on a device

```
$ sudo fdisk -l /dev/sda

Disk /dev/sda: 42.9 GB, 42949672960 bytes
255 heads, 63 sectors/track, 5221 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0004f229

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *           1          64      512000   83  Linux
Partition 1 does not end on cylinder boundary.
/dev/sda2              64        5222    41430016   8e  Linux LVM
```

### Create partitions on a device

```
$ sudo fdisk /dev/sdb
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0x1c0d7ee6.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.

Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').

Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
Partition number (1-4): 1
First cylinder (1-130, default 1):
Using default value 1
Last cylinder, +cylinders or +size{K,M,G} (1-130, default 130):
Using default value 130

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

### List supported file systems

Right columns are supported, those with 'nodev' on the left are not currently being used

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

### Create a file system (use `-n` to do a dry run)

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

### Manually mounting a file system

```
$ sudo mkdir /testmount
$ sudo mount -t ext4 /dev/sdb1 /testmount
```

### List mounted file systems

```
$ mount -l -t ext4
/dev/mapper/VolGroup-lv_root on / type ext4 (rw)
/dev/sda1 on /boot type ext4 (rw)
/dev/sdb1 on /testmount type ext4 (rw)
```

### Automatically mount file systems on boot

Using the device file

```
$ sudo echo '/dev/sdb1 /testmount ext4 defaults 0 0' >> /etc/fstab
```

Using the UUID (this is preferable since device file names can change)

```
$ sudo echo 'UUID=b72eb8b9-ce69-4e71-8a53-8ac81ccc7516 /testmount ext4 defaults 0 0' >> /etc/fstab
```

Get UUIDs of devices

```
$ sudo blkid
/dev/sda1: UUID="6eb6217d-f881-47c9-89b7-6eb9f7507295" TYPE="ext4"
/dev/sda2: UUID="TM0qy8-z9HK-c7B5-xGSk-Q11N-oF7S-ysS2MT" TYPE="LVM2_member"
/dev/sdb1: UUID="b72eb8b9-ce69-4e71-8a53-8ac81ccc7516" TYPE="ext4"
/dev/mapper/VolGroup-lv_root: UUID="097a9d08-afad-401d-af65-92a4c587a2e5" TYPE="ext4"
/dev/mapper/VolGroup-lv_swap: UUID="b7e866d5-831b-4ee8-b65d-be2589214a35" TYPE="swap"
```

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

```
$ sudo lsblk
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

Mount all file systems defined in `/etc/fstab` (happens on boot)

```
$ sudo mount -a
```

### Unmount file systems

Specify device file

```
$ sudo umount /dev/sdb1
```

Or specify the mountpoint

```
$ sudo umount /testmount
```
