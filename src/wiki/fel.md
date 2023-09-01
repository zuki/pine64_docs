# FEL

[オリジナル](https://linux-sunxi.org/FEL)

FELはAllwinnerデバイスのBootROMに含まれているる低レベルサブルーチン
です。USBを使用したデバイスの初期化プログラミングとリカバリに使用
されます。

このため、デバイスは**USBケーブル**でホスト（PC）に接続する必要が
あります。USBケーブルの他端はsunxiデバイスがUSB「スレーブ」
（デバイスモード）となるポートに接続します。通常、それは「OTG」
コネクタを意味します[^1]。

## FELモードと会話するためのツール

[ツールリポジトリ](https://github.com/linux-sunxi/sunxi-tools)にはFELモードを
扱うためのツールがあります。まだしていないのであれば、
[リポジトリを取得してビルド](https://linux-sunxi.org/Sunxi-tools#Building_from_Source)
してください。

## FELモードへの移行

FELモードに切り替える方法はいくつかありますが必ずしも同じでは
ありません。低レベルの初期化（Boot0とBoot1のロード）を行うものも
あれば、そうでないものもあります。

FELモードを使ってデバイス情報を取得する場合は、Boot1を初期化するFEL
モードに入る方法を選ぶ必要があります。

### デバイスの電源切断

FELモードに入る前にデバイスの**電源が本当にオフになっている**ことを
確認してください。ケーブルは外してください。

**警告**: 一般的な設計上の欠陥により、[UART（からのリーク電流）](https://linux-sunxi.org/UART#Board_won.27t_shut_down_completely)が
デバイスをわずかに動作している状態に保つことがよくあります。そのため、
デバイスの電源を再投入する前にUARTの接続を外し、再度接続してください。

### FELモードへの切り替え

#### 特別なFELボタンの押下

これは**リカバリ**や**uboot**、または**fel**と呼ばれるものです。
使用するデバイスにそのようなボタンがある場合は電源投入時にその
ボタンを押し続けるだけでデバイスがFELモードに入るはずです。

#### 標準ボタンの押下

これは通常、**VOL+**キーなどの標準的なタブレットのボタンです。

以下の方法で動作すると思われます。

- FEL（と思われる、または書かれている）キーを長押しする。
- 電源キーを約2秒間長押しする。
- 電源キーを離し、すぐに3回以上押す。

Boot1はこの方法で初期化されます。

#### シリアルコンソール経由

すでに[UART](https://linux-sunxi.org/UART)にアクセスできている場合は、
電源投入中に文字'1'（デバイスによっては'2'）をデバイスに送ります。

Boot1はこの方法で初期化されます。

後期のSoCではAllwinnerのU-bootが"efex"コマンドをサポートしています。

資料しているU-Bootで"efex"が利用できない場合はubootの"go"コマンドが
利用でき、リターンFELアドレスを引数として指定します。

```bash
=> go 0xffff0020
```

u-bootプロンプトでこのコマンドを入力するとFELモードに入ります。

**注意**: これは _FELモードに入るための単なる代替方法_ に過ぎません。
FEL自体はシリアル接続で会話することはできません。言い換えると、FELと
関連ツールを実際に使用できるようにするにはUSBケーブルを接続する必要が
あります。

#### 特別なSDカードイメージ経由

[sunxi-toolsリポジトリ](https://linux-sunxi.org/Sunxi-tools)には
FELにジャンプするだけの小さなSDCARDブートイメージがあります。

u-boot SPLでしたのと同じようにsdcardにインストールしてください
（**/dev/sdX はsdcardのある場所に変更してください**）。

```Bash
$ wget https://github.com/linux-sunxi/sunxi-tools/raw/master/bin/fel-sdboot.sunxi
$ dd if=fel-sdboot.sunxi of=/dev/sdX bs=1024 seek=8
```

#### 有効なブートイメージを与えない

[BROM](https://linux-sunxi.org/BROM)は有効なブートイメージを
見つけられなかった場合、自動的にFELモードに入ります。

したがって、ボード上にNANDやeMMCを搭載していないほとんどの
デバイスでは、SDカード/microSDカードを取り外すだけでこれが適用
されるはずです。

### FEFモードの確認

#### 新しいUSBデバイスの表示

`lsusb`コマンドを実行すると次の行があるはずです。

```bash
Bus 001 Device 074: ID 1f3a:efe8
```

#### sunxi-felツールの実行

```bash
> ./sunxi-fel version
AWUSBFEX soc=00162500(A13) 00000001 ver=0001 44 08 scratchpad=00007e00 00000000 00000000
```

#### シリアル出力

選択した方法でboot1が初期化された場合はシリアルに次のように
表示されるはずです。

```bash
HELLO! BOOT0 is starting!
boot0 version : .3.0
dram size =1024
Succeed in opening nand flash.
Succeed in reading Boot1 file head.
The size of Boot1 is 0x00036000.
The file stored in 0X00000000 of block 2 is perfect.
Check is correct.
Ready to disable icache.
Succeed in loading Boot1.
Jump to Boot1.
[       0.145] boot1 version : 1.3.1a
[       0.145] pmu type = 3
[       0.145] bat vol = 4117
[       0.176] axi:ahb:apb=3:2:2
[       0.176] set dcdc2=1400, clock=1008 successed
[       0.178] key
[       2.486] you can unclench the key to update now
[       2.486] key found, jump to fel
```

### よくある落とし穴

#### 失敗した！

もう一度実行してください。デバイスの電源を完全に落とし、イベントの
順序が正しいことを確認してください。きっとうまくいくはずです。

#### USB経由での読み込みに失敗する

次のようになった場合

```bash
> ./sunxi-fel read 0x43000000 0x20000 script.bin
libusb usb_bulk_send error -7
```

おそらく、boot0とboot1がロードされたときだけ初期化されるものを
読もうとしています。boot1を初期化するFELモードに入る別の方法を
試してみてください。

## FELプロトコル

FELは、[独自のUSBプロトコル](https://linux-sunxi.org/FEL/Protocol)を
実装した小さなUSBスタックです。

**その一部**は[ツールリポジトリ](https://github.com/linux-sunxi/sunxi-tools/blob/master/fel_lib.c)で
実装されており、リファレンスコードとして使用できます。

## ヒントとコツ

- 何かのコマンドを実行した際に `usb_bulk_send error -7` が表示された
  場合はsoc/felスタックがfelモードから抜けたか、クラッシュしたことを
  意味します。何かを起動したか、デバイスをハングアップさせたかの
  いずれかです。
- ウォッチドッグを有効にすれば、felでリセットできます。
    - A10互換の場合: `./sunxi-fel writel 0x01c20c94 3`
    - H3/H5互換の場合: `./sunxi-fel writel 0x01c20cb8 1`
- u-bootの場合: `mw 0x01c20c94 3;;;;`

これにより、デバイスのFELモード用のジャンパ/ボタンが正しく設定されて
いるか否に関係なく、FELモードに戻ります。何らかの理由でmwコマンドの
後には空白行が数行必要であることに注意してください。

## をも見よ参照

[FEL/USBBoot](usb_boot.md)

## 参考資料

[^1]: このルールには例外があり、特定のポートへの接続や非標準
ケーブルの使用が必要となるボードがあります。その最たるものが
[Pine64](https://linux-sunxi.org/Pine64#FEL_mode)です。
