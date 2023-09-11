# Pin4 A64+へのLinuxのインストール

```bash
$ lsb_release -d
Description:	Ubuntu 22.04.3 LTS
$ aarch64-linux-gnu-gcc --version
aarch64-linux-gnu-gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
```

## ATFのビルド

```bash
$ git clone https://github.com/ARM-software/arm-trusted-firmware.git --depth=1
$ export CROSS_COMPILE=aarch64-linux-gnu-
$ echo $CROSS_COMPILE
aarch64-linux-gnu-
$ cd arm-trusted-firmware/
$ make PLAT=sun50i_a64 DEBUG=1 bl31
$ ls build/sun50i_a64/debug/
bl31  bl31.bin  lib  libc  libfdt  libwrapper  romlib
export BL31=/<atf>/build/sun50i_a64/debug/bl31.bin
```

## SCP (Crust)のビルド

```bash
$ wget http://musl.cc/or1k-linux-musl-cross.tgz
$ tar xf or1k-linux-musl-cross.tgz
$ mv or1k-linux-musl-cross /usr/local
$ export PATH=/usr/local/or1k-linux-musl-cross/bin:$PATH
$ or1k-linux-musl-gcc --version
or1k-linux-musl-gcc (GCC) 11.2.1 20211120
$ git clone https://github.com/crust-firmware/crust --depth=1
$ cd crust
export CROSS_COMPILE=or1k-linux-musl-
$ make pine64_plus_defconfig
fatal: No names found, cannot describe anything.
$ make distclean
$ cp configs/pine64_plus_defconfig .config
$ make scp
fatal: No names found, cannot describe anything.
  HOSTCC  build/3rdparty/kconfig/conf.o
  HOSTCC  build/3rdparty/kconfig/confdata.o
  HOSTCC  build/3rdparty/kconfig/expr.o
  LEX     build/3rdparty/kconfig/lexer.lex.c
  YACC    build/3rdparty/kconfig/parser.tab.c
  HOSTCC  build/3rdparty/kconfig/lexer.lex.o
  HOSTCC  build/3rdparty/kconfig/parser.tab.o
  HOSTCC  build/3rdparty/kconfig/preprocess.o
  HOSTCC  build/3rdparty/kconfig/symbol.o
  HOSTCC  build/3rdparty/kconfig/util.o
  HOSTLD  build/3rdparty/kconfig/conf
  GEN     build/include/config/auto.conf
*
* Restart config...
*
*
* Crust Firmware Configuration
*
Platform selection
  1. A23/A33 (PLATFORM_A23) (NEW)
> 2. A64/H5 (PLATFORM_A64)
  3. A83T (PLATFORM_A83T) (NEW)
  4. H3 (PLATFORM_H3) (NEW)
  5. H6 (PLATFORM_H6) (NEW)
choice[1-5?]:
SoC
> 1. A64 (SOC_A64) (NEW)
  2. H5 (SOC_H5) (NEW)
choice[1-2?]:
*
* Device drivers
*
HDMI CEC (CEC) [N/y/?] (NEW)
CIR (infrared) receiver (CIR) [N/y/?] (NEW)
OSC24M clock source
> 1. X24M (OSC24M_SRC_X24M) (NEW)
choice[1]: 1
I2C controller pin selection
> 1. None (I2C_PINS_NONE) (NEW)
  2. PL0/PL1 (I2C_PINS_PL0_PL1) (NEW)
  3. PL8/PL9 (I2C_PINS_PL8_PL9) (NEW)
choice[1-3?]:
Multi-function driver for X-Powers PMICs (MFD_AXP20X) [Y/n/?] y
  *
  * X-Powers PMIC will communicate via RSB
  *
  X-Powers PMIC variant
    1. AXP223 (MFD_AXP223) (NEW)
  > 2. AXP803 (MFD_AXP803) (NEW)
    3. AXP805 (MFD_AXP805) (NEW)
  choice[1-3?]:
*
* GPIO-controlled CPU power supply
*
GPIO-controlled CPU power supply (REGULATOR_GPIO_CPU) [N/y/?] (NEW)
*
* GPIO-controlled DRAM power supply
*
GPIO-controlled DRAM power supply (REGULATOR_GPIO_DRAM) [N/y/?] (NEW)
*
* GPIO-controlled VCC-PLL power supply
*
GPIO-controlled VCC-PLL power supply (REGULATOR_GPIO_VCC_PLL) [N/y/?] (NEW)
*
* GPIO-controlled VDD-SYS power supply
*
GPIO-controlled VDD-SYS power supply (REGULATOR_GPIO_VDD_SYS) [N/y/?] (NEW)
Serial input/output support (SERIAL) [Y/n/?] (NEW)
  Device
  > 1. UART0 (SERIAL_DEV_UART0) (NEW)
    2. UART1 (SERIAL_DEV_UART1) (NEW)
    3. UART2 (SERIAL_DEV_UART2) (NEW)
    4. UART3 (SERIAL_DEV_UART3) (NEW)
    5. UART4 (SERIAL_DEV_UART4) (NEW)
    6. R_UART (SERIAL_DEV_R_UART) (NEW)
  choice[1-6?]:
Watchdog timer
> 1. Watchdog (R_WDOG) (WATCHDOG_SUN6I_A31_WDT) (NEW)
  2. Trusted watchdog (R_TWD) (WATCHDOG_SUN9I_A80_TWD) (NEW)
choice[1-2?]:
*
* Firmware features
*
Use PMIC for full hardware shutdown (PMIC_SHUTDOWN) [Y/n/?] (NEW)
*
* Debugging options
*
Enable runtime assertion checking (ASSERT) [Y/n/?] (NEW)
  Verbose logging of assertion failures (ASSERT_VERBOSE) [N/y/?] (NEW)
Allow compiling a firmware that does not run (COMPILE_TEST) [N/y/?] (NEW)
Compile the firmware with debug info (DEBUG_INFO) [Y/n/?] (NEW)
Print additional debug-level log messages (DEBUG_LOG) [N/y/?] (NEW)
Provide an interactive debug monitor while off/asleep (DEBUG_MONITOR) [N/y/?] (NEW)
Print battery consumption periodically while off/asleep (DEBUG_PRINT_BATTERY) [N/y/?] (NEW)
Print average latency after each state transition (DEBUG_PRINT_LATENCY) [N/y/?] (NEW)
Print the contents of Special Purpose Registers at boot (DEBUG_PRINT_SPRS) [N/y/?] (NEW)
Record steps of the suspend/resume process in the RTC (DEBUG_RECORD_STEPS) [N/y/?] (NEW)
Verify DRAM contents after controller resume (DEBUG_VERIFY_DRAM) [N/y/?] (NEW)
fatal: No names found, cannot describe anything.
  CPP     build/scp/arch/or1k/scp.ld.o
  CC      build/scp/arch/or1k/counter.o
  CC      build/scp/arch/or1k/exception.o
  AS      build/scp/arch/or1k/math.o
  AS      build/scp/arch/or1k/runtime.o
  AS      build/scp/arch/or1k/start.o
  CC      build/scp/common/debug.o
  CC      build/scp/common/delay.o
  CC      build/scp/common/device.o
  CC      build/scp/common/regulator_list.o
  CC      build/scp/common/scpi.o
  CC      build/scp/common/scpi_cmds.o
  CC      build/scp/common/simple_device.o
  CC      build/scp/common/system.o
  CC      build/scp/common/timeout.o
  CC      build/scp/drivers/clock/clock.o
  CC      build/scp/drivers/clock/ccu.o
  CC      build/scp/drivers/clock/ccu_helpers.o
  CC      build/scp/drivers/clock/r_ccu_common.o
  CC      build/scp/drivers/clock/sun50i-a64-ccu.o
  CC      build/scp/drivers/clock/sun8i-r-ccu.o
  CC      build/scp/drivers/counter/sun6i-a31-cnt64.o
  CC      build/scp/drivers/css/css.o
  CC      build/scp/drivers/css/css_default.o
  CC      build/scp/drivers/css/css_helpers.o
  CC      build/scp/drivers/css/css_power_state.o
  CC      build/scp/drivers/css/sun50i-a64-css.o
  CC      build/scp/drivers/dram/dram.o
  CC      build/scp/drivers/dram/sun8i-h3-dram.o
  CC      build/scp/drivers/gpio/gpio.o
  CC      build/scp/drivers/gpio/sunxi-gpio.o
  CC      build/scp/drivers/irq/irq.o
  CC      build/scp/drivers/irq/sun6i-a31-r-intc.o
  CC      build/scp/drivers/mfd/axp20x.o
  CC      build/scp/drivers/msgbox/msgbox.o
  CC      build/scp/drivers/msgbox/sunxi-msgbox.o
  CC      build/scp/drivers/pmic/pmic.o
  CC      build/scp/drivers/pmic/axp20x.o
  CC      build/scp/drivers/pmic/axp803.o
  CC      build/scp/drivers/regmap/regmap.o
  CC      build/scp/drivers/regmap/sunxi-rsb.o
  CC      build/scp/drivers/regulator/regulator.o
  CC      build/scp/drivers/regulator/axp20x.o
  CC      build/scp/drivers/regulator/axp803.o
  CC      build/scp/drivers/serial/serial.o
  CC      build/scp/drivers/serial/uart.o
  CC      build/scp/drivers/serial/sun50i-a64-uart.o
  CC      build/scp/drivers/watchdog/watchdog.o
  CC      build/scp/drivers/watchdog/sun6i-a31-wdt.o
  CC      build/scp/lib/bitfield.o
  AR      build/scp/lib.a
  LD      build/scp/scp.elf
  OBJCOPY build/scp/scp.bin
export SCP=/<crust>/build/scp/scp.bin
```

