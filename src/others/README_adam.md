# README

[オリジナル](https://github.com/AdamRLukaitis/pine64/blob/master/README)

（初めて）PINE64/PINE A64を手に入れたところです。

- [https://www.pine64.com](https://www.pine64.com)
- [http://wiki.pine64.org](http://wiki.pine64.org)
- [http://linux-sunxi.org/A64](http://linux-sunxi.org/A64)
- [http://linux-sunxi.org/Pine64](http://linux-sunxi.org/Pine64)

このボードはAllwinner A64を搭載しています。AllwinnerはBroadcomと同じく
通常はドキュメントを保護しています。しかし、最近のC.H.I.Pや今回の製品
（やその他の製品）についてはドキュメントを見つけるのは難しくありません。
Allwinnerもドキュメントを削除するつもりはないようです。

私は両ブランドには抵抗があり、当初はドキュメントの不足を理由にAllwinner
ベースの多くのボードを見送ってきました。しかし、C.H.I.P.はもう少し試して
みたいと思わせるものでした。C.H.I.P.はbrickableですが少し工夫が必要です。
このボードにはFELジャンパがあり、基本的にUSBを使ってチップにアクセスし、
再ロードすることができます。

このボードは今のところraspberry piと同じようにSDカードベースのようです。
ただし、raspiとは異なり、何か他のプロセッサではなくARMがシステムを起動します。

私の他の作品を見てもらえればわかると思いますが、当然ながら、私はLinuxを
走らせることには興味はなく、ベアメタルに興味があります。raspberry piと同じ
ように、動作する最初のソフトウェアを書いて、すべてを利用できるようにすることが
好きです。ドキュメントがあるかは知りませんが、十分なスタッフとソフトウェア
エンジニア、ハードウェア（チップや基板）が揃ってもDDRを立ち上げるのに10ヶ月は
かかることを知っています。理論的にはすでにコードがあり、実際にはBSPが利用
できるようです。だから、オペレーティングシステムを移植しようとした人と
同じようにブートローダから実行するつもりです。

ダウンロードに何時間もかかるような本格的なオペレーティングシステムの
SDカードイメージには興味はありませんでした。Wikiで見つけたのがこれです。

[https://www.stdin.xyz/downloads/people/longsleep/pine64-images/](https://www.stdin.xyz/downloads/people/longsleep/pine64-images/)

`simpleimage-pin64-latest.img.xz`を入手しました。

そして、`README.txtに`書いてあるように次のコマンドを実行しました。

```bash
xzcat simpleimage-pine64-20160207-1.xz | pv |sudo dd of=/dev/sdX bs=1M oflag=sync
```

ここで、`/dev/sdX`はSDカードです。いいですか、これを間違えるとハードドライブを
ふっとばすことになります。

[http://linux-sunxi.org/Pine64](http://linux-sunxi.org/Pine64)は次のように言っています。

> UART0はAllwinnerのファームウェアがブートとデバッグメッセージに
> 使用するメインUARTであり、Eulerコネクタの29ピン (TXD), 30ピン (RXD),
> 25/34ピン (GND)でアクセスできます（これは公式のコネクタの説明には
> 記載されていません）。

これは今のところ事実のようです。ある種のftdiブレークアウト (3.3V) か
ftdiケーブル（raspberry piやC.H.I.Pに使うものと同じ）を使えば、SDカードを
挿入した状態でブートするのを見ることができます。

このページ全体が興味深いものです。uartセクションではブート・シーケンス
においてsunxi-felを動かすために特別なA to A usbケーブルが必要とあります
（実際はそれほど特別なものではありませんが、誰でも持っているものでは
ありません）。

そうでない場合は、SDカードから32ビットモードで起動しようとします。

そこで、uart0に接続し、SDカードを用意して差し込みました。ちなみに、私の
ボードは袋の中でボタンが転がっている状態で出荷されていたので、最初は虫かと
思いました。その後、これは壊れた部品ではなく、基本的に電源とリセットの両方に
フィットするボタンを１つ提供しているものだという注意書きを見つけました。
どちらにするかを自分で選択してはんだ付けしてくださいとありました。すべての
人がそのための道具を持っているわけではないことは理解しています。ほとんどの
部分は簡単にできることですが、誰もが持っているものではりません。私は当然、
リセットの方にはんだ付けをしました。同じように、ユーザLED用のパッドも2つ
ありましたがどちらもつけませんでした。

任意のキーを押してオートブートを止め、sunxiプロンプトを表示させます。

多くのubootと違ってこれにはxmodemによるダウンロード機能がありません。
でも大丈夫です。回避できる方法はあります。それを行う前に

[http://linux-sunxi.org/A64](http://linux-sunxi.org/A64)

にあるデータシートを見ます。驚くことではないですが、UART0は他のallwinner
チップとほぼ同じ場所にあります。

```c
#define UART_BASE 0x01C28000
``````

よって、先のプロンプトから送信レジスタ（オフセット0）にバイトを書き込むと
それが表示されるはずです。

```bash
Hit any key to stop autoboot:  0
sunxi# mw.b 0x01c28000 0x55
Usunxi# mw.b 0x01c28000 0x56
Vsunxi#                         # U (0x55), V (0x56)が憑依されていることに注目
```

まだすることがあるようです。

`printenv`コマンドは次を示してくれます。

```bash
kernel_filename=pine64/Image
load_addr=41000000
```

では、まず`0x41000000`をRAMの基底アドレスとして少し試してみましょう。

LEDを取り付けてないのでLチカは最初の例になりません。uart01が最初の例です。

これはバイナリのダンプを端末からubootにカット&ペーストできるような
形で追加出力しています。

```bash
mw.l 0x41000000 0xEAFFFFFF
mw.l 0x41000004 0xE3A0D442
mw.l 0x41000008 0xE28DDA01
mw.l 0x4100000C 0xEB00003B
mw.l 0x41000010 0xEAFFFFFE
mw.l 0x41000014 0xE5801000
mw.l 0x41000018 0xE12FFF1E
mw.l 0x4100001C 0xE5900000
...
```

これらをすべてカット&ペーストして、goコマンドで0x41000000から実行すると
uart01は入力した内容をエコーバックします。

```bash
sunxi# mw.l 0x41000134 0xEAFFFFF6
sunxi# mw.l 0x41000138 0x12345678
sunxi# go 0x41000000
## Starting application at 0x41000000 ...
12345678
asdf
```

大きな進歩です。（uart経由で）プログラムをダウンロードできる
ブートローダが本当に欲しくなります。これまではxmodemやintel hexと
プロプリエタリな形式を使っていましたが、今はモトローラのsrecor形式に
切り替えています。

[https://en.wikipedia.org/wiki/SREC_(file_format)](https://en.wikipedia.org/wiki/SREC_(file_format))

私はフル32ビットアドレスのS3フォーマットが好きです。

bootloader01がそのブートローダです。

このローダはプログラムをダウンロードするためのスペースを少しだけ (128KB)
残します。

```assembly
.globl _start
_start:
    b   reset

.space 0x20000-4,0

reset:
    mov sp,#0x42000000
    bl notmain
hang: b hang
```

もちろん、必要であればスペースをもっと大きくすることができます。

mw.lのアプローチを使い、何らかのツールを使ってストレージである
sdカードに書き込むことができます。あるいは、これより良い方法は、
sdカードを抜いて、bootloader01ディレクトリのnotmain.binを
`/mount/whatever/BOOT/bootloader.img`としてコピーすることです。

どこにマウントしているかにおりますが、私の場合は3つのマウントが
表示され、`BOOT`が必要なマウントでした。

そのカードを挿入して、電源を入れるかリセットして、何かキーを押して
オートブートを止めます。

この部分はファイルシステム上のファイルに対しても行うことができますが
まだいじったことがありません。

```bash
sunxi# setenv bootcmd 'fatload mmc 0:1 41000000 bootloader.img; go 0x41000000' sunxi# saveenv
```

これでリセット/再起動すると、オートブートを止めるキーを押さなければ
このブートローダが起動します。

```bash
reading bootloader.img
131964 bytes read in 17 ms (7.4 MiB/s)
## Starting application at 0x41000000 ...
12345678
```

## SREC

これで、たとえば、uart01の例でいえば、notmain.srecファイルを使って
asciiまたはrawモード、あるいはあなたのダムターミナルがそう呼ぶもの
（またはカット＆ペースト）でダウンロードできるようになっています。
minicomでctrl-aを押し、次にsを押し、down（かup）でasciiを選択して
ファイルのパスを指定します。ダウンロードが終わったら（goを表す）gを
押すとダウンロードしたプログラムが起動します。 uart01の場合は
何かを入力するとエコーバックされます。

```bash
SREC
41000000
12345678
asdfasdf
```

これでプログラムをロードして実行する簡単な方法ができました。
これで、緩いボタンをリセット用にハンダ付けしたのが良いアイデアだった
理由もわかるでしょう。
