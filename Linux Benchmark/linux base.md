### runlevel
> runlevel - Print previous and current SysV runlevel
```
$ runlevel
3 5
$ man telinit
$ man init
$ init 3  //切换到多用户模式
$ init 5  //切换到X11模式，图形化界面
$ cat /etc/inittab
```
telinit是init的一个软链接。

### losetup
> losetup - set up and control loop devices

Print name of first unused loop device:
- losetup -f

Delete loop:
- losetup -d loopdev...

Delete all used loop devices:
- losetup -D

```
# losetup -f
/dev/loop0
# losetup /dev/loop0 rhel7.2.img
```

### mke2fs
> mke2fs - create an ext2/ext3/ext4 filesystem

格式化磁盘

### kpartx
> kpartx - Create device maps from partition tables

```
# kpartx -a /dev/loop0
# mke2fs -j /dev/mapper/loop0p1
```

### dump实例
```
# losetup -f
/dev/loop0
# losetup /dev/loop0 rhel7.2.img
#
# kpartx -a /dev/loop0
#
# kpartx -l /dev/loop0
loop0p1 : 0 41940992 /dev/loop0 2048
# dump -0uf rhel7.2.img.dump /dev/mapper/loop0p1
  DUMP: Date of this level 0 dump: Wed Apr 26 14:29:12 2017
  DUMP: Dumping /dev/mapper/loop0p1 (an unlisted file system) to rhel7.2.img.dump
  DUMP: Label: none
  DUMP: Writing 10 Kilobyte records
  DUMP: mapping (Pass I) [regular files]
  DUMP: mapping (Pass II) [directories]
  DUMP: estimated 3074909 blocks.
  DUMP: Volume 1 started with block 1 at: Wed Apr 26 14:29:23 2017
  DUMP: dumping (Pass III) [directories]
  DUMP: dumping (Pass IV) [regular files]
  DUMP: Closing rhel7.2.img.dump
  DUMP: Volume 1 completed at: Wed Apr 26 14:31:10 2017
  DUMP: Volume 1 3187210 blocks (3112.51MB)
  DUMP: Volume 1 took 0:01:47
  DUMP: Volume 1 transfer rate: 29787 kB/s
  DUMP: 3187210 blocks (3112.51MB) on 1 volume(s)
  DUMP: finished in 106 seconds, throughput 30068 kBytes/sec
  DUMP: Date of this level 0 dump: Wed Apr 26 14:29:12 2017
  DUMP: Date this dump completed:  Wed Apr 26 14:31:10 2017
  DUMP: Average transfer rate: 29787 kB/s
  DUMP: DUMP IS DONE

# kpartx -d /dev/loop0
# losetup -d /dev/loop0
# losetup -f
/dev/loop0
# losetup /dev/loop0 rhel7.2.img
# ll /images/sso/
total 6705768
-rw-r--r--. 1 root root 21474836480 Apr 26 14:19 rhel7.2.img
-rw-r--r--. 1 root root  3263703040 Apr 26 14:31 rhel7.2.img.dump


# kpartx -l /dev/loop0
loop0p1 : 0 41940992 /dev/loop0 2048
# losetup -d /dev/loop0


# mv rhel7.2.img rhel7.2.img.bak_20170426_1436
# ls /images/sso/
rhel7.2.img.bak_20170426_1436  rhel7.2.img.dump
# losetup -a
# losetup -l /dev/loop0
NAME       SIZELIMIT OFFSET AUTOCLEAR RO BACK-FILE
/dev/loop0                          0  0
```
### image作成
```
# dd if=/dev/zero of=/images/sso/rhel7.2.img bs=512M count=40
40+0 records in
40+0 records out
21474836480 bytes (21 GB) copied, 67.5232 s, 318 MB/s
```
### image清理
```
# losetup -f
/dev/loop0
# losetup /dev/loop0 /images/sso/rhel7.2.img
# kpartx -l /dev/loop0


# fdisk /dev/loop0
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0xf178fd5a.

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-41943039, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-41943039, default 41943039):
Using default value 41943039
Partition 1 of type Linux and of size 20 GiB is set

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 22: Invalid argument.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.


# kpartx -a /dev/loop0
# mke2fs -j /dev/mapper/loop0p1
mke2fs 1.42.9 (28-Dec-2013)
Discarding device blocks: done
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
1310720 inodes, 5242624 blocks
262131 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=4294967296
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
# e2label /dev/mapper/loop0p1  "/"

# mount /dev/mapper/loop0p1 /mnt_20170422_122945
# cd /mnt_20170422_122945

[root@localhost mnt_20170422_122945]# restore -rf /images/sso/rhel7.2.img.dump
restore: ./lost+found: File exists
[root@localhost mnt_20170422_122945]# rm -f restoresymtable
[root@localhost mnt_20170422_122945]#
[root@localhost mnt_20170422_122945]# cd /root/sh_cleanup_image/
#
#
# umount /mnt_20170422_122945
# rmdir /mnt_20170422_122945
# kpartx -d /dev/loop0
# losetup -d /dev/loop0
# ls /images/sso/
# mkdir -p /home/master_latest/domU/20170426_145500
# gzip -c /images/sso/rhel7.2.img > /home/master_latest/domU/20170426_145500/rhel7.2.img.gz
# ll /home/master_latest/domU/20170426_145500/rhel7.2.img.gz
-rw-r--r--. 1 root root 1366027699 Apr 26 15:02 /home/master_latest/domU/20170426_145500/rhel7.2.img.gz
```