## SPL/U-Bootのビルド

```bash
$ git clone https://github.com/u-boot/u-boot.git --depth=1
$ cd u-boot
$ echo $BL31
<stf>/build/sun50i_a64/debug/bl31.bin
$ echo $SCP
<crust>/build/scp/scp.bin
$ export CROSS_COMPILE=aarch64-linux-gnu-
$ make pine64_plus_defconfig
  HOSTCC  scripts/basic/fixdep
  HOSTCC  scripts/kconfig/conf.o
  YACC    scripts/kconfig/zconf.tab.c
  LEX     scripts/kconfig/zconf.lex.c
  HOSTCC  scripts/kconfig/zconf.tab.o
  HOSTLD  scripts/kconfig/conf
#
# configuration written to .config
#
$ make
error: command 'swig' failed: No such file or directory
$ sudo apt install swig
$ which swig
/usr/bin/swig
$ make
scripts/kconfig/conf  --syncconfig Kconfig
  CFG     u-boot.cfg
...
  MKIMAGE spl/sunxi-spl.bin
  MKIMAGE u-boot.img
  COPY    u-boot.dtb
  MKIMAGE u-boot-dtb.img
  BINMAN  .binman_stamp
  OFCHK   .config
$ ls
api    common     dts       Kconfig      post        tools           u-boot-dtb.img    u-boot.srec
arch   config.mk  env       lib          README      u-boot          u-boot.dtb.out    u-boot-sunxi-with-spl.bin
board  configs    examples  Licenses     scripts     u-boot.bin      u-boot.img        u-boot-sunxi-with-spl.fit.fit
boot   disk       fs        MAINTAINERS  spl         u-boot.cfg      u-boot.lds        u-boot-sunxi-with-spl.fit.itb
build  doc        include   Makefile     System.map  u-boot.dtb      u-boot.map        u-boot-sunxi-with-spl.map
cmd    drivers    Kbuild    net          test        u-boot-dtb.bin  u-boot-nodtb.bin  u-boot.sym
$ ls spl
arch   boot  common  drivers  env  include  sunxi-spl.bin  u-boot-spl      u-boot-spl.lds  u-boot-spl-nodtb.bin
board  cmd   disk    dts      fs   lib      u-boot.cfg     u-boot-spl.bin  u-boot-spl.map  u-boot-spl.sym
```

