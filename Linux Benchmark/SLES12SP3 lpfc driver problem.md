## SUSE12 GMC SP3版本下面无法识别FC卡连接的远程存储盘

切换到SUSE12 GMC SP2下面却能正常识别
SP2的lpfc驱动信息：
```
filename:       /lib/modules/4.4.21-68-default/kernel/drivers/scsi/lpfc/lpfc.ko
version:        0:11.1.0.1.
author:         Emulex Corporation - tech.support@emulex.com
description:    Emulex LightPulse Fibre Channel SCSI driver 11.1.0.1.
license:        GPL
srcversion:     3AC05E038B18B53D560B0DB
<省略>
```
SP2下面的磁盘信息：
```
linux-l3xb:~ # fdisk -l | egrep "Disk \/dev"
Disk /dev/sda: 136.2 GiB, 146263769088 bytes, 285671424 sectors
Disk /dev/sdb: 136.2 GiB, 146263769088 bytes, 285671424 sectors
Disk /dev/sdc: 136.2 GiB, 146263769088 bytes, 285671424 sectors
Disk /dev/sdd: 136.2 GiB, 146263769088 bytes, 285671424 sectors
Disk /dev/sde: 136.2 GiB, 146263769088 bytes, 285671424 sectors
Disk /dev/sdf: 136.2 GiB, 146263769088 bytes, 285671424 sectors
Disk /dev/sdg: 1.1 TiB, 1174673555456 bytes, 2294284288 sectors
Disk /dev/sdh: 1.1 TiB, 1174673555456 bytes, 2294284288 sectors
Disk /dev/mapper/rhel-swap: 10.7 GiB, 11475615744 bytes, 22413312 sectors
Disk /dev/mapper/rhel-home: 45.2 GiB, 48519708672 bytes, 94765056 sectors
Disk /dev/mapper/rhel-root: 50 GiB, 53687091200 bytes, 104857600 sectors
```
sdg和sdh为远程存储磁盘。


SP3的lpfc驱动信息：
```
filename:       /lib/modules/4.4.73-5-default/kernel/drivers/scsi/lpfc/lpfc.ko
version:        0:11.4.0.1
author:         Broadcom
description:    Emulex LightPulse Fibre Channel SCSI driver 11.4.0.1
license:        GPL
srcversion:     48C707A65B83503A81D44D2
<省略>
```
SP3下的磁盘信息：
```
linux-l3xb:~ # fdisk -l | egrep "Disk \/dev"
Disk /dev/sda: 136.2 GiB, 146263769088 bytes, 285671424 sectors
Disk /dev/sdb: 136.2 GiB, 146263769088 bytes, 285671424 sectors
Disk /dev/sdc: 136.2 GiB, 146263769088 bytes, 285671424 sectors
Disk /dev/sdd: 136.2 GiB, 146263769088 bytes, 285671424 sectors
Disk /dev/sde: 136.2 GiB, 146263769088 bytes, 285671424 sectors
Disk /dev/sdf: 136.2 GiB, 146263769088 bytes, 285671424 sectors
Disk /dev/mapper/rhel-swap: 10.7 GiB, 11475615744 bytes, 22413312 sectors
Disk /dev/mapper/rhel-home: 45.2 GiB, 48519708672 bytes, 94765056 sectors
Disk /dev/mapper/rhel-root: 50 GiB, 53687091200 bytes, 104857600 sectors
```
无法识别远程存储磁盘。
ps：Emulex Corporation在2015年被Broadcom公司收购。

