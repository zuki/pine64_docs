# Pin4 A64+へのLinuxのインストール

```bash
$ lsb_release -d
Description:	Ubuntu 22.04.3 LTS
$ aarch64-linux-gnu-gcc --version
aarch64-linux-gnu-gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
```

## ATFのインストール

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

## SCP (Crust)のインストール

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

## SPL/U-Bootのインストール

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

# Linuxのインストール

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
