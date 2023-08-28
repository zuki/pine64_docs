# Allwinner SoCベースのボード

[オリジナル](https://u-boot.readthedocs.io/en/latest/board/allwinner/sunxi.html)

Allwinner ARMベースのSoC（"sunxi"）を使用したボード用に、U-Bootビルド
システムは単一の統合イメージファイル: `i-boot-sunxi-with-spl.bin`を
生成します。このファイルはSDカードやeMMCデバイス、SPIフラッシュ、USB-OTG
ベースのブート方法（FEL）で使用できます。このファイルをビルドするには

- 64ビットSoCの場合は、まずTrusted Firmware（TF-A、以前はATFとして知られて
  いた）をビルドし、その内の`bl31.bin`が必要となります。詳細は以下を参照
  してください。
- 64ビットSoCのオプションとして[crust](https://github.com/crust-firmware/crust)
  管理プロセッサファームウェアをビルドします。詳細は以下を参照してください。
- U-Bootをビルドします。

  ```bash
  $ export BL31=/path/to/bl31.bin       # 64-bit SoCでは必須
  $ export SCP=/path/to/scp.bin         # 64-bit SoCではオプション
  $ make <yourboardname>_defconfig
  $ make
  ```

- (micro-)SDカードに転送します（詳細は下記参照）。

  ```bash
  $ sudo dd if=u-boot-sunxi-with-spl.bin of=/dev/sdX bs=8k seek=1
  ```

- ブートして楽しんで。

> **注意**: AllwinnerのBootROMがロードする従来のSDカードの位置は8KB
> （セクター16）です。これはほとんどのSDカードがフォーマットされている
> 古いMBRパーティションスキームで問題なく動作します。しかし、これはGPT
> パーティションテーブルの途中であり、このステップでは無効になります。
> 新しいSoC（2014年後半のH3から）は128KBからのブートもサポートしています。
> これはGPTの枠を超えており、より安全な場所です。

詳細や代替のブート場所、インストールについては以下を参照してください。

## Arm Trusted Firmware (TF-A)のビルド

64ビットSoc（A64、H5、H6、H616、R329）を使用するボードには、[Arm Trusted Firmware-A](https://www.trustedfirmware.org/projects/tf-a/)
ファームウェアのBL31ステージが必要です。これはPSCIとSMCCCサービスを提供する
Armv8-A用のセキュアソフトウェアのリファレンス実装を提供します。Allwinnerの
サポートは完全にメインライン化されています。`bl31.bin`は次のように
ビルドします。

```bash
$ git clone https://git.trustedfirmware.org/TF-A/trusted-firmware-a.git
$ cd trusted-firmware-a
$ make CROSS_COMPILE=aarch64-linux-gnu- PLAT=sun50i_a64 DEBUG=1
$ export BL31=$(pwd)/build/sun50i_a64/debug/bl31.bin
```

ターゲットプラットフォーム（PLAT=）には、A64 SoCとH5 SoCには`sun50i_a64`、
H6 SoCには`sun50i_h6`、H616 SoCには`sun50i_h616`、R329 SoCには`sun50i_r329`
を指定します。サポートされているすべてのプラットフォームは次のコマンで
見つけることができます。

```bash
$ find plat/allwinner -name platform.mk
```

TF-Aの[docs/plat/allwinner.rst](https://trustedfirmware-a.readthedocs.io/en/latest/plat/allwinner.html)にはより多くの情報とビルドオプションがあります。

## Crust管理プロセッサファームウェアのビルド

一部のSoCやボードではOpenRISC統合管理コントローラを使用して、RAMの
サスペンドを筆頭とする電源管理サービスを提供することができます。
[crust](https://github.com/crust-firmware/crust)と呼ばれるコミュニティが
サポートするオープンソースの実装があり、管理コントローラを搭載した
ほとんどのSoCで動作します。

このファームウェア部分はオプションであり、SCP環境変数に/dev/nullを設定する
と環境変数なしでビルドした場合の警告メッセージを回避できます。

crustの`scp.bin`をビルドするにはOpenRISC（or1k）用のクロスコンパイラが
必要です。

```bash
$ git clone https://github.com/crust-firmware/crust.git
$ cd crust
$ make <yourboard>_defconfig
$ make CROSS_COMPILE=or1k-none-elf- scp
$ export SCP=$(pwd)/build/scp/scp.bin
```

[configs/](https://github.com/crust-firmware/crust/tree/master/configs)
ディレクトリにサポートされているボード構成のリストがあります。
[crustのREADME](https://github.com/crust-firmware/crust/blob/master/README.md#building-the-firmware)には
OpenRISCクロスコンパイラの入手先など、ビルドプロセスに関する詳細情報が
あります。

## U-Bootイメージのビルド

まず、使用するボードのU-Boot defconfigファイルを見つけてください。
これらのファイルは`configs/`ディレクトリにあります。知っていれば
devicetreeファイルのスタブ名で、またはSoC名でgrepすることで正しい
バージョンを見つけることができます。

```bash
$ git grep -l MACH_SUN8I_H3 configs
$ git grep -l sun50i-h6-orangepi-3 configs
```

[linux-sunxi](https://linux-sunxi.org/)のwikiには各ボードのページに
defconfigファイルの名前が記載されています。このdefconfigファイルを使って
`.config`ファイルを作成し、イメージをビルドします。

```bash
$ make <yourboard>_defconfig
$ make
```

64ビットボードの場合、（上記のTF-Aビルドの例に示されているように）BL31環境
変数を設定するか、ビルドコマンドラインで指定する必要があります。

```bash
$ make BL31=/src/tf-a.git/build/sun50i_h616/debug/bl31.bin
```

これは(オプションの)SCPファームウェアについても同様です。

必要なすべてのものを含んだファイルは`u-boot-sunxi-with-spl.bin`という名前で、
U-Boot（ビルド）ツリーのルートフォルダにあります。生のNANDフラッシュ
デバイスを除いて、この同じファイルをどんなブートソースにも使えます。
このファイルには、BROMにより認識される適切な署名と必要なチェックサムを持つ
SPLイメージが含まれています。また、少なくとも、レガシーなU-Bootイメージ
フォーマットかFITイメージのいずれかにラップされているU-Bootが含まれています。
ボードのdevicetreeも含まれています。これはU-Bootイメージ本体に追加されているか
FITイメージに含まているかのいずれかです。SoCが必要とする場合はこのFIT
ファイルにはその他のファームウェアイメージも含まれます。

## U-Bootのインストール

Allwinner SoCはすべて最初のMMCコントローラに接続されたSDカードのセクタ16
（8KB）でブートイメージを見つけようとします。(micro-)SDカードリーダを持つ
Linuxデバイス(ボード自身を含む)から生成されたイメージをSDカードに転送する
には、次のようにタイプします。

```bash
$ sudo dd if=u-boot-sunxi-with-spl.bin of=/dev/sdX bs=1k seek=8
```

`/dev/sdx`はSDカードリーダのブロックデバイス名に置き換える必要があります。
一部のマシンではこれは`/dev/mmcblkX`になります。新しいSoC（2014年のH3から
始まり、すべてのARM64 SoCを含みます）はセクタ256（128KB）に署名がないかも
チェックします（8KBの場所をチェックした後）。ここにファームウェアを
インストールするとGPTパーティションテーブルと重ならないという利点があります。
上記のコマンドで"seek=8"を"seek=128"に置き換えるだけです。

既存の（mainline）U-Bootを使ってSDカードに書き込むこともできます。生成した
U-BootイメージをDRAMのどこかにロードし（`ext4load`、`fatload`、`tftpboot`の
いずれかを使用）、MMCデバイス0に書き込みます。

```bash
=> fatload mmc 0:1 $kernel_addr_r u-boot-sunxi-with-spl.bin
=> mmc dev 0
=> mmc write $kernel_addr_r 0x10 0x7f0
```

新しいSoCで別のブートロケーションを使用する場合は次のように指定します。

```bash
=> mmc write $kernel_addr_r 0x100 0x700
```

[以下、eMMC, SPI Flashへのインストールは省略]

### USB(-OTG) FELモード経由のブート

BROMがチェックしたブートロケーションのいずれにも媒体または有効な署名が
含まれていない場合、BROMはいわゆるFELモードに入り、SoCのUSB-OTGインタ
フェース上でホストからのコマンドを待ち受けます。これらのコマンドにより
任意のメモリロケーションからの読み取りと書き込みが可能になり、また、
任意のアドレスから実行を開始することができるため、USBケーブルだけで
ボードをブートストラップすることができます。一部のボードは有効なブート
ロケーションが存在してもFELモードを強制する"FEL"または"U-Boot"ボタンを
備えています。同じことがSDカードに[マジックバイナリ](https://github.com/linux-sunxi/sunxi-tools/raw/master/bin/fel-sdboot.sunxi)を
置くことで実現でき、任意のボードでFELモードに入ることができ瑠葉になります。

FELブートを使用するには上で説明したいずれかの方法（メディアなしのブート、
FELボタン、FELバイナリを有するSDカード）でボードをFELモードにし、USB
ケーブルをボードのUSB OTGポートに接続します。一部のボード（Pine64、
TVボックス）には独立したOTGポートがありません。この場合、ほとんどは
USB-Aポートの1つがUSB0に接続されており、非標準のUSB-A-USB-Aケーブルで
使用することができます。

通常、接続されたホストコンピュータに新しいUSBデバイスが表示される以外に
FELモードであることを示すボード上の表示はありません。USBベンダ/デバイスIDは
`1f3a:fe8`です。ほとんどの場合、これは"sunxi SoC OTG connector in FEL/flashing
mode"と認識されますが、古いディストリビューションでは"Onda (unverified) V972 tablet in flashing mode"と報告されることがあります。

[sunxi_fel](https://github.com/linux-sunxi/sunxi-tools)ツールは独自の
BROMプロトコルを実装しており、定評のある`u-boot-sunxi-with-spl.bin`を
提供するだけでU-Bootをブートストラップすることができます。

```bash
$ sudo apt-get install sunxi-tools
$ sunxi-fel uboot u-boot-sunxi-with-spl.bin
```

カーネルや初期ラムディスク、ブートスクリプトなどの追加バイナリもFEL経由で
アップロードすることができます。詳細はWikiの[FELページ](../wiki/usb_boot.md)を
確認してください。
