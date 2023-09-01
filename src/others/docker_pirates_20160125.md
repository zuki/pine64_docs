# Pine A64はDockerを実行する最も安価なARM64ビットプラットフォームになろうとしている

 Docker Pirates ARMed with explosive stuff: 2016年1月25日

[オリジナル](https://blog.hypriot.com/post/the-pine-a64-is-about-to-become=the-cheapest-ARM-64-bit-platform-to-run-Docker/)

先週の土曜日、我々の好奇心をそそった有望な[Kickstarterキャンペーン](https://www.kickstarter.com/projects/pine64/pine-a64-first-15-64-bit-single-board-super-comput)が
44日を経て終了し、 170万ドルを集めることができました。これは現在お金で
購入できる最も安価な64ビットARM ボードに資金を提供するキャンペーンでした。

**Pine A64 は 15ドルで販売されています。**

では、5ドルで販売されているPi Zeroのようなボードがあるのになぜそれが
注目に値するのでしょうか?

価格を考慮するとそれは当然です。Pine以前に知られていた最も安価な64
ビットARMボードは約200ドルで販売されている[NVIDIA ShieldTV](https://blog.hypriot.com/post/getting-docker-running-on-a-highend-arm-gaming-console-for-fun-and-profit/)
でした。次はもっと高価な[AMD Opteron A1100シリーズ](http://www.slashgear.com/amd-starts-shipping-seattle-arm-server-chips-14423259/)で、
個々のチップだけの価格が150ドルから始まります。

つまり、Pine A64は本物の64ビットARMハードウェアを手に入れるための最も
手頃な方法であるということです。それもダントツです。では、そのような
デバイスを使って実際に何ができるのでしょうか?

もちろんいろいろなことができますが、**我々Hypriotが最初に考えるのは
「その上でDockerを実行できるか」**ということです。その短い答えは、
おそらくできる、です。しかし、それについては後で詳しく説明します。

## ハードウェア

核心的な詳細を深く掘り下げる前に、まず一歩下がって、Pine A64が
ハードウェア的に何を提供しているかを見てみましょう。

Pine A64は以下を搭載しています。

- 1.2GHzの**クアッドコアCortex A53 CPU**
- **64ビット**プロセッサアーキテクチャ
- 最大**2GBのDDR3メモリ**
- **ギガビットイーサネット**搭載バージョン
- 4Kビデの再生機能

ここで我々を最も興奮させるのは、2GBのメモリとギガビットイーサネットを
組み合わせた64ビットアーキテクチャです。

これらの仕様により、Pine A64は残念なことに100メガビットイーサネットしか
提供していいないRaspberry Pi 2とは明らかに異なります。ただし、これらの
仕様はPineのハイエンドバージョンでしか得られないことに注意してください。
これは、A64+と呼ばれ、価格は19ドル (1GB バージョン) 、または、29ドル
(2GBバージョン) と少し高くなります。**1GBバージョンでも19ドルですがこれはRaspberry Pi 2の価格のほぼ半額です**。

ボードのビルド品質とサポートがRaspberry Piと同等であることが判明したら、
それは確かに素晴らしい成果と言えるでしょう。

我々は幸運にもデバイスの初期開発者バージョンを確保することができ、
数日前に到着しました。百聞は一見にしかずということで、1 GBのPine A64+
ボードを写した写真をいくつか用意しました。

![クローズアップ Pine 64 正面]()
![クローズアップ Pine 64 背面]()
![クローズアップ01]
![クローズアップ02]
![クローズアップ03]
![クローズアップ04]
![Pine A64とRaspberry Pi 2. ODroid C1の比較]

## ソフトウェア

Pine A64の現在の制約の1つはAndroidでしか実行できないことです。Pineの
メーカーはLinuxもサポートすると約束してくれました。それでも、我々に
にとってはLinuxではなくAndroidが最初であることは少し迷惑です。

我々はせっかちなのでデバイスを入手したらすぐにその上でHypriotOSを
実行してこの問題を修正し始めました。そして、これは簡単な作業ではない
ことが比較的すぐに明らかになりました。

なぜか。[Allwinner Technology](http://www.allwinnertech.com/index.html)と
オープンソースコミュニティとの関係はこれまで常にベストだったわけではない
ようです。これは基本的にAllwinner関連のSoCに対する[既存の友好的で協力的なオープンソースコミュニティ](http://linux-sunxi.org/)がAllwinnerベースの
デバイスで適切なLinuxサポートを行うには限られた情報とサポートで作業しな
ければならないことが多いことを意味します。このような状況にもかかわらず、
Linux Sunxiコミュニティの一部のメンバーはすでにPine A64の使用に熱心に
取り組んでいます。

## （うさぎの穴を降りる）長い旅は最初の一歩から始まる

このブログポストの残りの部分ではPine A64においてLinuxとDockerの至福の
時間を実現するための準備段階について説明します。あなたは十分な警告を
受けていると思ってください - この旅は気の弱い人向けではないからです …:)

### ウサギの穴を降りる: UART0コンソールでfooを出力する

ではこの新しい素晴らしいものを自分のものにしていきましょう。まず知って
おくべきことはPine A64ボードに電力を供給する方法です。このボードは
標準的なmicroUSBポートを備えており、5V/2Aの電源アダプタで使用できます。
これはおそらくあなたがRaspberry Piで使用しているものと同じです。

我々がボードで最初に低レベル操作をする際には、UART-to-USBコネクタ
ケーブルでUART0コンソールに接続することを好みます。Adafruitの
[USB to TTLシリアルケーブル](https://www.adafruit.com/product/954)などを
使用することを勧めます。Adafruitはこのようなケーブルのドライバーを
さまざまなオペレーティングシステムにインストールする方法に関して
[優れたチュートリアル](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-5-using-a-console-cable)を提供しています。

UART0経由でPineに接続するには基本的に2つの方法があります。`Euler bus`の
ピン経由か、`EXT`コネクタ経由のいずれかす。それぞれの位置は次の画像で
確認できます。詳細については[Linux-sunxi Wiki: シリアルポート/UART](http://linux-sunxi.org/Pine64#Serial_port_.2F_UART)を
参照してください。

- Euler busによるシリアルコンソール

![Euler bus]()

- Extコネクタによるシリアルコンソール

![Ext connector]()

実際にUART0コンソールを使用するにはUART-to-USBケーブルをLinuxまたは
Macに接続し、好みのターミナルプログラムを起動する必要があります。

Mac OS Xのコマンドは次のとおりです。

```bash
$ sudo screen /dev/cu.usbserial 115200
```

UART-to--USBケーブルをPineに接続してもコンソール画面に起動の兆候が
まったく表示されないことに驚かないでください。見ることできるのは
ボード上の電源LEDが緑色に変わったことだけです。

これはA64 SOCが初期ブートローダをmicroSDカードからメモリに読み込んで
起動しているためです。ブートローダはUART0ポートを初期化し、最初の
ブートメッセージを出力します。

Pineが沈黙したままの場合は2つの理由が考えられます。SDカードが存在しない
か、SDカードに動作するブートローダプログラムが見つからないかのいずれか
です。

なので両方があることを確認しましょう。

SDカードを入手して適切なイメージを書き込んでください。Andre Przywaraの
おかげで、すでに使用可能な[最初の実験的なLinuxイメージ](https://github.com/apritzel/pine64)が存在します。

```bash
# sdxを使用するSDカードデバイスファイルに置き換えてください

$ xzcat pine64_linux-20160121.img.xz | dd of=/dev/sdx bs=1M
$ sync
```

このSDカードイメージはオリジナルのPine 64 Androidイメージの
コンポーネントと最新のLinux 4.4.0-rc8カーネル、initrdを組み合わせた
ハイブリッドです。これらすべては、多かれ少なかれ、Linuxが稼働する
最初のプロトタイプSDカードイメージを得るためにまとめられたものに
すぎません。

このイメージには完全なルートファイルシステムは含まれていないないので
最初に思いついたアイデアはARM64用にHypriotOSルートファイルシステムを
含めることでした。さてこれが本当に動くか見てみましょう。

SDカードをPine A64に挿入して起動すると、予想どおりの起動メッセージが
表示されました。数秒後、次のようにLinuxブートプロンプトが表示されました。

```bash
BusyBox v1.22.1 (Debian 1:1.22.0-9+deb8u1) built-in shell (ash)
Enter 'help' for a list of built-in commands.

/ #
/ #
/ #
/ # uname -a
Linux (none) 4.4.0-rc8 #20 SMP PREEMPT Mon Jan 18 01:05:25 GMT 2016 aarch64 GNU/Linux
/ # df -h
Filesystem                Size      Used Available Use% Mounted on
none                     78.5M     72.0K     78.4M   0% /run
devtmpfs                381.8M         0    381.8M   0% /dev
/ #
```

OK、これが第一ステップです。次はHypriotOSルートファイルシステムの
起動が本当に可能かどうかを確認しよう。Pineを再起動します。U-Boot
メッセージが表示されたらすぐに何かキーを押して自動ブート処理を
停止します。これにより対話型のU-Bootプロンプトが現れます。

あとはAndreのドキュメントに従って我々の[ARM64用の汎用HypriotOS](https://github.com/hypriot/os-rootfs/releases/tag/v0.6.0)/が
ある`dev/sda10`のルートファイルシステムを起動するようにU-Bootに
指示するだけです。

```bash
sunxi# run load_env
sunxi# run load_dtb
sunxi# run set_cmdline
sunxi# setenv kernel_part mainline
sunxi# run load_kernel
sunxi# run boot_kernel
```

やった。動作しました。

![screen]()

UART0コンソールに表示されたブートログの一部を次に示します。完全なブート
ログは[GitHub gist](https://gist.github.com/DieterReuter/93a5d10dae6a62911b71)に保存しました。

```bash
HELLO! BOOT0 is starting!
boot0 commit : 045061a8bb2580cb3fa02e301f52a015040c158f

boot0 version : 4.0.0
set pll start
set pll end
...
NOTICE:  BL3-1: v1.0(debug):045061a
NOTICE:  BL3-1: Built : 14:30:28, Dec  3 2015
NOTICE:  BL3-1 commit: 045061a8bb2580cb3fa02e301f52a015040c158f

INFO:    BL3-1: Initializing runtime services
INFO:    BL3-1: Preparing for EL3 exit to normal world
INFO:    BL3-1: Next image address = 0x4a000000
INFO:    BL3-1: Next image spsr = 0x1d3


U-Boot 2014.07 (Dec 03 2015 - 14:30:33) Allwinner Technology
...
Starting kernel ...

INFO:    BL3-1: Next image address = 0x41080000
INFO:    BL3-1: Next image spsr = 0x3c5
Booting Linux on physical CPU 0x0
Initializing cgroup subsys cpu
Linux version 4.4.0-rc8 (aprzywara@slackpad) (gcc version 4.9.3 (GCC) ) #21 SMP PREEMPT Wed Jan 20 22:43:20 GMT 2016
Boot CPU: AArch64 Processor [410fd034]
...
Welcome to Debian GNU/Linux 8 (jessie)!
...
[  OK  ] Reached target Multi-User System.
[  OK  ] Reached target Graphical Interface.

Debian GNU/Linux 8 black-pearl ttyS0

black-pearl login: pirate
Password:
Linux black-pearl 4.4.0-rc8 #21 SMP PREEMPT Wed Jan 20 22:43:20 GMT 2016 aarch64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
HypriotOS/arm64: pirate@black-pearl in ~
$ uname -a
Linux black-pearl 4.4.0-rc8 #21 SMP PREEMPT Wed Jan 20 22:43:20 GMT 2016 aarch64 GNU/Linux
$ cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 8 (jessie)"
NAME="Debian GNU/Linux"
VERSION_ID="8"
VERSION="8 (jessie)"
ID=debian
HOME_URL="http://www.debian.org/"
SUPPORT_URL="http://www.debian.org/support/"
BUG_REPORT_URL="https://bugs.debian.org/"
HYPRIOT_OS="HypriotOS/arm64"
HYPRIOT_TAG="dirty"
HypriotOS/arm64: root@black-pearl in ~
```

**成功!** Pine A64ボード上でのHypriotOS/arm64の最初のブートです。

### うさぎの穴のさらに奥深くへ: FELモードを使用してPine A64と通信する

OK。楽しかったですね。UART-to-USBコネクタケーブルとAndreのSDカード
イメージを使った小さなな実験によりPine上でLinuxを初めて体験することが
できました。しかし、もちろんまだ本物ではありません。Linuxを適切に
サポートするまでにはまだ険しい道が待っています。

より多くの知識を目指す我々の道はウサギの穴のさらに奥深くへと導きます。

より深い知識を得るために[FEL](http://linux-sunxi.org/FEL)モードと
呼ばれるもう一つの強力なツールを紹介します。FELモードはすべての
AllwinnerデバイスのブートROMにある低レベルのサブルーチンであり、
Pineに関する貴重な洞察を収集するために使用できます。これがどのように
機能するかについては後ほど説明します。

FELには情報の収集以外にも便利な応用があります。開発用のコンピュータから
USBケーブル経由でイメージを直接起動することを可能にします。このため、
SDカードを何度もフラッシュしたり挿入したりする煩わしいプロセスが不要に
なります。

Pine A64をFELモードにするのは非常に簡単です。SDカードスロットを空の
ままにして、適切なUSBケーブルでPine A64をコンピュータに接続するだけです。
次に、Pineの電源を入れます。特別なUSBケーブルをPineのUSBポートの上段に
接続する必要があることを知っておくことが重要です。これはこのUSBポート
しかFEL信号を使用するための配線がされていないためです。

![FELモードのためのUSB接続]()

この特別なUSB A-オス/A-オス ケーブルはAmazonで購入したり、[自分で作成](http://www.instructables.com/id/Male-to-Male-A-to-A-USB-Cable/)
することができます。Pine A64が起動するとホストコンピュータ上では
`USB ID 1f3a:efe8`の新規USBデバイスとして見つけることができます。

`lsusb`を実行すると次のようにデバイスが表示されるはずです。

```bash
$ lsusb
Bus 001 Device 004: ID 1f3a:efe8
```

FELモード経由でさらにアクセスするには`sunxi-tools`の最新バージョンを
入手してコンパイルする必要があります。我々は開発用コンピューターとして
HypriotOSが稼働するRaspberry Piを使用しています。そこで、必要な開発依存
関係をすべてインストールしてからsunxi-toolsリポジトリをクローンして
ソースからコンパイルします。

```bash
$ sudo apt-get update
$ sudo apt-get install -y make gcc pkg-config libusb-1.0-0-dev
$ makedir -p ~/code
$ cd ~/code
$ git clone https://github.com/linux-sunxi/sunxi-tools
$ cd sunxi-tools
$ make
$ sudo make install
```

では、sunxi-toolsを使用してPineに関するバージョン情報を抽出できるか
見てみましょう。

```bash
$ sudo sunxi-fel version
AWUSBFEX soc=00001689(unknown) 00000001 ver=0001 44 08 scratchpad=00017e00 00000000 00000000
```

やった。Pine A64ボードが接続されており、基本的なバージョン情報が取得
されました。`SOC ID #1689`はAllwinner A64 SOCであることを示しています。
デバイス自体は`sunxi-fel`ツールが認識するには新しすぎて"unknown"と
分類されています。

FELモードによるこの低レベルのアクセスにより何ができるでしょうか?

たとえば、デバイスメモリの128バイトを読み取り、16進ダンプとして画面に
表示できます。アドレス0x0000から表示してみましょう。

```bash
$ sudo sunxi-fel hexdump 0x0000 128
00000000: 08 00 00 ea 06 00 00 ea 05 00 00 ea 04 00 00 ea  ................
00000010: 03 00 00 ea 02 00 00 ea 11 00 00 ea 00 00 00 ea  ................
00000020: 13 00 00 ea fe ff ff ea 01 00 a0 e3 00 10 a0 e3  ................
00000030: 00 20 a0 e3 00 30 a0 e3 00 40 a0 e3 00 50 a0 e3  . ...0...@...P..
00000040: 00 60 a0 e3 00 70 a0 e3 00 80 a0 e3 00 90 a0 e3  .`...p..........
00000050: 00 a0 a0 e3 00 b0 a0 e3 00 c0 a0 e3 00 d0 a0 e3  ................
00000060: e8 f0 9f e5 04 e0 4e e2 ff 5f 2d e9 1f 07 00 eb  ......N.._-.....
00000070: ff 9f fd e8 d2 20 a0 e3 02 f0 21 e1 d0 d0 9f e5  ..... ....!.....
```

デバイスメモリを開発コンピュータのファイルにコピーして、`hexdump`コマンドで
その内容をローカルで表示することもできます。

```bash
$ sudo sunxi-fel dump 0x0000 128 > memory-dump.bin
$ hexdump -C memory-dump.bin
00000000  08 00 00 ea 06 00 00 ea  05 00 00 ea 04 00 00 ea  |................|
00000010  03 00 00 ea 02 00 00 ea  11 00 00 ea 00 00 00 ea  |................|
00000020  13 00 00 ea fe ff ff ea  01 00 a0 e3 00 10 a0 e3  |................|
00000030  00 20 a0 e3 00 30 a0 e3  00 40 a0 e3 00 50 a0 e3  |. ...0...@...P..|
00000040  00 60 a0 e3 00 70 a0 e3  00 80 a0 e3 00 90 a0 e3  |.`...p..........|
00000050  00 a0 a0 e3 00 b0 a0 e3  00 c0 a0 e3 00 d0 a0 e3  |................|
00000060  e8 f0 9f e5 04 e0 4e e2  ff 5f 2d e9 1f 07 00 eb  |......N.._-.....|
00000070  ff 9f fd e8 d2 20 a0 e3  02 f0 21 e1 d0 d0 9f e5  |..... ....!.....|
00000080
```

すでに説明したように、Allwinner SOCはブートROM ([BROM](http://linux-sunxi.org/BROM))を
ロードすることでブートプロセスを開始し、通常のブートを実行するか、FEL
サブルーチンを開始するかを検知します。FELモードと`sunxi-fel`ツールの助けを
借りてBROMのブートコード自体をコピーして分析することもできます。

では、BROMを直接Pineからダウンロードして後で検査できるように開発用
コンピュータ上のローカルファイルに保存しましょう。そのためにはBROMが開始
するメモリアドレスを知る必要があります。

残念ながら、A10やA20などの既存のAllwinner SoCで機能するメモリアドレスは
機能しません。しかし、[Linux-Sunxi wiki](http://linux-sunxi.org/EGON)には
重要な手がかりがあり、BROMの開始アドレスの付近でユニークな文字列`eGON.BRM`を
検索する必要があることを教えてくれます。

この知識と`sunxi-fel hexdump`コマンドを使ってこの文字列が見つかるまで
メモリをスキャンできます。そして幸運なことにアドレス**0x2c00**で始まる
BROMを見つけることができました。

```bash
$ sudo sunxi-fel hexdump 0x2c00 128
00002c00: 07 00 00 ea 07 00 00 ea 65 47 4f 4e 2e 42 52 4d  ........eGON.BRM
00002c10: 24 00 00 00 31 31 30 30 31 31 30 30 31 36 33 33  $...110011001633
00002c20: 00 00 00 00 00 00 00 ea 01 00 00 ea 00 60 a0 e3  .............`..
00002c30: 03 00 00 ea 5c 60 a0 e3 0e 00 00 ea e8 01 9f e5  ....\`..........
00002c40: 00 f0 90 e5 b0 0f 10 ee 03 10 00 e2 00 00 51 e3  ..............Q.
00002c50: f9 ff ff 1a ff 1c 00 e2 00 00 51 e3 f6 ff ff 1a  ..........Q.....
00002c60: c8 11 9f e5 c8 21 9f e5 00 30 91 e5 03 00 52 e1  .....!...0....R.
00002c70: 00 00 00 1a f0 ff ff ea 50 00 a0 e3 01 00 50 e2  ........P.....P.
```

では、BROMバイナリBLOBの32Kバイト全体をダウンロードして見てみましょう。

```bash
$ sudo sunxi-fel dump 0x2c00 32768 > pine64-a64-brom.bin
$ head -c 128 pine64-a64-brom.bin | hexdump -C
00000000  07 00 00 ea 07 00 00 ea  65 47 4f 4e 2e 42 52 4d  |........eGON.BRM|
00000010  24 00 00 00 31 31 30 30  31 31 30 30 31 36 33 33  |$...110011001633|
00000020  00 00 00 00 00 00 00 ea  01 00 00 ea 00 60 a0 e3  |.............`..|
00000030  03 00 00 ea 5c 60 a0 e3  0e 00 00 ea e8 01 9f e5  |....\`..........|
00000040  00 f0 90 e5 b0 0f 10 ee  03 10 00 e2 00 00 51 e3  |..............Q.|
00000050  f9 ff ff 1a ff 1c 00 e2  00 00 51 e3 f6 ff ff 1a  |..........Q.....|
00000060  c8 11 9f e5 c8 21 9f e5  00 30 91 e5 03 00 52 e1  |.....!...0....R.|
00000070  00 00 00 1a f0 ff ff ea  50 00 a0 e3 01 00 50 e2  |........P.....P.|
00000080
```

これらの例はすべてFELモードと`sunxi-fel`ツールに基づいています。これは
デバイスに関する情報を収集する方法を示しています。

FELモードについての紹介で、FELモードのもう一つの応用について触れました。
SDカードのフラッシュとブートを必要としないため、開発サイクルを短縮することが
できます。

このモードをPineで機能させるには、まず`sunix-fel`ツールにパッチを適用する
必要があります。Linux-Sunxiコミュニティの[Siarhei Siamashka](https://github.com/ssvb)の作業のおかげでこれは簡単です。

```bash
$ cd ~/code/sunxi-tools
$ mv fel.c fel.c.org
$ wget https://github.com/ssvb/sunxi-tools/raw/dc77476014669a6f9010a3160357391450a5196e/fel.c
$ make
$ sudo make install
```

やった、うまくいきました。`sunxi-fel`ツールがPine A64とAllwinner A64 SOCを
正しく検出するようになりました。

```bash
$ sudo sunxi-fel version
AWUSBFEX soc=00001689(A64) 00000001 ver=0001 44 08 scratchpad=00017e00 00000000 00000000
```

この準備が整ったのでセカンダリプログラムローダ (SPL または boot0)、U-Boot、
Linux カーネルとそのモジュールなどに関する作業がより便利になります。

### 不思議の国の内側: UART0コンソールとFELモードの合体

ここでこれまでに学んできたさまざまなことをまとめる時が来ました。USB A-オス/
A-オスケーブルとUART-to-USBコネクタケーブルを同時に使用するとソフトウェアを
Pine に送信して、それがどのように実行されるかを確認できます。これにより
迅速なフィードバックサイクルが可能になります。

boot0ブートローダのアップロードと起動でこれを試してみます。

簡単のために独自のboot0ブートローダを作成する代わりにAndreのLinux SDカード
イメージにある既存のブートローダを抽出します。これはセクタ16 (8Kバイト) に
あり、合計サイズは64セクタ、すなわち、32Kバイトです。

```bash
$ xzcat pine64_linux-20160121.img.xz | dd of=pine64-boot0.bin bs=512 count=64 skip=16
```

このファイルが本当に探している正しいboot0バイナリblobであるかどうかを
再確認しましょう。ファイルはバイト#5の文字列 "eGON.BT0" で始まるはずです。
結果はこうでした。

```bash
$ hexdump -C pine64-boot0.bin
00000000  cc 00 00 ea 65 47 4f 4e  2e 42 54 30 31 7a f6 a8  |....eGON.BT01z..|
00000010  00 80 00 00 30 00 00 00  00 00 00 00 00 00 01 00  |....0...........|
00000020  00 00 01 00 00 00 00 00  00 00 34 2e 30 2e 30 00  |..........4.0.0.|
00000030  00 00 00 00 01 00 00 00  a0 02 00 00 03 00 00 00  |................|
...
```

`sunxi-fel`の助けを借りるとこのboot0プログラムをPine A64に直接送信できます。

```bash
$ sudo sunxi-fel spl pine64-boot0.bin
```

UART0コンソールの出力を見るとプログラムのアップロードと起動が期待どおりに
動作していることがわかります。

```bash
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
[mmc]: mmc 0 cmd 8 timeout, err 00000100
[mmc]: mmc 0 cmd 8 err 00000100
[mmc]: mmc 0 send if cond failed
[mmc]: mmc 0 cmd 55 timeout, err 00000100
[mmc]: mmc 0 cmd 55 err 00000100
[mmc]: mmc 0 send app cmd failed
[mmc]: ***Try MMC card 0***
[mmc]: mmc 0 cmd 1 timeout, err 00000100
[mmc]: mmc 0 cmd 1 err 00000100
[mmc]: mmc 0 send op cond failed
[mmc]: mmc 0 Card did not respond to voltage select!
[mmc]: ***SD/MMC 0 init error!!!***
[mmc]: mmc 0 register failed
Fail in Init sdmmc.
Ready to disable icache.
```

コードをPineに送信して実行できることが確認できたので、さらなるステップへの
扉が開かれました。たとえば、新しいU-Bootブートローダの開発です。

数日前、SiarheiはA64 SOC用の最初に動作するU-Bootを作成し、Pine A64ボードに
アップロードして起動することができました。彼はこの
[開発中のU-Bootブートローダ](https://gist.github.com/ssvb/67ebb38e8f8f2b9b5ee6)
の完全なブートログを公開しました。

以下はすでに何が動作しているかを示すブートログの一部です。

```bash
U-Boot SPL 2016.01-00352-ge77e0e4-dirty (Jan 24 2016 - 10:26:33)
DRAM:DRAM driver version: V1.0
DRAM Type = 3 (2:DDR2,3:DDR3,6:LPDDR2,7:LPDDR3)
DRAM clk = 672 MHz
DRAM zq value: 3b3bbb
DRAM single rank full DQ OK
DRAM size = 1024 MB
DRAM init ok
 1024 MiB
Trying to boot from MMC

U-Boot 2016.01-00352-ge77e0e4-dirty (Jan 24 2016 - 10:26:33 +0200) Allwinner Technology

CPU:   Allwinner A64 (SUN50I)
DRAM:  1 GiB
MMC:   SUNXI SD/MMC: 0
...
```

### 予備試験の終わり

これで (少し交わし位ですが) このブログポストは終わりです。

我々の目標はPine A64におけるLinuxサポートの現状についての第一印象を与える
ことでした。また、それを実現するために必要なツールと知識についても紹介
したいと思っていました。

このブログポストはLinux-Sunxiコミュニティの偉大な方々なしには不可能でした。
特に、 [Andre Przywara](https://github.com/apritzel) (apritzel)氏と
[Siarhei Siamashka](https://github.com/ssvb) (ssvb)氏の助けは非常に貴重
でした。Allwinner Technologyからのサポートがほとんどない状態でどのように
このようなものをリバースエンジニアリングしなければならなかったかを思うと
最大の敬意を払う必要があります。Pineチームから[発表されたサポート](http://forum.pine64.org/forumdisplay.php?fid=17)が
物事を前進させるのに役立つことを心から願っています。

我々はこの開発を注意深く監視し、HypriotOSの初期サポートをできるだけ早く
公開したいと考えています。これによりDockerは非常に強力な開発者ボードに
アクセスできるようになります。我々はそれを早い段階でサポートしたいと考えて
います。

Pine Linux Wonderlandを巡る目まぐるしいツアーを楽しんでいただけたでしょうか。

いつものように以下のコメントを使用してフィードバックを提供してください。また、
HackerNewsでこの投稿について議論し、TwitterやGoogle、Facebookでこの投稿を
共有してください。

Dieter @Quintus23M と Govinda @beagile_
