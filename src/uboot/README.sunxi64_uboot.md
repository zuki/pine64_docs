# U-Boot: Allwinner 64-bitボートのREADME

[オジリナル](https://github.com/u-boot/u-boot/blob/master/board/sunxi/README.sunxi64)

新しいAllwinnerのSoCはARMv8コア（ARM Cortex-A53）を搭載し、64ビットの
AArch64モードとARMv7互換の32ビットのAArch32モードの両方をサポートして
います。Allwinner A64（たとえばPine64ボードで使用）やAllwinner H5 SoC
（OrangePi PC 2で使用）がその例です。これらのSoCはリセット時はAArch32
モードで起動し、ブートROM（BROM）から32ビットコードを実行するように
配線されています。これはU-Bootに影響を与えるため、このファイルでは
64ビット機能をフルに活用する方法を説明します。

# クイックスタート / 概要

- ATF (ARM Trusted Firmware)バイナリをビルドします（下の[ARM Trusted Firmware (ATF)](#arm_trusted_firmware_atf)を参照）
  ```bash
  $ cd /src/arm-trusted-firmware
  $ make PLAT=sun50i_a64 DEBUG=1 bl31
  ```
- SCPファームウェアバイナリをビルドします（下の[SCPファームウェア (Crust)](#scpファームウェア_crust)を参照）
  ```bash
  $ cd /src/crust
  $ make pine64_plus_defconfig && make -j5 scp
  ```
- U-Bootをビルドします（下の[SPL/U-Boot](#spl_u-boot)を参照）
  ```bash
  $ export BL31=/path/to/bl31.bin
  $ export SCP=/src/crust/build/scp/scp.bin
  $ make pine64_plus_defconfig && make -j5
  ```
- mivtoSDカードに書き込みます（下の[microSDカード](#microSDカード)を参照）
  ```bash
  $ dd if=u-boot-sunxi-with-spl.bin of=/dev/sdx bs=8k seek=1
  ```
- ブートして楽しんで!

# ファームウェアのビルド

Allwinner A64/H5/H6のファームウェアは次のようにいくつかの部分で構成
されています: U-BootのSPL、ARM Trusted Firmware (ATF)、オプションの
System Control Processor (SCP)ファームウェア（Crustなど）、そして、
U-Boot本体です。

SPLは他のすべてのファームウェアバイナリを適切なデバイスツリーブログ
(.dtb) とともにRAMにロードし、実行を（EL3で）ATFに渡します。SCP
ファームウェアがロードされていた場合、ATFはSCPの電源を投入し、ブート
するのを待ちます。その後、ATFは（EL2で）U-Bootに入ります。

ATFバイナリとSCPファームウェアはU-Bootイメージファイルの一部となる
ため、まずそれらをビルドする必要があります。

## ARM Trusted Firmware (ATF)

[公式ATFリポジトリ][link1]から最新のマスターブランチをチェックアウトし、
ビルドします。

```bash
$ export CROSS_COMPILE=aarch64-linux-gnu-
$ make PLAT=sun50i_a64 DEBUG=1 bl31
```

できあがったバイナリは`build/sun50i_a64/debug/bl31.bin`です。この
ファイルの場所を環境変数`BL31`に入れるか、U-Bootのビルドディレクトリの
ルートにコピーします（またはシンボリックリンクを作成します）。

```bash
$ export BL31=/src/arm-trusted-firmware/build/sun50i_a64/debug/bl31.bin
```

（実際のパスに従って調整してください）プラットフォームターゲット
"sun50i_a64" はAllwinner A64またはH5 SoCを搭載しているすべてのボードに
対応しています（両者は非常によく似ているためです）。Allwinner H6 SoCを
搭載しているボードには "sun50i_h6" を使用してください。

できあがったU-Bootイメージファイルのサイズに問題がある場合は、bl31.binを
ビルドする際に "DEBUG=0 "を指定して、リリースビルドを使用するとよいかも
しれません。ATFのビルドプロセスでは使用するツールチェーンについて少し
うるさいことがあります。この問題があった場合、または、ATFをビルドするのが
面倒な場合は、[ファームウェアリポジトリ][link3]に動作可能なバイナリが
あります。これを使用するのは単に利便性のためです。

## SCPファームウェア (Crust)

SCPファームウェアはシステムのサスペンド/レジュームと（PMICを持たない
ボードで）ソフトパワーオフ/オンの実装を担当します。ATFにはCPUの電源を制御
するフォールバックコードが含まれているため、これらの機能を必要としない
場合はSCPファームウェアはオプションです。SCPファームウェアはARMではなく
or1k CPUのAR100上で動作するため別のクロスツールチェインが必要です。

現在利用可能なSCPファームウェアの実装の1つはCrustです。

```bash
$ git clone https://github.com/crust-firmware/crust
$ cd crust
$ export CROSS_COMPILE=or1k-linux-musl-
$ make pine64_plus_defconfig
$ make scp
```

一般に、同じSoC（A64、H5、H6）を搭載したボードはどれでも同じconfig
で動作するので使用するボード用のconfigがない場合は類似したボード用の
configを使用してください。

ATFの場合と同様に、U-Bootは環境変数経由でSCPファームウェアのバイナリを
見つけます。

```bash
$ export SCP=/src/crust/build/scp/scp.bin
```

SCPファームウェアを使いたくない場合は、空のファイルを指定することで
binmanからの警告を消すことができます。

```bash
$ export SCP=/dev/null
```

## SPL/U-Boot

U-Boot本体もSPLも64ビットモードを使っています。ブートROMはAArch32の
セキュアSVCモードのままSPLに入るため、非常に早い段階でAArch64に入る
ためのシムコードが存在します。その後、SPLはAArch64 EL3で動作します。
U-Boot本体はEL2で動作し、任意のAArch64コード（"go"コマンドを使用）、
EFIアプリケーション（"bootefi"コマンドを使用）、arm64 Linuxカーネルイメージ
（通常は"Image"という名前）（"booti"コマンドを使用）をロードできます。

```bash
$ make clean
$ export CROSS_COMPILE=aarch64-linux-gnu-
$ make pine64_plus_defconfig
$ make
```

これはSPLである`spl/sunxi-spl.bin`と残りのファームウェアを含む
`u-boot.itb`と呼ばれるFITイメージをビルドします。
`u-boot-sunxi-with-spl.bin`はこれら2つのコンポーネントを1つの
イメージファイルに結合したものです。

# ブートプロセス

CPU内蔵のBROMコードはファームウェアをロードして実行するために
いくつかの方法を試みます。Pine64のような典型的なボードでは次のような
ブート順になります。

1. microSDカードのセクタ16 (@8K) からSRAM A1に32KBを読み込みます。
   BROMは先頭バイトにマジックヘッダー "eGON" を見つけたらそのコードを
   実行します。そうでない場合（SDカードが挿入されていないか、無効な
   マジックの場合は）、2を実行します。
2. MMC2コントローラ（通常はオンボードのeMMCチップ）に接続されている
   メモリのセクタ16 (@8K) から32KBの読み込みを試みます。eMMCがないか
   有効なブートヘッダが含まれていない場合は、3を実行します。
3. SPI0コントローラを初期化し（CS0ピンを使用して）接続されている
   NORフラッシュへのアクセスを試みます。フラッシュチップが見つかったら
   BROMは最初の32KBを（オフセット0から）SRAM A1にロードします。
   次に、マジックヘッダー eGON とチェックサムをチェックし、問題がなければ
   コードを実行します。問題があった場合は、4を実行します。
4. USB OTGコントローラを初期化し、ホストが接続するのを待ちます。
   ホストとはAllwinner独自の（しかし解読されている）"FEL" USBプロトコルを
   使用します。

Pine64ボードをブートするにはU-Bootと上の方法のいずれかを使用することが
できます。

## FELブート (USB OTG)

FELとは、ほとんどのAllwinner SoCのマスクROMに組み込まれている
Allwinner定義のUSBブートプロトコルの名前です。USB-OTGインタフェースと
他のコンピュータのホストポートを使用して、ボードをブートストラップ
することができます。FELモードはブートROMによって制御されるため、
AArch32で実行されることが期待されます。今のところ、AArch64 SPLは
正しくFELモードに復帰することができないため、この機能はconfigで無効に
なっています。[このリポジトリ][link3]には、32ビットARMコードを生成する
非公開ブランチを使用してビルドされた、FEL対応のSPLバイナリが含まれて
います（再作成する方法の説明もあります）。

## microSDカード

SPLとU-Boot FITイメージを次のコマンドで直接microSDカードに転送します。

```bash
# dd if=spl/sunxi-spl.bin of=/dev/sdx bs=8k seek=1
# dd if=u-boot.itb of=/dev/sdx bs=8k seek=5
# sync
```

（/dev/sdxは手元の環境のSDカードデバイスファイルに置き換えてください。
/dev/mmcblk[x] のような名前かもしれません）。

別の方法として、SPLとU-Boot FITイメージを一つにまとめたファイルを転送する
事もできます。

```bash
# dd if=u-boot-sunxi-with-spl.bin of=/dev/sdx bs=8k seek=1
```

microSDカードをパーティショニングすることができますが、最初のMBは
未割り当てのままにしておきます（ほとんどのパーティショニングツールは
いずれにせよこのようにしています）。

## NORフラッシュ

一部のボード（SoPine、Pinebook、OrangePi PC2など）にはSPI NORフラッシュ
チップが搭載されています。Pine64などの他のボードではそのようなチップを
PI-2ヘッダーのSPI0/CS0ピンに接続することができます。SDカード用に上で
説明したようにSPLとFITイメージを作成します。次に、Pine64のUSBポートの
上段に"A to A" USBケーブルを接続するか、アダプタを入手して通常の
"A-microB"ケーブルを接続してください。他のボードでは通常、USB OTBポートに
適切なmicroB USBソケットが接続されています。microSDカードをスロットから
取り出してボードの電源を入れてください。ホストコンピューターで
[sunxi-toolsパッケージ][link2]をダウンロード・ビルドし、"sunxi-fel"を
使ってボードにアクセスします。

```bash
$ ./sunxi-fel ver -v -p
```

この結果、`AWUSBFEX soc=00001689(A64) ...`で始まる出力がされるはずです。
そうしたら、sunxi-felツールを使ってNORフラッシュに書き込みます。

```bash
$ ./sunxi-fel spiflash-write 0 spl/sunxi-spl.bin
$ ./sunxi-fel spiflash-write 32768 u-boot.itb
```

SDカードを挿入しないでボートをブートします。すると、シリアルコンソールに
U-Bootのプロントが現れるはずです。

## (Legacy) boot0による方法

`boot0`はAllwinnerのセカンダリプログラムローダです。SPLの代わりとして
U-BootをmicroSDカードから起動させるために使うことができます。しばらくの間、
boot0の使用がPine64を起動させる唯一の選択肢でした。U-BootのSPLでDRAM init
コードが動作するようになったため、この方法は必要なくなりましたが、
念のためここで説明しておきます。この方法は、A64ベースのボードに同梱されて
いるboot0ファイルでしか動作しないことに注意してください。互換性のない
レイアウトを使用しているH5はこの方法はサポートされていません。

boot0バイナリは32 KByteのblobであり、Pine64、または、Allwinnerが配布
している公式のPine64イメージに含まれています。マイクロSDカードや
イメージファイルから簡単に抽出できます。

```bash
# dd if=/dev/sd<x> of=boot0.bin bs=8k skip=1 count=4
```

ここで、`/dev/sd<x>`はmicroSDカードのデバイス名かイメージファイル名の
いずれかです。どうやらAllwinnerはこのプロプライエタリなコードを"そのまま"
再配布することを許可しているようです。このboot0 blobは、DRAMの初期化を
行い、残りのファームウェア部分をロードし、コアをAArch64モードに切り替え
ます。オリジナルのboot0コードはmicroSDカードの特定の場所 (19096KB) で
U-Bootを探し、そこにマジックバイトとチェックサムを含むヘッダがあることを
期待します。[boot0img][link3]というツールがあり、`boot0.bin`イメージと
コンパイルされたU-Bootバイナリ (とその他のバイナリ) を受け取り、それに
応じてヘッダを生成します。マジックヘッダ用のスペースを確保するために、
`pine64_plus_defconfig`はU-Bootバイナリの先頭に十分なスペースを設けます。
boot0imgは様々なバイナリをmicroSDカードの適切な場所に配置し、トランポリン
コードを使って未使用だが必須の部分を回避することも行います。詳細は
"boot0img -h"の出力を見てください。boot0imgは19MBからのU-Bootのロードを
避け、代わりにboot0バイナリのすぐ後ろからフェッチするようにboot0に
パッチを当てることもできます（-Bオプション）。

```bash
$ ./boot0img -o firmware.img -B boot0.img -u u-boot-dtb.bin -e -s bl31.bin \
-a 0x44008 -d trampoline64:0x44000
```

そして、このイメージをmicroSDカードに書き込みます。/dev/sdxは正しい
デバイスファイル（上記参照）に置き換えてください。

```bash
$ dd if=firmware.img of=/dev/sdx bs=8k seek=1
```


[link1]: https://github.com/ARM-software/arm-trusted-firmware.git
[link2]: git://github.com/linux-sunxi/sunxi-tools.git
[link3]: https://github.com/apritzel/pine64/
