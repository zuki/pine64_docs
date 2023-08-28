# aritzel/pine64のREADME

[オリジナル](https://github.com/apritzel/pine64/blob/master/README.md)

## 画像ファイルの使用方法

### クイックスタート

ファームウェアイメージを取り出してmicroSDカードにフラッシュします。
`sdX`はSDカードのデバイスファイル名に置き換えてください。

**警告**:

この操作によりSDカードの先頭の1メガバイトのほとんどが上書きされますが
MBRパーティションテーブルはそのまま残ります。最初のパーティションが
1MB以下から始まる場合はデータが失われますので、まずSDカード二保存されてい
いる休日の写真をバックアップすることをお勧めします。

```bash
# xzcat pine64_firmware-xxxxx.img.xz | dd of=/dev/sdx bs=1k seek=8
# sync
```

このファームウェアイメージにはプライマリブートローダ（U-Boot SPL）、
ARM Trusted Firmware（ATF）、デバイスツリーファイル（DTB）、実際の
U-Bootイメージが含まれています。

- PCをPine64のUART0に接続し（[Pine64 linux-sunxi Wiki](http://linux-sunxi.org/Pine64#Serial_port_.2F_UART)を参照）、ターミナルプログラムを115200n8の
  設定で接続します。または、モニターをHDMIポートに接続し、USBキーボードを
  Pine64の下段のUSBソケットに接続します。
- Pine64にmicroSDカードを入れ、電源を入れます。
- U-Bootプロンプトをタイムアウトさせると、SDカード、USB大容量記憶装置
  （ペンドライブ、ハードディスク）にあるカーネルを探し始めます。または
  PXEネットワークブートを試みます。デバイスから標準準拠のEFIアプリ
  ケーションを自動的に検出するので（カーネルが64ビットのAllwinner
  デバイスをサポートしていれば）ディストリビューションのインストーラも
  動作するはずです。

### ルートファイルシステムの追加

イメージにはファームウェアしか含まれておらず、それ以上のデータや
パーティションは含まれていません。1MB以下の領域がそのまま残されていれば
任意のパーティションを追加（または再利用）してもかまいません（ほとんどの
パーティショニングツールはそうなっています）。推奨されるレイアウトは
約100MBのEFIシステムパーティション（ESP）を最初のパーティションとし、
その後にLinuxのルートパーティションとデータパーティションを配置する
ことです。

通常、.dtbファイルを用意する必要はありません。U-Bootイメージに含まれている
バージョンを使うことができるからです。DTBのロードアドレスを指定する必要が
ある場合は`$fdtcontroladdr`を使用するだけです。

```bash
=> booti $kernel_addr_r - $fdtcontroladdr
```

### ファームウェアイメージの再構築

Pine64ファームウェアは次の4つの部分で構成されています。

- オンチップブートROM（BROM）。これは変更できません。コードをロードする
  最初のステップを行います。A64 SoCの一部であるため、ここには含まれて
  いません。
- セカンダリログラムローダ（SPL）。SPLの主な役割はDRAMを初期化し、ファーム
  ウェアの残りの部分をロードすることです。BROMの制限により、SPLのサイズは
  32Kに制限されています。SPLはU-Bootの一部であり、U-Bootビルド後に
  `spl/sunxi-spl.bin`として見つけることができます。
- EL3ランタイムファームウェア。このコードの役割はPSCIのようなランタイム
  サービスを提供することです。生涯常駐し、LinuxのようなOSから
  呼び出すことができ、セカンダリコアの有効/無効化やその他のサービスを
  要求することができます。また、低レベルのCPU初期化やエラッタ処理も
  行います。A64 SoCのサポートは公式のmainline ATFリポジトリにあります。
- U-Bootブートローダ。ユーザインタフェースを提供し、カーネルやその他の
  データをメモリにロードして最終的にシステムを起動できるようにします。
  Pine64ボードはバージョン2016.07-rc1以降、upstream U-Bootでサポート
  されています。

ファームウェアイメージを再構築するには、以下が必要です。

#### ARM Trusted Firmware (ATF)のビルド

最新バージョンをチェックしてコンパイルします。

```bash
$ git clone https://github.com/ARM-software/arm-trusted-firmware.git
$ cd arm-trusted-firmware
$ export CROSS_COMPILE=aarch64-linux-gnu-
$ make PLAT=sun50i_a64 DEBUG=1 bl31
```

できあがったバイナリは`build/sun50i_a64/debug/bl31.bin`です。この
ファイルをU-Bootのソースディレクトリのルートにコピーするか、絶対
ファイル名を環境変数に`BL31`に設定します。

#### U-Bootのビルド

最新のupstream HEADをチェックアウトしてコンパイルします。

```bash
$ git clone git://git.denx.de/u-boot.git
$ cd u-boot
$ export CROSS_COMPILE=aarch64-linux-gnu-
$ make pine64_plus_defconfig
$ make
```

SPL部分は`spl/sunxi-spl.bin`に、ファームウェアの残りの部分
（ATFバイナリ、DTB、U-Boot本体を含む）は`u-boot.itb`になります。

`u-boot-sunxi-with-spl.bin`は1つのイメージファイルに上の2つを含んでおり
SDカードにそのまま書き込む事ができます。

#### SDカードへの書き込み

これでできあがったイメージをマイクロSDカードのセクタ16に書き込むことが
できます。

```bash
# dd if=u-boot-sunxi-with-spl.bin of=/dev/sdX bs=1k seek=8
```

**警告**: これによりSDカードの一部が上書きされますので事前にデータを
バックアップしてください！ **警告**

`dev/sdX`は死傷するSDカードのブロックデバイス名に置き換えてください。
ほとんどのARMボードと非USBカードリーダでは`/dev/mmcblk0`（末尾の番号は
異なる場合があります）になります。

#### SDカードのパーティショニング

ファームウェアはSDカードの8KBから最大1MBまでの領域を占有します。
カードの残りの部分は自由に使うことができます。たとえば、先頭に
MBRパーティションテーブルを置くことができます。最初のパーティションは
1MBから始めてください。
