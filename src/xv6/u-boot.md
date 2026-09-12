# u-bootによるシステム起動

## TFTPサーバー

1. 起動

```bash
sudo launchctl load -w /System/Library/LaunchDaemons/tftp.plist
```

2. 接続確認

```bash
$ sudo lsof -i:69
COMMAND PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
launchd   1 root   50u  IPv6 0x378d20ac2bcba9e8      0t0  UDP *:tftp
launchd   1 root   51u  IPv4 0xfdeb2b26c3009f26      0t0  UDP *:tftp
launchd   1 root   52u  IPv6 0x378d20ac2bcba9e8      0t0  UDP *:tftp
launchd   1 root   53u  IPv4 0xfdeb2b26c3009f26      0t0  UDP *:tftp
```

3. 停止

```bash
sudo launchctl unload -w /System/Library/LaunchDaemons/tftp.plist
```

4. ファイルの配置ディレクトリ

```bash
/private/tftpboot/
```

## システム作成

```bash
$ cat main.c
int _start()
{
	for (;;)
		*(unsigned long *)0x1c28000 = 'U';
}
$ cat Makefile
CROSS_DEV_PREFIX := /opt/homebrew/bin/aarch64-elf-

serial_test: main.c
	$(CROSS_DEV_PREFIX)gcc -Wl,-Ttext=0x42000000 -nostdlib $< -o $@
	$(CROSS_DEV_PREFIX)objdump -ld $@

serial_test.img: serial_test
	$(CROSS_DEV_PREFIX)objcopy -Obinary $< $@

test: serial_test.img
	sudo cp $< /private/tftpboot/
$ make test
$ ls /private/tftpboot/
serial_test.img
```

## bootpによるシステム起動


```bash
U-Boot SPL 2023.10-rc4 (Oct 04 2023 - 14:04:19 +0900)
DRAM: 1024 MiB
Trying to boot from MMC1
NOTICE:  BL31: v2.9(debug):5f01b0b
NOTICE:  BL31: Built : 10:14:49, Sep  3 2023
NOTICE:  BL31: Detected Allwinner A64/H64/R18 SoC (1689)
NOTICE:  BL31: Found U-Boot DTB at 0x20a44d0, model: Pine64+
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


U-Boot 2023.10-rc4 (Oct 04 2023 - 14:04:19 +0900) Allwinner Technology

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
Hit any key to stop autoboot:  0        // 改行
=>
=> printenv
arch=arm
baudrate=115200
board=sunxi
board_name=sunxi
boot_a_script=load ${devtype} ${devnum}:${distro_bootpart} ${scriptaddr} ${prefix}${script}; source ${scriptaddr}
boot_efi_binary=load ${devtype} ${devnum}:${distro_bootpart} ${kernel_addr_r} efi/boot/bootaa64.efi; if fdt addr -q ${fdt_addr_r}; then bootefi i
boot_efi_bootmgr=if fdt addr -q ${fdt_addr_r}; then bootefi bootmgr ${fdt_addr_r};else bootefi bootmgr;fi
boot_extlinux=sysboot ${devtype} ${devnum}:${distro_bootpart} any ${scriptaddr} ${prefix}${boot_syslinux_conf}
boot_net_usb_start=usb start
boot_prefixes=/ /boot/
boot_script_dhcp=boot.scr.uimg
boot_scripts=boot.scr.uimg boot.scr
boot_syslinux_conf=extlinux/extlinux.conf
boot_targets=fel mmc0 usb0 pxe dhcp
bootcmd=run distro_bootcmd
bootcmd_dhcp=devtype=dhcp; run boot_net_usb_start; if dhcp ${scriptaddr} ${boot_script_dhcp}; then source ${scriptaddr}; fi;setenv efi_fdtfile $;
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
ethaddr=02:ba:cd:42:77:0b
fdt_addr_r=0x4FA00000
fdtcontroladdr=79f271f0
fdtfile=allwinner/sun50i-a64-pine64-plus.dtb
fdtoverlay_addr_r=0x4FE00000
kernel_addr_r=0x40080000
kernel_comp_addr_r=0x44000000
kernel_comp_size=0xb000000
load_efi_dtb=load ${devtype} ${devnum}:${distro_bootpart} ${fdt_addr_r} ${prefix}${efi_fdtfile}
loadaddr=0x42000000
mmc_boot=if mmc dev ${devnum}; then devtype=mmc; run scan_dev_for_boot_part; fi
mmc_bootdev=0
partitions=name=loader1,start=8k,size=32k,uuid=${uuid_gpt_loader1};name=loader2,size=984k,uuid=${uuid_gpt_loader2};name=esp,size=128M,bootable,u;
preboot=usb start
pxefile_addr_r=0x4FD00000
ramdisk_addr_r=0x4FF00000
scan_dev_for_boot=echo Scanning ${devtype} ${devnum}:${distro_bootpart}...; for prefix in ${boot_prefixes}; do run scan_dev_for_extlinux; run sc;
scan_dev_for_boot_part=part list ${devtype} ${devnum} -bootable devplist; env exists devplist || setenv devplist 1; for distro_bootpart in ${devt
scan_dev_for_efi=setenv efi_fdtfile ${fdtfile}; for prefix in ${efi_dtb_prefixes}; do if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefie
scan_dev_for_extlinux=if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefix}${boot_syslinux_conf}; then echo Found ${prefix}${boot_syslinui
scan_dev_for_scripts=for script in ${boot_scripts}; do if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefix}${script}; then echo Found U-e
scriptaddr=0x4FC00000
serial#=92c000bacd42770b
soc=sunxi
stderr=serial,vidconsole
stdin=serial,usbkbd
stdout=serial,vidconsole
usb_boot=usb start; if usb dev ${devnum}; then devtype=usb; run scan_dev_for_boot_part; fi
uuid_gpt_esp=c12a7328-f81f-11d2-ba4b-00a0c93ec93b
uuid_gpt_system=b921b045-1df0-41c3-af44-4c6f280d3fae

Environment size: 4469/65532 bytes
=> bootp 192.168.10.105:/private/tftpboot/serial_test.img
ethernet@1c30000 Waiting for PHY auto negotiation to complete....... done
BOOTP broadcast 1
BOOTP broadcast 2
BOOTP broadcast 3
BOOTP broadcast 4
DHCP client bound to address 192.168.10.114 (2263 ms)
Using ethernet@1c30000 device
TFTP from server 192.168.10.105; our IP address is 192.168.10.114
Filename '/private/tftpboot/serial_test.img'.
Load address: 0x42000000
Loading: #
         0 Bytes/s
done
Bytes transferred = 20 (14 hex)
=> go 0x42000000
## Starting application at 0x42000000 ...
UUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU
```