### 问题定位
通过拷贝SP2的内核文件/lib/modules/4.4.21-68-default到SP3系统的/lib/modules,SP3的为/lib/modules/4.4.73-5-default/。然后拷贝SP2的/boot/initrd-4.4.21-68-default，/boot/vmlinuz-4.4.21-68-default文件到SP3系统的/boot,修改/boot/grub2/grub.cfg，追加SP2 4.4.21的启动项，以SP2的内核启动，发现能够识别远程磁盘。
于是尝试更改lpfc的版本。
以下命令sample，后面会用到：
```
# rmmod lpfc  #删除lpfc驱动模块
# modprobe -r lpfc #删除lpfc驱动模块
# modprobe lpfc #追加lpfc驱动模块
# modinfo lpfc #lpfc驱动模块信息
# modinfo /lib/modules/4.4.21-68-default/kernel/drivers/scsi/lpfc/lpfc.ko
# insmod lpfc #插入lpfc驱动模块
```
将SP2的lpfc.ko文件拷贝到/lib/modules/4.4.73-5-default/kernel/drivers/scsi/lpfc/lpfc.ko
```
# modprobe lpfc
modprobe: ERROR: could not insert 'lpfc': Exec format error
# insmod /lib/modules/4.4.73-5-default/kernel/drivers/scsi/lpfc/lpfc.ko
insmod: ERROR: could not insert module /lib/modules/4.4.73-5-default/kernel/drivers/scsi/lpfc/lpfc.ko: Invalid module format
```
于是考虑重新安装驱动。
从官网上下载lpfc suse的驱动：
https://www.broadcom.com/support/download-search?dk=lpfc

下载了elx-lpfc-dd-sles12sp-11.2.156.18-1.tar.gz
重新安装：
```
# ./elx_lpfc_install.sh
Unpacking lpfc installation files...
Installing lpfc package, lpfc-sles12sp-11.2.156.18/elx-lpfc-kmp-default-11.2.156.18_k4.4.21_69-1.sles12sp2.x86_64.rpm ...
Preparing...                          ################################# [100%]
Updating / installing...
   1:elx-lpfc-kmp-default-11.2.156.18_################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:elx-lpfc-vector-map-1-1          ################################# [100%]

Note: This output shows SysV services only and does not include native
systemd services. SysV configuration data might be overridden by native
systemd configuration.

If you want to list systemd services use 'systemctl list-unit-files'.
To see services enabled on particular target use
'systemctl list-dependencies [target]'.

SCSI host6 SLI3 SlowPath vector 62 mapped to CPU0
SCSI host6 SLI3 FastPath vector 63 mapped to CPU0
SCSI host7 SLI3 SlowPath vector 65 mapped to CPU0
SCSI host7 SLI3 FastPath vector 66 mapped to CPU1
Loaded driver lpfc
lpfc installation complete
```
重启机器之后，lpfc的版本并没有改变：
```
filename:       /lib/modules/4.4.73-5-default/kernel/drivers/scsi/lpfc/lpfc.ko
version:        0:11.4.0.1
author:         Broadcom
description:    Emulex LightPulse Fibre Channel SCSI driver 11.4.0.1
license:        GPL
srcversion:     48C707A65B83503A81D44D2
```
磁盘还是无法检测。

