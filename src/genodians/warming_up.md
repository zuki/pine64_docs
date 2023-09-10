# Pineを楽しむためのウォーミングアップ

[December 10 2020 by Norman Feske](https://genodians.org/nfeske/2020-12-10-pine-fun-warmup)

 私は最近、Pine64ボードとにPinePhoneを手に入れ、このプラットフォームに
 Sculpt OSを導入したいという願望を持っています。これはそのような
 移植作業のプロセスを記録する非常にありがたい機会です。

[前回のポスト](http://genodians.org/nfeske/2020-04-30-porting-sculpt)
でGenode、中でもSculpt OSを新しいARM SoCに導入するために必要な基本的な
手順について書きました。このような作業には、複雑すぎるハードウェアの
内部機能、適切なツールや手法の選択、十分な情報に基づくドライバの移植
か開発かの決定、そしてこれらすべてをGenodeに関連付けるという、非常に
多くの不確実性が伴います。

これらの不確定要素が組み合わさることで大きな障壁となります。Genode
Labsでは過去に何度かこの障壁を克服してきました。[最近では](https://genode.org/documentation/release-notes/20.11#Sculpt_OS_on_64-bit_ARM_hardware__i.MX8_EVK_)、NXP i.MX8
SoCをサポートしています。しかし、新しいハードウェアへのGenodeの移植
はGenode Labsだけの活動として放置すべきではありません。Genodeの
内部サークル以外の開発者にも楽しんでいただけるよう、私たちが知って
いることを共有したいと思います。この共有はガイドとして機能し、可能な
限り摩擦を取り除く深遠な文書という形を取る必要があります。

そのためには過去の経験をただ語るだけではなく、実際に行った手順を
書き留めながら一歩一歩歩いてみるべきだと考えました。そして良さそうな
ハードを探しながら歩を進めていたところ、[Pine64](https://www.pine64.org/)が目に留まりました。

### なぜ、Pine64なのか

私がPine64に興奮した理由はいくつかあります。

第一に、PinePhoneやA64開発ボードのようなフォームファクターの
デバイスが手頃な価格で入手できることです。Pine64のウェブサイトには、
コミュニティやオープン性、持続可能性、透明性、マーケティングなしを
強調する非常に前向きなメッセージが掲載されています。

第二に、製品がハッキングしやすいように設計されていることです。
これは活気のある開発者コミュニティ、mainline Linuxカーネルのサポート、
文字通り十数種類のLinuxディストリビューションが利用可能であること
からも明らかです。PinePhoneはSDカードから直接起動できます。なんと
クールなことだろう！

第三に、使用されているAllwinner SoCは2015年という早い時期に導入
され、枯れていることです。最先端のハードウェアとは異なり、未開拓の
領域を探索する必要はないものと思われます。他の人がほとんどの落とし
穴を私より先に発見しているはずです。このSoCは最新の機能（64ビット、
マルチコア、仮想化）と適度な複雑さのバランスがとれているようです。
SoCの性能はスマートフォンの製品カテゴリーの中でも特に低い方です。
オペレーティングシステム開発者の立場からすると、これは短所ではなく、
むしろ歓迎すべき挑戦です。Genodeはこのような制約のあるデバイスで
輝くことができるでしょうか。それを確かめましょう。

このSoCの特筆すべき唯一の欠点は、I/Oデバイスやドライバーの暴走に
対する保護機構としてのIO-MMUがないことです。そのため、デバイス
ドライバのサンドボックス化は決して完璧なものにはなりません。

## 第一印象

[Pine64-LTS](https://pine64.com/product-category/pinephone/)ボードと[PinePhone](https://pine64.com/product-category/pinephone/)、PinePhone用[シリアルケーブル](https://pine64.com/product/pinebook-pinephone-pinetab-serial-console/)をオンライン
ストアに直接注文しました。安全上の理由からPinePhoneは別に注文する
必要がありました。今から思えばPine64-LTSボード用の電源も注文して
おけばよかったです。私たちは何Kgもの他のボード用のAC電源ををすでに
持っていたので注文しませんでした。しかし、このボードにはあまり主流
ではない3.5mmコネクタがついており、5mmコネクタがついた何Kgもの電源は
ほとんど役に立たないことが判明しました。このような細部は時として
重要であす。

PinePhoneを触るのは純粋に楽しいことでした。特に、[p-boot](https://xnux.eu/p-boot/)ブートローダや
¹枚のSDカードに12種類以上のLinuxディストリビューションを収録した
すぐに使える[マルチディストロ](https://xnux.eu/p-boot-demo/)イメージなど、[Ondrej Jirman](https://xnux.eu/log/)の作業を
探求するのは楽しいことでした。モバイル向けのGNU/Linux開発の多様性を
知るにはまさにうってつけでした。個人的にはLune OS（Palm WebOSの
子孫）、Sailfish OS、Ubuntu touchが特に興味深かいものでした。

技術的な作業で自分の手を汚すために、PinePhoneは暫くの間は横に置き、
Pine-A64-LTSボードに注力する必要があるでしょう。[Pine64 wiki](https://wiki.pine64.org/index.php/PINE_A64-LTS/SOPine_Main_Page)は
完璧な出発点を提供してくれます。

### 公式にサポートされているGNU/Linuxイメージの起動

wikiにはすぐに使えるLinux[ディストリビューション](https://wiki.pine64.org/wiki/SOPINE_Software_Release)が多数掲載されて
います。私はArmbianを選びました。<https://dl.armbian.com/pine64so/Buster_current> からディスクイメージをダウンロードしてSDカードに
書き込み、HDMIディスプレイとUSBキーボードを接続してSDカードを挿入
したボードを起動すると、Armbianのログインが表示され、rootユーザと
してログインできるようになりました。ここまで数分しかかかりません
でした。

この時点で最も興味のあるのはハードウェアの概要を知ることです。
以下の情報が参考になります。

```bash
root@pine64so:/# cat /proc/cpuinfo
...
root@pine64so:/# cat /proc/meminfo
```

まあ、驚くことではありません。儀式のようなものです。

```bash
root@pine64so:/# dmesg | less
```

カーネルのブートログはすごくおしゃべりです。以下の行が目に留まりました。

```bash
[    2.228675] sun4i-drm display-engine: bound 1100000.mixer (ops 0xffff800010da9a30)
[    2.230477] sun4i-drm display-engine: bound 1200000.mixer (ops 0xffff800010da9a30)
[    2.231001] sun4i-drm display-engine: No panel or bridge found... RGB output disabled
[    2.231018] sun4i-drm display-engine: bound 1c0c000.lcd-controller (ops 0xffff800010da5360)
[    2.231227] sun4i-drm display-engine: bound 1c0d000.lcd-controller (ops 0xffff800010da5360)
[    2.231293] sun8i-dw-hdmi 1ee0000.hdmi: Couldn't get regulator
[    2.231734] sun4i-drm display-engine: Couldn't bind all pipelines components
```

...グラフィックスが表示できるようになったら、Linuxカーネルで
"sun4i-drm"と"sun8i-dw-hdmi"をgrepする必要があります。sun4iとsun8iが
何を意味するのかはわかりません。"dw"はDesignwareの略でしょうか。
一瞬ぞっとしました...

```bash
[    2.250163] 1c28000.serial: ttyS0 at MMIO 0x1c28000 (irq = 31, base_baud = 1500000) is a 16550A
[    2.250239] printk: console [ttyS0] enabled
[    2.250893] sun50i-a64-pinctrl 1c20800.pinctrl: supply vcc-pg not found, using dummy regulator
[    2.251327] 1c28400.serial: ttyS1 at MMIO 0x1c28400 (irq = 32, base_baud = 1500000) is a 16550A
[    2.251471] serial serial0: tty port ttyS1 registered
```

...Linuxカーネルはデフォルトで0x1c28000のシリアルコントローラを
使用しています。これがドライバを必要とする最初のデバイスになるで
しょう。"16550A"というデバイスは聞いたことがありませんが...

```bash
[    2.277178] ehci-platform 1c1b000.usb: EHCI Host Controller
[    2.277210] ehci-platform 1c1b000.usb: new USB bus registered, assigned bus number 3
[    2.277359] ehci-platform 1c1b000.usb: irq 22, io mem 0x01c1b000
[    2.289613] ehci-platform 1c1b000.usb: USB 2.0 started, EHCI 1.00
...
[    2.291208] ohci-platform 1c1b400.usb: Generic Platform OHCI controller
[    2.291228] ohci-platform 1c1b400.usb: new USB bus registered, assigned bus number 4
[    2.291342] ohci-platform 1c1b400.usb: irq 23, io mem 0x01c1b400
```

...OHCI USBコントローラ、過去の楽しい出来事をちょっと思い出しました...

```bash
[    2.384988] sunxi-mmc 1c0f000.mmc: initialized, max. request size: 16384 KB, uses new timings mode
[    2.410167] sunxi-mmc 1c10000.mmc: initialized, max. request size: 16384 KB, uses new timings mode
[    2.422925] mmc0: Problem switching card into high-speed mode!
[    2.423025] mmc0: new SDHC card at address 0001
```

...2つのマルチメディアカード（MMC）デバイスは明らかにAllwinner専用
コントローラで駆動されています。"Problem switching card into
high-speed mode!"。MMCとProbleはほぼ同義語です。Allwinnerが私たちを
積極的に驚かせることはないでしょう...。

```bash
[    3.412571] dwmac-sun8i 1c30000.ethernet: IRQ eth_wake_irq not found
```

...良いニュースは、単なるUSBネットワークデバイスではなく、専用の
イーサネットコントローラがあることです。悪いニュースは、その
コントローラがDesignwareから購入したIPコアだということです。
Raspberry PiのUSBで深い傷を負ったので"dw"の付くものには二度と
触れたくないと思っていたのですが...

```bash
[    9.189128] Call trace:
[    9.191219]  ktime_get_update_offsets_now+0x5c/0x100
[    9.193340]  hrtimer_interrupt+0xa0/0x2f0
[    9.195466]  sun50i_a64_read_cntpct_el0+0x30/0x38
[    9.197542]  arch_counter_read+0x18/0x28
[    9.199712]  arch_timer_handler_phys+0x34/0x48
[    9.201813]  handle_percpu_devid_irq+0x84/0x148
[    9.203971]  ktime_get_update_offsets_now+0x5c/0x100
[    9.206022]  hrtimer_interrupt+0xa0/0x2f0
[    9.208071]  generic_handle_irq+0x30/0x48
[    9.210150]  __handle_domain_irq+0x64/0xc0
... many more lines ...
```

...Linuxカーネルスレッドが起動中に死んでいます。シンボル"sun50i"は
Allwinner関連のドライバの問題を示唆しています。それでもカーネルは
動き続けています...

```bash
[    9.703995] lima 1c40000.gpu: gp - mali400 version major 1 minor 1
```

...プロプライエタリなブロブを必要とせずにGPUを手に入れることが
できて本当にうれしいです。Limaプロジェクトによるリバースエンジニア
リングの努力のおかげです。

ハードウェアに関する情報を明らかにするのはカーネルログだけでは
ありません。

```bash
root@pine64so:/# cat /proc/iomem

01000000-0100ffff : 1000000.clock clock@0
01100000-011fffff : 1100000.mixer mixer@100000
01200000-012fffff : 1200000.mixer mixer@200000
...
...
40000000-bdffffff : System RAM
```

これにより、実際のRAMだけでなく、すべてのメモリマップドデバイスの
ロケーションを含む、物理メモリレイアウトの完全なビューを得ることが
できました。（ほぼ）2GBの物理メモリは`0`からではなく`0x40000000`から
始まっています。

```bash
oot@pine64so:/# cat /proc/interrupts
```

これにより、Linuxカーネルで構成されているデバイスと割り込み番号、
CPUの間の関係（割り込みルーティング）を知ることができます。

もう1つの注目点は */proc/device-tree* でみることができるデバイス
ツリーです。これは実際には */sys/firmware/devicetree/base* への
シンボリックリンクです。

現時点でこの情報をすべて消化するには早すぎます。後のために保存して
おきましょう。最も簡単な方法はUSBメモリにデータを保存することです。

1. USBメモリを2番目のUSBポートに差し込むと、カーネルのdmesg出力は
   それが */dev/sdb* として検出されたこと、そして、たとえば最初の
   パーティションが */dev/sdb1* であることなど、パーティションに
   ついても教えてくれます。

2. パーティションのデバイス名がわかれば、*mount /dev/sdb1 /mnt*
   を実行することでファイルシステムを */mnt* にマウントできます。

3. これで興味のあるファイルを */mnt/* にコピーできます。

追加の機能テストとして、ネットワークインターフェイスをすぐに試して
みることができます。ネットワークケーブルをローカルネットワークに
接続すると、ネットワークPHYのLEDが嬉しそうに点滅し始め、`ifconfig`
コマンドを実行することでボードがローカルのDHCPサーバーからIP
アドレスを取得していることがわかります。すぐに
`wget https://genode.org`を実行すると期待通りに動作しました。

### シリアル・ライン

LinuxベースのOSを実行した際にボードが完全に機能することが確認できた
ので、このボードを組込み開発ターゲットとして使用することを目指さ
なければなりません。シリアルによるテキスト出力はそのための最も重要な
前提条件です。開発ボードが9ピンのD-SUBコネクターを備えていた時代は
とうの昔に過ぎ去りました。現在ではボードの拡張ソケットから適切な
ピンを探す必要があります。このボードにはそれのピンが複数あります。
なのでここは[ボードの回路図](https://files.pine64.org/doc/SOPINE-A64/PINE%20A64-TLS-20180130.pdf)に精通するいい機会です。

回路図はいくつかのシリアルデバイス（UART）を示唆しています。たとえば、
SDIO WIFI + BTピンヘッダのUART1です。定番の解決策は明らかでは
ありません。幸いなことにウェブを少し検索したところ[Pine64のUART](https://linux-sunxi.org/Pine64#Serial_port_.2F_UART)に
ついて説明した素晴らしいwikiページにたどり着きました。特に「近くに
あるEXPコネクタにある UART0を使用することがいつでも最善です。7ピン
(TXD)、8ピン (RXD)、 9ピン (GND)でアクセスできます」ということを学び
ました。

誰でも[TTL-232R-RPi](https://eu.mouser.com/productdetail/ftdi/ttl-232r-rpi?qs=3tuk1l7PSbNqj0mOeA5IPw%3D%3D)ケーブルの2,3本は持っているはずです。持って
いなければすぐに注文してください。信号レベルに注意してください。
今回の場合、ボードには3.3Vのケーブルが必要です。必要な線は
TXとRX、RXとTXのクロス接続とグランド接続だけです。

Linuxベースの開発マシンでは、私たちは通常、シリアル端末プログラムと
して[picocom](https://www.mankier.com/1/picocom)を使用しています。USBケーブルを接続するとLinuxカーネルの
dmesg出力が新しいデバイス */dev/ttyUSB0* について教えてくれますので
picocomで簡単にアクセスすることができます。

```bash
picocom --baud 115200 /dev/ttyUSB0
```

エンターキーを押すとArmbianのログイン画面が表示されます。

以後のステップではディスプレイとキーボードはもう必要ありません。
必要なのはシリアル回線だけです。

### JTAG

ほとんどのデバッグ作業にはシリアル出力で十分だと思います。しかし、
キャッシュコヒーレンシーの問題に直面したときのような絶望的な状況では、
LauterbachやFlyswatterなどのJTAGデバッガがあると本当に何日（あるいは
何週間）も時間を節約節約できます。そのため、新しいボードに出会った
ときには常にJTAGデバッグピンがあるか否を確認します。もしあれば、
最後の手段としてこのオプションが使えるという安心感を得ることが
できます。

Pine64の場合、この安心感なしで生きなければなりません。フォーラム
<https://forum.pine64.org> を検索したところ、SoCには確かにJTAGピンが
装備されているがPineボートの配線ではアクセスできないことを知りました。
どうやら、コミュニティ全体がJTAGへの関心が低すぎるようでした。
ほとんどのユーザはJTAGが選択のツールになるような低レベルはいじらない
からです。

## U-Bootブートローダ

 [U-Boot](https://www.denx.de/wiki/U-Boot)はARMプラットフォーム用の*正規*ブートローダとして広く認知
 されており、私たちGenode開発者も同意見です。私たちが高く評価する
 第一の理由は、U-BootがTFTPサーバからネットワーク経由でブート
 イメージを取り込むことができることです。これは私たちの作業フローの
 基本です。

第二の理由は、U-Bootはハードウェアをブートしたオペレーティング
システムにとって便利な状態にすることです。たとえば、U-Bootはシリアル
経由でメッセージを出力するため、シリアルコントローラを正しく初期化する
必要があります。これにはボーレートの設定やUSB FUEへの電源供給などの
面倒な作業が必要です。これらの準備はブートローダが行うため、Genodeの
ドライバはこれらのステップを省略してもうまく動作させることができます。

U-Bootの3つ目の大きな利点は、プロジェクトがサポートしているドライバの
豊富さです。もちろん、私たちはこれらのドライバのほとんどは実際には
使っていません。しかし、他の人たちは使っています。そのため、ドライバは
確実に動作し、よくメンテナンスされており、通常、Linuxカーネルの
ドライバほど複雑ではありません。このため、Genode用のドライバを開発
する際にこれらのドライバが非常に有用なリファレンスになります。

ArmbianはU-Bootを使用しているので原則としてU-Bootを使い続けることが
できます。ブート中にシリアルターミナルで<space>を押すと自動ブートを
止めることができます。すると対話型のU-Bootプロンプトが表示されます。

### U-Bootのソースからのビルド

ブートローダをソースからビルドすることは単に名誉なことだけではなく、
ブートプロセスについて理解し、完全にコントロールできるように
することでもあります。ブートローダをコントロールする能力は力を与え、
実験の場として機能します。Allwinnerベースのデバイス用にU-Bootを
手動でビルドする手順は[優れたドキュメント](https://linux-sunxi.org/Mainline_U-Boot)に記載されています。

参考までに、私が行った手順を以下に示します。

1. gitリポジトリをクローンして、最新リリースブランチをchekoutします。
   ```bash
   $ git clone git://git.denx.de/u-boot.git
   $ cd u-boot
   u-boot$ git checkout -b v2020.10 v2020.10
   ```
2. Pine64-LTSボートに適したデフォルトの構成を探します。おそらく名前に
   "pine"が含まれているものだと思われます。
   ```bash
   u-boot$ find configs/ | grep -i pine
   configs/pinebook-pro-rk3399_defconfig
   configs/sopine_baseboard_defconfig
   configs/pine64_plus_defconfig
   configs/pine64-lts_defconfig
   configs/pinebook_defconfig
   configs/pine_h64_defconfig
   ```
   幸運なことにPine64ボード用に*pine64-lts_defconfig* があるようです。
   ただし、PinePhone用がないのが目立ちます。<https://linux-sunxi.org/PinePhone> を見ると状況がはっきりします: 「現在のところ、この
   デバイス用のU-Boot configはないので、ハックとして、 一時的に
   pine64-lts_defconfig をビルドターゲットとして使用してください」。
   私はそれでいいと思います。

3. ARM Trusted Firmwareのビルド

   ARM Trusted Firmwareは低レベルのファームウェアインタフェース
   （セカンダリCPUコアの起動を考えてほしい）をSoCベンダ間で統一する
   取り組みです。Stefan Kalkowski氏による最近の記事でより詳しく
   説明されています。

   linux-sunxi.orgでビルド手順が説明されているのでそれに従うだけです。
   ビルド出力は私たちの注意を引く非常に教育的なものでした。
   ```bash
   $ make CROSS_COMPILE=aarch64-linux-gnu- PLAT=sun50i_a64 DEBUG=1 bl31
   ...
   CC      drivers/allwinner/axp/axp803.c
   CC      drivers/allwinner/axp/common.c
   CC      drivers/allwinner/sunxi_msgbox.c
   CC      drivers/allwinner/sunxi_rsb.c
   ...
   CC      plat/allwinner/sun50i_a64/sunxi_power.c
   CC      plat/common/plat_gicv2.c
   ...
   Built /home/no/pine64/arm-trusted-firmware/build/sun50i_a64/debug/bl31.bin successfully
   ```
   他にもたくさん出力されてますが、興味深い詳細を示しています。
   たとえば、*drivers/allwinner/axp/axp803.c*はAXP電源管理チップの
   デフォルト設定を含んでおり、*plat/allwinner/sun50i_a64/sunxi_power.c*は
   メモリマップドI/Oを介してAXPチップにアクセスする方法を教えて
   くれます。

4. ブートローダのSDカードへのインストール

   手順は <https://linux-sunxi.org/Bootable_SD_card> に詳しく説明
   されています。私にとっては、すでにPCハードウェアのSculpt OSで
   使用しているGPTパーティショニングスキームを使用するオプションが
   あることが素晴らしいことです。これは後の段階で便利なものになる
   でしょう。

### U-Bootの便利なコマンド

準備したSDカードからU-Bootを起動すると、U-Bootが初期化され、
たくさんのデバイスをプローブしているのを見ることができます。
現在の状況では、**ネットワーク経由のブート**が最も重要な機能です。
そこで、bootpコマンドに注目しました。

```bash
=> help bootp
bootp - boot image via network using BOOTP/TFTP protocol

Usage:
bootp [loadAddress] [[hostIPaddr:]bootfilename]
```

早速試してみましょう。開発マシンのIPアドレスは10.0.0.32で、ローカル
ネットワーク内にあり、たまたまTFTPサーバが稼動しています。試しに、
*something*と名前の小さなファイルをTFTPディレクトリに置き、U-Bootに
以下のコマンドを発行してみました。

```bash
=> bootp 10.0.0.32:/var/lib/tftpboot/something

TFTP from server 10.0.0.32; our IP address is 10.0.0.178
Filename '/var/lib/tftpboot/something'.
Load address: 0x42000000
```

もちろん、起動のたびに手動でこのコマンドを入力したくはありません。
U-Bootにコマンドを自動的に実行するように指示した方がずっとマシです。
これはU-Bootの`bootcmd`環境変数をカスタマイズすることで可能です。

```bash
=> help editenv
editenv - edit environment variable

Usage:
editenv name
    - edit environment variable 'name'

=> editenv bootcmd
edit: bootp 10.0.0.32:/var/lib/tftpboot/something
```

`bootcmd`を自分好みにカスタマイズしたら、新しい設定を保存
しましょう。U-Bootには`saveenv`というコマンドがあり、MMC/SDカードの
あらかじめ定義された場所に設定を保存することができます。

```bash
=> saveenv
Saving Environment to FAT... Card did not respond to voltage select!
Failed (1)
```

あれ、これは予想通りには動きませんでした。その理由はMMCデバイスが
2つ存在するからです。SDカードは1つ目のMMCコントローラに接続されて
いますが、U-Bootは2つ目のMMCコントローラ経由で環境を保存するように
設定されているようです。幸いなことに、後者の設定はU-Bootのビルド
構成で設定できます。

*u-boot/.config*に`CONFIG_ENV_FAT_DEVICE_AND_PART`という構成
変数をみつけました。対話的な`menuconfig`における対応する構成は
*Environment*サブメニューにあります。

```bash
(1:auto) Device and partition for where to store the environemt in FAT
```

設定を0:autoに変更すればうまくいくはずです。もちろん、U-Bootを
ビルドしてSDカードに書き込む手順をもう一度踏む必要があります。
しかしそれは私たちを待っている便利さの代償としては小さなものです。

次にU-Bootを使う際に`bootcmd`を自分好みに編集し直して`saveenv`
コマンドを実行すると笑顔になれます。

```bash
=> saveenv
Saving Environment to FAT... OK
```

これ以後はブート時のキーストローク数を節約できます。最後にもう一つ
手を加えれば、さらに快適さが増します。デフォルトではU-Bootはブート時に
USBコントローラを初期化します。これには数秒かかり、ブート時間が
遅れます。私たちの開発ワークフローではUSBデバイスからブートする
予定はないので**USBの初期化はスキップ**した方がよいです。
これは`preboot`環境変数を"usb start"から"nothing"に変更し、
もちろん、`saveenv`コマンドで変更を永続化することで可能です。

次のステップは、このブートメカニズムを使って、プリミティブな
[シリアル出力を行う](https://genodians.org/nfeske/2020-12-17-pine-fun-serial)カスタムコードをロードして実行することです。