## 自動実行

```bash
Hit any key to stop autoboot:  0
=> editenv bootcmd
edit: bootp 192.168.10.105:/private/tftpboot/serial_test.img; go 0x42000000
=> editenv preboot
edit: nothing
=> saveenv
Saving Environment to FAT... OK
=> printenv
bootcmd=bootp 192.168.10.105:/private/tftpboot/serial_test.img; go 0x42000000
preboot=nothing
```

- 再実行

```bash
Welcome to minicom 2.10

OPTIONS:
Compiled on Feb 22 2025, 10:10:35.
Port /dev/cu.usbserial-AI057C9L, 10:19:34 [F]

Press Meta-Z for help on special keys                       // 電源オン


U-Boot SPL 2023.10-rc4 (Oct 04 2023 - 14:04:19 +0900)
DRAM: 1024 MiB
Trying to boot from MMC1
NOTICE:  BL31: v2.9(debug):5f01b0b
NOTICE:  BL31: Built : 10:14:49, Sep  3 2023
NOTICE:  BL31: Detected Allwinner A64/H64/R18 SoC (1689)
NOTICE:  BL31: Found U-Boot DTB at 0x20a44d0, model: Pine64+
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


U-Boot 2023.10-rc4 (Oct 04 2023 - 14:04:19 +0900) Allwinner Technology

CPU:   Allwinner A64 (SUN50I)
Model: Pine64+
DRAM:  1 GiB
Core:  68 devices, 20 uclasses, devicetree: separate
WDT:   Not starting watchdog@1c20ca0
MMC:   mmc@1c0f000: 0
Loading Environment from FAT... OK
In:    serial,usbkbd
Out:   serial,vidconsole
Err:   serial,vidconsole
Net:   eth0: ethernet@1c30000
Unknown command 'nothing' - try 'help'
Hit any key to stop autoboot:  0
ethernet@1c30000 Waiting for PHY auto negotiation to complete...... done
BOOTP broadcast 1
BOOTP broadcast 2
DHCP client bound to address 192.168.10.114 (255 ms)
Using ethernet@1c30000 device
TFTP from server 192.168.10.105; our IP address is 192.168.10.114
Filename '/private/tftpboot/serial_test.img'.
Load address: 0x42000000
Loading: #####
         335.9 KiB/s
done
Bytes transferred = 65704 (100a8 hex)
## Starting application at 0x42000000 ...
Hello world! Hello world! Hello world! Hello world! Hello world! Hello world! Hello world! Hello world! Hello world! Hello world! Hello world! Hd
```