[u-bootのmakeログ](uboot_make.log)

# Linuxのビルド

```bash
$ wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.1.47.tar.xz
$ tar xf linux-6.1.47.tar.xz && cd linux-6.1.47
$ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make defconfig
  HOSTCC  scripts/basic/fixdep
  HOSTCC  scripts/kconfig/conf.o
  HOSTCC  scripts/kconfig/confdata.o
  HOSTCC  scripts/kconfig/expr.o
  LEX     scripts/kconfig/lexer.lex.c
  YACC    scripts/kconfig/parser.tab.[ch]
  HOSTCC  scripts/kconfig/lexer.lex.o
  HOSTCC  scripts/kconfig/menu.o
  HOSTCC  scripts/kconfig/parser.tab.o
  HOSTCC  scripts/kconfig/preprocess.o
  HOSTCC  scripts/kconfig/symbol.o
  HOSTCC  scripts/kconfig/util.o
  HOSTLD  scripts/kconfig/conf
*** Default configuration is based on 'defconfig'
#
# configuration written to .config
#
$ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make menuconfig
$ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make -j2 Image
  SYNC    include/config/auto.conf.cmd
  WRAP    arch/arm64/include/generated/uapi/asm/kvm_para.h
  WRAP    arch/arm64/include/generated/uapi/asm/errno.h
...
  LD      vmlinux
  NM      System.map
  SORTTAB vmlinux
  OBJCOPY arch/arm64/boot/Image
$ ls -l arch/arm64/boot/
total 20672
drwxrwxr-x 35 vagrant vagrant     4096 Aug 24 00:52 dts
-rw-rw-r--  1 vagrant vagrant 21442568 Aug 24 15:18 Image
-rwxrwxr-x  1 vagrant vagrant      962 Aug 24 00:52 install.sh
-rw-rw-r--  1 vagrant vagrant     1198 Aug 24 00:52 Makefile
$ mkdir -p <HOME>/pine64_linux/usr
$ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make -j2 dtbs
$ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make -j2 modules
$ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=<HOME>/pine64_linux/usr make modules modules_install
$ ARCH=arm64 INSTALL_HDR_PATH=<HOME>/pine64_linux/usr make headers_install
```

