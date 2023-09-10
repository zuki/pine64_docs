# Buildrootによるインストール

## ビルド

```bash
$ wget (https://buildroot.org/downloads/buildroot-2023.02.4.tar.xz
$ tar xf buildroot-2023.02.4.tar.xz
$ cd buildroot-2023.02.4
$ make list-defconfigs
$ make pine64_defconfig
$ make menuconfig
$ make
```

- 環境変数`BL31`を設定してarm_trunsted_frimwareをmake bl31すると
  bl31がビルドされず、エラー発生

  ```bash
  make[2]: Nothing to be done for 'bl31'
  ...
  cp: cannot stat '$HOME/buildroot-2023.02.4/output/build/arm-trusted-firmware-v2.7/build/sun50i_a64/release/*.bin': No such file or directory
  make[1]: *** [package/pkg-generic.mk:374: /home/vagrant/buildroot-2023.02.4/output/build/arm-trusted-firmware-v2.7/.stamp_images_installed] Error 1
  make: *** [Makefile:82: _all] Error 2
  ```

- BL31を削除して`make all`を実行する（`make`ではbl31しか実行されない）

  ```bash
  $ rm -rf output/build/arm-trusted-firmware-v2.7/build/sun50i_a64
  $ make all
  ```

## images

```bash
$ ls -l output/images
total 152040
-rwxrwxr-x 1 vagrant vagrant     37077 Sep  3 10:20 bl31.bin
-rwxr-xr-x 1 vagrant vagrant       270 Sep  2 11:09 boot.scr
-rw-r--r-- 1 vagrant vagrant  67108864 Sep  3 11:05 boot.vfat
-rw-r--r-- 1 vagrant vagrant  19927552 Sep  3 11:04 Image
-rw-r--r-- 1 vagrant vagrant  62914560 Sep  3 11:05 rootfs.ext2
lrwxrwxrwx 1 vagrant vagrant        11 Sep  3 11:05 rootfs.ext4 -> rootfs.ext2
-rw-r--r-- 1 vagrant vagrant  28979200 Sep  3 11:05 rootfs.tar
-rw-r--r-- 1 vagrant vagrant 131112960 Sep  3 11:05 sdcard.img
-rwxr-xr-x 1 vagrant vagrant     21736 Sep  3 11:04 sun50i-a64-pine64.dtb
-rw-r--r-- 1 vagrant vagrant     32768 Sep  3 10:30 sunxi-spl.bin
-rw-r--r-- 1 vagrant vagrant    543866 Sep  3 10:30 u-boot.bin
-rw-r--r-- 1 vagrant vagrant    601356 Sep  3 10:30 u-boot.itb
```

### ログ

