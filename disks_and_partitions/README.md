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

### Show partition setup

```
$ sudo yum install parted -y
$ sudo parted
GNU Parted 2.1
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) p
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sda: 42.9GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  525MB   524MB   primary  ext4         boot
 2      525MB   42.9GB  42.4GB  primary               lvm

(parted) q
```
