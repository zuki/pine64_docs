# Pineを楽しむ - Linuxを散歩に連れ出す

[May 12 2021 by Norman Feske](https://genodians.org/nfeske/2021-05-12-pine-fun-linux)

LinuxのドライバをGenodeに移植する準備としてドライバの自然な習性に関する
知識を収集する必要があります。この記事では選択したドライバに合わせて
カスタマイズされたカスタムLinuxシステムを構築する手順について説明します。

GenodeのデバイスドライバのほとんどはGenode用に一から書かれたものではなく、
Linuxカーネルなどの実績のあるコードベースから移植されたものです。ほとんどの
ARM SoCのドライバの場合、ドライバコードはSoCベンダーによって提供されたもので
あり、通常、ハードウェアがどのように動作するかを示す唯一の参照になります。
デバイスの驚くべき複雑さと製品サイクルの速さ、使用されるIPコアの多様性を
考慮すると、ドライバを一から開発することは題外です。私は[以前の記事](https://genodians.org/nfeske/2019-11-20-arm-soc-landscape)でパワー
ダイナミクスとインセンティブについて書いています。

## モチベーション

以前、私たちは単なる直感に従ってドライバの移植作業に直接取り組み、Linux
カーネルコードの断片をGenodeコンポーネントに移植してコードを生き返らせる
ことがありました。Intel GPUドライバの最初のポートのようないくつかの
ケースでは、事前にドライバコードをLinux上でテストすることなく、幸運にも
Genodeでドライバを起動できました。ただし、これはまれな例外にすぎません
でした。ほとんどの場合、特に予測可能な成功と開発スケジュールが望まれる
場合はドライバ移植作業の一環としてカスタムLinuxシステムの構築と統合に
取り組む必要がありました。Linuxに立ち返るのは追加作業ではありますが、
それにより3つの貴重な利点が得られます。

第1に、ドライバがリファレンスハードウェア上でどのように動作するかがわかり、
**安心感**が得られます。機能セット、安定性、パフォーマンスに関する期待を
設定できます。ドライバが移植できたらLinuxとGenodeの両システムでドライバの
ベンチマークを実行して成功を検討することができます。

第2に、ユーザレベルのインタフェースおよびハードウェアとドライバとの
相互作用を**研究**し、**計測**できるようになります。ユーザランドと
ドライバの相互作用を把握するには常識的なやり方（`ioctl`の呼び出しを
検討してください）を用いてドライバインタフェースを鍛えると便利です。

第3に、最も重要なことは、自然環境におけるドライバの動作と隔離された
Genodeコンポーネント内での実行との**相互相関**が可能になることです。
結局のところ、ドライバがうまく動くためにはLinuxカーネルAPIを模倣する
必要があります。両環境で実行しているドライバの比較はドライバ環境を作成
する際に役立ちます。

## 具体的な目標を選ぶ

さまざまなクラスの周辺機器の中でも*ネットワーク*デバイスは最初に移植する
ドライバとして魅力的な対象です。[前の記事](https://genodians.org/nfeske/2021-04-29-platform-driver)で
説明したPIOデバイスとは対照的に、ネットワークデバイスは移植アプローチから
具体的なメリットを得ることが*できるほどには複雑*です。逆に、ネットワーク
ドライバは圧倒されない*程度にはにシンプル*です。さらに、ネットワーク接続は
自動化された方法で*簡単にテストできる*ため、面倒な手によるワークフローに
煩わされることがありません。最後に、ネットワークサポートを有効にすることで
得られる*メリット*は非常に大きいです。一つのドライバによりGenodeシステム
シナリオの世界全体が開かれます。

以下では、次の手順で複雑さを最小限に抑えたカスタムLinuxシステムを構築して
いきます。

- U-Bootを使用して*任意の*LinuxベースのOSを起動する方法を学習する
- Busyboxに基づくカスタムinitrdを使用する
- *カスタムビルドした*カーネルを起動する
- *ネットワークデバイス*を動作させる
- *カーネル構成*を削減する
- 調査結果を*再現可能*にする

以下にまとめた情報の収集は必ずしもスムーズに進んだわけではないことに
注意してください。[Stefan](https://genodians.org/skalk/index)の貴重な
ヒントと技術的・精神的なサポートに感謝します。

## U-Bootを使用したLinuxのブートストラップ

私たちの開発ワークフローでは以前、[ここ](warming_up.md)と
[ここ](baremeta.md)で説明したネットワークブートの利便性を維持したいと
考えています。Linuxのブートはこれまでのような単一のバイナリのブートとは
異なります。これらの違いを理解するには動作確認されている要素に頼るのが
良いでしょう。

[Armbian](https://www.armbian.com/)ディストリビューションのイメージが
ボード上でどのようにうまく動いたのを[思い出せば](warming_up.md)、どこを
見ればよいかがわかります。イメージの内容にアクセスする方法は複数あります。
Gnomeを使用している場合はImageをクリックしてマウントできるでしょう。
ここではイメージを手作業でループマウントしました。まず、パーティション
テーブルを調べてファイルシステムがどこから始まるかを見つけます。

```bash
$ fdisk -l Armbian_21.05....img
Disk Armbian_21.05....img: 1,7 GiB, 1782579200 bytes, 3481600 sectors
...
Armbian_21.05....img1  8192 3481599 3473408  1,7G 83 Linux
```

ブロック8192から始まっています。通常のブロックサイズは512なので、計算機の
助けを借りるとファイルシステムをループマウントするための引数であるバイト
オフセットを知ることができます。

```bash
$ python -c "print 8192*512"
4194304
$ sudo mount -oloop,offset=4194304 Armbian_21.05....img /mnt
```

ファイルシステムの*/boot/*ディレクトリで対象となるファイルは次のとおりです。

`Image`

> Linuxカーネルへのシンボリックリンクです。ここでは*vmlinuz-5.10.34-sunxi64*
> を指しています。21MiBのファイルです。

`config-5.10.34-sunxi64`

> カーネル構成ファイルです。ソースからカーネルを構築すると作成され、
> 便利なものになります。後で使用するために保存します。

`dtb/allwinner/`

> いわゆるデバイスツリーファイルのディレクトリです。これについては後ほど
> 説明します。現時点では、各dtbファイルには特定のボードのハードウェア
> 記述が含まれていることを知っていれば十分です。ディレクトリに存在する
> 48個のdtbファイルのうち、Pine-A64-LTSボード用のファイルを次のように
> 特定しました。

   ```bash
   $ ls -1 /mnt/boot/dtb/allwinner/ | grep a64 | grep lts
   sun50i-a64-pine64-lts.dtb
   ```

`initrd.img-5.10.34-sunxi64`

> ユーザランドのブートストラップに使用される初期RAMディスクです。

Linuxをブートするには、カーネル (イメージ)、ボード用のdtbファイル
(pine64.dtb)、initrdをTFTP ディレクトリ経由で提供する必要がありあす。
これは次のU-Bootコマンドを使用してこれらを行うことができます。
アドレスはカーネル用のU-Bootのデフォルトのロードアドレスを使用して、
重複しないように選択しています（アドレスはオリジナルの数値では
カーネルとinitrdが重なったので最新バージョンに合わせて修正した）。

1. カーネルをロードする
    ```bash
    => bootp 0x43000000 10.0.0.32:/var/lib/tftpboot/Image
    ```
2. 初期RAMディスクをロードする
   ```bash
   => bootp 0x41000000 10.0.0.32:/var/lib/tftpboot/initrd
   ```
3. デバイスツリーファイルをロードする
   ```bash
   => bootp 0x42f00000 10.0.0.32:/var/lib/tftpboot/pine64.dtb
   ```
4. デバイスツリーを変更してカーネルパラメタを提供する。まず、U-Bootの
   FDTツールにDTBデータの場所を伝える
   ```bash
   => fdt addr 0x42f00000
   ```
   追加コンテンツ用のスペースを作成する
   ```bash
   => fdt resize 0x1000
   ```
   initrdの配置場所（開始アドレスとサイズ）などのカーネルパラメタを
   含む、`chosen`デバイスツリーノードを変更する
   ```bash
   => fdt chosen 0x41000000 0x115f3c1
   ```
   さらに、カーネルコマンドラインを指定する
   ```bash
   => fdt set /chosen bootargs "rdinit=/bin/sh"
   ```
5. カーネルアドレスとDTBアドレスを指定してカーネルを起動する。すでに
   `chosen`DTBノードを介してinitrdの場所をカーネルに伝えているので
   2番目 (initrd) 引数に `-` を指定できる
   ```bash
   => booti 0x42000000 - 0x41f00000
   ```

言うまでもなく、この一連のコマンドを手作業で実行するとエラーが発生
しやすく、不安です。幸いなことに、U-Bootでは次のように一連のコマンドを
環境変数に保存できます。

```bash
=> setenv lx 'command; another command; ...'
```

`lx`という名前の変数にセミコロンで区切られた一連のコマンドが設定されます。
`lx`変数の値は[以前](warming_up.md)説明したメカニズムで永続化することが
できます。

```bash
=> saveenv
```

変数`lx`に保存されている一連のコマンドは`run`コマンドで実行することが
できます。

```bash
=> run lx
```

次のように各コマンドが実行されていきます。

```bash
ethernet@1c30000 Waiting for PHY auto negotiation to complete........ done
...
Using ethernet@1c30000 device
TFTP from server 10.0.0.32; our IP address is 10.0.0.178
Filename '/var/lib/tftpboot/Image'.
Load address: 0x42000000
Loading: #################################################################
        #################################################################
        ....
        #####################################
        1.6 MiB/s
done
Bytes transferred = 32020992 (1e89a00 hex)
BOOTP broadcast 1
DHCP client bound to address 10.0.0.178 (3 ms)
Using ethernet@1c30000 device
TFTP from server 10.0.0.32; our IP address is 10.0.0.178
Filename '/var/lib/tftpboot/initrd'.
Load address: 0x41000000
Loading: #################################################################
        ....
        ##############
        2.2 MiB/s
done
Bytes transferred = 10052488 (996388 hex)
BOOTP broadcast 1
DHCP client bound to address 10.0.0.178 (2 ms)
Using ethernet@1c30000 device
TFTP from server 10.0.0.32; our IP address is 10.0.0.178
Filename '/var/lib/tftpboot/pine64.dtb'.
Load address: 0x41f00000
Loading: ##
        1.2 MiB/s
done
Bytes transferred = 28418 (6f02 hex)
## Flattened Device Tree blob at 41f00000
Booting using the fdt blob at 0x41f00000
EHCI failed to shut down host controller.
Loading Device Tree to 0000000049ff5000, end 0000000049ffffff ... OK

Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x410fd034]
[    0.000000] Linux version 5.10.34-sunxi64 ...
...
... a few hundred lines of boot messages ...
...
[    6.593071] Freeing unused kernel memory: 5888K
[   37.865818] vcc-1v2-hsic: disabling
```

Enterを押すとshellのプロンプトが現れます。

```bash
#
```

ルートのシェルがコマンドの入力を待ちます。素晴らしいですね。

## Busyboxを使ったカスタムinitrd

U-BootによるLinuxのネットワークブートプロセスが明確になったので、3要素の
各々を段階的に置き換えることができます。

まず、ArmbianのinitrdをBusyboxを使ってカスタム構築したRAMディスクに置き
換えます。これにより、Armbianイメージから取り出したinitrdの10分の1の
サイズの機能する再現可能でカスタマイズ可能なユーザランドが得られます。
私たちは最終的にはすべてのバッテリーをボードに搭載し、動的モジュールの
スイッチをオフにした最小限のLinuxカーネルの作成を目指しているので、結局の
ところ、カーネルモジュールのキャリアとしてのinitrdは必要としません。

Busyboxの構築には[Sebastianの記事](https://genodians.org/ssumpf/2020-02-24-linux-vm)で説明されている手順が参考になります。

TFTPディレクトリのArmbianのinitrdをカスタムinitrdに置き換えたら、`chosen`
デバイスツリーノードを変更しているU-Bootコマンドでサイズを変更するために
少し調整する必要があります。

## ベンダーのカーネルソース

ほとんどのSoCでは、チップベンダが作成したLinuxカーネルソースがインターネット
上のどこかで提供されています。これらのいわゆるベンダーカーネルは、通常、
いくぶん苦い妥協策です。ベンダーカーネルは通常、デバイスドライバに関する
ベンダーのノウハウが詰め込まれており、ハードウェアに合わせて調整されている
一方で、その「調整」は Linuxカーネル開発の明文化されたルールにも明文化
されていないルールにも常に従っているわけではないので、ベンダーコードを
上流のカーネルにきれいに統合する妨げとなっています。従おうとさえしない
ベンダーも存在します。したがって、特定のチップ用のベンダーカーネルは
公開後はカーネル開発が止まり、それ以上愛されることはないと考えなければ
なりません。ベンダーのドライバが得られたという我々の状況ではSoCのベンダー
カーネルは参照として使用する必要があります。

Allwinner A64 SoC の場合、[ベンダーカーネル](https://linux-sunxi.org/Pine64#BSP)はバージョン 3.10という古いLinuxに基づいています。ただし、
SoCベンダーとは独立して活動する[Sunxi](https://linux-sunxi.org/)オープン
ソースコミュニティの多大な努力のおかげで、現在ではA64はアップストリーム
Linuxカーネルによって完全にサポートされています。A64 SoCに適したカーネルを
構築する[手順](https://linux-sunxi.org/Pine64#Linux_Kernel)は
デフォルトの構成を使用してAARCH64アーキテクチャ用にLinuxカーネルを
コンパイルすることになります。

肝心なのは<https://kernel.org>から最新のカーネルを選択して使用できると
いうことです。

```bash
$ wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.12.1.tar.xz
 ...
$ tar xf linux-5.12.1.tar.xz
```

これによりカーネルソースが*linux-5.12.1*サブディレクトリに展開されます。
以下ではこのディレクトリを`LX_DIR`と呼びます。環境変数にこのパスを覚え
させましょう。単に便利だからです。

```bash
export LX_DIR=$PWD/linux-5.12.1
```

## カスタムカーネルのビルドと起動

デフォルトのカーネル構成を試してみましょう。精神衛生上の理由から
私はソースツリーとは別にビルドディレクトリを使用することを好みます。
まだ存在しない場所を指すビルドディレクトリを`LX_BUILD_DIR`と呼びましょう。

```bash
export LX_BUILD_DIR=$PWD/lx_build
```

AARCH64アーキテクチャをターゲットとするLinuxビルドシステムを使用する場合、
makeの引数として`ARCH=arm64`を常に指定する必要があります。この引数は
`make help`の出力も変更することに注意してください。したがって、この引数を
一貫して適用する必要があります。何度も繰り返す必要がないように環境変数に
設定しましょう。

```bash
export ARCH=arm64
```

`ARCH`引数と同様、カーネルのクロスコンパイルに使用するツールチェーンに
ついてもカーネルのビルドシステムに伝える必要があります。ここではGenodeの
ツールチェーンを使用します。これは通常、*/usr/local/genode/tool/<version>/bin/*にインストールされ、 `genode-aarch64-`というプレフィックスが付けられて
います。カーネルのビルドシステムはこの情報を`CROSS_COMPILE`引数または
環境変数として期待します。

```bash
export CROSS_COMPILE=/usr/local/genode/tool/21.05/bin/genode-aarch64-
```

これらの予防措置を講じたら、次のコマンドでビルドディレクトリをデフォルト
構成で初期化することができます。

```bash
make -C $LX_DIR O=$LX_BUILD_DIR defconfig
```

カーネル構成はビルドディレクトリの`.config`ファイルにあります。
「デフォルト」構成がどのようなものかは、有効なオプションの数でわかります。

```bash
$ grep =y $LX_BUILD_DIR/.config | wc -l
2482
```

私はこの数字を*私の頭では処理できない*と解釈しました。

このデフォルト設定とArmbianイメージに含まれていたconfig-5.10.21-sunxi64を
比較するとさらなる敗北感が生まれます。

```bash
$ diff $LX_BUILD_DIR/.config config-5.10.21-sunxi64 | wc -l
10363
```

双方の構成を互に関連付けても駄目なようです。

難しい作業はなしに、カーネルイメージは次のように構築できます。

```bash
$ make -C $LX_BUILD_DIR Image -j8
  ...

  LD vmlinux
  SORTTAB vmlinux
  SYSMAP System.map
  OBJCOPY arch/arm64/boot/image
```

私のラップトップでは約20分かかりました。ビルドシステムのメッセージが示して
いるように作成されたカーネル`Image`は*Arch/arm64/boot/*にあります。
サイズは32MiBです。

ちなみに、カーネルのビルドシステムは圧縮されたカーネルイメージなど、他にも
多数の便利なターゲットを提供してます。オプションを検討してみる価値はあります。

```bash
$ make -C $LX_BUILD_DIR help
```

ここで、DTBファイルを作成するための便利なターゲットが存在することが
わかります。

```bash
$ make -C $LX_BUILD_DIR dtbs
```

これによりボードに合ったdtbファイルを*Arch/arm64/boot/dts/*から取り出す
ことができます。ここではソースから作成された一つのバリアントである
*allwinner/sun50i-a64-pine64-lts.dtb* でブートプロセスの3番目の魔法の
パズルのピースが置き換えられます。

ボード上で焼きたてのカーネルを試してみると、カーネルが起動し、シリアル
回線を介してカーネルログが表示され、ルートシェルが表示されることが
わかりました。私たちには歩くためのしっかりとした地面があるようです。

次に、ネットワークに注目してみましょう。最初の衝動はネットワークデバイスの
存在を確認するために`ifconfig`を発行することです。残念、何もありません。
悪いニュースです。

この問題はいくつかの角度から調べることができます。

- ALLWINNER、SUNXI、NET などのパターンで *.config*ファイルをgrepする
- 手がかりを探して`make menuconfig`のメニューをさまよう
- Armbianカーネルとカスタムビルドカーネルのブートログを比較し、ネット
  ワーク関連のメッセージを探す

これらのうち、おそらく最後の項目が最も無駄とならないでしょう。ただし、
成功と幸福への別のルートがあります。ボードのデバイスツリーを詳しく見て
みましょう。

## 宝の山デバイスツリー

いわゆるデバイス ツリー ([PDF](https://github.com/devicetree-org/devicetree-specification/releases/download/v0.3/devicetree-specification-v0.3.pdf) ) は
ペリフェラルやバス、割り込みコントローラ、クロックなどで構成される
ハードウェアプラットフォームの準形式的な記述です。Linuxの文脈では
これは2つの目的を果たします。第1二、実行時には信頼性の高い方法でデバイスを
プローブできないようなフォームでの動的にデバイスを検出する際のギャップを
埋めます。これはほとんどのARM SoCに当てはまります。デバイスツリーを参照する
ことでカーネルはどのドライバーを使用するかを学習します。第2に、デバイス
ツリーは個々のドライバーをパラメタ化します。これによりドライバはソース
コードにハードコーディングされたベンダー固有のパラメータを避けることが
できます。デバイスツリーからパラメタを取得することにより、ドライバは
SoCの他のリビジョンをより簡単に再ターゲットすることができます。

ドライバをGenodeに移植するという私たちの願望にとって、デバイスツリーは
祝福です。これらはSoCベンダーによって監督されているため、他のどこにも
見ることができないハードウェアのドキュメントを含んでいます。さらに、
大まかなドキュメント（そもそもあればですが）とは対照的に、デバイスツリーに
含まれる情報は正しいもとと想定できます。実際にハードウェアの駆動に役に
立っているからです。ある意味、デバイスツリーはベンダーからハードウェア
ドキュメントを奪い取るためのソーシャルエンジニアリングトリックのように
思えます。宝物を得るにはソースツリーの*arch/arm64/boot/dts/*を調べる
必要があります。

```bash
$ find $LX_DIR/arch/arm64/boot/dts
```

そこには約800個のファイルが見つかります。ただし、森はよく整備されています。
ベンダーを見つけて検索を絞り込むのは簡単です。ここではこうです。

```bash
$ find $LX_DIR/arch/arm64/boot/dts | grep allwinner
...
$ find $LX_DIR/arch/arm64/boot/dts | grep allwinner | grep pine
...
$ find $LX_DIR/arch/arm64/boot/dts | grep allwinner | grep pine | grep lts
.../arch/arm64/boot/dts/allwinner/sun50i-a64-pine64-lts.dts
```

このファイルを調べるとCプリプロセッサを使用して多数のファイルを
インクルードしていることがわかります。全体像を把握するにはCの前処理
を行う必要があります。

```bash
$ cpp -I $LX_DIR/include -x assembler-with-cpp -P \
       $LX_DIR/arch/arm64/boot/dts/allwinner/sun50i-a64-pine64-lts.dts \
       > flat_pine64lts.dts
```

`-x Assemblyler-with-cpp`引数は、Cプリプロセッサが先頭に#が付いている行を
プリプロセッサディレクティブとして誤って解釈するのを防ぐために必要です。
`-P`引数は出力からラインマーカーを削除します。結果をファイル
(*flat_pine64lts.dts*) に保存すると後でいつでも参照できるので便利です。
Pine-A64-LTSボードの場合、このファイルは1600行の洞察、すなわち、多くの
重要な数値と人間可読な用語との関係で構成されています。

## ネットワークサポートを有効にする

使用するボードのデバイスツリーの研究を終えてリフレッシュしたので、
ボードのネットワーキングを有効にするというトピックに戻る良い機会です。
フラット化されたdtsファイルをざっと確認すると"ethernet"という用語を
取り上げている次のノードが注目を集めます。

```bash
emac: ethernet@1c30000 {
  compatible = "allwinner,sun50i-a64-emac";
  syscon = <&syscon>;
  reg = <0x01c30000 0x10000>;
  interrupts = <0 82 4>;
  interrupt-names = "macirq";
  resets = <&ccu 13>;
  reset-names = "stmmaceth";
  clocks = <&ccu 36>;
  clock-names = "stmmaceth";
  status = "disabled";
  MDIO: mdio {
   compatible = "snps,dwmac-mdio";
   #address-cells = <1>;
   #size-cells = <0>;
  };
 };
```

`compatible`と呼ばれるプロパティを定義している2行に注目してください。
これらのプロパティはこのハードウェアと通信するために使用するドライバを
示します。実際にソースコードとの直接のつながりを示しています。デバイス
このツリーノードを言い換えると次のようになります。

*このボードでイーサネットを使用するには、カーネル構成で "allwinner,sun50i-a64-emac"と"snps,dwmac-mdio"に関連するソースコードを含める必要があります。*

"「"allwinner,sun50i-a64-emac"から始めましょう。

```bash
$ grep -r "allwinner,sun50i-a64-emac" $LX_DIR/drivers/net
.../stmmac/dwmac-sun8i.c: { .compatibility = "allwinner,sun50i-a64-emac",
```

どうやら、大ハンマーが干し草の山から針を選び出すのに最適な道具であることも
あるようです。コードを見るのにあまり時間を無駄にしないようにしましょう。
私たちにとって重要な情報はコンパイルユニット`dwmac-sun8i.c`の名前だけです。
慣例として、対応するオブジェクトファイルは通常、サブシステムのmakeファイル
に存在します。

```bash
$ grep "dwmac-sun8i.o" $LX_DIR/drivers/net/ethernet/stmicro/stmmac/Makefile
obj-$(CONFIG_DWMAC_SUN8I) += dwmac-sun8i.o
```

何という啓示でしょう。デバイスツリーから判明したノードとカーネル構成
オプションへの接続を知ることができました。カーネルの`.config`ファイルを
見ると、ドライバがモジュールとして構成されているため、カーネルが組み込み
ドライバでイーサネットデバイスを駆動していないことがわかります。

```bash
$ grep CONFIG_DWMAC_SUN8I $LX_BUILD_DIR/.config
CONFIG_DWMAC_SUN8I=m
```
多くの場合、カーネルオプションには依存関係があります。`CONFIG_DWMAC_SUN8I`
ドライバの依存関係を把握するには付属の*Kconfig*ファイルを参照するか、
`make menuconfig`対話型カーネル構成ツールのメニューで対応する項目の
ヘルプを表示します。

```bash
$ make -C $LX_BUILD_DIR menuconfig
```

検索 (`/`} を使用して目的のオプション（`CONFIG_DWMAC_SUN8I`など）を検索し、
メニュー階層内のオプションの場所を確認し、yでマークされていない依存関係を
探します。ここでの具体的なケースでは`DWMAC_SUN8I`を満たすために2つの
オプション `STMMAC_ETH`と`STMMAC_PLATFORM` を有効にする必要がありました。

オプションを有効にするには、対話型メニュー構成ツールを使用することが
できますが、私はスクリプトによる非対話型の方を好みます。カーネルソース
ツリーには次のようにオプションを有効/無効にできる*scripts/config*という
便利なツールがあります。

```bash
$ $LX_DIR/scripts/config --file $LX_BUILD_DIR/.config \
       --enable STMMAC_ETH --enable STMMAC_PLATFORM --enable DWMAC_SUN8I
```

このツールは単に個々のオプションを設定または削除するだけです。潜在的な
不整合や依存関係を解決するには続けて`make olddefconfig`を実行する必要が
あります。

```bash
$ make -C $LX_BUILD_DIR olddefconfig
```

作成された`.config`を見るとすべての依存関係が実際に満たされていることを
確認できます。

```bash
$ grep CONFIG_DWMAC_SUN8I $LX_BUILD_DIR/.config
CONFIG_DWMAC_SUN8I=y
```

上記の手順は長く感じるかもしれません。しかし、カーネル構成を行き当たり
ばったりに判断する方法とは対照的に、この手順は決定論的で説明可能な
プロセスです。

"allwinner,sun50i-a64-emac"デバイスツリーノードのドライバーをカバーできた
ので次は"snps,dwmac-mdio"ノードに対してこのプロセスを繰り返す必要が
あります。このノードはすでに`STMMAC_PLATFORM`でカバーされていることが
わかりました。

ネットワークを備えたLinuxカーネルを迅速にテストする便利な方法として
カーネルにブートプロセスの一部としてDHCP要求を発行するように指示できます。
これはカーネルコマンドラインに引数`ip=dhcp`を指定することで有効にできます。
参考までに、U-Bootのfdtコマンドを使用すると`chosen`デバイスツリーノードを
次のコマンドで調整できます。

```bash
=> fdt set /chosen bootargs "rdinit=/bin/sh ip=dhcp"
```

ドライバの依存関係がすべて解決されると`ifconfig`はイーサネットデバイスと
そのIPアドレスを報告します。この時点でネットワークパケットが両方向に
正常に送信されていることがわかります。

```bash
# ifconfig
ifconfig: /proc/net/dev: No such file or directory
eth0      Link encap:Ethernet  HWaddr 02:BA:FE:7B:59:38
        inet addr:10.0.0.178  Bcast:10.0.0.255  Mask:255.255.255.0
        UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
        Interrupt:39

lo        Link encap:Local Loopback
        inet addr:127.0.0.1  Mask:255.0.0.0
        UP LOOPBACK RUNNING  MTU:65536  Metric:1
```

## カーネル構成の削減

カスタムビルドしたカーネルは原理的に動作しましたが、状況は満足のいくもの
ではありません。まず、単にネットワークドライバをいじくりたいだけである
ことを考えると32 MiBのカーネルイメージは大きすぎます。次に、デフォルトの
構成では私たちにとって興味深いと思われる機能が意図的に除外されていることが
わかりました。

この状況を克服するにはLinuxカーネルの構成という世界に膝まで深く飛び込む
必要があります。ただし、まず動作することがわかっている`.config`ファイルを
バックアップしましょう。

```bash
cp $LX_BUILD_DIR/.config $LX_BUILD_DIR/config_works
```

`.config`ファイルで2482のオプションを有効にする`defconfig`ビルドターゲット
以外に、カーネルのビルドシステムは323のオプションだけが有効になる`tinyconfig`ターゲットも備えています。この構成を使用するとイメージの
ビルド時間は20分からわずか1.5分に短縮され、イメージのサイズはわずか1.6 MiB
になります。それは私の好みにとても合っています。

この`tiny`カーネルの唯一の欠点は私たちのボード上で起動してもライフサインが
表示されないことです。ただし、これは想定内のことです。この構成では特定の
SoCがそのまま使用できるわけではないからです。動くようにするには適切な
オプションを有効にするだけです、ですよね。最初の目標はシリアル回線経由で
ブートログを取得するために必要なオプションを見つけることです。それが完了
したら、initrdのサポートに注意を向けることができます。それも機能したら、
ネットワークサポートを復活させることができます。

この時点でシリアル出力が`defconfig`カーネルで動作することは知っています。
defconfigで有効になっている2482のオプションの中に正しいオプションが含まれて
いるはずです。最も明らかなことはこのSoC (ARCH_SUNXI) のプラットフォーム
サポートを有効にする必要があることです。少しの間、直感に任せてみましょう。
defconfigにあるカーネルオプションのうち、カーネルコンソールのシリアル出力に
影響を与える可能性があるものはどれでしょうか。*config_works*ファイルを
"SERIAL"などの単語で検索します。

```bash
$ grep =y $LX_BUILD_DIR/config_works | grep SERIAL
...
```

たとえば、`SERIAL_EARLYCON` は確かに便利そうです。

適切なオプションを見つけるための第2の戦略は、ネットワークデバイスに
使用したデバイスツリー駆動のアプローチです。DTSファイルで"serial"を検索
すると"snps,dw-apb-uart"を参照しているノードが見つかります。これは
*8250_dw.c* と *8250_early.c*に関連しており、`CONFIG_SERIAL_8250_DW` と
`CONFIG_SERIAL_8250_CONSOLE` に目を向けさせます。さらに進むと、次の
オプションがデバイスツリーから直接推測できました: `SERIAL_8250`,
`SERIAL_8250_16550A_VARIANTS`, `SERIAL_8250_DW`, `SERIAL_8250_CONSOLE`
です。

![lx_kernel_config.png](figs/lx_kernel_config.png)

上から見ても、下から見ても、どのように見ても、まだ、構成オプションには
埋めるべきギャップがあります。欠落しているオプションを見つけるには、
次のコマンドで示すように構成を二分する総当り攻撃を適用できます。

1. 新しいtinyconfigを作成する
   ```bash
   $ make -C $LX_BUILD_DIR tinyconfig
   ```
2. 確実に必要であることがすでにわかっているオプションを有効にする
   ```bash
    $ $LX_DIR/scripts/config --file $LX_BUILD_DIR/.config \
       --enable SERIAL_8250 ...
    ```
3. バックアップした defconfig で見つかった候補の半分を有効にする
   ```bash
   $ grep =y $LX_BUILD_DIR/config_works |\
      head -n 1200 |\
      sed "s/=y//" |\
      xargs -ixxx $LX_DIR/scripts/config \
                      --file $LX_BUILD_DIR/.config --enable xxx
   ```
   このコマンドは*config_works*ファイルで有効になっていることが判明している
   最初の1200個の構成オプション (有効なオプションの上位半分) を有効にする
   ことにより *.config* ファイルを調整する
4. *.config*ファイルをサニタイズする
   ```bash
   $ make -C $LX_BUILD_DIR olddefconfig
   ```
5. カーネルを構築して作成されたイメージをTFTP ディレクトリにコピーする
   ```bash
   $ make -C $LX_BUILD_DIR dtbs Image -j8 \
     && cp $LX_BUILD_DIR/arch/arm64/boot/Image /var/lib/tftpboot/
   ```
6. ボードを起動する。カーネルがシリアル上でライフサインを示した場合、
   必要なオプションがすべて最初の1200個のオプションでカバーされている
   ことがわかる。そうでなければ、欠落しているオプションは1200を超えた
   ところにあることがわかる。したがって、検索範囲を半分にして最初の
   ステップを繰り返す

たとえば、Pine-A64-LTSボードの場合はシリアル出力には最初の1200の
オプションで十分であることがわかりました。そこで、次の反復では最初の
600のオプションを選択しました。

検索範囲は反復されるたびに半分になるため欠落しているカーネルオプションの
謎を解決するには約10回の反復と少しの忍耐が必要になります。このプロセスを
使用することで`CONFIG_PRINTK`, `CONFIG_BINFMT_ELF`, `BLK_DEV_INITRD`が
必要であることが明らかになりました。これらは後から考えると明らかですが、
grepや直感、デバイスツリーの分析といった方法では見つけられる可能性が
非常に低いと思われます。

このプロセスに従うことで、最終的にネットワークとシリアル出力が有効に
なるカーネルを見つけ出すことができました。二分検索は骨の折れる作業では
ではなく、機械的作業です。それが予測可能な成功につながることがわかった
ことは素晴らしいことです。約500のオプションを有効にし、イメージサイズが
（非圧縮で）4.2 MiBになったカーネルは今後の移植および計装作業の実行可能な
基盤となります。

## 調査結果を再現可能にする

Genode関連の他の作業トピックと同様に、上記の発見の本質を他の人や私に
とって簡単に再現できるようにしています。こうすることで、次の開発者は
私が放置したトピックを取り上げることができます。

Genodeのポートツールを使用してカーネルをダウンロードするために
*allwinner/ports/a64_linux.port*においた初期ポートファイルから開始
できます。接頭辞`a64`はAllwinner SoCの名前を指しており、ダウンロード
したバージョンのLinuxカーネルがこの特定のSoCの使用に適しているという
意図を表しています。

```bash
LICENSE   := GPLv2
VERSION   := 5.12.1
DOWNLOADS := a64_linux.archive

URL(a64_linux) := https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-$(VERSION).tar.xz
SHA(a64_linux) := 123
DIR(a64_linux) := src/linux
```

SHAハッシュはこの時点では不明なので任意の数値123を設定しています。
付属の`allwinner/ports/a64_linux.hash`ハッシュファイルは架空の番号を
使用して作成できます。

```bash
456
```

ポート記述ファイルとハッシュファイルを用意したら`a64_linux`ポートを
試してみることができます。

```bash
genode$ ./tool/ports/prepare_port a64_linux
a64_linux download https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.12.1.tar.xz
Error: Hash sum check for a64_linux failed
```
ハッシュサムチェックは予想通り失敗しましたが、アーカイブのダウンロードには
成功しています。これは`genode/contrib/`ディレクトリ配下のポートファイルに
ちなんで名付けられたサブディレクトリにあります。

```bash
genode$ ls -lh contrib/a64_linux-456.incomplete/linux-5.12.1.tar.xz
... 113M ... contrib/a64_linux-456.incomplete/linux-5.12.1.tar.xz
```

正しいSHAハッシュ値は`sha256sum`を1回実行するだけです。

```bash
genode$ sha256sum contrib/a64_linux-456.incomplete/linux-5.12.1.tar.xz
c0fc1cf...fe5f37 contrib/a64_linux-456.incomplete/linux-5.12.1.tar.xz
```

これによりポート記述ファイルのダミー値 123 を正しい値に置き換えて
`prepare_port`を再度実行します。

```bash
genode$ ./tool/ports/prepare_port a64_linux
a64_linux extract linux-5.12.1.tar.xz (a64_linux)
a64_linux generate a64_linux.hash
Error: allwinner/ports/a64_linux.port is out of date, expected 155f...
```
今回は抽出ステップが成功しました。ただし、ポートツールはポートハッシュ
(ポート記述ファイルのハッシュ) について正しく主張しています。ハッシュは
ポートがGenodeソースツリーと一致していることを保証します。これは
`ports/update_hash`ツールを使用して簡単に更新できます。

```bash
genode$ ./tool/ports/update_hash a64_linux
generate a64_linux.hash
```
ハッシュが更新されると、`prepare_port`の次の試行は成功します。

```bash
genode$ ./tool/ports/prepare_port a64_linux
a64_linux  extract linux-5.12.1.tar.xz (a64_linux)
a64_linux  generate a64_linux.hash
genode$ ls contrib/a64_linux-155f8b01cd911f42f23178571d70a2220612b634/
a64_linux.hash  linux-5.12.1.tar.xz  src
```

*src/linux/サブディレクトリにカーネルのソースツリーがあります。

```bash
genode$ ls contrib/a64_linux-155f8b01cd911f42f23178571d70a2220612b634/src/linux/
arch     CREDITS        fs       Kbuild   LICENSES     net      security  virt
block    crypto         include  Kconfig  MAINTAINERS  README   sound
certs    Documentation  init     kernel   Makefile     samples  tools
COPYING  drivers        ipc      lib      mm           scripts  usr
```

最後に、移植作業に合わせたLinuxカーネルの構成と構築に関する情報を
保存するために[*allwinner/src/a64_linux/target.mk*](https://github.com/genodelabs/genode-allwinner/blob/master/src/a64_linux/target.mk) と
[*target.inc*](https://github.com/genodelabs/genode-allwinner/blob/master/src/a64_linux/target.inc)に
Genodeビルドターゲットを追加しました。これはカーネル構成を適用し、
カーネルイメージを構築します。この*target.mk*ファイルのおかげで次の方法で
Genodeのビルドディレクトリ内からカスタムLinuxカーネルをビルドできます。

```bash
build/arm_v8a$ make a64_linux
```