# SDカードの初期化

```bash
$ diskutil list

/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *31.3 GB    disk2
   1:             Windows_FAT_32                         16.8 MB    disk2s1
   2:                      Linux                         31.2 GB    disk2s2

$ diskutil unmountDisk /dev/disk2
Unmount of all volumes on disk2 was successful
$ sudo dd if=/dev/zero of=${card} bs=1024*1024 count=1                # 1. 先頭 1MiBを0クリア
$ sudo dd if=u-boot-sunxi-with-spl.bin of=/dev/disk2 bs=1024 seek=8   # 2. 先頭から 8KiBにu-bootをインストール
$ # blockdev --rereadpt ${card}                                       # Macにはblockdevコマンドがないので1.でパーティションテーブルも0クリア
$ cat <<EOT | sudo sfdisk ${card}                                     # 3. パーティションを3つ (1MiB, 16Mib, 残りすべて)作成
1M,16M,c
,,L
EOT
$ sudo mkfs.vfat /dev/disk2s1                                         # 4. パーティション1にFAT fsを作成
mkfs.fat 4.2 (2021-01-31)
$ sudo /usr/local/opt/e2fsprogs/sbin/mkfs.ext4 /dev/disk2s2           # 5. パーティション2にext4 faを作成
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 7629056 4k blocks and 1908736 inodes
Filesystem UUID: de3dff1e-edce-4c80-9fc5-df3a202647c4
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
$
```

