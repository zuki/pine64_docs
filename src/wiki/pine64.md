# Pine64

[オリジナル](https://linux-sunxi.org/Pine64)

Pine64は、ARMv8（64ビットARM）コアを搭載したコスト最適化ボードです。
64ビットAllwinnerチップを搭載した最初のボードの一つであり、64ビット
ARMコアを搭載した最初の手頃なボードの一つでもあります。

## 識別

microSDカードスロットの横には松ぼっくりのようなロゴがあり、ロゴの
下には`Pine64`と書かれています。さらに、SoCには大きく`A64`とプリント
されているのが目立ちます。

ボードの背面には次のようにプリントされています。

```
Designed in Silicon Valley, California. Built in Silicon Delta, China.
```

PCBには次のシルクスクリーンが施されています。

```
A64-DB-Rev B
2015-12-17
```

アンドロイドで「設定」→「タブレットについて」とたどると以下が
確認できます。

- Model Number: _Pine A64_
- Build Number: _tulip_t1-eng_ _5.1.1 LVY4BE 20151210 test-keys_

### モデル

現在のところ3つのモデルがあります。

- Pine64 512MB DRAM
- Pine64+ 1GB DRAM
- Pine64+ 2GB DRAM

後者の2つは搭載されているDRAM容量を除いて同じです。Pine64とPine64+の
違いは次のとおりです。

- Pine64はファーストイーサネットしかサポートしていません。そのため、
  PHYチップは大型モデルのRealtek 8211ではなく、8201です。8211は
  RGMIIを話しますが、8201はMIIインタフェースを使用します。このため、
  DTに違いがあります。
- Pine64には、タッチスクリーン、LCDスクリーン、カメラポート用の
  コネクターがありません。

## Sunxiのサポート

### 現在の状況

A64 SoC用のメインラインカーネルとファームウェアのサポートは非常に
成熟しており、Pine64は最もよくサポートされているボードの一つです。
SoCとボードのほぼすべての機能がサポートされており、多くの場合、
汎用のディストリビューションはPine64を明確にサポートしています。

ARM64対応ボードの初期例という性質上、多くのLinuxイメージが出回って
おり、Allwinnerが提供したBSPカーネルを更新・拡張したバージョンを使用
しているものもあります。ただし、現在においては、BSPカーネルを使う
理由はほとんどありません。

### イメージ

#### エンドユーザ

以下は、現在コミュニティがサポートしているイメージへのリンクです：

- [LongsleepによるUbuntu基本イメージ (pine64.pro)](http://wiki.pine64.org/index.php/Pine_A64_Software_Release#Xenial_Base_Image)
- [ayufanによるUbuntu最小イメージ (pine64.pro)](http://wiki.pine64.org/index.php/Pine_A64_Software_Release#Xenial_Minimal_Image)
- [Arch Linux イメージ Mainline XFCE (pine64.org)](http://wiki.pine64.org/index.php/Pine_A64_Software_Release#Arch_Linux_mainline_XFCE)
- [Lenny Raposo氏によるDebian Jessie with Mate DE (pine64.org)](http://wiki.pine64.org/index.php/Pine_A64_Software_Release#Debian_Jessie_Mate)
- [PINE64用のArmbian](https://www.armbian.com/pine64/)

（このページの[Manufacturer Images](https://linux-sunxi.org/Pine64#Manufacturer_Images)からリンクされているWikiページも
チェックしてください。）

#### 開発者

まず、apritzelのgithubにある[基本イメージ](https://github.com/apritzel/pine64)を
入手してください。手順についてはとりあえずそこにある**README.md**を
見てください。

longsleepもBSPカーネルと組み合わせた最小限のUbuntuイメージを構築して
おり、[ここ](https://www.stdin.xyz/downloads/people/longsleep/pine64-images/ubuntu/README.txt)から
ダウンロードできます。セットアップの方法は[こちら](https://www.stdin.xyz/downloads/people/longsleep/pine64-images/ubuntu/README.txt)で
説明されています。

このイメージは開発者向けであり、以下が添付されています。

- BSP Linuxカーネル 3.10.65+
- BSP Uブート
- Ubuntu Ubuntu 16.04 (Xenial Xerus) aarch64
- 1080PのHDMI
- HDMIアナログオーディオ（alsa、pulseaudio）
- イーサネット（1000Mを含む）
- USB
- Wifi

### BSP

AllwinnerのBSPにはLinaroのLSK-3.10.65 (LinaroとAndroidのパッチを含む)
をベースにしたarm64 Linuxカーネルが含まれています。これには（AArch64
EL3によるカーネルへの投入やセキュアEL1での実行などの）厄介な実験の
痕跡（コメントや構成なしのコード）があります。このリリース/リーク
されたコードは、提供されているAndroidイメージにあるものとは正確には
一致しません。BSPカーネルは非セキュアなEl1で投入されているため
（KVMなど）あらゆる種類の仮想化はできません。

### マニュアルビルド

[マニュアルビルドの手引](manual_build.md)に従い、以下で示す構成から選択することで自分で
ビルドすることができます。

#### U-Boot

##### Sunxi/Legacy U-Boot

Allwinnerが提供するBSPパッケージには2014.07リリースに基づいた32ビット
ポートであるU-Bootのソースコードが含まれいます。tarボールに含まれて
いるコードはコンパイルすらできず、ポート自体もMMCからAndroidシステムを
ブートできる程度であり、ひどく不自由です。ブートサポートがない
（カーネルイメージを直接ロードできず、Androidカーネルイメージしか
使えない）、ネットワークサポートがない、Androidパーティションからしか
データをロードできない（ファイルシステムをサポートしていない）、完全に
非標準のDTバインディング、簡単なFDTロードのサポートがない、など多くの
制限があります。完全な暴言はWikiの履歴を参照してください ;-)

しかし、既存のコードベースはlongsleepによって修正・拡張され（bootiと
適切なFDTサポートを使用して）カーネルを直接ロードできるようになり、
ファイルシステムのサポートが追加されました。最新のコードベースは
[ここ](https://github.com/longsleep/u-boot-pine64/commits/pine64-hacks)
にあります。

現時点では、BSPカーネルをブートするにはこのU-Bootバージョンが必要です。

##### メインラインU-boot

このボードはv2017.07から完全にサポートされるようになりました。U-Boot
イメージのビルドには`pine64_plus_defconfig`ターゲットを使用して
ください。これには512MBの"non-plus"バージョンのサポートが含まれており、
実行時に検出されます。ATF (ARM Trusted Firmware)のビルド（bl31.bin、
下記参照）が必要ですが、これはFITイメージに含まれています。

Linux arm64カーネルイメージをロードするbootiコマンドがサポートされて
おり、さらにbootefiは（grub2などの）Aarch64 EFIアプリケーションの
起動にも利用できます。32ビットカーネルの起動はサポートされていませんが、
技術的には可能であり、将来追加されるかもしれません。

BSP版とは対照的にメインラインポートは64ビット版であるため、Allwinnerが
提供するboot0/ATFペアは、さらなる変更/パッチなしでは起動しません。

U-BootツリーにはREADMEファイルがあり、このボード用にU-Bootをコンパイル
する方法が詳しく説明されています。その内容は現在U-Bootコードベースが
サポートするものと同期しているはずです。

#### Trusted Firmware-A (TF-A, 従来はATFと呼ばれていた)

すべての64ビットAllwinner SoCにはTrusted FirmwareのBL31部分のビルドが
必要です。これは適切なSMPハンドリングを提供しており、中でも、PSCI
ランタイムのリファレンス実装とエラッタの回避などが含まれています。
TF-AはBSDライセンスのオープンソースプロジェクトです。

必要なbl31.binファイルをビルドするには、次のように、masterブランチを
チェックアウトし、AArch64クロスコンパイラでA64ターゲット用にビルド
します。

```bash
$ git clone https://git.trustedfirmware.org/TF-A/trusted-firmware-a.git
$ cd trusted-firmware-a
$ make CROSS_COMPILE=aarch64-linux-gnu- PLAT=sun50i_a64 DEBUG=1
```

build/sun50i_a64/debugディレクトリにあるbl31.binファイルをU-Boot
ツリーのルートにコピーするか、`BL31`環境変数でこのファイルを指す
ようにしてください。

#### Linuxカーネル

メインラインのLinuxカーネルは、3Dグラフィックス、ビデオアクセラ
レーション、DVFSなどの高度な機能を含め、Pine64を非常によくサポート
しています。

- 基本的なサポートはLinux v4.10-rc1でマージされましたが、これはMMCを
  サポートしていないのでinitrdでしか使用できません。また、I2CとUART
  （シリアルコンソール）  以外の周辺機器はサポートしていません。
- MMCとUSBのサポートが4.11-rc1にマージされました。これにより、ある
  程度使える状態になりましたが、イーサネットはまだサポートされて
  いません。
- イーサネットドライバが4.15リリースでついに追加されました。

全容は[ステータスマトリックス](https://linux-sunxi.org/Mainlining_Effort#Status_Matrix)を
チェックしてください。

基本的なサポートであればU-Bootのデバイスツリーをカーネルにも使う
ことができ、$fdtcontroladdrをbooti引数として指定するだけです。

このボード用のカーネルをビルドするには、最近のメインラインLinux
ツリーをクローンし、次のように`defconfig`カーネルをビルドします。

```bash
$ export CROSS_COMPILE=aarch64-linux-gnu-
$ ARCH=arm64 make clean defconfig
$ ARCH=arm64 make -j4 Image
```

Pine64の各バージョン用に生成されるdtbファイルの名前は各々、
`sun50i-a64-pine64-lts.dtb`、`sun50i-a64-pine64-plus.dtb`、
`sun50i-a64-pine64.dtb`です。

## ヒント、コツ、注意点

### ブートシーケンス

A64 SoCは32ビット・セキュア・スーパーバイザモードでリセットから
復帰するように配線されています。他のAllwinnerデバイスと同様に、
A64 SoCはBROMコード（アドレス0にマッピングされている）の実行を
開始します。これはARM32のコードです。ROMは最初の64KBをカバーして
いますが、通常の非セキュア・ブート・バージョンではより少ないスペース
しか使用しません。BROMコードはFEL状態（SDカードがないか、SDカードが
挿入されていない）を検出しないと、microSDカードのセクタ16（8KByte）
から32KBをSRAMにロードし、これを実行します。このコードの少なくとも
最初の命令までは依然として32ビットARMコードである必要があります。

メインラインのU-Bootはできるだけ早い段階で
[AArch64にリセット](http://git.denx.de/?p=u-boot.git;a=blob;f=arch/arm/include/asm/arch-sunxi/boot0.h)し、
以降のコード（ATFとU-Boot本体、それからカーネルを含む）を64ビット
モードで実行します。

#### レガシーBSPブートシーケンス

AllwinnerファームウェアはほとんどがAArch32で動作し、boot0はmicroSD
カードのセクタ38192（19096KByte）からU-Boot（32ビット）をロードします。
boot0はさらにARM Trusted Firmware（ATF）の（ハック）バージョンを
DRAMに、arisc管理コアのコードをSRAMに各々ロードします。最後に、
RMR書き込みを行ってAArch64実行状態にSoCをウォームリセットして、
RVBARレジスタにそのアドレスを入れてATFエントリポイントにジャンプします。
ATFはブート・コアを非セキュア実行用に初期化し、非セキュアなAArch32
EL1に移行して、U-Bootを実行します。

U-Bootは32ビットで問題なく実行されます。U-Bootはカーネルを起動する直前に
カスタムsmcサービスコールを使用して（Allwinnerバージョンの）ARM
Trusted Firmwareに戻り、カーネルのエントリポイントを渡します。ATF
コードは _AArch64_ 非セキュアEL1に戻りますが、U-Bootには戻らず、
提供されたカーネルエントリポイントを使用します。

### FELモード

Pine64ボードはmicroSDカードスロットにカードが存在しないことを検知する
とFELモードにフェイルオーバーします。

**注意**: トリッキーで混乱しそうなのはただ一つあるMicro USBコネクタ
（"POWER JACK"と表示されている）はボードへの電源供給専用であり、
SoCのどのUSBコントローラにも接続されて _いない_ ことです。実際には、
SoCのUSB **OTG**コントローラが**上段のUSBホストコネクタ**に接続
されています。そのため、Pine64ボードを[sunxi-felツール](https://linux-sunxi.org/FEL/USBBoot)を
実行しているデスクトップPCに接続するには少々特殊なUSBケーブル
（AオスーAオス）またはアダプタ（AオスーMini/Micro Bメス）が必要です。

Pine64をFELモードで起動するとすぐに（思い出してください、SDカードは
挿入しないで）新しいUSBデバイスが見つかるはずです。

```bash
$ lsusb
   Bus 001 Device 005: ID 1f3a:efe8
```

```bash
$ ./sunxi-fel version
   AWUSBFEX soc=00001689(A64) 00000001 ver=0001 44 08 scratchpad=00017e00 00000000 00000000
```

実際にFELモードでソフトウェアをロードする場合は、汎用の
[A64 FELブート](https://linux-sunxi.org/A64#FEL_booting)手順を
参照してください。

### SPI NOR Flash

Raspberry Pi互換の拡張ヘッダーにプラグイン可能なハードウェア
アクセサリを追加して、SPI0ピンに小さなSPI NORフラッシュを追加する
ことができるはずです。これにはブート可能なファームウェアを保存する
ことができ、AArch64サーバーグレードのLinuxディストリビューションを
実行するための[流行りの"業界標準"](https://en.wikipedia.org/wiki/Server_Base_System_Architecture)互換性を提供することができます
（正確には今はできませんが、将来的にはおそらくできるでしょう）。
[ブータブルSPIフラッシュ](https://linux-sunxi.org/Bootable_SPI_flash)
のページではさらに詳しく説明されています。

[u-boot用のドライバモデル互換のSPIドライバ](https://github.com/StephanvanSchaik/u-boot/tree/sunxi-spi)が
現在開発中です。Pine64+ボードはテスト済みで、このドライバで完全に
サポートされています。

### 拡張ヘッダー

ピンの割り当てやその他の仕様（ボードの物理的寸法など）についての
文書は[Pine64の公式Wiki](http://wiki.pine64.org/index.php/Main_Page)の
[ハードウェアセクション](http://wiki.pine64.org/index.php/Main_Page#Pine_A64_Hardware_PCB_information)に
あります。

### AXP803 PMIC

コールドブート後のデフォルトのリセット電圧の一部はボードの仕様と
正確には一致していません。たとえば、Eulerコネクタの"3.3V"ピンの
電圧はAllwinner社のブートローダがPMICを構成するまでは実際には3.0V
（DCDC1）です。現在の"アップストリーム"ファームウェアスタックでは
ARM Trusted FirmwareがPMICを設定し、DCDC1を指定された3.3Vに
プログラムしています。また、Ethernet PHYに電力を供給できるように
DC1SWを有効にしている。

DRAM電圧はDCDC5から供給され、AXP803のマニュアルによればデフォルトで
1.5Vに設定されるはずです。さらに、AXP803のマニュアルではDCDC5をDRAM
専用として使用することをはっきりと推奨しています。これは1.35VのDDR3L
チップでも安全です。[1.5Vとも互換性がある](http://superuser.com/a/564205)
ためです。しかし、少なくともPine64ボードの製品前のバッチでは
[デフォルトのリセット電圧は実際には1.24Vに設定されているようです](https://irclog.whitequark.org/linux-sunxi/2016-10-01#17765715;)。
DCDC5SETピンがフローティングのままになっているからです。

### DC5V/BAT Powerジャンパ

1GB版と2GB版のPine64+では、DC5V/BAT POWERスイッチを使用することで
MT3608昇圧コンバータをバイパスすることができます（入力電圧は5V）。
ボードがDC-IN（micro-USBまたはEulerコネクタ）から給電されている場合、
DC5V設定では入力電圧をUSB電源レールに接続し、BAT設定では接続されて
いる電源のいずれか（バッテリーまたはDC-INなど）から5Vが生成されます。
どちらの設定でもUSBポートは1ポートあたり約650mAに電流制限されています。

ジャンパをDC5Vの位置で使用すると、不十分な電源電圧がUSBポートに直接
現れることに注意してください。Pine64+がバッテリーで動作している場合、
USBポートはBAT設定になっていないと給電されません。

### Gigabit PHYの問題

Pine64+（1GB版と2GB版の両方）にはGigabit Ethernet問題が発生している
ものがあります。GbEモードで、転送速度が遅く、再送が多く、パケットが
失われるという問題です。とりあえず、[これはハードウェアの問題である](http://forum.pine64.org/showthread.php?tid=835&pid=19704#pid19704)
ことが確認されています。この問題が発生した場合は、Pine64+を強制的に
Fast Ethernetモードにすることができます（ _ethtool -s eth0 speed 100 duplex full_ を使うか、2対のケーブルしかないEthernetケーブルを使うか
のいずれかの方法が使えます）が、ソフトウェアの修正でこの問題が解決
する可能性は低いと思われます。Pine64が解決策を思いつくかどうかは
前述のスレッドを参照してください。

### CPUクロック速度の限界

Allwinner A64の電圧-周波数テーブルはA64 SDKに含まれているFEXファイルに
あります。

```bash
; dvfs voltage-frequency table configuration
;
; max_freq: cpu maximum frequency, based on Hz
; min_freq: cpu minimum frequency, based on Hz
;
; lv_count: count of lv_freq/lv_volt, must be < 16
;
; lv1: core vdd is 1.30v if cpu frequency is (1104Mhz, 1152Mhz]
; lv2: core vdd is 1.26v if cpu frequency is (1008Mhz, 1104Mhz]
; lv3: core vdd is 1.20v if cpu frequency is (816Mhz,  1008Mhz]
; lv4: core vdd is 1.10v if cpu frequency is (648Mhz,   816Mhz]
; lv5: core vdd is 1.04v if cpu frequency is (480Mhz,   648Mhz]
; lv6: core vdd is 1.04v if cpu frequency is (480Mhz,   648Mhz]
; lv7: core vdd is 1.04v if cpu frequency is (480Mhz,   648Mhz]
; lv8: core vdd is 1.04v if cpu frequency is (480Mhz,   648Mhz]
```

この表のデータに基づくと、1152MHz@1.3Vが最速のcpufreq動作ポイント
です。さらに、AXP803 PMICはDCDC2/DCDC3（VDD-CPU）用に1.1Vのデフォルト
電圧を使用します。これはPMICが初期化される前にCPUを816MHzまで安全に
クロックアップできることを意味します。

### USBコントローラとポート

A64 SoCは2つのUSB 2.0 EHCI/OHCIホストコントローラを搭載しています。
第一のホストコントローラ (HCI0) はPHYスイッチに接続されており、
通常のUSB PHY（ボードの下段のUSBコネクタに接続されている）のドライブと
オンボードのUSBペリフェラル（通常はハブ、ただし、これはボード上では
使用されない）を接続できるSoC上のHSICピンのドライブとを切り替える
ことができます。

第二のホストコントローラ (USB-OTG-HCI) はUSB PHYを（別の!）OTG
コントローラと共有しており、Pine64ボードの上段のコネクタに接続されて
います。そのため、このソケットは通常のホストコントローラインタフェース
でも、OTGコントローラ（これはホストモードも提供するが、問題がない
わけではないらしい）でもドライルすることができます。

### シリアルポート/UART

このボードはSoCが持つ4つのUARTを簡単にアクセスできるヘッダーピンに
接続しています。RPiコネクタにはUART2が、EulerコネクタにはUART3と
UART4があります。UART0はAllwinnerのファームウェアがブートとデバッグ
メッセージに使用するメインUARTであり、Eulerコネクタの29ピン (TXD)、
30ピン (RXD)、25/34ピン (GND) でアクセスできます（これは公式の
コネクタ解説には記載されていません）。近くにあるEXPコネクタにある
UART0を使用することが**いつでも**最善です。7ピン (TXD)、8ピン (RXD)、
9ピン (GND)でアクセスできます。このRXピンはFETを介してSoCのピンに
接続されているため、このラインからの電力流入を防ぎます。

すべてのUARTは3.3Vの電圧レベルを使用します。詳しくは[UART Howto](https://linux-sunxi.org/UART)を
ご覧ください。

Eulerピンに接続されたUARTケーブルは電力をリークするのでいくつかの
問題を引き起こします。たとえば、電源ケーブルを抜き差してもボードは
正しく再起動しません。そのため、EXPコネクタのUART0を使用することを
強く勧めます。それでもEulerコネクタを使用したい場合は、合理的な
ソフトウェア開発を行うためにリセットボタンを使用することを推奨します。

このボードには標準ではハードウェアリセットボタンがありませんが、
拡張コネクタの適切なピンにボタンを簡単に接続することができます。
また、基板上のUSBソケットの隣に標準的なマイクロスイッチ（正立型）を
ハンダ付けすることで初期開発を容易にすることができます ;-)

[以下、省略]
