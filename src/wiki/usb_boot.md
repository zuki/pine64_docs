# FEL/USBBoot

[オリジナル](https://linux-sunxi.org/FEL/USBBoot)

[対応しているAllwinner SoC](https://linux-sunxi.org/FEL/USBBoot#SoC_support_status)では
USB OTG経由でブートすることができます。これは「マニュアルビルドの
手引」にあるビルドステップと比較して最小限の変更しか必要としません。
この変更についてこのWikiページで説明します。

USB OTG経由でブートすることによりSDカードを完全に省くことができます。[UART](https://linux-sunxi.org/UART)がないデバイスやSDカードと多重化
されているデバイスでは特に便利です。[micro-SDブレークアウトアダプタ](https://linux-sunxi.org/MicroSD_Breakout)を
使用すればシリアルポートにアクセスできます。

## ツールのインストール

[sunxi-toolsリポジトリ](https://linux-sunxi.org/Sunxi-tools)に
`sunxi-fel`というユーティリティがあります。このユーティリティを
使用してUSB経由でシステムを起動できます。そのためにはまず
インストールする必要があります。

`sunxi-fel`ユーティリティのコマンドライン構文は次のとおりです。

```bash
Usage: sunxi-fel [options] command arguments... [command...]
        -v, --verbose                   Verbose logging
        -p, --progress                  "write" transfers show a progress bar
        -l, --list                      Enumerate all (USB) FEL devices and exit
        -d, --dev bus:devnum            Use specific USB bus and device number
            --sid SID                   Select device by SID key (exact match)

        spl file                        Load and execute U-Boot SPL
                If file additionally contains a main U-Boot binary
                (u-boot-sunxi-with-spl.bin), this command also transfers that
                to memory (default address from image), but won't execute it.

        uboot file-with-spl             like "spl", but actually starts U-Boot
                U-Boot execution will take place when the fel utility exits.
                This allows combining "uboot" with further "write" commands
                (to transfer other files needed for the boot).

        hex[dump] address length        Dumps memory region in hex
        dump address length             Binary memory dump
        exe[cute] address               Call function address
        reset64 address                 RMR request for AArch64 warm boot
        memmove dest source size	    Copy <size> bytes within device memory
        readl address                   Read 32-bit value from device memory
        writel address value            Write 32-bit value to device memory
        read address length file        Write memory contents into file
        write address file              Store file contents into memory
        write-with-progress addr file   "write" with progress bar
        write-with-gauge addr file      Output progress for "dialog --gauge"
        write-with-xgauge addr file     Extended gauge output (updates prompt)
        multi[write] # addr file ...    "write-with-progress" multiple files,
                                        sharing a common progress status
        multi[write]-with-gauge ...     like their "write-with-*" counterpart,
        multi[write]-with-xgauge ...      but following the 'multi' syntax:
                                          <#> addr file [addr file [...]]
        echo-gauge "some text"          Update prompt/caption for gauge output
        ver[sion]                       Show BROM version
        sid                             Retrieve and output 128-bit SID key
        clear address length            Clear memory
        fill address length value       Fill memory

	spiflash-info                       Retrieves basic information
	spiflash-read addr length file      Write SPI flash contents into file
	spiflash-write addr file            Store file contents into SPI flash
```

**警告**: Linuxディストリビューションの中には`sunxi-tools`パッケージ
を提供しているものもあります。しかし、各ディストリビューションの判断で
特定のツールを削除したり、実行ファイルの名前を変更したりしているものが
あります。使用しているディストロのsunxi-toolsパッケージで問題が発生
した場合は[githubリポジトリ](https://github.com/linux-sunxi/sunxi-tools)
から直接sunxi-toolsを入手するのも一つの方法です。

## デバイスのFELモードへの切り替え

`sunxi-fel`ツールを使って実際にデバイスを操作するには、まず
"USB A to USB mini/micro B"ケーブルを使ってデバイスをPCに接続する
必要があります。

次に、デバイスをFELモードに切り替える必要があります。FELモードで
ブートする方法については[FELの手引](https://linux-sunxi.org/FEL)を
参照してください。デバイスのページにはFELモードに切り替えるボタンに
ついても記載されているはずです。

```bash
$ sunxi-fel version
```

を実行すると、次のような結果が返ります。

```bash
AWUSBFEX soc=00162500(A13) 00000001 ver=0001 44 08 scratchpad=00007e00 00000000 00000000
```

これでデバイスはFELモードに切り替わり、USB経由でコマンドを受け取ったり、
システムをロードしたりする準備が整いました。

## USB経由のシステムのブート

### Mainline U-Boot (v2015.04以降のバージョン)

#### mainline U-Bootのソースの取得

U-Bootのソースを取得するには最新のU-Bootのmasterブランチを
クローンします。

```bash
$ git clone https://source.denx.de/u-boot/u-boot
$ cd u-boot
```

あるいは、sunxiが管理するリポジトリの"next"ブランチをクローンします。

```bash
$ git clone -b next https://source.denx.de/u-boot/custodians/u-boot-sunxi
$ cd u-boot-sunxi
```

これらのブランチはどちらも開発版であるので時にバグが含まれている
可能性があります。問題が発生した場合は、あきらめる前に最新の正式な
U-Bootリリースのtarballを試してください。

#### USB経由のu-Bootのブート

通常通りの方法でU-Bootをビルドします。

```bash
$ make CROSS_COMPILE=arm-linux-gnueabihf- Cubietruck_defconfig
$ make CROSS_COMPILE=arm-linux-gnueabihf- -j$(nporc)
```

64ビットのSoCでは次のようにビルドします。

```bash
$ make CROSS_COMPILE=aarch64-linux-gnu- <your-board>_defconfig
$ make CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc) BL31=/path/to/ trusted-firmware/build/sun50i-a64/debug/bl31.bin SCP=/dev/null
```

正しいTrusted-Firmware (TF-A, あるいはATF)を選択してください。
A64とH5では`sun50i-a64`、H6では`sun50i-h6`、H616/H313では
`sun50i-h616`です。

そしてUSBでブートします（'sunxi-fel uboot'コマンドは最新の
sunxi-toolsが必要です）。

```bash
$ sunxi-fel uboot u-boot-sunxi-with-apl.bin
```

#### USB経由の全システムのブート (U-Boot + カーネル + initramfs)

これには（'sunxi-fel'によってRAMにアップロードされた'boot.scr'
ブロブを自動的に見つけることができる）v2015.10以降のU-Bootを使用
する必要があります。

カーネルイメージとハードウェア用のdtbブロブ、[ブートスクリプトブロブ](https://linux-sunxi.org/Mainline_U-Boot#Install_U-Boot)、
initrdイメージを用意すれば、'sunxi-fel'を1回実行するするだけでこれら
すべてをブートすることができます。

```bash
$ sunxi-fel -v uboot u-boot-sunxi-with-spl.bin \
             write 0x42000000 uImage \
             write 0x43000000 sun7i-a20-cubietruck.dtb \
             write 0x43100000 boot.scr \
             write 0x43300000 rootfs.cpio.lzma.uboot
```

64ビットSoCの場合は、

```bash
$ sunxi-fel -v uboot u-boot-sunxi-with-spl.bin \
             write 0x40200000 Image \
             write 0x4fa00000 sun50i-a64-pine64-lts.dtb \
             write 0x4fc00000 boot.scr \
             write 0x4ff00000 rootfs.cpio.lzma.uboot
```

上のコマンドラインで使用されているすべてのマジックアドレスについて疑問が
ある場合、正しい値は[U-Bootのソース](http://git.denx.de/?p=u-boot.git;a=blob;f=include/configs/sunxi-common.h)の`MEM_LAYOUT_ENV_SETTINGS`定義で見つける
ことができます（64ビット値については`#ifdef CONFIG_ARM64`を参照して
ください）。

```bash
bootm_size     = 0xa000000  (0xf000000)
kernel_addr_r  = 0x40080000 (0x42000000)
fdt_addr_r     = 0x4FA00000 (0x43000000)
scriptaddr     = 0x4FC00000 (0x43100000)
pxefile_addr_r = 0x4FD00000 (0x43200000)
ramdisk_addr_r = 0x4FF00000 (0x43300000)
```

**訳注**: 現在のarm64値。()内はオリジナルの値

U-Bootはv2017.03とv2017.07の間のどこかで新しいSPLフォーマットに切り替わり、
これはいくつかのディストロが提供するリリース <=1.4.2のsunxi-toolsと互換性が
ありません。この場合、ツールは次のようなエラーメッセージを表示します。

```bash
sunxi SPL version mismatch: found 0x02 > maximum supported 0x01.
You need a more recent version of this (sunxi-tools) fel utility.
```

この問題を避けるにはgitから最新のsunxi-toolsを入手する必要があります。

**bootcmdの例**

| 32ビットボート | 64ビットボード |
|:---------------|:--------------
| env set fdt_high ffffffff<br/>bootm 0x42000000 0x43300000 0x43000000 | booti 0x40200000 0x4ff00000 0x4fa00000 |

Buildroot（32ビットシステム用）を使用している場合は、以下のオプションを
有効にする必要があるかもしれません。

```bash
BR2_LINUX_KERNEL_UIMAGE=y
BR2_LINUX_KERNEL_UIMAGE_LOADADDR=0x42000000
BR2_TARGET_ROOTFS_CPIO=y
BR2_TARGET_ROOTFS_CPIO_LZMA=y
BR2_TARGET_ROOTFS_CPIO_UIMAGE=y
BR2_TARGET_ROOTFS_INITRAMFS=y
```

#### uEnvスタイルのデータによる環境変数の上書き

v1.4以降のsunxi-felユーティリティは。FEL経由でU-Bootに環境データを渡すことが
できるようになりました。

テキスト表現の`key=<value>`ペアを指定すると**デフォルト環境にマージ**されます。
U-Bootの`import -t`コマンドや古いU-BootがuEnv.txtファイルを使ってautoboot上で
行っていたことに似ています。

> 注意: これには最新のU-Boot（v2016.09以降）が必要であり、さらにデータは
> 特別な「マジック」署名で開始する必要があります

  ```bash
  #=uEnv
  ```

この署名はsunxi-felの"write"コマンドがuEnvスタイルのデータを検知できる
ようにします。そして、それに応じてこの転送にフラグを立て、それにより
U-Boot v2016.09+にインポートを要求します。この方法により`bootcmd`を含む
任意の環境変数を上書きすることができます。

##### 例

テキストエディタを使って以下を _my.env_ ファイルに保存します。

```bash
#=uEnv
myvar=world
bootcmd=echo "Helo $myvar."
```

次のコマンドでテストします。

```bash
$ ./sunxi-fel uboot u-boot-sunxi-with-spl.bin write 0x43100000 my.env
```

U-Bootのautobootが対応するメッセージを表示し、プロンプトに戻るはずです。
これはデフォルトの"bootcmd"の上書きに成功したことを示しています。

#### 新しいSoC用の早期カーネル開発

FELブートはBROMコードによりサポートされているため、すぐに利用できます。
`sunxi-fel`ツールにはSRAMのレイアウト情報を追加するためにパッチを当てる
必要があるだけです。これは[通常非常に簡単](https://github.com/linux-sunxi/sunxi-tools/commit/65bcc0501382db606ab5a5b5eb39f6f1c06b2df8)であり、
このページに詳細に記載されています。比較的難しいのはU-Bootのサポートです。
SPL用のDRAM初期化コードが必要だからです。しかし、U-Bootのブートローダがすでに
利用できると仮定すると、カーネルにイーサネットやMMCのサポートがまだない
場合でもFELブートをカーネル開発に使用することができます。便利なので
[fel-sdboot.sunxi](https://github.com/linux-sunxi/sunxi-tools/blob/master/bin/fel-sdboot.sunxi)を
SDカードに書いておくのがベストです（`/dev/sdX`は使用するカードリーダの
ブロックデバイス名に置き換えてください）。

```bash
$ wget https://github.com/linux-sunxi/sunxi-tools/raw/master/bin/fel-sdboot.sunxi
$ dd if=fel-sdboot.sunxi of=/dev/sdX bs=1024 seek=8
```

あとは[U-Boot + カーネル + initramfsに関するセクション](USB経由の全システムのブート_U-Boot_+_カーネル_+_initramfs)の指示を使うだけです。そして、カーネルで
MMCのサポートが実装されたら、rootfsをSDカードに移してinitrdイメージを削除する
ことが可能になります。しかし、イーサネットかUSBのどちらかがネットワーク
経由でのブートに十分対応できるようになるまではFEL USBブートは役にたち続ける
でしょう。

[以下、Legacy関係とWindows関係は省略]

## 新しいSoCのサポートの追加

既にすべてが問題なく動作しているのであれば、ここで読むのをやめても
構いません :-) しかし、上記のセクションの命令を実行すると
**"SPL: Unsupported SoC type"** や
**"Warning: no 'soc_sram_info' data for your SoC"** といったメッセージが
表示されることがあります。この場合、何らかのSoCのサポートを追加するための
作業が必要になるかもしれません。以下のSoCサポート状況表をまずチェックする
のも良いアイデアです。

### SoCのサポート状態

| SoC名 | sunxi-tools | U-Boot | "sunxi-fel write"速度 | USB DFU速度 | MMuの設定 | スタックポインタ | 注記 |
|:-----:|:------------|:-------|:-----------|:------|:-------:|:----|:-------|
| A64  | サポート済 | サポート済 | 〜510KB/s | &nbsp; | 不要 | sp_irq=0x12000, <br/>sp=0x15E08 | SPLは（0x0ではなく）0x10000にロードする必要がある |

**訳注**: 注の翻訳は省略。

[以下省略]