# U-Bootを実行

```bash
Welcome to minicom 2.8

OPTIONS:
Compiled on Jan  4 2021, 00:04:46.
Port /dev/cu.usbserial-AI057C9L, 17:43:57
Using character set conversion

Press Meta-Z for help on special keys


U-Boot SPL 2023.10-rc3-g291055ef (Aug 24 2023 - 12:30:35 +0900)
DRAM: 1024 MiB
Trying to boot from MMC1
NOTICE:  BL31: v2.9(debug):f56da5d
NOTICE:  BL31: Built : 11:16:37, Aug 24 2023
NOTICE:  BL31: Detected Allwinner A64/H64/R18 SoC (1689)
NOTICE:  BL31: Found U-Boot DTB at 0x20a4460, model: Pine64+
INFO:    ARM GICv2 driver initialized
INFO:    Configuring SPC Controller
INFO:    PMIC: Probing AXP803 on RSB
INFO:    PMIC: dcdc1 voltage: 3.300V
INFO:    PMIC: dcdc5 voltage: 1.360V
INFO:    PMIC: dcdc6 voltage: 1.100V
INFO:    PMIC: dldo1 voltage: 3.300V
INFO:    PMIC: dldo2 voltage: 3.300V
INFO:    PMIC: dldo4 voltage: 3.300V
INFO:    PMIC: fldo1 voltage: 1.200V
INFO:    PMIC: Enabling DC SW
INFO:    BL31: Platform setup done
INFO:    BL31: Initializing runtime services
INFO:    BL31: cortex_a53: CPU workaround for 843419 was applied
INFO:    BL31: cortex_a53: CPU workaround for 855873 was applied
INFO:    BL31: cortex_a53: CPU workaround for 1530924 was applied
SCP/INF: Crust v0.5.10000
INFO:    PSCI: Suspend is available via SCPI
INFO:    BL31: Preparing for EL3 exit to normal world
INFO:    Entry point address = 0x4a000000
INFO:    SPSR = 0x3c9


U-Boot 2023.10-rc3-g291055ef (Aug 24 2023 - 12:30:35 +0900) Allwinner Technology

CPU:   Allwinner A64 (SUN50I)
Model: Pine64+
DRAM:  1 GiB
Core:  68 devices, 20 uclasses, devicetree: separate
WDT:   Not starting watchdog@1c20ca0
MMC:   mmc@1c0f000: 0
Loading Environment from FAT... Unable to read "uboot.env" from mmc0:1...
In:    serial,usbkbd
Out:   serial,vidconsole
Err:   serial,vidconsole
Net:   eth0: ethernet@1c30000
starting USB...
Bus usb@1c1a000: USB EHCI 1.00
Bus usb@1c1a400: USB OHCI 1.0
Bus usb@1c1b000: USB EHCI 1.00
Bus usb@1c1b400: USB OHCI 1.0
scanning bus usb@1c1a000 for devices... 1 USB Device(s) found
scanning bus usb@1c1a400 for devices... 1 USB Device(s) found
scanning bus usb@1c1b000 for devices... 1 USB Device(s) found
scanning bus usb@1c1b400 for devices... 1 USB Device(s) found
       scanning usb for storage devices... 0 Storage Device(s) found
Hit any key to stop autoboot:  0
=> bootp 0x42000000 192.168.10.103:serial_test.img                    # TFTP経由でプログラムをロード
ethernet@1c30000 Waiting for PHY auto negotiation to complete....... done
BOOTP broadcast 1
BOOTP broadcast 2
DHCP client bound to address 192.168.10.121 (256 ms)
Using ethernet@1c30000 device
TFTP from server 192.168.10.103; our IP address is 192.168.10.121
Filename 'serial_test.img'.
Load address: 0x42000000
Loading: #
         2 KiB/s
done
Bytes transferred = 20 (14 hex)
=> printenv bootcmd
bootcmd=run distro_bootcmd
=> editenv bootcmd
edit: bootp 0x42000000 192.168.10.103:serial_test.img
=> printenv bootcmd
bootcmd=bootp 0x42000000 192.168.10.103:serial_test.img
=> saveenv
Saving Environment to FAT... OK
=> help mmc
mmc - MMC sub system

Usage:
mmc info - display info of the current MMC device
mmc read addr blk# cnt
mmc write addr blk# cnt
mmc erase blk# cnt
mmc rescan [mode]
mmc part - lists available partition on current mmc device
mmc dev [dev] [part] [mode] - show or set current mmc device [partition] and set mode
  - the required speed mode is passed as the index from the following list
    [MMC_LEGACY, MMC_HS, SD_HS, MMC_HS_52, MMC_DDR_52, UHS_SDR12, UHS_SDR25,
    UHS_SDR50, UHS_DDR50, UHS_SDR104, MMC_HS_200, MMC_HS_400, MMC_HS_400_ES]
mmc list - lists available devices
mmc wp [PART] - power on write protect boot partitions
  arguments:
   PART - [0|1]
       : 0 - first boot partition, 1 - second boot partition
         if not assigned, write protect all boot partitions
mmc hwpartition <USER> <GP> <MODE> - does hardware partitioning
  arguments (sizes in 512-byte blocks):
   USER - <user> <enh> <start> <cnt> <wrrel> <{on|off}>
        : sets user data area attributes
   GP - <{gp1|gp2|gp3|gp4}> <cnt> <enh> <wrrel> <{on|off}>
        : general purpose partition
   MODE - <{check|set|complete}>
        : mode, complete set partitioning completed
  WARNING: Partitioning is a write-once setting once it is set to complete.
  Power cycling is required to initialize partitions after set to complete.
mmc setdsr <value> - set DSR register value

=> mmc part

Partition Map for MMC device 0  --   Partition Type: DOS

Part    Start Sector    Num Sectors     UUID            Type
  1     2048            32768           c6d66597-01     0c
  2     34816           61032448        c6d66597-02     83
=> fatls
fatls - list files in a directory (default /)

Usage:
fatls <interface> [<dev[:part]>] [directory]
    - list files from 'dev' on 'interface' in a 'directory'
=> fatls mmc 0:1
    65536   uboot.env

1 file(s), 0 dir(s)

=> help ext4ls
ext4ls - list files in a directory (default /)

Usage:
ext4ls <interface> <dev[:part]> [directory]
    - list files from 'dev' on 'interface' in a 'directory'
=> ext4ls mmc 0:2
<DIR>       4096 .
<DIR>       4096 ..
<DIR>      16384 lost+found
=> help fatinfo
fatinfo - print information about filesystem

Usage:
fatinfo <interface> [<dev[:part]>]
    - print information about filesystem from 'dev' on 'interface'
=> fatinfo mmc 0:1
Interface:  mmc
  Device 0: Vendor: Man 000027 Snr 2d2a6b01 Rev: 3.9 Prod: SD32G`
            Type: Removable Hard Disk
            Capacity: 29818.0 MB = 29.1 GB (61067264 x 512)
