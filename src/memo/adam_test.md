# AdamRLukaitis/pine64の追試

## Linuxイメージの取得

```bash
$ wget https://www.stdin.xyz/downloads/people/longsleep/pine64-images/simpleimage-pine64-latest.img.xz
$ unxz simpleimage-pine64-latest.img.xz
$ Etcherでsdカードに書き込み
$ cd /Volumes/BOOT
$ ls -l
total 1074
-rwxrwxrwx 1 dspace staff 1094464  3 11  2017 initrd.img
drwxrwxrwx 1 dspace staff    2048  3 11  2017 pine64
-rwxrwxrwx 1 dspace staff     111  3 11  2017 uEnv.txt
$ ls -l pine64/
total 11764
-rwxrwxrwx 1 dspace staff 11835840  3 11  2017 Image
-rwxrwxrwx 1 dspace staff    69414  3 11  2017 sun50i-a64-pine64-plus.dtb
-rwxrwxrwx 1 dspace staff    69434  3 11  2017 sun50i-a64-pine64-so.dtb
-rwxrwxrwx 1 dspace staff    69322  3 11  2017 sun50i-a64-pine64.dtb
```

## 実行

### U-Bootのコマンドプロンプトの実験

```bash
$ minicom

HELLO! BOOT0 is starting!
boot0 commit : 045061a8bb2580cb3fa02e301f52a015040c158f

boot0 version : 4.0.0
set pll start
set pll end
rtc[0] value = 0x00000000
rtc[1] value = 0x00000000
rtc[2] value = 0x00000000
rtc[3] value = 0x00000000
rtc[4] value = 0x00000000
rtc[5] value = 0x00000000
DRAM driver version: V1.1
rsb_send_initseq: rsb clk 400Khz -> 3Mhz
PMU: AXP81X
ddr voltage = 1500 mv
DRAM Type = 3 (2:DDR2,3:DDR3,6:LPDDR2,7:LPDDR3)
DRAM clk = 672 MHz
DRAM zq value: 003b3bbb
DRAM single rank full DQ OK
DRAM size = 1024 MB
DRAM init ok
dram size =1024
card boot number = 0, boot0 copy = 0
card no is 0
sdcard 0 line count 4
[mmc]: mmc driver ver 2015-05-08 20:06
[mmc]: sdc0 spd mode error, 2
[mmc]: Wrong media type 0x00000000
[mmc]: ***Try SD card 0***
[mmc]: HSSDR52/SDR25 4 bit
[mmc]: 50000000 Hz
[mmc]: 29818 MB
[mmc]: ***SD/MMC 0 init OK!!!***
sdcard 0 init ok
The size of uboot is 000e8000.
sum=e88028eb
src_sum=e88028eb
Succeed in loading uboot from sdmmc flash.
boot0: start load other image
boot0: Loading BL3-1
Loading file 0 at address 0x40000000,size 0x00008400 success
boot0: Loading scp
Loading file 2 at address 0x00040000,size 0x00019c00 success
set arisc reset to de-assert state
Ready to disable icache.
Jump to secend Boot.


U-Boot 2014.07-7-pine64-longsleep (Mar 11 2017 - 17:28:41) Allwinner Technology

uboot commit : 360fbc1b502997e626843b6699a330879a296d62

rsb: secure monitor exist
[      0.335]pmbus:   ready
[      0.338][ARISC] :arisc initialize
[      0.668][ARISC] :arisc_dvfs_cfg_vf_table: support only one vf_table
[      0.783][ARISC] :sunxi-arisc driver startup succeeded
[      0.817]PMU: AXP81X
[      0.819]PMU: AXP81X found
bat_vol=368, ratio=100
[      0.825]PMU: dcdc2 1100
[      0.829]PMU: cpux 1008 Mhz,AXI=336 Mhz
PLL6=600 Mhz,AHB1=200 Mhz, APB1=100Mhz AHB2=300Mhz MBus=400Mhz
device_type = 3253, onoff=1
dcdc1_vol = 3300, onoff=1
dcdc2_vol = 1100, onoff=1
dcdc6_vol = 1100, onoff=1
aldo1_vol = 2800, onoff=0
aldo2_vol = 1800, onoff=1
aldo3_vol = 3000, onoff=1
dldo1_vol = 3300, onoff=0
dldo2_vol = 3300, onoff=0
dldo3_vol = 2800, onoff=0
dldo4_vol = 3300, onoff=1
eldo1_vol = 1800, onoff=1
eldo2_vol = 1800, onoff=0
eldo3_vol = 1800, onoff=0
fldo1_vol = 1200, onoff=0
fldo2_vol = 1100, onoff=1
gpio0_vol = 3100, onoff=0
vbus not exist
no battery, limit to dc
run key detect
no key found
no uart input
DRAM:  1008 MiB
fdt addr: 0x76ebef60
Relocation Offset is: 35f11000
In:    serial
Out:   serial
Err:   serial
gic: sec monitor mode
[      1.640]start
drv_disp_init
init_clocks: finish init_clocks.
enable power vcc-hdmi-33, ret=0
drv_disp_init finish
boot_disp.output_disp=0
boot_disp.output_type=3
boot_disp.output_mode=10
fetch script data boot_disp.auto_hpd fail
disp0 device type(4) enable
attched ok, mgr0<-->device1, type=4, mode=10
[      2.028]end
workmode = 0,storage type = 1
[      2.032]MMC:        0
[mmc]: mmc driver ver 2015-06-03 13:50:00
SUNXI SD/MMC: 0
[mmc]: start mmc_calibrate_delay_unit, don't access device...
[mmc]: delay chain cal done, sample: 200(ps)
[mmc]: media type 0x0
[mmc]: Wrong media type 0x0
[mmc]: ************Try SD card 0************
[mmc]: host caps: 0x27
[mmc]: MID 27 PSN 392d2a6b
[mmc]: PNM SD32G -- 0x53-44-33-32-47
[mmc]: PRV 6.0
[mmc]: MDT m-8 y-2022
[mmc]: speed mode     : HSSDR52/SDR25
[mmc]: clock          : 50000000 Hz
[mmc]: bus_width      : 4 bit
[mmc]: user capacity  : 29818 MB
[mmc]: ************SD/MMC 0 init OK!!!************
[mmc]: erase_grp_size      : 0x1WrBlk*0x200=0x200 Byte
[mmc]: secure_feature      : 0x0
[mmc]: secure_removal_type : 0x0
[      2.226]sunxi flash init ok
[mmc]: Has init
[      2.263]---drivers/mmc/mmc.c 2733 mmc_init
reading uboot.env

** Unable to read "uboot.env" from mmc0:1 **
Using default environment

--------fastboot partitions--------
mbr not exist
base bootcmd=run mmcbootcmd
bootcmd set setargs_mmc
key 0
recovery key high 12, low 10
fastboot key high 6, low 4
no misc partition is found
to be run cmd=run mmcbootcmd
update dtb dram start
update dtb dram  end
serial is: ffffffffffffffffffff
get Pine64 model from DRAM size and used storage
DRAM >512M
Pine64 model: pine64-plus
no battery exist
sunxi_bmp_logo_display
[mmc]: Has init
[      2.481]---drivers/mmc/mmc.c 2733 mmc_init
reading bootlogo.bmp
** Unable to read file bootlogo.bmp **
sunxi bmp info error : unable to open logo file bootlogo.bmp
[      2.498]inter uboot shell
Hit any key to stop autoboot:  0

sunxi# mw.b 0x01c28000 0x55     # UARTに0x55('U')を送信 => 次の行の先頭に'U'が表示
Usunxi# mw.b 0x01c28000 0x56    # UARTに0x56('V')を送信 => 次の行の先頭に'V'が表示
Vsunxi# printenv                $ printenvコマンド

baudrate=115200
boot_disk=0
boot_kernel=booti ${kernel_addr} ${initrd_addr}:${initrd_size} ${fdt_addr}
boot_key=0
boot_part=0:1
bootcmd=run mmcbootcmd
bootdelay=3
bootenv_filename=uEnv.txt
console=ttyS0,115200
fdt_addr=45000000
fdt_filename_prefix=pine64/sun50i-a64-
fdt_filename_suffix=.dtb
import_bootenv=env import -t ${load_addr} ${filesize}
initrd_addr=45300000
initrd_filename=initrd.img
kernel_addr=41080000
kernel_filename=pine64/Image
load_addr=41000000
load_bootenv=fatload mmc ${boot_part} ${load_addr} ${bootenv_filename}
load_bootscript=fatload mmc ${boot_part} ${load_addr} ${script}
load_dtb=if test ${fdt_filename} = ""; then setenv fdt_filename ${fdt_filename_prefix}${pe
load_initrd=fatload mmc ${boot_part} ${initrd_addr} ${initrd_filename}; setenv initrd_siz}
load_kernel=fatload mmc ${boot_part} ${kernel_addr} ${kernel_filename}
mmcboot=run load_dtb load_kernel load_initrd set_cmdline boot_kernel
mmcbootcmd=if run load_bootenv; then echo Loading boot environment ...; run import_booteni
pine64_model=pine64-plus
root=/dev/mmcblk0p2
script=boot.scr
scriptboot=source ${load_addr}
set_cmdline=setenv bootargs console=${console} ${optargs} earlycon=uart,mmio32,0x01c28000t
sunxi_serial=ffffffffffffffffffff

Environment size: 1605/131067 bytes
sunxi#
```