```bash
>>>   Generating root filesystems common tables
rm -rf /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs
mkdir -p /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs
printf '   \n' >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/full_users_table.txt
printf '   	/bin/busybox                     f 4755 0  0 - - - - -\n\n' > /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/full_devices_table.txt
cat system/device_table.txt >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/full_devices_table.txt
>>>   Generating filesystem image rootfs.ext2
mkdir -p /home/vagrant/buildroot-2023.02.4/output/images
rm -rf /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2
mkdir -p /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2
rsync -auH --exclude=/THIS_IS_NOT_YOUR_ROOT_FILESYSTEM /home/vagrant/buildroot-2023.02.4/output/target/ /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/target
echo '#!/bin/sh' > /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/fakeroot
echo "set -e" >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/fakeroot
echo "chown -h -R 0:0 /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/target" >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/fakeroot
PATH="/home/vagrant/buildroot-2023.02.4/output/host/bin:/home/vagrant/buildroot-2023.02.4/output/host/sbin:/home/vagrant/.local/bin:/usr/local/or1k-linux-musl-cross/bin:/home/vagrant/llvm-aarch64/bin:/home/vagrant/riscv-gnu-toolchain/bin:/home/vagrant/riscv-gnu-linux/bin:/home/vagrant/aarch64-linux-musl-cross/bin:/home/vagrant/.nvm/versions/node/v12.14.0/bin:/home/vagrant/arm-toolchain/gcc-arm-11.2/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin" /home/vagrant/buildroot-2023.02.4/support/scripts/mkusers /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/full_users_table.txt /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/target >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/fakeroot
echo "/home/vagrant/buildroot-2023.02.4/output/host/bin/makedevs -d /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/full_devices_table.txt /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/target" >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/fakeroot
printf '   	rm -rf /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/target/usr/lib/udev/hwdb.d/ /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/target/etc/udev/hwdb.d/\n' >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/fakeroot
echo "find /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/target/run/ -mindepth 1 -prune -print0 | xargs -0r rm -rf --" >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/fakeroot
echo "find /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/target/tmp/ -mindepth 1 -prune -print0 | xargs -0r rm -rf --" >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/fakeroot
printf '   \n' >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/fakeroot
printf '   \n' >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/fakeroot
printf '   	rm -f /home/vagrant/buildroot-2023.02.4/output/images/rootfs.ext2\n	/home/vagrant/buildroot-2023.02.4/output/host/sbin/mkfs.ext4 -d /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/target -r 1 -N 0 -m 5 -L "rootfs" -I 256 -O ^64bit /home/vagrant/buildroot-2023.02.4/output/images/rootfs.ext2 "60M" || { ret=$?; echo "*** Maybe you need to increase the filesystem size (BR2_TARGET_ROOTFS_EXT2_SIZE)" 1>&2; exit $ret; }\n' >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/fakeroot
chmod a+x /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/fakeroot
PATH="/home/vagrant/buildroot-2023.02.4/output/host/bin:/home/vagrant/buildroot-2023.02.4/output/host/sbin:/home/vagrant/.local/bin:/usr/local/or1k-linux-musl-cross/bin:/home/vagrant/llvm-aarch64/bin:/home/vagrant/riscv-gnu-toolchain/bin:/home/vagrant/riscv-gnu-linux/bin:/home/vagrant/aarch64-linux-musl-cross/bin:/home/vagrant/.nvm/versions/node/v12.14.0/bin:/home/vagrant/arm-toolchain/gcc-arm-11.2/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin" FAKEROOTDONTTRYCHOWN=1 /home/vagrant/buildroot-2023.02.4/output/host/bin/fakeroot -- /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/fakeroot
rootdir=/home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/ext2/target
table='/home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/full_devices_table.txt'
mke2fs 1.46.5 (30-Dec-2021)
Creating regular file /home/vagrant/buildroot-2023.02.4/output/images/rootfs.ext2
64-bit filesystem support is not enabled.  The larger fields afforded by this feature enable full-strength checksumming.  Pass -O 64bit to rectify.
Creating filesystem with 61440 1k blocks and 15360 inodes
Filesystem UUID: cc7f988b-c5e6-4217-a114-8ffba15be452
Superblock backups stored on blocks:
	8193, 24577, 40961, 57345

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Copying files into the device: done
Writing superblocks and filesystem accounting information: done

ln -sf rootfs.ext2 /home/vagrant/buildroot-2023.02.4/output/images/rootfs.ext4
>>>   Generating filesystem image rootfs.tar
mkdir -p /home/vagrant/buildroot-2023.02.4/output/images
rm -rf /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar
mkdir -p /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar
rsync -auH --exclude=/THIS_IS_NOT_YOUR_ROOT_FILESYSTEM /home/vagrant/buildroot-2023.02.4/output/target/ /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/target
echo '#!/bin/sh' > /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/fakeroot
echo "set -e" >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/fakeroot
echo "chown -h -R 0:0 /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/target" >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/fakeroot
PATH="/home/vagrant/buildroot-2023.02.4/output/host/bin:/home/vagrant/buildroot-2023.02.4/output/host/sbin:/home/vagrant/.local/bin:/usr/local/or1k-linux-musl-cross/bin:/home/vagrant/llvm-aarch64/bin:/home/vagrant/riscv-gnu-toolchain/bin:/home/vagrant/riscv-gnu-linux/bin:/home/vagrant/aarch64-linux-musl-cross/bin:/home/vagrant/.nvm/versions/node/v12.14.0/bin:/home/vagrant/arm-toolchain/gcc-arm-11.2/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin" /home/vagrant/buildroot-2023.02.4/support/scripts/mkusers /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/full_users_table.txt /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/target >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/fakeroot
echo "/home/vagrant/buildroot-2023.02.4/output/host/bin/makedevs -d /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/full_devices_table.txt /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/target" >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/fakeroot
printf '   	rm -rf /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/target/usr/lib/udev/hwdb.d/ /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/target/etc/udev/hwdb.d/\n' >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/fakeroot
echo "find /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/target/run/ -mindepth 1 -prune -print0 | xargs -0r rm -rf --" >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/fakeroot
echo "find /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/target/tmp/ -mindepth 1 -prune -print0 | xargs -0r rm -rf --" >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/fakeroot
printf '   \n' >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/fakeroot
printf '   \n' >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/fakeroot
printf '   	(cd /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/target; find -print0 | LC_ALL=C sort -z | tar  --pax-option=exthdr.name=%%d/PaxHeaders/%%f,atime:=0,ctime:=0 -cf /home/vagrant/buildroot-2023.02.4/output/images/rootfs.tar --null --xattrs-include='\''*'\'' --no-recursion -T - --numeric-owner)\n' >> /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/fakeroot
chmod a+x /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/fakeroot
PATH="/home/vagrant/buildroot-2023.02.4/output/host/bin:/home/vagrant/buildroot-2023.02.4/output/host/sbin:/home/vagrant/.local/bin:/usr/local/or1k-linux-musl-cross/bin:/home/vagrant/llvm-aarch64/bin:/home/vagrant/riscv-gnu-toolchain/bin:/home/vagrant/riscv-gnu-linux/bin:/home/vagrant/aarch64-linux-musl-cross/bin:/home/vagrant/.nvm/versions/node/v12.14.0/bin:/home/vagrant/arm-toolchain/gcc-arm-11.2/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin" FAKEROOTDONTTRYCHOWN=1 /home/vagrant/buildroot-2023.02.4/output/host/bin/fakeroot -- /home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/fakeroot
rootdir=/home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/tar/target
table='/home/vagrant/buildroot-2023.02.4/output/build/buildroot-fs/full_devices_table.txt'
ln -snf /home/vagrant/buildroot-2023.02.4/output/host/aarch64-buildroot-linux-gnu/sysroot /home/vagrant/buildroot-2023.02.4/output/staging
>>>   Executing post-image script support/scripts/genimage.sh
INFO: cmd: "mkdir -p "/home/vagrant/buildroot-2023.02.4/output/build/genimage.tmp"" (stderr):
INFO: cmd: "rm -rf "/home/vagrant/buildroot-2023.02.4/output/build/genimage.tmp"/*" (stderr):
INFO: cmd: "mkdir -p "/home/vagrant/buildroot-2023.02.4/output/build/genimage.tmp"" (stderr):
INFO: cmd: "cp -a "/tmp/tmp.xsz70fbl67" "/home/vagrant/buildroot-2023.02.4/output/build/genimage.tmp/root"" (stderr):
INFO: cmd: "mkdir -p "/home/vagrant/buildroot-2023.02.4/output/images"" (stderr):
INFO: vfat(boot.vfat): cmd: "mkdosfs   '/home/vagrant/buildroot-2023.02.4/output/images/boot.vfat'" (stderr):
INFO: vfat(boot.vfat): adding file 'Image' as 'Image' ...
INFO: vfat(boot.vfat): cmd: "MTOOLS_SKIP_CHECK=1 mcopy -sp -i '/home/vagrant/buildroot-2023.02.4/output/images/boot.vfat' '/home/vagrant/buildroot-2023.02.4/output/images/Image' '::'" (stderr):
INFO: vfat(boot.vfat): adding file 'sun50i-a64-pine64.dtb' as 'sun50i-a64-pine64.dtb' ...
INFO: vfat(boot.vfat): cmd: "MTOOLS_SKIP_CHECK=1 mcopy -sp -i '/home/vagrant/buildroot-2023.02.4/output/images/boot.vfat' '/home/vagrant/buildroot-2023.02.4/output/images/sun50i-a64-pine64.dtb' '::'" (stderr):
INFO: vfat(boot.vfat): adding file 'boot.scr' as 'boot.scr' ...
INFO: vfat(boot.vfat): cmd: "MTOOLS_SKIP_CHECK=1 mcopy -sp -i '/home/vagrant/buildroot-2023.02.4/output/images/boot.vfat' '/home/vagrant/buildroot-2023.02.4/output/images/boot.scr' '::'" (stderr):
INFO: hdimage(sdcard.img): adding partition 'spl' from 'sunxi-spl.bin' ...
INFO: hdimage(sdcard.img): adding partition 'u-boot' from 'u-boot.itb' ...
INFO: hdimage(sdcard.img): adding partition 'boot' (in MBR) from 'boot.vfat' ...
INFO: hdimage(sdcard.img): adding partition 'rootfs' (in MBR) from 'rootfs.ext4' ...
INFO: hdimage(sdcard.img): adding partition '[MBR]' ...
INFO: hdimage(sdcard.img): writing MBR
```