Filesystem: FAT16 "NO NAME    "
=> printenv bootcmd
bootcmd=bootp 0x42000000 192.168.10.103:serial_test.img
=> go 0x42000000                                                    # tftp経由でロードしたプログラムを実行
## Starting application at 0x42000000 ...
UUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU?
# 電源オフ

# 電源オン
...
Hit any key to stop autoboot:  0    # 自動的にbootcmdが実行されている
ethernet@1c30000 Waiting for PHY auto negotiation to complete....... done
BOOTP broadcast 1
BOOTP broadcast 2
DHCP client bound to address 192.168.10.121 (254 ms)
Using ethernet@1c30000 device
TFTP from server 192.168.10.103; our IP address is 192.168.10.121
Filename 'serial_test.img'.
Load address: 0x42000000
Loading: #
         2 KiB/s
done
Bytes transferred = 20 (14 hex)
=> go 0x42000000
## Starting application at 0x42000000 ...
UUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU
```

# Armbianを探る

```bash
$ fdisk -l Armbian_23.8.1_Pine64_bookworm_current_6.1.47.img
ディスク Armbian_23.8.1_Pine64_bookworm_current_6.1.47.img: 1.75 GiB, 1879048192 バイト, 3670016 セクタ
単位: セクタ (1 * 512 = 512 バイト)
セクタサイズ (論理 / 物理): 512 バイト / 512 バイト
I/O サイズ (最小 / 推奨): 512 バイト / 512 バイト
ディスクラベルのタイプ: dos
ディスク識別子: 0x0072e104