看来只有重新编译内核了。编译内核之前需要先下载lpfc 11.1的源码，然后替换到内核的lpfc源码，重新编译。
```
# zypper se kernel-source #查找源码
# zypper lr
[Thu Jul 20 15:07:50.139 2017] Repository priorities are without effect. All enabled repositories share the same priority.
[Thu Jul 20 15:07:50.139 2017]
[Thu Jul 20 15:07:50.139 2017] # | Alias             | Name              | Enabled | GPG Check | Refresh
[Thu Jul 20 15:07:50.139 2017] --+-------------------+-------------------+---------+-----------+--------
[Thu Jul 20 15:07:50.155 2017] 1 | SLES12-SP3-12.3-0 | SLES12-SP3-12.3-0 | Yes     | (r ) Yes  | No
# zypper rr 1
[Thu Jul 20 15:07:55.200 2017] Removing repository 'SLES12-SP3-12.3-0' --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------[\]Removing repository 'SLES12-SP3-12.3-0' --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------[\]Removing repository 'SLES12-SP3-12.3-0' --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------[|]Removing repository 'SLES12-SP3-12.3-0' ...............................................................................................................................................................................................[done]
[Thu Jul 20 15:07:55.208 2017] Repository 'SLES12-SP3-12.3-0' has been removed.
# mount -o loop /mnt1/SUSE12/SLE-12-SP3-Server-DVD-x86_64-GMC-DVD1.iso /mnt
# zypper ar file:///mnt SLES12-SP3
[Thu Jul 20 15:08:08.805 2017] Adding repository 'SLES12-SP3' -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------[\]Adding repository 'SLES12-SP3' -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------[\]Adding repository 'SLES12-SP3' ................................................................................................<50%>======================================================================================================[|]Adding repository 'SLES12-SP3' .................................................................................................................................................................................<90%>=====================[/]Adding repository 'SLES12-SP3' .....................................................................................................................................................................................................<100%>[-]Adding repository 'SLES12-SP3' .....................................................................................................................................................................................................<100%>[\]Adding repository 'SLES12-SP3' ........................................................................................................................................................................................................[done]
[Thu Jul 20 15:08:08.821 2017] Repository 'SLES12-SP3' successfully added
[Thu Jul 20 15:08:08.821 2017]
[Thu Jul 20 15:08:08.821 2017] URI         : file:/mnt
[Thu Jul 20 15:08:08.821 2017] Enabled     : Yes
[Thu Jul 20 15:08:08.821 2017] GPG Check   : Yes
[Thu Jul 20 15:08:08.821 2017] Autorefresh : No
[Thu Jul 20 15:08:08.821 2017] Priority    : 99 (default priority)
[Thu Jul 20 15:08:08.821 2017]
# zypper in kernel-soucerce #安装内核源码
# ls /usr/src/ #源码位置
linux  linux-4.4.73-5  linux-4.4.73-5-obj  linux-obj  packages
```
下载需要的驱动源码：
https://www.broadcom.com/support/download-search?dk=lpfc
elx-lpfc-11.1.218.0-1_sles12sp2_plus.src.rpm

```
# rpm -i elx-lpfc-11.1.218.0-1_sles12sp2_plus.src.rpm #安装源码
# ls /usr/src/packages/SOURCES/ #安装好的源码位置
lpfcdriver-4-11.1.218.0.tar.gz
# tar zxf lpfcdriver-4-11.1.218.0.tar.gz #解压缩源码
# cd lpfcdriver-4-11.1.218.0/
# ls
.tmp_versions  ChangeLog    Makefile  lpfc_attr.c  lpfc_bsg.h     lpfc_crtn.h  lpfc_debugfs.c  lpfc_disc.h  lpfc_hbadisc.c  lpfc_hw4.h   lpfc_logmsg.h  lpfc_mem.c  lpfc_nportdisc.c  lpfc_scsi.h  lpfc_sli.h   lpfc_version.h  lpfc_vport.h
COPYING        GNUmakefile  lpfc.h    lpfc_bsg.c   lpfc_compat.h  lpfc_ct.c    lpfc_debugfs.h  lpfc_els.c   lpfc_hw.h       lpfc_init.c  lpfc_mbox.c    lpfc_nl.h   lpfc_scsi.c       lpfc_sli.c   lpfc_sli4.h  lpfc_vport.c
```

讲驱动源码替换掉内核的源码，内核源码位置在：
```
# ls /usr/src/linux/drivers/scsi/lpfc/
.built-in.o.cmd  lpfc.h       lpfc_bsg.c     lpfc_crtn.h     lpfc_debugfs.h  lpfc_hbadisc.c  lpfc_ids.h     lpfc_mbox.c  lpfc_nportdisc.c  lpfc_nvmet.c  lpfc_scsi.h  lpfc_sli4.h     lpfc_vport.h
Makefile         lpfc_attr.c  lpfc_bsg.h     lpfc_ct.c       lpfc_disc.h     lpfc_hw.h       lpfc_init.c    lpfc_mem.c   lpfc_nvme.c       lpfc_nvmet.h  lpfc_sli.c   lpfc_version.h
built-in.o       lpfc_attr.h  lpfc_compat.h  lpfc_debugfs.c  lpfc_els.c      lpfc_hw4.h      lpfc_logmsg.h  lpfc_nl.h    lpfc_nvme.h       lpfc_scsi.c   lpfc_sli.h   lpfc_vport.c
```