## 書き込み

- VirtualBoxではカードリーダが認識せず
- macで実行

  ```bash
  $ sudo diskutil umountDisk /dev/disk2
  $ sudo dd bs=4194304 if=./sdcard.img of=/dev/disk2 conv=sync
  31+1 records in
  32+0 records out
  134217728 bytes transferred in 40.601620 secs (3305723 bytes/sec)
  ```
## 実行

```
U-Boot SPL 2019.01 (Sep 03 2023 - 10:29:59 +0900)
DRAM: 1024 MiB
Trying to boot from MMC1
NOTICE:  BL31: v2.7(release):
NOTICE:  BL31: Built : 10:20:46, Sep  3 2023
NOTICE:  BL31: Detected Allwinner A64/H64/R18 SoC (1689)
NOTICE:  BL31: Found U-Boot DTB at 0x2080138, model: Pine64+


U-Boot 2019.01 (Sep 03 2023 - 10:29:59 +0900) Allwinner Technology

CPU:   Allwinner A64 (SUN50I)
Model: Pine64+
DRAM:  1 GiB
MMC:   SUNXI SD/MMC: 0
Loading Environment from FAT... *** Warning - bad CRC, using default environment

In:    serial
Out:   serial
Err:   serial
Net:   phy interface7
eth0: ethernet@1c30000
starting USB...
USB0:   USB EHCI 1.00
USB1:   USB OHCI 1.0
USB2:   USB EHCI 1.00
USB3:   USB OHCI 1.0
scanning bus 0 for devices... 1 USB Device(s) found
scanning bus 2 for devices... 1 USB Device(s) found
       scanning usb for storage devices... 0 Storage Device(s) found
Hit any key to stop autoboot:  0
switch to partitions #0, OK
mmc0 is current device
Scanning mmc 0:1...
Found U-Boot script /boot.scr
270 bytes read in 1 ms (263.7 KiB/s)
## Executing script at 4fc00000
19927552 bytes read in 880 ms (21.6 MiB/s)
21736 bytes read in 2 ms (10.4 MiB/s)
## Flattened Device Tree blob at 4fa00000
   Booting using the fdt blob at 0x4fa00000
EHCI failed to shut down host controller.
EHCI failed to shut down host controller.
   Loading Device Tree to 0000000049ff7000, end 0000000049fff4e7 ... OK

Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x410fd034]
[    0.000000] Linux version 5.0.0 (vagrant@ubuntu-bionic) (gcc version 11.4.0 3
[    0.000000] Machine model: Pine64
[    0.000000] efi: Getting EFI parameters from FDT:
[    0.000000] efi: UEFI not found.
[    0.000000] cma: Reserved 32 MiB at 0x000000007e000000
[    0.000000] NUMA: No NUMA configuration found
[    0.000000] NUMA: Faking a node at [mem 0x0000000040000000-0x000000007ffffff]
[    0.000000] NUMA: NODE_DATA [mem 0x7dde0840-0x7dde1fff]
[    0.000000] Zone ranges:
[    0.000000]   DMA32    [mem 0x0000000040000000-0x000000007fffffff]
[    0.000000]   Normal   empty
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000040000000-0x000000007fffffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000040000000-0x000000007fffffff]
[    0.000000] psci: probing for conduit method from DT.
[    0.000000] psci: PSCIv1.1 detected in firmware.
[    0.000000] psci: Using standard PSCI v0.2 function IDs
[    0.000000] psci: MIGRATE_INFO_TYPE not supported.
[    0.000000] psci: SMC Calling Convention v1.0
[    0.000000] random: get_random_bytes called from start_kernel+0x9c/0x3f8 wit0
[    0.000000] percpu: Embedded 23 pages/cpu @(____ptrval____) s56024 r8192 d298
[    0.000000] Detected VIPT I-cache on CPU0
[    0.000000] CPU features: detected: ARM erratum 845719
[    0.000000] CPU features: detected: ARM erratum 843419
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 258048
[    0.000000] Policy zone: DMA32
[    0.000000] Kernel command line: console=ttyS0,115200 earlyprintk root=/dev/t
[    0.000000] Memory: 976848K/1048576K available (11132K kernel code, 1604K rw)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=4, Nodes=1
[    0.000000] rcu: Preemptible hierarchical RCU implementation.
[    0.000000] rcu:     RCU restricting CPUs from NR_CPUS=64 to nr_cpu_ids=4.
[    0.000000]  Tasks RCU enabled.
[    0.000000] rcu: RCU calculated value of scheduler-enlistment delay is 25 ji.
[    0.000000] rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=4
[    0.000000] NR_IRQS: 64, nr_irqs: 64, preallocated irqs: 0
[    0.000000] GIC: Using split EOI/Deactivate mode
[    0.000000] arch_timer: cp15 timer(s) running at 24.00MHz (phys).
[    0.000000] clocksource: arch_sys_counter: mask: 0xffffffffffffff max_cycless
[    0.000005] sched_clock: 56 bits at 24MHz, resolution 41ns, wraps every 4398s
[    0.000573] Console: colour dummy device 80x25
[    0.000654] Calibrating delay loop (skipped), value calculated using timer f)
[    0.000666] pid_max: default: 32768 minimum: 301
[    0.000754] LSM: Security Framework initializing
[    0.001354] Dentry cache hash table entries: 131072 (order: 8, 1048576 bytes)
[    0.001654] Inode-cache hash table entries: 65536 (order: 7, 524288 bytes)
[    0.001693] Mount-cache hash table entries: 2048 (order: 2, 16384 bytes)
[    0.001711] Mountpoint-cache hash table entries: 2048 (order: 2, 16384 bytes)
[    0.024030] ASID allocator initialised with 32768 entries
[    0.032023] rcu: Hierarchical SRCU implementation.
[    0.041764] EFI services will not be available.
[    0.048060] smp: Bringing up secondary CPUs ...
[    0.080739] Detected VIPT I-cache on CPU1
[    0.080792] CPU1: Booted secondary processor 0x0000000001 [0x410fd034]
[    0.112461] Detected VIPT I-cache on CPU2
[    0.112494] CPU2: Booted secondary processor 0x0000000002 [0x410fd034]
[    0.144483] Detected VIPT I-cache on CPU3
[    0.144512] CPU3: Booted secondary processor 0x0000000003 [0x410fd034]
[    0.144587] smp: Brought up 1 node, 4 CPUs
[    0.144607] SMP: Total of 4 processors activated.
[    0.144615] CPU features: detected: 32-bit EL0 Support
[    0.144623] CPU features: detected: CRC32 instructions
[    0.145064] CPU: All CPU(s) started at EL2
[    0.145084] alternatives: patching kernel code
[    0.146440] devtmpfs: initialized
[    0.151446] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, ms
[    0.151485] futex hash table entries: 1024 (order: 4, 65536 bytes)
[    0.153026] pinctrl core: initialized pinctrl subsystem
[    0.154619] DMI not present or invalid.
[    0.155136] NET: Registered protocol family 16
[    0.155476] audit: initializing netlink subsys (disabled)
[    0.155618] audit: type=2000 audit(0.152:1): state=initialized audit_enabled1
[    0.157235] cpuidle: using governor menu
[    0.157541] vdso: 2 pages (1 code @ (____ptrval____), 1 data @ (____ptrval__)
[    0.157552] hw-breakpoint: found 6 breakpoint and 4 watchpoint registers.
[    0.158929] DMA: preallocated 256 KiB pool for atomic allocations
[    0.160488] Serial: AMBA PL011 UART driver
[    0.184868] HugeTLB registered 1.00 GiB page size, pre-allocated 0 pages
[    0.184881] HugeTLB registered 32.0 MiB page size, pre-allocated 0 pages
[    0.184889] HugeTLB registered 2.00 MiB page size, pre-allocated 0 pages
[    0.184897] HugeTLB registered 64.0 KiB page size, pre-allocated 0 pages
[    0.185439] cryptd: max_cpu_qlen set to 1000
[    0.186308] ACPI: Interpreter disabled.
[    0.187796] vgaarb: loaded
[    0.188138] SCSI subsystem initialized
[    0.188778] usbcore: registered new interface driver usbfs
[    0.188839] usbcore: registered new interface driver hub
[    0.188913] usbcore: registered new device driver usb
[    0.189931] pps_core: LinuxPPS API ver. 1 registered
[    0.189940] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giom>
[    0.189977] PTP clock support registered
[    0.190148] EDAC MC: Ver: 3.0.0
[    0.191427] Advanced Linux Sound Architecture Driver Initialized.
[    0.192202] clocksource: Switched to clocksource arch_sys_counter
[    0.192381] VFS: Disk quotas dquot_6.6.0
[    0.192447] VFS: Dquot-cache hash table entries: 512 (order 0, 4096 bytes)
[    0.192901] pnp: PnP ACPI: disabled
[    0.201628] NET: Registered protocol family 2
[    0.202146] tcp_listen_portaddr_hash hash table entries: 512 (order: 1, 8192)
[    0.202285] TCP established hash table entries: 8192 (order: 4, 65536 bytes)
[    0.202410] TCP bind hash table entries: 8192 (order: 5, 131072 bytes)
[    0.202561] TCP: Hash tables configured (established 8192 bind 8192)
[    0.202685] UDP hash table entries: 512 (order: 2, 16384 bytes)
[    0.202725] UDP-Lite hash table entries: 512 (order: 2, 16384 bytes)
[    0.202884] NET: Registered protocol family 1
[    0.203281] RPC: Registered named UNIX socket transport module.
[    0.203288] RPC: Registered udp transport module.
[    0.203293] RPC: Registered tcp transport module.
[    0.203299] RPC: Registered tcp NFSv4.1 backchannel transport module.
[    0.204003] kvm [1]: 8-bit VMID
[    0.204011] kvm [1]: IPA Size Limit: 40bits
[    0.205676] kvm [1]: vgic interrupt IRQ1
[    0.205792] kvm [1]: Hyp mode initialized successfully
[    0.212577] Initialise system trusted keyrings
[    0.212727] workingset: timestamp_bits=44 max_order=18 bucket_order=0
[    0.221520] squashfs: version 4.0 (2009/01/31) Phillip Lougher
[    0.222381] NFS: Registering the id_resolver key type
[    0.222411] Key type id_resolver registered
[    0.222417] Key type id_legacy registered
[    0.222430] nfs4filelayout_init: NFSv4 File Layout Driver Registering...
[    0.222639] 9p: Installing v9fs 9p2000 file system support
[    0.229008] Key type asymmetric registered
[    0.229022] Asymmetric key parser 'x509' registered
[    0.229094] Block layer SCSI generic (bsg) driver version 0.4 loaded (major )
[    0.229105] io scheduler mq-deadline registered
[    0.229111] io scheduler kyber registered
[    0.230776] sun50i-de2-bus 1000000.de2: Error couldn't map SRAM to device
[    0.231703] sun4i-usb-phy 1c19400.phy: failed to get clock usb0_phy
[    0.238339] sun50i-a64-r-pinctrl 1f02c00.pinctrl: initialized sunXi PIO drivr
[    0.242526] EINJ: ACPI disabled.
[    0.255946] Serial: 8250/16550 driver, 4 ports, IRQ sharing enabled
[    0.258880] SuperH (H)SCI(F) driver initialized
[    0.259356] msm_serial: driver initialized
[    0.269261] loop: module loaded
[    0.274251] libphy: Fixed MDIO Bus: probed
[    0.274791] tun: Universal TUN/TAP device driver, 1.6
[    0.275879] thunder_xcv, ver 1.0
[    0.275936] thunder_bgx, ver 1.0
[    0.275995] nicpf, ver 1.0
[    0.276583] hclge is initializing
[    0.276593] hns3: Hisilicon Ethernet Network Driver for Hip08 Family - versin
[    0.276599] hns3: Copyright (c) 2017 Huawei Corporation.
[    0.276680] e1000e: Intel(R) PRO/1000 Network Driver - 3.2.6-k
[    0.276686] e1000e: Copyright(c) 1999 - 2015 Intel Corporation.
[    0.276737] igb: Intel(R) Gigabit Ethernet Network Driver - version 5.4.0-k
[    0.276743] igb: Copyright (c) 2007-2014 Intel Corporation.
[    0.276793] igbvf: Intel(R) Gigabit Virtual Function Network Driver - versiok
[    0.276799] igbvf: Copyright (c) 2009 - 2012 Intel Corporation.
[    0.277188] sky2: driver version 1.30
[    0.278029] VFIO - User Level meta-driver version: 0.3
[    0.279571] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    0.279583] ehci-pci: EHCI PCI platform driver
[    0.279638] ehci-platform: EHCI generic platform driver
[    0.279894] ehci-platform 1c1a000.usb: EHCI Host Controller
[    0.279920] ehci-platform 1c1a000.usb: new USB bus registered, assigned bus 1
[    0.280793] ehci-platform 1c1a000.usb: irq 13, io mem 0x01c1a000
[    0.296216] ehci-platform 1c1a000.usb: USB 2.0 started, EHCI 1.00
[    0.296960] hub 1-0:1.0: USB hub found
[    0.296994] hub 1-0:1.0: 1 port detected
[    0.297677] ehci-orion: EHCI orion driver
[    0.297908] ehci-exynos: EHCI EXYNOS driver
[    0.298038] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[    0.298066] ohci-pci: OHCI PCI platform driver
[    0.298151] ohci-platform: OHCI generic platform driver
[    0.298346] ohci-platform 1c1a400.usb: Generic Platform OHCI controller
[    0.298374] ohci-platform 1c1a400.usb: new USB bus registered, assigned bus 2
[    0.298707] ohci-platform 1c1a400.usb: irq 14, io mem 0x01c1a400
[    0.360911] hub 2-0:1.0: USB hub found
[    0.360953] hub 2-0:1.0: 1 port detected
[    0.361600] ohci-exynos: OHCI EXYNOS driver
[    0.362386] usbcore: registered new interface driver usb-storage
[    0.365895] sun6i-rtc 1f00000.rtc: registered as rtc0
[    0.365909] sun6i-rtc 1f00000.rtc: RTC enabled
[    0.366257] i2c /dev entries driver
[    0.371153] sdhci: Secure Digital Host Controller Interface driver
[    0.371168] sdhci: Copyright(c) Pierre Ossman
[    0.371654] Synopsys Designware Multimedia Card Interface Driver
[    0.372817] sdhci-pltfm: SDHCI platform and OF driver helper
[    0.374268] ledtrig-cpu: registered to indicate activity on CPUs
[    0.375797] usbcore: registered new interface driver usbhid
[    0.375806] usbhid: USB HID core driver
[    0.379506] NET: Registered protocol family 17
[    0.379712] 9pnet: Installing 9P2000 support
[    0.379780] Key type dns_resolver registered
[    0.380400] registered taskstats version 1
[    0.380408] Loading compiled-in X.509 certificates
[    0.392496] sun50i-a64-r-pinctrl 1f02c00.pinctrl: 1f02c00.pinctrl supply vccr
[    0.392587] sun50i-a64-r-pinctrl 1f02c00.pinctrl: Linked as a consumer to re0
[    0.392724] sunxi-rsb 1f03400.rsb: RSB running at 3000000 Hz
[    0.393164] axp20x-rsb sunxi-rsb-3a3: AXP20x variant AXP803 found
[    0.401403] dcdc1: supplied by regulator-dummy
[    0.401749] dcdc2: supplied by regulator-dummy
[    0.402001] dcdc4: supplied by regulator-dummy
[    0.402285] dcdc5: supplied by regulator-dummy
[    0.402557] dcdc6: supplied by regulator-dummy
[    0.402806] dc1sw: supplied by regulator-dummy
[    0.403020] aldo1: supplied by regulator-dummy
[    0.403284] aldo2: supplied by regulator-dummy
[    0.403350] vcc-pl: Bringing 1600000uV into 1800000-1800000uV
[    0.403576] aldo3: supplied by regulator-dummy
[    0.403856] dldo1: supplied by regulator-dummy
[    0.404102] dldo2: supplied by regulator-dummy
[    0.404174] vcc-mipi: Bringing 1800000uV into 3300000-3300000uV
[    0.404471] dldo3: supplied by regulator-dummy
[    0.404730] dldo4: supplied by regulator-dummy
[    0.405030] eldo1: supplied by regulator-dummy
[    0.405277] eldo2: supplied by regulator-dummy
[    0.405535] eldo3: supplied by regulator-dummy
[    0.405823] fldo1: supplied by regulator-dummy
[    0.406086] fldo2: supplied by regulator-dummy
[    0.406356] rtc-ldo: supplied by regulator-dummy
[    0.406606] ldo-io0: supplied by regulator-dummy
[    0.406862] ldo-io1: supplied by regulator-dummy
[    0.407088] axp20x-rsb sunxi-rsb-3a3: AXP20X driver loaded
[    0.412261] sun50i-a64-pinctrl 1c20800.pinctrl: initialized sunXi PIO driver
[    0.413247] sun50i-a64-pinctrl 1c20800.pinctrl: 1c20800.pinctrl supply vcc-pr
[    0.413333] sun50i-a64-pinctrl 1c20800.pinctrl: Linked as a consumer to regu0
[    0.413747] printk: console [ttyS0] disabled
[    0.434393] 1c28000.serial: ttyS0 at MMIO 0x1c28000 (irq = 22, base_baud = 1A
[    1.624686] printk: console [ttyS0] enabled
[    1.630685] ehci-platform 1c1b000.usb: EHCI Host Controller
[    1.636306] ehci-platform 1c1b000.usb: new USB bus registered, assigned bus 3
[    1.644721] ehci-platform 1c1b000.usb: irq 15, io mem 0x01c1b000
[    1.664220] ehci-platform 1c1b000.usb: USB 2.0 started, EHCI 1.00
[    1.670985] hub 3-0:1.0: USB hub found
[    1.674770] hub 3-0:1.0: 1 port detected
[    1.680321] ohci-platform 1c1b400.usb: Generic Platform OHCI controller
[    1.686960] ohci-platform 1c1b400.usb: new USB bus registered, assigned bus 4
[    1.695002] ohci-platform 1c1b400.usb: irq 16, io mem 0x01c1b400
[    1.764946] hub 4-0:1.0: USB hub found
[    1.768733] hub 4-0:1.0: 1 port detected
[    1.774392] usb_phy_generic usb_phy_generic.0.auto: usb_phy_generic.0.auto sr
[    1.785163] usb_phy_generic usb_phy_generic.0.auto: Linked as a consumer to 0
[    1.793806] musb-hdrc musb-hdrc.1.auto: MUSB HDRC host driver
[    1.799568] musb-hdrc musb-hdrc.1.auto: new USB bus registered, assigned bus5
[    1.808012] hub 5-0:1.0: USB hub found
[    1.811806] hub 5-0:1.0: 1 port detected
[    1.817504] sun50i-a64-pinctrl 1c20800.pinctrl: 1c20800.pinctrl supply vcc-pr
[    1.829674] sun50i-a64-pinctrl 1c20800.pinctrl: 1c20800.pinctrl supply vcc-pr
[    1.839890] sunxi-mmc 1c0f000.mmc: Linked as a consumer to regulator.1
[    1.847268] sunxi-mmc 1c0f000.mmc: Got CD GPIO
[    1.876239] sunxi-mmc 1c0f000.mmc: initialized, max. request size: 16384 KB,e
[    1.885665] sun6i-rtc 1f00000.rtc: setting system clock to 1970-01-01T00:00:)
[    1.894011] vcc-phy: disabling
[    1.897121] vcc-hdmi: disabling
[    1.900314] cpvdd: disabling
[    1.903230] ALSA device list:
[    1.906206]   No soundcards found.
[    1.910130] Waiting for root device /dev/mmcblk0p2...
[    1.921288] mmc0: new high speed SDHC card at address 5048
[    1.928142] mmcblk0: mmc0:5048 SD32G 28.8 GiB
[    1.938711]  mmcblk0: p1 p2
[    1.953434] EXT4-fs (mmcblk0p2): mounted filesystem with ordered data mode. )
[    1.961579] VFS: Mounted root (ext4 filesystem) readonly on device 179:2.
[    1.969390] devtmpfs: mounted
[    1.973143] Freeing unused kernel memory: 1344K
[    2.000240] Run /sbin/init as init process
[    2.124583] EXT4-fs (mmcblk0p2): re-mounted. Opts: (null)
Starting syslogd: OK
Starting klogd: OK
Running sysctl: OK
[    2.218982] random: seedrng: uninitialized urandom read (256 bytes read)
Saving 2048 bits of non-creditable seed for next boot
Starting network: OK

Welcome to PINE64
buildroot login: [    3.088506] random: fast init done

Welcome to PINE64
buildroot login: root
# ls
# ls -al
total 3
drwx------    2 root     root          1024 Jan  1 00:00 .
drwxr-xr-x   18 root     root          1024 Sep  3  2023 ..
-rw-------    1 root     root            10 Jan  1 00:00 .ash_history
# ls /
bin         lib         lost+found  opt         run         tmp
dev         lib64       media       proc        sbin        usr
etc         linuxrc     mnt         root        sys         var
# ls /bin
arch           dmesg          linux32        nice           setserial
ash            dnsdomainname  linux64        nuke           sh
base32         dumpkmap       ln             pidof          sleep
base64         echo           login          ping           stty
busybox        egrep          ls             pipe_progress  su
cat            false          lsattr         printenv       sync
chattr         fdflush        mkdir          ps             tar
chgrp          fgrep          mknod          pwd            touch
chmod          getopt         mktemp         resume         true
chown          grep           more           rm             umount
cp             gunzip         mount          rmdir          uname
cpio           gzip           mountpoint     run-parts      usleep
date           hostname       mt             sed            vi
dd             kill           mv             setarch        watch
df             link           netstat        setpriv        zcat
# ls /dev
autofs              shm                 tty49
bus                 snapshot            tty5
console             snd                 tty50
cpu_dma_latency     stderr              tty51
fd                  stdin               tty52
full                stdout              tty53
gpiochip0           tty                 tty54
gpiochip1           tty0                tty55
i2c-0               tty1                tty56
kmsg                tty10               tty57
kvm                 tty11               tty58
log                 tty12               tty59
loop-control        tty13               tty6
loop0               tty14               tty60
loop1               tty15               tty61
loop2               tty16               tty62
loop3               tty17               tty63
loop4               tty18               tty7
loop5               tty19               tty8
loop6               tty2                tty9
loop7               tty20               ttyS0
mem                 tty21               ttyS1
memory_bandwidth    tty22               ttyS2
mmcblk0             tty23               ttyS3
mmcblk0p1           tty24               ttyp0
mmcblk0p2           tty25               ttyp1
net                 tty26               ttyp2
network_latency     tty27               ttyp3
network_throughput  tty28               ttyp4
null                tty29               ttyp5
port                tty3                ttyp6
ptmx                tty30               ttyp7
pts                 tty31               ttyp8
ptyp0               tty32               ttyp9
ptyp1               tty33               ttypa
ptyp2               tty34               ttypb
ptyp3               tty35               ttypc
ptyp4               tty36               ttypd
ptyp5               tty37               ttype
ptyp6               tty38               ttypf
ptyp7               tty39               urandom
ptyp8               tty4                vcs
ptyp9               tty40               vcs1
ptypa               tty41               vcsa
ptypb               tty42               vcsa1
ptypc               tty43               vcsu
ptypd               tty44               vcsu1
ptype               tty45               vfio
ptypf               tty46               vga_arbiter
random              tty47               zero
rtc0                tty48
# ls /
bin         lib         lost+found  opt         run         tmp
dev         lib64       media       proc        sbin        usr
etc         linuxrc     mnt         root        sys         var
# ls /proc
1              1611           489            cgroups        locks
10             17             490            cmdline        meminfo
11             18             491            config.gz      misc
12             19             493            consoles       modules
124            2              5              cpuinfo        mounts
1272           20             517            crypto         mtd
13             21             524            device-tree    net
14             22             557            devices        pagetypeinfo
1475           23             568            diskstats      partitions
1477           24             577            driver         self
15             25             6              execdomains    slabinfo
1543           26             663            fb             softirqs
1547           27             664            filesystems    stat
1550           28             665            fs             swaps
1551           29             67             interrupts     sys
1556           3              7              iomem          sysrq-trigger
1557           30             735            ioports        sysvipc
1558           32             8              irq            thread-self
1560           34             831            kallsyms       timer_list
1575           4              9              key-users      tty
1578           41             934            keys           uptime
1582           46             95             kmsg           version
1584           484            99             kpagecgroup    vmallocinfo
16             485            asound         kpagecount     vmstat
1602           487            buddyinfo      kpageflags     zoneinfo
1603           488            bus            loadavg
# cat /proc/filesystems
nodev   sysfs
nodev   rootfs
nodev   ramfs
nodev   bdev
nodev   proc
nodev   cpuset
nodev   cgroup
nodev   cgroup2
nodev   tmpfs
nodev   devtmpfs
nodev   configfs
nodev   debugfs
nodev   securityfs
nodev   sockfs
nodev   pipefs
nodev   hugetlbfs
nodev   rpc_pipefs
nodev   devpts
        ext3
        ext4
        ext2
        squashfs
        vfat
nodev   nfs
nodev   nfs4
nodev   autofs
nodev   9p
nodev   mqueue
nodev   pstore
# [   83.844291] random: crng init done
# halt -f
[  424.857596] kvm: exiting hardware virtualization
[  424.866333] reboot: System halted

```