デバイス                                           起動 開始位置 終了位置  セクタ サイズ Id タイプ
Armbian_23.8.1_Pine64_bookworm_current_6.1.47.img1          8192  3670015 3661824   1.7G 83 Linux

$ sudo mount -oloop,offset=4194304 Armbian_23.8.1_Pine64_bookworm_current_6.1.47.img /mnt
$ ls /mnt
bin         dev   initrd.img      lost+found  opt   run      srv  usr      vmlinuz.old
boot        etc   initrd.img.old  media       proc  sbin     sys  var
core.14218  home  lib             mnt         root  selinux  tmp  vmlinuz
$ ls /mnt/boot
armbianEnv.txt                  config-6.1.47-current-sunxi64      System.map-6.1.47-current-sunxi64
armbian_first_run.txt.template  dtb                                uInitrd
boot.bmp                        dtb-6.1.47-current-sunxi64         uInitrd-6.1.47-current-sunxi64
boot.cmd                        Image                              vmlinuz-6.1.47-current-sunxi64
boot.scr                        initrd.img-6.1.47-current-sunxi64
$ ls -l /mnt/boot
total 61288
-rw-r--r-- 1 1014 1014      155 Aug 31 23:35 armbianEnv.txt
-rw-r--r-- 1 root root     1536 Aug 31 23:34 armbian_first_run.txt.template
-rw-r--r-- 1 root root   230454 Aug 31 23:33 boot.bmp
-rw-r--r-- 1 1014 1014     3187 Aug 31 23:28 boot.cmd
-rw-rw-r-- 1 root root     3259 Aug 31 23:35 boot.scr
-rw-r--r-- 1 root root   207780 Aug 30 11:05 config-6.1.47-current-sunxi64
lrwxrwxrwx 1 root root       26 Aug 31 23:32 dtb -> dtb-6.1.47-current-sunxi64
drwxr-xr-x 3 root root     4096 Aug 31 23:32 dtb-6.1.47-current-sunxi64
lrwxrwxrwx 1 root root       30 Aug 31 23:32 Image -> vmlinuz-6.1.47-current-sunxi64
-rw-r--r-- 1 root root 18215873 Aug 31 23:38 initrd.img-6.1.47-current-sunxi64
-rw-r--r-- 1 root root  3464899 Aug 30 11:05 System.map-6.1.47-current-sunxi64
lrwxrwxrwx 1 root root       30 Aug 31 23:38 uInitrd -> uInitrd-6.1.47-current-sunxi64
-rw-r--r-- 1 root root 18215937 Aug 31 23:38 uInitrd-6.1.47-current-sunxi64
-rw-r--r-- 1 root root 22390792 Aug 30 11:05 vmlinuz-6.1.47-current-sunxi64
$ cat /mnt/boot/armbianEnv.txt
verbosity=1
bootlogo=false
console=both
disp_mode=1920x1080p60
overlay_prefix=sun50i-a64
rootdev=UUID=07188c97-2963-45ea-a263-1bc1f5530078
rootfstype=ext4
$ cat /mnt/boot/boot.cmd
# DO NOT EDIT THIS FILE
#
# Please edit /boot/armbianEnv.txt to set supported parameters
#

