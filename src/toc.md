# Summary

# Sunxi Wiki

- [Sunxi Pine64](wiki/pine64.md)
- [マニュアルビルドの手引](wiki/manual_build_howto.md)
- [Mainlineカーネルの手引](wiki/mainline_kernel_howto.md)
- [U-Boot](wiki/uboot.md)
- [ブート可能なSDカード](wiki/bootable_sd.md)
- [初期Ramdisk](wiki/initial_ramdisk.md)
- [A64](wiki/a64.md)
- [FEL](wiki/fel.md)
- [FEL/USBBoot](wiki/usb_boot.md)
- [Sun8i emac](wiki/sun8i_emac.md)

# U-Boot

- [TPL/SPLからのブート](uboot/booting.md)
- [環境変数](uboot/env.md)
- [FIT](uboot/fit/README.md)
  - [FITフォーマット](uboot/fit/format.md)
- [シェルコマンド](uboot/commands/README.md)
  - [bootd](uboot/commands/bootd.md)
  - [bootm](uboot/commands/bootm.md)
- [Allwinner SoCベースのボード](uboot/allwinner.md)
- [U-Boot: README.sunxi64](uboot/README.sunxi64_uboot.md)

# Crust

- [README](crust/README.md)
- [ABI](crust/abi.md)

# Busybox

- [mdev](busybox/mdev.md)

# Allwiner A64ユーザマニュアル

- [3.1 メモリマッピング](usermanual/memory_mapping.md)
- [3.3 CCU](usermanual/ccu.md)
- [3.6 タイマー](usermanual/timer.md)
- [3.8 RTC](usermanual/rtc.md)
- [3.9 ハイスピードタイマー](usermanual/hs_timer.md)
- [3.11 DMA](usermanual/dma.md)
- [3.12 GIC](usermanual/gic.md)
- [3.13 メッセージボックス](usermanual/message_box.md)
- [3.21 ポートコントローラ (CPUx-PORT)](usermanual/port_b-h.md)
- [3.22 ポートコントローラ (CPUs-PORT)](usermanual/port_l.md)
- [4.3 SD-MMCホストコントローラ](usermanual/sd-mmc_host_controller.md)
- [7.3 UART](usermanual/uart.md)
- [7.5 USB](usermanual/usb.md)
- [7.9 EMAC](usermanual/emac.md)

# GIC　: Generic Interrupt Controller 関係マニュアル

- [GICv2アーキテクチャマニュアル](gic/gicv2_manual.md)
- [GIC400テクニカルマニュアル](gic/gic400_tech_manual.md)

# Pineを楽しむ: Genodians

- [ウォーミングアップ](genodians/warming_up.md)
- [ベアメタルシリアル出力](genodians/baremetal.md)
- [カーネルスケルトン](genodians/kernel.md)
- [Linuxを散歩に連れ出す](genodians/taking_linux.md)

# その他

- [AdamRLukaitis/pine64: README](others/README_adam.md)
- [apritzel/pine64: README](others/README_apritzel.md)
- [apritzel/pine64: Booting](others/Booting_apritzel.md)
- [Pine A64はDockerを実行する最も安価なARM64ビットプラットフォームになろうとしている](others/docker_pirates_20160125.md)
- [Pine64+のデバイスツリー](others/devicetree.md)

# 作業記録

- [AdamRLukaitis/pine64の追試](memo/adam_test.md)
- [Pine A64+へのLinuxのインストール](memo/install.md)
- [Buildrootによるインストール](memo/buildroot.md)
- [sunxi-felコマンドの実行](memo/fel.md)
- [tftp経由のプログラムロード](memo/tftp.md)
- [U-BootからLinuxを実行](memo/uboot_linux.md)
- [MacのSDXCカードスロットをVirtualBoxから使用する](memo/virtualbox.md)

# pine64+でxv6を動かす

- [macでu-bootをSDカードに書き込む](xv6/dd_uboot.md)
- [u-bootからプログラムを起動する](xv6/u-boot.md)