# sunxi-felコマンドの実行

```bash
$ lsusb
Bus 020 Device 011: ID 1f3a:efe8 1f3a Composite Device
$ sudo ./sunxi-fel version      # current verのsunxi-felははじめからa64を認識する
Password:
AWUSBFEX soc=00001689(A64) 00000001 ver=0001 44 08 scratchpad=00007e00 00000000 00000000
$ sudo ./sunxi-fel hexdump 0x0000 128                   # この場合の1st opは hexdump
00000000: 08 00 00 ea 06 00 00 ea 05 00 00 ea 04 00 00 ea  ................
00000010: 03 00 00 ea 02 00 00 ea 11 00 00 ea 00 00 00 ea  ................
00000020: 13 00 00 ea fe ff ff ea 01 00 a0 e3 00 10 a0 e3  ................
00000030: 00 20 a0 e3 00 30 a0 e3 00 40 a0 e3 00 50 a0 e3  . ...0...@...P..
00000040: 00 60 a0 e3 00 70 a0 e3 00 80 a0 e3 00 90 a0 e3  .`...p..........
00000050: 00 a0 a0 e3 00 b0 a0 e3 00 c0 a0 e3 00 d0 a0 e3  ................
00000060: e8 f0 9f e5 04 e0 4e e2 ff 5f 2d e9 1f 07 00 eb  ......N.._-.....
00000070: ff 9f fd e8 d2 20 a0 e3 02 f0 21 e1 d0 d0 9f e5  ..... ....!.....
$ sudo ./sunxi-fel dump 0x0000 128 > memory-dump.bin    # この場合の1st opは dump
$ ls -l memory-dump.bin
-rw-r--r-- 1 dspace staff 128  9  2 18:19 memory-dump.bin
$ xxd memory-dump.bin
00000000: 0800 00ea 0600 00ea 0500 00ea 0400 00ea  ................
00000010: 0300 00ea 0200 00ea 1100 00ea 0000 00ea  ................
00000020: 1300 00ea feff ffea 0100 a0e3 0010 a0e3  ................
00000030: 0020 a0e3 0030 a0e3 0040 a0e3 0050 a0e3  . ...0...@...P..
00000040: 0060 a0e3 0070 a0e3 0080 a0e3 0090 a0e3  .`...p..........
00000050: 00a0 a0e3 00b0 a0e3 00c0 a0e3 00d0 a0e3  ................
00000060: e8f0 9fe5 04e0 4ee2 ff5f 2de9 1f07 00eb  ......N.._-.....
00000070: ff9f fde8 d220 a0e3 02f0 21e1 d0d0 9fe5  ..... ....!.....
$ sudo ./sunxi-fel hexdump 0x2c00 128
00002c00: 07 00 00 ea 07 00 00 ea 65 47 4f 4e 2e 42 52 4d  ........eGON.BRM
00002c10: 24 00 00 00 31 31 30 30 31 31 30 30 31 36 33 33  $...110011001633
00002c20: 00 00 00 00 00 00 00 ea 01 00 00 ea 00 60 a0 e3  .............`..
00002c30: 03 00 00 ea 5c 60 a0 e3 0e 00 00 ea e8 01 9f e5  ....\`..........
00002c40: 00 f0 90 e5 b0 0f 10 ee 03 10 00 e2 00 00 51 e3  ..............Q.
00002c50: f9 ff ff 1a ff 1c 00 e2 00 00 51 e3 f6 ff ff 1a  ..........Q.....
00002c60: c8 11 9f e5 c8 21 9f e5 00 30 91 e5 03 00 52 e1  .....!...0....R.
00002c70: 00 00 00 1a f0 ff ff ea 50 00 a0 e3 01 00 50 e2  ........P.....P.
$ sudo ./sunxi-fel dump 0x2c00 32768 > pine64-a64-brom.bin
dspace@mini:~/pine64/sunxi-tools$ head -c 128 pine64-a64-brom.bin | xxd
00000000: 0700 00ea 0700 00ea 6547 4f4e 2e42 524d  ........eGON.BRM
00000010: 2400 0000 3131 3030 3131 3030 3136 3333  $...110011001633
00000020: 0000 0000 0000 00ea 0100 00ea 0060 a0e3  .............`..
00000030: 0300 00ea 5c60 a0e3 0e00 00ea e801 9fe5  ....\`..........
00000040: 00f0 90e5 b00f 10ee 0310 00e2 0000 51e3  ..............Q.
00000050: f9ff ff1a ff1c 00e2 0000 51e3 f6ff ff1a  ..........Q.....
00000060: c811 9fe5 c821 9fe5 0030 91e5 0300 52e1  .....!...0....R.
00000070: 0000 001a f0ff ffea 5000 a0e3 0100 50e2  ........P.....P.
```

## pine64の電源とUSBケーブル接続

- pine64の電源をOFFでUSBケーブルを繋いでもpine64は見えない。
- pine64の電源をONにして初めてpine64が見えるようになる（USBデバイスモード）。
- pine64の電源をOFFにしても電源LEDは消えない。
- USBケーブルを外して初めて電源LEDが消える。

## 最新のu-bootを使う

- 端末1でminicom
- 端末2で`sunxi-fel uboot u-boot-sunxi-with-spl.bin`

### ログ