# default values
setenv load_addr "0x45000000"
setenv overlay_error "false"
setenv rootdev "/dev/mmcblk0p1"
setenv verbosity "1"
setenv rootfstype "ext4"
setenv console "both"
setenv docker_optimizations "on"
setenv bootlogo "false"

# Print boot source
itest.b *0x10028 == 0x00 && echo "U-boot loaded from SD"
itest.b *0x10028 == 0x02 && echo "U-boot loaded from eMMC or secondary SD"
itest.b *0x10028 == 0x03 && echo "U-boot loaded from SPI"

echo "Boot script loaded from ${devtype}"

if test -e ${devtype} ${devnum} ${prefix}armbianEnv.txt; then
	load ${devtype} ${devnum} ${load_addr} ${prefix}armbianEnv.txt
	env import -t ${load_addr} ${filesize}
fi

if test "${console}" = "display" || test "${console}" = "both"; then setenv consoleargs "console=ttyS0,115200 console=tty1"; fi
if test "${console}" = "serial"; then setenv consoleargs "console=ttyS0,115200"; fi
if test "${bootlogo}" = "true"; then
	setenv consoleargs "splash plymouth.ignore-serial-consoles ${consoleargs}"
else
	setenv consoleargs "splash=verbose ${consoleargs}"
fi

# get PARTUUID of first partition on SD/eMMC it was loaded from
# mmc 0 is always mapped to device u-boot (2016.09+) was loaded from
if test "${devtype}" = "mmc"; then part uuid mmc 0:1 partuuid; fi

setenv bootargs "root=${rootdev} rootwait rootfstype=${rootfstype} ${consoleargs} consoleblank=0 loglevel=${verbosity} ubootpart=${partuuid} usb-storage.quirks=${usbstoragequirks} ${extraargs} ${extraboardargs}"

if test "${docker_optimizations}" = "on"; then setenv bootargs "${bootargs} cgroup_enable=memory swapaccount=1"; fi

load ${devtype} ${devnum} ${fdt_addr_r} ${prefix}dtb/${fdtfile}
fdt addr ${fdt_addr_r}
fdt resize 65536
for overlay_file in ${overlays}; do
	if load ${devtype} ${devnum} ${load_addr} ${prefix}dtb/allwinner/overlay/${overlay_prefix}-${overlay_file}.dtbo; then
		echo "Applying kernel provided DT overlay ${overlay_prefix}-${overlay_file}.dtbo"
		fdt apply ${load_addr} || setenv overlay_error "true"
	fi
done
for overlay_file in ${user_overlays}; do
	if load ${devtype} ${devnum} ${load_addr} ${prefix}overlay-user/${overlay_file}.dtbo; then
		echo "Applying user provided DT overlay ${overlay_file}.dtbo"
		fdt apply ${load_addr} || setenv overlay_error "true"
	fi
done
if test "${overlay_error}" = "true"; then
	echo "Error applying DT overlays, restoring original DT"
	load ${devtype} ${devnum} ${fdt_addr_r} ${prefix}dtb/${fdtfile}
else
	if load ${devtype} ${devnum} ${load_addr} ${prefix}dtb/allwinner/overlay/${overlay_prefix}-fixup.scr; then
		echo "Applying kernel provided DT fixup script (${overlay_prefix}-fixup.scr)"
		source ${load_addr}
	fi
	if test -e ${devtype} ${devnum} ${prefix}fixup.scr; then
		load ${devtype} ${devnum} ${load_addr} ${prefix}fixup.scr
		echo "Applying user provided fixup script (fixup.scr)"
		source ${load_addr}
	fi
fi

load ${devtype} ${devnum} ${ramdisk_addr_r} ${prefix}uInitrd
load ${devtype} ${devnum} ${kernel_addr_r} ${prefix}Image

booti ${kernel_addr_r} ${ramdisk_addr_r} ${fdt_addr_r}

# Recompile with:
# mkimage -C none -A arm -T script -d /boot/boot.cmd /boot/boot.scr

```