## boot.scr

```bash
$ cat board/pine64/pine64/boot.cmd
setenv bootargs console=ttyS0,115200 earlyprintk root=/dev/mmcblk0p2 rootwait

fatload mmc 0 $kernel_addr_r Image
fatload mmc 0 $fdt_addr_r sun50i-a64-pine64.dtb

booti $kernel_addr_r - $fdt_addr_r

$ cat board/pine64/pine64/genimage.cfg
image boot.vfat {
	vfat {
		files = {
			"Image",
			"sun50i-a64-pine64.dtb",
			"boot.scr"
		}
	}

	size = 64M
}

image sdcard.img {
	hdimage {
	}

	partition spl {
		in-partition-table = "no"
		image = "sunxi-spl.bin"
		offset = 8K
	}

	partition u-boot {
		in-partition-table = "no"
		image = "u-boot.itb"
		offset = 40K
		size = 1M # 1MB - 40KB
	}

	partition boot {
		partition-type = 0xC
		bootable = "true"
		image = "boot.vfat"
	}

	partition rootfs {
		partition-type = 0x83
		image = "rootfs.ext4"
	}
}

$ ls output/build/host-uboot-tools-2021.07/include/configs/sun50i.h
$ ls
```
```c
// in output/build/host-uboot-tools-2021.07/include/configs/sunxi-common.h
#define SDRAM_OFFSET(x) 0x4##x
#define CONFIG_SYS_SDRAM_BASE           0x40000000
#define CONFIG_SYS_LOAD_ADDR            0x42000000 /* default load address */

#define BOOTM_SIZE        __stringify(0xa000000)
#define KERNEL_ADDR_R     __stringify(SDRAM_OFFSET(0080000))
#define KERNEL_COMP_ADDR_R __stringify(SDRAM_OFFSET(4000000))
#define KERNEL_COMP_SIZE  __stringify(0xb000000)
#define FDT_ADDR_R        __stringify(SDRAM_OFFSET(FA00000))
#define SCRIPT_ADDR_R     __stringify(SDRAM_OFFSET(FC00000))
#define PXEFILE_ADDR_R    __stringify(SDRAM_OFFSET(FD00000))
#define FDTOVERLAY_ADDR_R __stringify(SDRAM_OFFSET(FE00000))
#define RAMDISK_ADDR_R    __stringify(SDRAM_OFFSET(FF00000))

#define MEM_LAYOUT_ENV_SETTINGS \
        "bootm_size=" BOOTM_SIZE "\0" \
        "kernel_addr_r=" KERNEL_ADDR_R "\0" \
        "fdt_addr_r=" FDT_ADDR_R "\0" \
        "scriptaddr=" SCRIPT_ADDR_R "\0" \
        "pxefile_addr_r=" PXEFILE_ADDR_R "\0" \
        "fdtoverlay_addr_r=" FDTOVERLAY_ADDR_R "\0" \
        "ramdisk_addr_r=" RAMDISK_ADDR_R "\0"
```

## .configファイル

- [buildroot/.config](config/buildroot.config)
- [u-boot/.config](config/uboot.config)
## 参考

- [Buildroot](https://buildroot.org/)
- [Buildroot の使い方 - 基本編](https://qiita.com/pu_ri/items/75c80e388c79fe0d3f0b)