```
U-Boot SPL 2023.10-rc3-g291055ef (Aug 24 2023 - 12:30:35 +0900)
DRAM: 1024 MiB
Trying to boot from FEL
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
Loading Environment from FAT... MMC: no card present
** Bad device specification mmc 0 **
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
MMC: no card present

Device 0: unknown device
ethernet@1c30000 Waiting for PHY auto negotiation to complete........ done
BOOTP broadcast 1
BOOTP broadcast 2
DHCP client bound to address 192.168.10.121 (255 ms)
*** ERROR: `serverip' not set
Cannot autoload with TFTPGET
missing environment variable: pxeuuid
Retrieving file: pxelinux.cfg/01-02-ba-cd-42-77-0b
*** ERROR: `serverip' not set
Retrieving file: pxelinux.cfg/C0A80A79
*** ERROR: `serverip' not set
Retrieving file: pxelinux.cfg/C0A80A7
*** ERROR: `serverip' not set
Retrieving file: pxelinux.cfg/C0A80A
*** ERROR: `serverip' not set
Retrieving file: pxelinux.cfg/C0A80
*** ERROR: `serverip' not set
Retrieving file: pxelinux.cfg/C0A8
*** ERROR: `serverip' not set
Retrieving file: pxelinux.cfg/C0A
*** ERROR: `serverip' not set
Retrieving file: pxelinux.cfg/C0
*** ERROR: `serverip' not set
Retrieving file: pxelinux.cfg/C
*** ERROR: `serverip' not set
Retrieving file: pxelinux.cfg/default-arm-sunxi-sunxi
*** ERROR: `serverip' not set
Retrieving file: pxelinux.cfg/default-arm-sunxi
*** ERROR: `serverip' not set
Retrieving file: pxelinux.cfg/default-arm
*** ERROR: `serverip' not set
Retrieving file: pxelinux.cfg/default
*** ERROR: `serverip' not set
Config file not found
BOOTP broadcast 1
DHCP client bound to address 192.168.10.121 (31 ms)
*** ERROR: `serverip' not set
Cannot autoload with TFTPGET
BOOTP broadcast 1
DHCP client bound to address 192.168.10.121 (27 ms)
*** ERROR: `serverip' not set
Cannot autoload with TFTPGET
=> printenv
arch=arm
baudrate=115200
board=sunxi
board_name=sunxi
boot_a_script=load ${devtype} ${devnum}:${distro_bootpart} ${scriptaddr} ${prefix}${script}; source}
boot_efi_binary=load ${devtype} ${devnum}:${distro_bootpart} ${kernel_addr_r} efi/boot/bootaa64.efii
boot_efi_bootmgr=if fdt addr -q ${fdt_addr_r}; then bootefi bootmgr ${fdt_addr_r};else bootefi booti
boot_extlinux=sysboot ${devtype} ${devnum}:${distro_bootpart} any ${scriptaddr} ${prefix}${boot_sys}
boot_net_usb_start=usb start
boot_prefixes=/ /boot/
boot_script_dhcp=boot.scr.uimg
boot_scripts=boot.scr.uimg boot.scr
boot_syslinux_conf=extlinux/extlinux.conf
boot_targets=fel mmc0 usb0 pxe dhcp
bootcmd=run distro_bootcmd
bootcmd_dhcp=devtype=dhcp; run boot_net_usb_start; if dhcp ${scriptaddr} ${boot_script_dhcp}; then ;
bootcmd_fel=if test -n ${fel_booted} && test -n ${fel_scriptaddr}; then echo '(FEL boot)'; source $i
bootcmd_mmc0=devnum=0; run mmc_boot
bootcmd_pxe=run boot_net_usb_start; dhcp; if pxe get; then pxe boot; fi
bootcmd_usb0=devnum=0; run usb_boot
bootdelay=2
bootfile=boot.scr.uimg
bootm_size=0xa000000
console=ttyS0,115200
cpu=armv8
dfu_alt_info_ram=kernel ram 0x40080000 0x1000000;fdt ram 0x4FA00000 0x100000;ramdisk ram 0x4FF000000
distro_bootcmd=for target in ${boot_targets}; do run bootcmd_${target}; done
efi_dtb_prefixes=/ /dtb/ /dtb/current/
ethact=ethernet@1c30000
ethaddr=02:ba:cd:42:77:0b
fdt_addr_r=0x4FA00000
fdtcontroladdr=79f271f0
fdtfile=allwinner/sun50i-a64-pine64-plus.dtb
fdtoverlay_addr_r=0x4FE00000
fel_booted=1
kernel_addr_r=0x40080000
kernel_comp_addr_r=0x44000000
kernel_comp_size=0xb000000
load_efi_dtb=load ${devtype} ${devnum}:${distro_bootpart} ${fdt_addr_r} ${prefix}${efi_fdtfile}
loadaddr=0x42000000
mmc_boot=if mmc dev ${devnum}; then devtype=mmc; run scan_dev_for_boot_part; fi
partitions=name=loader1,start=8k,size=32k,uuid=${uuid_gpt_loader1};name=loader2,size=984k,uuid=${uu;
preboot=usb start
pxefile_addr_r=0x4FD00000
ramdisk_addr_r=0x4FF00000
scan_dev_for_boot=echo Scanning ${devtype} ${devnum}:${distro_bootpart}...; for prefix in ${boot_pr;
scan_dev_for_boot_part=part list ${devtype} ${devnum} -bootable devplist; env exists devplist || set
scan_dev_for_efi=setenv efi_fdtfile ${fdtfile}; for prefix in ${efi_dtb_prefixes}; do if test -e ${e
scan_dev_for_extlinux=if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefix}${boot_syslinux_ci
scan_dev_for_scripts=for script in ${boot_scripts}; do if test -e ${devtype} ${devnum}:${distro_booe
scriptaddr=0x4FC00000
serial#=92c000bacd42770b
soc=sunxi
stderr=serial,vidconsole
stdin=serial,usbkbd
stdout=serial,vidconsole
usb_boot=usb start; if usb dev ${devnum}; then devtype=usb; run scan_dev_for_boot_part; fi
uuid_gpt_esp=c12a7328-f81f-11d2-ba4b-00a0c93ec93b
uuid_gpt_system=b921b045-1df0-41c3-af44-4c6f280d3fae

Environment size: 4515/65532 bytes
=> help
?         - alias for 'help'
base      - print or set address offset
bdinfo    - print Board Info structure
blkcache  - block cache diagnostics and control
boot      - boot default, i.e., run 'bootcmd'
bootd     - boot default, i.e., run 'bootcmd'
bootefi   - Boots an EFI payload from memory
bootelf   - Boot from an ELF image in memory
bootflow  - Boot flows
booti     - boot Linux kernel 'Image' format from memory
bootm     - boot application image from memory
bootp     - boot image via network using BOOTP/TFTP protocol
bootvx    - Boot vxWorks from an ELF image
cls       - clear screen
cmp       - memory compare
coninfo   - print console devices and information
cp        - memory copy
crc32     - checksum calculation
cyclic    - Cyclic
dhcp      - boot image via network using DHCP/TFTP protocol
dm        - Driver model low level access
echo      - echo args to console
editenv   - edit environment variable
eficonfig - provide menu-driven UEFI variable maintenance interface
env       - environment handling commands
exit      - exit script
ext2load  - load binary file from a Ext2 filesystem
ext2ls    - list files in a directory (default /)
ext4load  - load binary file from a Ext4 filesystem
ext4ls    - list files in a directory (default /)
ext4size  - determine a file's size
false     - do nothing, unsuccessfully
fatinfo   - print information about filesystem
fatload   - load binary file from a dos filesystem
fatls     - list files in a directory (default /)
fatmkdir  - create a directory
fatrm     - delete a file
fatsize   - determine a file's size
fatwrite  - write file into a dos filesystem
fdt       - flattened device tree utility commands
fstype    - Look up a filesystem type
fstypes   - List supported filesystem types
go        - start application at address 'addr'
gpio      - query and control gpio pins
gpt       - GUID Partition Table
gzwrite   - unzip and write memory to block device
help      - print command description/usage
iminfo    - print header information for application image
imxtract  - extract a part of a multi-image
itest     - return true/false on integer compare
lcdputs   - print string on video framebuffer
ln        - Create a symbolic link
load      - load binary file from a filesystem
loadb     - load binary file over serial line (kermit mode)
loads     - load S-Record file over serial line
loadx     - load binary file over serial line (xmodem mode)
loady     - load binary file over serial line (ymodem mode)
loop      - infinite loop on address range
ls        - list files in a directory (default /)
lzmadec   - lzma uncompress a memory region
md        - memory display
mdio      - MDIO utility commands
mii       - MII utility commands
mm        - memory modify (auto-incrementing address)
mmc       - MMC sub system
mmcinfo   - display MMC info
mw        - memory write (fill)
net       - NET sub-system
nm        - memory modify (constant address)
panic     - Panic with optional message
part      - disk partition related commands
ping      - send ICMP ECHO_REQUEST to network host
pinmux    - show pin-controller muxing
printenv  - print environment variables
pxe       - commands to get and boot from pxe files
To use IPv6 add -ipv6 parameter
random    - fill memory with random pattern
reset     - Perform RESET of the CPU
run       - run commands in an environment variable
save      - save file to a filesystem
saveenv   - save environment variables to persistent storage
setcurs   - set cursor position within screen
setenv    - set environment variables
setexpr   - set environment variable as the result of eval expression
showvar   - print local hushshell variables
size      - determine a file's size
sleep     - delay execution for some time
source    - run script from memory
sysboot   - command to get and boot from syslinux files
test      - minimal test like /bin/sh
tftpboot  - load file via network using TFTP protocol
true      - do nothing, successfully
unlz4     - lz4 uncompress a memory region
unzip     - unzip a memory region
usb       - USB sub-system
usbboot   - boot from USB device
version   - print monitor, compiler and linker version
=> version
U-Boot 2023.10-rc3-g291055ef (Aug 24 2023 - 12:30:35 +0900) Allwinner Technology

aarch64-linux-gnu-gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
GNU ld (GNU Binutils for Ubuntu) 2.38
=> bdinfo
boot_params = 0x0000000040000100
DRAM bank   = 0x0000000000000000
-> start    = 0x0000000040000000
-> size     = 0x0000000040000000
flashstart  = 0x0000000000000000
flashsize   = 0x0000000000000000
flashoffset = 0x0000000000000000
baudrate    = 115200 bps
relocaddr   = 0x000000007df51000
reloc off   = 0x0000000033f51000
Build       = 64-bit
current eth = ethernet@1c30000
ethaddr     = 02:ba:cd:42:77:0b
IP addr     = <NULL>
fdt_blob    = 0x0000000079f271f0
new_fdt     = 0x0000000079f271f0
fdt_size    = 0x0000000000009ba0
Video       = sunxi_de2 inactive
lmb_dump_all:
 memory.cnt = 0x1 / max = 0x10
 memory[0]      [0x40000000-0x7fffffff], 0x40000000 bytes flags: 0
 reserved.cnt = 0x2 / max = 0x10
 reserved[0]    [0x78f23000-0x7fffffff], 0x070dd000 bytes flags: 0
 reserved[1]    [0x79f22bc0-0x7fffffff], 0x060dd440 bytes flags: 0
devicetree  = separate
serial addr = 0x0000000001c28000
 width      = 0x0000000000000004
 shift      = 0x0000000000000002
 offset     = 0x0000000000000000
 clock      = 0x00000000016e3600
arch_number = 0x0000000000000000
TLB addr    = 0x000000007fff0000
irq_sp      = 0x0000000079f271e0
sp start    = 0x0000000079f271e0
Early malloc usage: 740 / 2000

```

