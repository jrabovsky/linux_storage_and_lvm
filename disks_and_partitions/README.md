# Disks and Partitions

## Everything in Linux is a file, even your terminal

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

In this case, SCSI drive "a" has partitions "1" and "2":

```
$ ls -l /dev | grep sd
brw-rw----. 1 root    disk      8,   0 Apr 30 18:07 sda
brw-rw----. 1 root    disk      8,   1 Apr 30 18:07 sda1
brw-rw----. 1 root    disk      8,   2 Apr 30 18:07 sda2
```

### File types
Indicated by the first letter of each row in the output of `ls -l`:
  - Regular file `-`
  - Directory files `d`
  - Block file `b`
  - Character device file `c`
  - Named pipe file or just a pipe file `p`
  - Symbolic link file `l`
  - Socket file `s`

### Finding disk free space and disk usage

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

## Partitions

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