## 実行

- 端末1

```bash
$ ../sunxi-tools/sunxi-fel -v uboot u-boot-sunxi-with-spl.bin \
> write 0x40080000 Image \
> write 0x4fa00000 sun50i-a64-pine64-plus.dtb \
> write 0x4fc00000 boot.scr \
> write 0x4ff00000 initramfs.cpio.gz
found DT name in SPL header: sun50i-a64-pine64-plus
Stack pointers: sp_irq=0x00012000, sp=0x00015E08
MMU is not enabled by BROM
=> Executing the SPL... done.
loading image "ARM Trusted Firmware" (45277 bytes) to 0x44000
loading image "SCP firmware" (13644 bytes) to 0x50000
loading image "U-Boot (64-bit)" (672864 bytes) to 0x4a000000
loading DTB "sun50i-a64-pine64-plus" (39840 bytes)
Starting U-Boot (0x00044000).
Store entry point 0x00044000 to RVBAR 0x017000A0, and request warm reset with RMR mode 3... done.
```

- 端末2

```bash
U-Boot SPL 2023.10-rc3-g291055ef (Aug 24 2023 - 12:30:35 +0900)
DRAM: 1024 MiB
Trying to boot from FEL
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
Loading Environment from FAT... MMC: no card present
** Bad device specification mmc 0 **
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
MMC: no card present

Device 0: unknown device
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
missing environment variable: pxeuuid
Retrieving file: pxelinux.cfg/01-02-ba-cd-42-77-0b
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
Retrieving file: pxelinux.cfg/00000000
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
Retrieving file: pxelinux.cfg/0000000
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
Retrieving file: pxelinux.cfg/000000
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
Retrieving file: pxelinux.cfg/00000
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
Retrieving file: pxelinux.cfg/0000
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
Retrieving file: pxelinux.cfg/000
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
Retrieving file: pxelinux.cfg/00
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
Retrieving file: pxelinux.cfg/0
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
Retrieving file: pxelinux.cfg/default-arm-sunxi-sunxi
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
Retrieving file: pxelinux.cfg/default-arm-sunxi
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
Retrieving file: pxelinux.cfg/default-arm
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
Retrieving file: pxelinux.cfg/default
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
Config file not found
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
ethernet@1c30000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
=>

=> printenv
arch=arm
baudrate=115200
board=sunxi
board_name=sunxi
boot_a_script=load ${devtype} ${devnum}:${distro_bootpart} ${scriptaddr} ${prefix}${script}; source ${scriptaddr}
boot_efi_binary=load ${devtype} ${devnum}:${distro_bootpart} ${kernel_addr_r} efi/boot/bootaa64.efi; if fdt addr -q ${fdt_addr_r}; ti
boot_efi_bootmgr=if fdt addr -q ${fdt_addr_r}; then bootefi bootmgr ${fdt_addr_r};else bootefi bootmgr;fi
boot_extlinux=sysboot ${devtype} ${devnum}:${distro_bootpart} any ${scriptaddr} ${prefix}${boot_syslinux_conf}
boot_net_usb_start=usb start
boot_prefixes=/ /boot/
boot_script_dhcp=boot.scr.uimg
boot_scripts=boot.scr.uimg boot.scr
boot_syslinux_conf=extlinux/extlinux.conf
boot_targets=fel mmc0 usb0 pxe dhcp
bootcmd=run distro_bootcmd
bootcmd_dhcp=devtype=dhcp; run boot_net_usb_start; if dhcp ${scriptaddr} ${boot_script_dhcp}; then source ${scriptaddr}; fi;setenv e;
bootcmd_fel=if test -n ${fel_booted} && test -n ${fel_scriptaddr}; then echo '(FEL boot)'; source ${fel_scriptaddr}; fi
bootcmd_mmc0=devnum=0; run mmc_boot
bootcmd_pxe=run boot_net_usb_start; dhcp; if pxe get; then pxe boot; fi
bootcmd_usb0=devnum=0; run usb_boot
bootdelay=2
bootm_size=0xa000000
console=ttyS0,115200
cpu=armv8
dfu_alt_info_ram=kernel ram 0x40080000 0x1000000;fdt ram 0x4FA00000 0x100000;ramdisk ram 0x4FF00000 0x4000000
distro_bootcmd=for target in ${boot_targets}; do run bootcmd_${target}; done
efi_dtb_prefixes=/ /dtb/ /dtb/current/
ethact=ethernet@1c30000
ethaddr=02:ba:cd:42:77:0b
fdt_addr_r=0x4FA00000
fdtcontroladdr=79f271f0
fdtfile=allwinner/sun50i-a64-pine64-plus.dtb
fdtoverlay_addr_r=0x4FE00000
fel_booted=1
kernel_addr_r=0x40080000
kernel_comp_addr_r=0x44000000
kernel_comp_size=0xb000000
load_efi_dtb=load ${devtype} ${devnum}:${distro_bootpart} ${fdt_addr_r} ${prefix}${efi_fdtfile}
loadaddr=0x42000000
mmc_boot=if mmc dev ${devnum}; then devtype=mmc; run scan_dev_for_boot_part; fi
partitions=name=loader1,start=8k,size=32k,uuid=${uuid_gpt_loader1};name=loader2,size=984k,uuid=${uuid_gpt_loader2};name=esp,size=128;
preboot=usb start
pxefile_addr_r=0x4FD00000
ramdisk_addr_r=0x4FF00000
scan_dev_for_boot=echo Scanning ${devtype} ${devnum}:${distro_bootpart}...; for prefix in ${boot_prefixes}; do run scan_dev_for_extl;
scan_dev_for_boot_part=part list ${devtype} ${devnum} -bootable devplist; env exists devplist || setenv devplist 1; for distro_bootpt
scan_dev_for_efi=setenv efi_fdtfile ${fdtfile}; for prefix in ${efi_dtb_prefixes}; do if test -e ${devtype} ${devnum}:${distro_bootpe
scan_dev_for_extlinux=if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefix}${boot_syslinux_conf}; then echo Found ${prefix}${i
scan_dev_for_scripts=for script in ${boot_scripts}; do if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefix}${script}; then ee
scriptaddr=0x4FC00000
serial#=92c000bacd42770b
soc=sunxi
stderr=serial,vidconsole
stdin=serial,usbkbd
stdout=serial,vidconsole
usb_boot=usb start; if usb dev ${devnum}; then devtype=usb; run scan_dev_for_boot_part; fi
uuid_gpt_esp=c12a7328-f81f-11d2-ba4b-00a0c93ec93b
uuid_gpt_system=b921b045-1df0-41c3-af44-4c6f280d3fae

Environment size: 4492/65532 bytes
=> echo $kernel_addr_r
0x40080000
=> echo $ramdisk_addr_r
0x4FF00000
=> echo $fdt_addr_r
0x4FA00000
=> booti $kernel_addr_r $ramdisk_addr_r $fdt_addr_r
Moving Image from 0x40080000 to 0x40200000, end=416f0000
Wrong Ramdisk Image Format
Ramdisk image is corrupt or invalid
```

## initramfs.cpio.gzをmkimage

```
$ mkimage -A arm64 -T ramdisk -d initramfs.cpio.gz initrd.img
```

- 端末1

```bash
$ ../sunxi-tools/sunxi-fel -v uboot u-boot-sunxi-with-spl.bin write 0x40080000 Image write 0x4fa00000 sun50i-a64-pine64-plus.dtb write 0x4fc00000 boot.scr write 0x4ff00000 initrd.img
```

- 端末2

```
# 個々までは一緒で、boot.scrが認識されていない
# rootfsは認識され、カーネルのboot開始
=> booti $kernel_addr_r $ramdisk_addr_r $fdt_addr_r
Moving Image from 0x40080000 to 0x40200000, end=416f0000
## Loading init Ramdisk from Legacy Image at 4ff00000 ...
   Image Name:
   Image Type:   AArch64 Linux RAMDisk Image (gzip compressed)
   Data Size:    8503229 Bytes = 8.1 MiB
   Load Address: 00000000
   Entry Point:  00000000
   Verifying Checksum ... OK
## Flattened Device Tree blob at 4fa00000
   Booting using the fdt blob at 0x4fa00000
Working FDT set to 4fa00000
   Loading Ramdisk to 497e4000, end 49ffffbd ... OK
   Loading Device Tree to 00000000497da000, end 00000000497e3ee8 ... OK
Working FDT set to 497da000

Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x410fd034]
[    0.000000] Linux version 6.1.47 (vagrant@ubuntu-bionic) (aarch64-linux-gnu-gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0, GNU ld (GN3
[    0.000000] Machine model: Pine64+
[    0.000000] NUMA: No NUMA configuration found
[    0.000000] NUMA: Faking a node at [mem 0x0000000040000000-0x000000007fffffff]
[    0.000000] NUMA: NODE_DATA [mem 0x7fdcea00-0x7fdd0fff]
[    0.000000] Zone ranges:
[    0.000000]   DMA      [mem 0x0000000040000000-0x000000007fffffff]
[    0.000000]   DMA32    empty
[    0.000000]   Normal   empty
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000040000000-0x000000007fffffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000040000000-0x000000007fffffff]
[    0.000000] cma: Reserved 32 MiB at 0x000000007cc00000
[    0.000000] psci: probing for conduit method from DT.
[    0.000000] psci: PSCIv1.1 detected in firmware.
[    0.000000] psci: Using standard PSCI v0.2 function IDs
[    0.000000] psci: MIGRATE_INFO_TYPE not supported.
[    0.000000] psci: SMC Calling Convention v1.4
[    0.000000] percpu: Embedded 19 pages/cpu s39144 r8192 d30488 u77824
[    0.000000] Detected VIPT I-cache on CPU0
[    0.000000] CPU features: detected: ARM erratum 843419
[    0.000000] CPU features: detected: ARM erratum 845719
[    0.000000] alternatives: applying boot alternatives
[    0.000000] Fallback order for Node 0: 0
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 258048
[    0.000000] Policy zone: DMA
[    0.000000] Kernel command line:
[    0.000000] Dentry cache hash table entries: 131072 (order: 8, 1048576 bytes, linear)
[    0.000000] Inode-cache hash table entries: 65536 (order: 7, 524288 bytes, linear)
[    0.000000] mem auto-init: stack:off, heap alloc:off, heap free:off
[    0.000000] Memory: 965568K/1048576K available (12608K kernel code, 1420K rwdata, 4392K rodata, 2368K init, 478K bss, 50240K rese)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=4, Nodes=1
[    0.000000] rcu: Preemptible hierarchical RCU implementation.
[    0.000000] rcu:     RCU event tracing is enabled.
[    0.000000] rcu:     RCU restricting CPUs from NR_CPUS=256 to nr_cpu_ids=4.
[    0.000000]  Trampoline variant of Tasks RCU enabled.
[    0.000000]  Tracing variant of Tasks RCU enabled.
[    0.000000] rcu: RCU calculated value of scheduler-enlistment delay is 25 jiffies.
[    0.000000] rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=4
[    0.000000] NR_IRQS: 64, nr_irqs: 64, preallocated irqs: 0
[    0.000000] Root IRQ handler: gic_handle_irq
[    0.000000] GIC: Using split EOI/Deactivate mode
[    0.000000] rcu: srcu_init: Setting srcu_struct sizes based on contention.
[    0.000000] arch_timer: Enabling global workaround for Allwinner erratum UNKNOWN1
[    0.000000] arch_timer: CPU0: Trapping CNTVCT access
[    0.000000] arch_timer: cp15 timer(s) running at 24.00MHz (phys).
[    0.000000] clocksource: arch_sys_counter: mask: 0xffffffffffffff max_cycles: 0x588fe9dc0, max_idle_ns: 440795202592 ns
[    0.000001] sched_clock: 56 bits at 24MHz, resolution 41ns, wraps every 4398046511097ns
[    0.000230] clocksource: timer: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 79635851949 ns
[    0.000814] Console: colour dummy device 80x25
[    0.001336] printk: console [tty0] enabled
[    0.001427] Calibrating delay loop (skipped), value calculated using timer frequency.. 48.00 BogoMIPS (lpj=96000)
[    0.001461] pid_max: default: 32768 minimum: 301
[    0.001548] LSM: Security Framework initializing
[    0.001679] Mount-cache hash table entries: 2048 (order: 2, 16384 bytes, linear)
[    0.001713] Mountpoint-cache hash table entries: 2048 (order: 2, 16384 bytes, linear)
[    0.003677] cblist_init_generic: Setting adjustable number of callback queues.
[    0.003711] cblist_init_generic: Setting shift to 2 and lim to 1.
[    0.003820] cblist_init_generic: Setting adjustable number of callback queues.
[    0.003842] cblist_init_generic: Setting shift to 2 and lim to 1.
[    0.004127] rcu: Hierarchical SRCU implementation.
[    0.004145] rcu:     Max phase no-delay instances is 1000.
[    0.005611] smp: Bringing up secondary CPUs ...
[    0.008172] Detected VIPT I-cache on CPU1
[    0.008306] arch_timer: CPU1: Trapping CNTVCT access
[    0.008327] CPU1: Booted secondary processor 0x0000000001 [0x410fd034]
[    0.009876] Detected VIPT I-cache on CPU2
[    0.009966] arch_timer: CPU2: Trapping CNTVCT access
[    0.009978] CPU2: Booted secondary processor 0x0000000002 [0x410fd034]
[    0.011162] Detected VIPT I-cache on CPU3
[    0.011254] arch_timer: CPU3: Trapping CNTVCT access
[    0.011265] CPU3: Booted secondary processor 0x0000000003 [0x410fd034]
[    0.011347] smp: Brought up 1 node, 4 CPUs
[    0.011460] SMP: Total of 4 processors activated.
[    0.011476] CPU features: detected: 32-bit EL0 Support
[    0.011491] CPU features: detected: CRC32 instructions
[    0.011582] CPU: All CPU(s) started at EL2
[    0.011598] alternatives: applying system-wide alternatives
[    0.013357] devtmpfs: initialized
[    0.020786] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
[    0.020852] futex hash table entries: 1024 (order: 4, 65536 bytes, linear)
[    0.022295] pinctrl core: initialized pinctrl subsystem
[    0.023801] NET: Registered PF_NETLINK/PF_ROUTE protocol family
[    0.025087] DMA: preallocated 128 KiB GFP_KERNEL pool for atomic allocations
[    0.025351] DMA: preallocated 128 KiB GFP_KERNEL|GFP_DMA pool for atomic allocations
[    0.025501] DMA: preallocated 128 KiB GFP_KERNEL|GFP_DMA32 pool for atomic allocations
[    0.025587] audit: initializing netlink subsys (disabled)
[    0.025774] audit: type=2000 audit(0.024:1): state=initialized audit_enabled=0 res=1
[    0.026319] thermal_sys: Registered thermal governor 'step_wise'
[    0.026327] thermal_sys: Registered thermal governor 'power_allocator'
[    0.026388] cpuidle: using governor menu
[    0.026510] hw-breakpoint: found 6 breakpoint and 4 watchpoint registers.
[    0.026635] ASID allocator initialised with 65536 entries
[    0.027422] Serial: AMBA PL011 UART driver
[    0.032592] platform 1c0c000.lcd-controller: Fixed dependency cycle(s) with /soc/bus@1000000/mixer@200000
[    0.032651] platform 1c0c000.lcd-controller: Fixed dependency cycle(s) with /soc/bus@1000000/mixer@100000
[    0.032914] platform 1c0d000.lcd-controller: Fixed dependency cycle(s) with /soc/hdmi@1ee0000
[    0.040104] platform 1ee0000.hdmi: Fixed dependency cycle(s) with /hdmi-connector
[    0.040679] KASLR disabled due to lack of seed
[    0.051436] HugeTLB: registered 1.00 GiB page size, pre-allocated 0 pages
[    0.051471] HugeTLB: 0 KiB vmemmap can be freed for a 1.00 GiB page
[    0.051491] HugeTLB: registered 32.0 MiB page size, pre-allocated 0 pages
[    0.051507] HugeTLB: 0 KiB vmemmap can be freed for a 32.0 MiB page
[    0.051525] HugeTLB: registered 2.00 MiB page size, pre-allocated 0 pages
[    0.051541] HugeTLB: 0 KiB vmemmap can be freed for a 2.00 MiB page
[    0.051559] HugeTLB: registered 64.0 KiB page size, pre-allocated 0 pages
[    0.051575] HugeTLB: 0 KiB vmemmap can be freed for a 64.0 KiB page
[    0.053835] iommu: Default domain type: Translated
[    0.053867] iommu: DMA domain TLB invalidation policy: strict mode
[    0.054221] SCSI subsystem initialized
[    0.054644] usbcore: registered new interface driver usbfs
[    0.054704] usbcore: registered new interface driver hub
[    0.054757] usbcore: registered new device driver usb
[    0.055056] pps_core: LinuxPPS API ver. 1 registered
[    0.055073] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    0.055108] PTP clock support registered
[    0.055253] EDAC MC: Ver: 3.0.0
[    0.056050] FPGA manager framework
[    0.056160] Advanced Linux Sound Architecture Driver Initialized.
[    0.057135] vgaarb: loaded
[    0.057429] clocksource: Switched to clocksource arch_sys_counter
[    0.057686] VFS: Disk quotas dquot_6.6.0
[    0.057751] VFS: Dquot-cache hash table entries: 512 (order 0, 4096 bytes)
[    0.066222] NET: Registered PF_INET protocol family
[    0.066515] IP idents hash table entries: 16384 (order: 5, 131072 bytes, linear)
[    0.067861] tcp_listen_portaddr_hash hash table entries: 512 (order: 1, 8192 bytes, linear)
[    0.067950] Table-perturb hash table entries: 65536 (order: 6, 262144 bytes, linear)
[    0.067985] TCP established hash table entries: 8192 (order: 4, 65536 bytes, linear)
[    0.068089] TCP bind hash table entries: 8192 (order: 6, 262144 bytes, linear)
[    0.068412] TCP: Hash tables configured (established 8192 bind 8192)
[    0.068566] UDP hash table entries: 512 (order: 2, 16384 bytes, linear)
[    0.068620] UDP-Lite hash table entries: 512 (order: 2, 16384 bytes, linear)
[    0.068812] NET: Registered PF_UNIX/PF_LOCAL protocol family
[    0.069276] RPC: Registered named UNIX socket transport module.
[    0.069297] RPC: Registered udp transport module.
[    0.069311] RPC: Registered tcp transport module.
[    0.069325] RPC: Registered tcp NFSv4.1 backchannel transport module.
[    0.069348] PCI: CLS 0 bytes, default 64
[    0.069747] Unpacking initramfs...
[    0.078259] hw perfevents: enabled with armv8_cortex_a53 PMU driver, 7 counters available
[    0.080052] Initialise system trusted keyrings
[    0.080337] workingset: timestamp_bits=42 max_order=18 bucket_order=0
[    0.089003] squashfs: version 4.0 (2009/01/31) Phillip Lougher
[    0.089968] NFS: Registering the id_resolver key type
[    0.090032] Key type id_resolver registered
[    0.090048] Key type id_legacy registered
[    0.090157] nfs4filelayout_init: NFSv4 File Layout Driver Registering...
[    0.090178] nfs4flexfilelayout_init: NFSv4 Flexfile Layout Driver Registering...
[    0.144323] Key type asymmetric registered
[    0.144374] Asymmetric key parser 'x509' registered
[    0.144498] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 246)
[    0.144524] io scheduler mq-deadline registered
[    0.144540] io scheduler kyber registered
[    0.158370] Serial: 8250/16550 driver, 4 ports, IRQ sharing enabled
[    0.168232] loop: module loaded
[    0.169191] megasas: 07.719.03.00-rc1
[    0.173885] tun: Universal TUN/TAP device driver, 1.6
[    0.174666] thunder_xcv, ver 1.0
[    0.174733] thunder_bgx, ver 1.0
[    0.174785] nicpf, ver 1.0
[    0.175209] hns3: Hisilicon Ethernet Network Driver for Hip08 Family - version
[    0.175232] hns3: Copyright (c) 2017 Huawei Corporation.
[    0.175317] hclge is initializing
[    0.175355] e1000: Intel(R) PRO/1000 Network Driver
[    0.175370] e1000: Copyright (c) 1999-2006 Intel Corporation.
[    0.175436] e1000e: Intel(R) PRO/1000 Network Driver
[    0.175451] e1000e: Copyright(c) 1999 - 2015 Intel Corporation.
[    0.175514] igb: Intel(R) Gigabit Ethernet Network Driver
[    0.175531] igb: Copyright (c) 2007-2014 Intel Corporation.
[    0.175586] igbvf: Intel(R) Gigabit Virtual Function Network Driver
[    0.175603] igbvf: Copyright (c) 2009 - 2012 Intel Corporation.
[    0.175750] sky2: driver version 1.30
[    0.176180] VFIO - User Level meta-driver version: 0.3
[    0.178435] usbcore: registered new interface driver usb-storage
[    0.181318] sun6i-rtc 1f00000.rtc: registered as rtc0
[    0.181388] sun6i-rtc 1f00000.rtc: setting system clock to 1970-01-01T00:03:03 UTC (183)
[    0.181575] sun6i-rtc 1f00000.rtc: RTC enabled
[    0.181702] i2c_dev: i2c /dev entries driver
[    0.184059] sdhci: Secure Digital Host Controller Interface driver
[    0.184098] sdhci: Copyright(c) Pierre Ossman
[    0.184525] Synopsys Designware Multimedia Card Interface Driver
[    0.185255] sdhci-pltfm: SDHCI platform and OF driver helper
[    0.186263] ledtrig-cpu: registered to indicate activity on CPUs
[    0.186645] SMCCC: SOC_ID: ID = jep106:091e:1689 Revision = 0x00000001
[    0.187261] usbcore: registered new interface driver usbhid
[    0.187282] usbhid: USB HID core driver
[    0.192451] NET: Registered PF_PACKET protocol family
[    0.192494] can: controller area network core
[    0.192587] NET: Registered PF_CAN protocol family
[    0.192680] Key type dns_resolver registered
[    0.193040] registered taskstats version 1
[    0.193083] Loading compiled-in X.509 certificates
[    0.217591] sun50i-a64-pinctrl 1c20800.pinctrl: initialized sunXi PIO driver
[    0.219434] sun50i-a64-r-pinctrl 1f02c00.pinctrl: initialized sunXi PIO driver
[    0.219906] sun50i-a64-pinctrl 1c20800.pinctrl: supply vcc-pb not found, using dummy regulator
[    0.240824] 1c28000.serial: ttyS0 at MMIO 0x1c28000 (irq = 153, base_baud = 1500000) is a U6_16550A
[    0.664431] Freeing initrd memory: 8300K
[    0.668914] printk: console [ttyS0] enabled
[    1.389619] ehci-platform 1c1a000.usb: EHCI Host Controller
[    1.389975] ehci-platform 1c1b000.usb: EHCI Host Controller
[    1.390553] ohci-platform 1c1a400.usb: Generic Platform OHCI controller
[    1.390579] ohci-platform 1c1a400.usb: new USB bus registered, assigned bus number 1
[    1.390723] ohci-platform 1c1a400.usb: irq 156, io mem 0x01c1a400
[    1.390740] usb_phy_generic usb_phy_generic.1.auto: supply vcc not found, using dummy regulator
[    1.390868] usb_phy_generic usb_phy_generic.1.auto: dummy supplies not allowed for exclusive requests
[    1.391449] ohci-platform 1c1b400.usb: Generic Platform OHCI controller
[    1.391471] ohci-platform 1c1b400.usb: new USB bus registered, assigned bus number 2
[    1.391536] musb-hdrc musb-hdrc.2.auto: MUSB HDRC host driver
[    1.391554] musb-hdrc musb-hdrc.2.auto: new USB bus registered, assigned bus number 3
[    1.391564] ohci-platform 1c1b400.usb: irq 158, io mem 0x01c1b400
[    1.392307] hub 3-0:1.0: USB hub found
[    1.392339] hub 3-0:1.0: 1 port detected
[    1.394264] sun50i-a64-pinctrl 1c20800.pinctrl: supply vcc-ph not found, using dummy regulator
[    1.395280] ehci-platform 1c1a000.usb: new USB bus registered, assigned bus number 4
[    1.396154] thermal_sys: Failed to find 'trips' node
[    1.396160] thermal_sys: Failed to find trip points for thermal-sensor id=1
[    1.396169] sun8i-thermal: probe of 1c25000.thermal-sensor failed with error -22
[    1.397958] sun50i-a64-r-pinctrl 1f02c00.pinctrl: supply vcc-pl not found, using dummy regulator
[    1.398336] sunxi-rsb 1f03400.rsb: RSB running at 4000000 Hz
[    1.398684] axp20x-rsb sunxi-rsb-3a3: AXP20x variant AXP803 found
[    1.399608] axp20x-rsb sunxi-rsb-3a3: mask_invert=true is deprecated; please switch to unmask_base
[    1.400842] ehci-platform 1c1b000.usb: new USB bus registered, assigned bus number 5
[    1.407597] ehci-platform 1c1a000.usb: irq 154, io mem 0x01c1a000
[    1.410231] axp20x-rsb sunxi-rsb-3a3: AXP20X driver loaded
[    1.416393] ehci-platform 1c1b000.usb: irq 155, io mem 0x01c1b000
[    1.417509] sun50i-a64-pinctrl 1c20800.pinctrl: supply vcc-pf not found, using dummy regulator
[    1.417523] ALSA device list:
[    1.417529]   No soundcards found.
[    1.418343] sunxi-mmc 1c0f000.mmc: Got CD GPIO
[    1.437441] ehci-platform 1c1a000.usb: USB 2.0 started, EHCI 1.00
[    1.441443] sunxi-mmc 1c0f000.mmc: initialized, max. request size: 16384 KB, uses new timings mode
[    1.447149] hub 4-0:1.0: USB hub found
[    1.453442] ehci-platform 1c1b000.usb: USB 2.0 started, EHCI 1.00
[    1.617949] hub 4-0:1.0: 1 port detected
[    1.622758] hub 2-0:1.0: USB hub found
[    1.626558] hub 2-0:1.0: 1 port detected
[    1.631151] hub 1-0:1.0: USB hub found
[    1.634946] hub 1-0:1.0: 1 port detected
[    1.642909] hub 5-0:1.0: USB hub found
[    1.646710] hub 5-0:1.0: 1 port detected
[    1.661141] /dev/root: Can't open blockdev
[    1.665325] VFS: Cannot open root device "(null)" or unknown-block(0,0): error -6
[    1.672843] Please append a correct "root=" boot option; here are the available partitions:
[    1.681230] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
[    1.689504] CPU: 0 PID: 1 Comm: swapper/0 Not tainted 6.1.47 #1
[    1.695437] Hardware name: Pine64+ (DT)
[    1.699281] Call trace:
[    1.701736]  dump_backtrace.part.0+0xdc/0xf0
[    1.706036]  show_stack+0x18/0x30
[    1.709369]  dump_stack_lvl+0x68/0x84
[    1.713054]  dump_stack+0x18/0x34
[    1.716387]  panic+0x184/0x344
[    1.719459]  mount_block_root+0x17c/0x22c
[    1.723491]  mount_root+0x204/0x240
[    1.726997]  prepare_namespace+0x130/0x170
[    1.731111]  kernel_init_freeable+0x258/0x284
[    1.735484]  kernel_init+0x24/0x12c
[    1.738988]  ret_from_fork+0x10/0x20
[    1.742580] SMP: stopping secondary CPUs
[    1.746514] Kernel Offset: disabled
[    1.750009] CPU features: 0x00000,00c00080,0000420b
[    1.754896] Memory Limit: none
[    1.757964] ---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0) ]---
```

## bootargsをsetenvして実行

```
=> echo $bootargs

=> setenv bootargs console=ttyS0,115200 earlyprintk root=/dev/ram0
=> booti $kernel_addr_r $ramdisk_addr_r $fdt_addr_r

# 結果は変わらず

[    1.666355] /dev/root: Can't open blockdev
[    1.670523] VFS: Cannot open root device "ram0" or unknown-block(0,0): error -6
[    1.677844] Please append a correct "root=" boot option; here are the available partitions:
[    1.686221] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
...
[    1.762647] ---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0) ]---
```
