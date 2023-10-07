# Pineを楽しむ - ベアメタルシリアル出力

[December 17 2020 by Norman Feske](https://genodians.org/nfeske/2020-12-17-pine-fun-serial)

低レベルのカーネルブートストラップ作業には、シリアル接続を介して
デバッグメッセージをプリントする原始的な方法が必要です。このポスト
では、カーネルを持たないベアメタルハードウェア上でカスタムコードを
実行し、UARTデバイスレジスタを直接突くことでシリアル出力を得る手順を
説明します。

[前回の記事](warming_up.md)では、Pine64ハードウェアを知り、Linuxを
使ってシリアル接続を確立し、U-Bootブートローダの使い方を探りました。
これにより、このデバイス上でGenodeのカーネルを実行する方向に向かう
ことができるようになりました。しかし、Genodeに触れる前に2つの注意を
払う必要があります。

1. ブートローダからロードされたカーネルコードへ実行を渡す方法を
   理解する必要があります。

2. カスタムコードが正しく実行されていることを知るためには情報を
   取得する方法が必要です。

この2つの問題に対処するため、あらかじめ定義された物理メモリ
アドレスにコピーして、実行するとシリアルライン上に文字を出力する
カスタムコードブロブを作成します。

### 収集する情報

[最初の探索](https://genodians.org/nfeske/2020-12-10-pine-fun-warmup)において、
明らかにLinuxカーネルでデフォルトそして使用されている、アドレス
`0x1c28000`にある"16550A"タイプのシリアルデバイスを偶然発見しました。
シリアル接続を使ってU-BootやArmbianシステムとやりとりした際に、
このデバイスが動作するのをすでに確認しています。参考までに、対応する
dmesgの出力をもう一度示します。

```bash
[    2.250163] 1c28000.serial: ttyS0 at MMIO 0x1c28000
               (irq = 31, base_baud = 1500000) is a 16550A
```

この特定のデバイスについて詳しく調べる方法はいくつかあります。
たとえば、チップベンダーのドキュメントを参照したくなるかもしれません。
しかし、これは泥臭い方法です。なによりほとんど場合これはできません。
ARMベースのSoCは公開されているドキュメントがほとんどないか、
存在してもドキュメントは不確実であったり、エラーがあったりします。
可能な限り、動作している既知のコードを参照として読みながら真実の道を
進みたいと思います。*u-boot/.config*ファイルにあるU-Bootのビルド
設定を調べてみます。文字列 "Serial"で検索するとすぐに次の行に
たどり着ります。

```bash
CONFIG_SYS_NS16550=y
```

ドライバの名前には"NS16550"のような文字列が含まれているに違い
ありません。そこでソースツリーをgrepして、この文字列にちなんだ
名前を持つファイルを探してみます。

```bash
$ cd u-boot
$ find | grep -i NS16550
./drivers/serial/ns16550.c
./drivers/serial/ns16550.su
./drivers/serial/.ns16550.o.cmd
./drivers/serial/ns16550.o
./drivers/serial/serial_ns16550.c
./include/ns16550.h
./include/config/sys/ns16550.h
./spl/drivers/serial/ns16550.su
./spl/drivers/serial/serial_ns16550.o
./spl/drivers/serial/.ns16550.o.cmd
./spl/drivers/serial/ns16550.o
./spl/drivers/serial/serial_ns16550.su
./spl/drivers/serial/.serial_ns16550.o.cmd
```

これは期待できそうです。現時点では、UARTデバイスアドレス`0x1c28000`
への接続に特に興味があります。U-Bootのビルドシステムにどのようにして
`PLAT=sun50i_a64`を指定したかを覚えていますか。この"sun50i_a64"は
SoCを指しているはずです。そこで、ソースツリーをgrepして、"sun"と
"NS16550"の間の接続を探してみます。

```bash
grep -r NS16550 | grep -i sun
...
include/configs/sunxi-common.h:# define CONFIG_SYS_NS16550_COM1  SUNXI_UART0_BASE
include/configs/sunxi-common.h:# define CONFIG_SYS_NS16550_COM2  SUNXI_UART1_BASE
```

次は、`SUNXI_UART0_BASE`です。

```bash
grep -r SUNXI_UART0_BASE
...
arch/arm/include/asm/arch-sunxi/cpu_sun9i.h:#define SUNXI_UART0_BASE (REGS_APB1_BASE + 0x0000)
arch/arm/include/asm/arch-sunxi/cpu_sun4i.h:#define SUNXI_UART0_BASE 0x01c28000
```

`0x01c28000`というアドレスを見ると、正しいデバイスとそれに対応する
ドライバコードを探していることは正しいことがわかります。

次のステップは複数の情報をクロスチェックすることです。ウェブで
"NS16550 pdf"を検索するとこのデバイスのデータシートが見つかります
（<http://caro.su/msx/ocm_de1/16550.pdf>）。SoCチップベンダーの
ドキュメントとは対照的に、このような個々のIPコアのデータシートは
公開されていれば、その質が高いものが多いのが普通です。だから
ラッキーでした。データシートに目を通すと、オフセット0のレジスタは
いわゆるTransmitter holdingレジスタ（THR）であることがわかります。
文字をプリントするにはこのレジスタに書き込まなければなりません。
すべてのデバイスレジスタが8ビット幅であることが興味深いです。
これらのレジスタがARM SoCのシステムバスアドレスにどのように
マッピングされているのかという疑問が湧いてきます。その答えは
Pine64 wikiからリンクされている[Allwinner A64のマニュアル](https://files.pine64.org/doc/datasheet/pine64/Allwinner_A64_User_Manual_V1.0.pdf)で
知ることができます。このマニュアルからは個々のレジスタが32ビットに
アライメントされたメモリマップドI/Oレジスタにマッピングされている
ことがわかります。したがって、NS16550データシートに書かれている
レジスタオフセットは4倍する必要があることがわかります。THRレジスタは
もちろん、オフセット0にマップされます。この情報をクロスチェック
するには、*drivers/serial/ns16550.c*にあるU-Bootのドライバコードが
便利です。

NS16550のデータシートではTHRレジスタについて次のように書かれています。

> このレジスタを書き込む前にユーザは、たとえば、LSRのTHR Empty
> フラグがセットされているか確認するなど、UARTがデータ送信の準備が
> できていることを確認する必要があります。

LSRはLine Statusレジスタの略です。データシートによると5番目の
レジスタです。したがって、ARMシステムバスのオフセット 5*4 = 0x14 で
アクセスできるはずです。また、言及されている"Empty"ラグはLSRの
ビット5にあることも知りました。

### 20バイトで "U"と叫ぶ

予備テストとして、無限ループの中で無条件に文字`U`（ASCII値`0x55`）を
THRレジスタに書き込んでみましょう。対応するCプログラム（*main.c*
ファイルとして保存）は次のようになります。

```c
int _start()
{
    for (;;)
        *(unsigned logn *)0x1c28000 = 'U';
}
```

最後には、Genodeのツールチェーンを使わなければならなくなるので
今、インストールしておくのは良いタイミングです。ツールチェーンは
AARCH64をサポートしていています。すべてのユーティリティは
*/usr/local/genode/tool/current/bin/* にあります。この長いパスを
入力することを避けるために、シェルのPATH変数にこのディレクトリを
追加するを検討してください。ただし、それは単なる便宜上の問題です。

次のGCCの呼び出しはこの小さなCプログラムをELFバイナリにコンパイル
します。


```bash
genode-aarch64-gcc -nostdlib main.c -o serial_test
```

`-nostdlib`フラグはCのランタイムやデフォルトのスタートアップコードを
リンクしたくないことをコンパイラーに伝えます。`objdump`を使って
バイナリを逆アセンブルして結果を調べてみます。

```bash
$ genode-aarch64-objdump -ld serial_test

serial_test:     file format elf64-littleaarch64

Disassembly of section .text:

0000000000400000 <_start>:
_start():
    400000:  d2900000   mov  x0, #0x8000                  // #32768
    400004:  f2a03840   movk  x0, #0x1c2, lsl #16
    400008:  d2800aa1   mov  x1, #0x55                    // #85
    40000c:  f9000001   str  x1, [x0]
    400010:  17fffffc   b  400000 <_start>
```

命令は私には異質に見えますが（現時点ではAARCH64 ISAにあまり詳しく
ない）、これはきわめて合理的に見えます。生成されたコードがスタック
ポインタに依存していないのは問題ありません。正しいスタックを持つ
ことが保証できないからです。しかし、リンクアドレス`0x400000`が気に
なります。A64 SoCのRAMの基底アドレスは`0x40000000`より低くは
ないからです。Linuxの */proc/iomem*を見た際に次の行があったことを
思い出してください。

```bash
40000000-bdffffff : System RAM
```

そのため、リンカの引数を少し調整する必要があります。U-Bootを使った
実験からU-Bootのデフォルトのロードアドレスである`0x42000000`はこの
範囲内にあることを知りました。リンカ引数 `-Ttext` を使用すると
テキスト（コード）セグメントに対して希望するリンクアドレスを明示的に
指定できます。

```bah
genode-aarch64-gcc -Wl,-Ttext=0x42000000 -nostdlib main.c -o serial_test
```

`-Wl`という接頭辞はGCCフロントエンドに続く引数をリンカに渡すように
指示するために必要なだけです。この微調整により逆アセンブルした
バイナリはより良く見えます。

```bash
Disassembly of section .text:

0000000042000000 <_start>:
_start():
    42000000:  d2900000   mov  x0, #0x8000                  // #32768
    42000004:  f2a03840   movk  x0, #0x1c2, lsl #16
    42000008:  d2800aa1   mov  x1, #0x55                    // #85
    4200000c:  f9000001   str  x1, [x0]
    42000010:  17fffffc   b  42000000 <_start>
```

`serial_test`はあらゆる種類のメタデータを含む完全なELFバイナリです。
この命令をターゲット上で実行するにはELFローダ（もちろんU-Bootが
やってくれます）か、裸足で丘を登るかのいずれかが必要です。後者の方が
より多くのコントロールができます。それでは、`objcopy`を使ってELF
バイナリを生のバイナリに変換します。

```bash
genode-aarch64-objcopy -Obinary serial_test serial_test.img
```

この生のバイナリは *serial_test.img*と名付けました。サイズを
チェックすると純粋に有用なのはわずか20バイトであることがわかります。
すごくスリリングです。オーバーヘッドがありません。

次のステップは、U-BootのTFTPサポートを使ってイメージをフェッチする
ことです。開発マシンで動いているTFTPサーバはディレクトリ
`/var/lib/tftpboot/`を提供しています。そこで、U-Bootのコンソールに
向かう前に *serial_test.img* をこのディレクトリにコピーする必要が
あります。

```bash
=> bootp 10.0.0.32:/var/lib/tftpboot/serial_test.img
BOOTP broadcast 1
BOOTP broadcast 2
BOOTP broadcast 3
DHCP client bound to address 10.0.0.178 (1100 ms)
Using ethernet@1c30000 device
TFTP from server 10.0.0.32; our IP address is 10.0.0.178
Filename '/var/lib/tftpboot/serial_test.img'.
Load address: 0x42000000
Loading: #
    4.9 KiB/s
done
Bytes transferred = 20 (14 hex)
```

プログラム全体が指定された場所に到達したようです。さあ、
ジャンプする時が来ました。

```bash
=> go 0x42000000
## Starting application at 0x42000000 ...
UUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU...
```

シリアルコンソールはUの文字で溢れます。なんとも嬉しい瞬間です。

この実験で得たものをもう一度まとめておきます。

- 独自のCプログラムをターゲット上で動作するバイナリコードに
  コンパイルする方法を知りました。

- バイナリをターゲットにロードし、ブートローダからコードに制御を
  渡すことに成功しました。

- コードからはポジティブなライフサインが返りました。

### 原初の声から言葉への進化

これまでは、UARTデバイスの状態を聞くことなく、ただTHRレジスタを
激しくたたくだけだけでした。プログラムが単なる声ではなく言葉を
発するようにするにはこの無知を止めなければなりません。

プログラムを修正する際にはスタックを使わないように注意しなければ
なりません。このような実験を繰り返しているとコンパイルのたびに逆
アセンブルしたプログラムを表示する、ちょっとしたMakefileが便利に
なってきます。

```makefile
CROSS_DEV_PREFIX := /usr/local/genode/tool/current/bin/genode-aarch64-

serial_test: main.c
  $(CROSS_DEV_PREFIX)gcc -Wl,-Ttext=0x42000000 -nostdlib $< -o $@
  $(CROSS_DEV_PREFIX)objdump -ld $@

serial_test.img: serial_test
  $(CROSS_DEV_PREFIX)objcopy -Obinary $< $@

test: serial_test.img
  cp $< /var/lib/tftpboot/
```

 この小さなワークフローツールは、生活をとても便利にしてくれるだけで
 なく、未来の私のために様々なコマンドの使い方を記録してくれています。私はこのツールを単なる個人的なツールと考えているので、TFTP
 ディレクトリへのイメージのコピーのようなコマンドをそこに置くことも
 ためらいません。さて、`make test`を発行するとコンパイル、アセンブリ
 コードの表示、rawバイナリの作成、TFTPディレクトリへのコピーと
 いったすべてのステップを一度にやってくれます。

実際のプログラムに戻ると、次の小さなステップは1文字の代わりに
文字列を出力することです。

```c
static char const *text = "Aye aye.\n\r";
static char const *s;

for (;;)
    for (s = text; *s; s++)
        *(unsigned int volatile *)0x1c28000 = *s;
```

なぜ変数`text`と`s`にstaticがついているのか不思議に思うかも
しれません。ローカル変数にしたら（通常、それは良い習慣ですが）
コンパイラがスタックフレームを生成するかもしれません。
たとえば、`for`ループを単に次のように率直な形に変えると

```c
for (char const *s = text; *s; s++)
```

対応するアセンブリプログラムは、スタックポインタレジスタを変更
したり、デリファレンスしたりする命令を生成します。

```armasm
  42000000:  d10043ff   sub  sp, sp, #0x10
  42000004:  90000080   adrp  x0, 42010000 <_start+0x10000>
  42000008:  91018000   add  x0, x0, #0x60
  4200000c:  f9400000   ldr  x0, [x0]
  42000010:  f90007e0   str  x0, [sp, #8]
```

スタックは持っていないのでこれは大きな間違いです。`static`
キーワードは変数をバイナリのデータ（またはbss）セグメントに静的に
割り当てるようコンパイラに指示します。バイナリセグメントについて
いえば、少しびっくりですが、バイナリサイズを見てください。

```bash
$ ls -la serial_test.img
-rwxrwxr-x 1 no no 65640 Dez 17 15:30 serial_test.img
```

当惑しませんか。この変更でバイナリサイズは20バイトから64キロバイト
以上に膨れ上がりました。この影響は初期バージョンにはまったく
なかった変数を使用したことに起因します。少なくとも1つ変数を使用
すると、コンパイラ/リンカはテキスト（コード）セグメントに加えて
データセグメントを生成するように指示します。デフォルトでは、リンカは
デフォルトのアライメントを使用して各セグメントをアライメントされた
アドレスに配置します。AARCH64ではこのデフォルトのアライメントは
64 KiBです。そのため、仮想メモリを使用する場合、セグメントは常に
MMUページの先頭から始まります。このデフォルトの動作のため、変数が
次の64 KiB境界で始まる前に、数個の命令の後にほぼ64 KiBのゼロが
続きます。今のところ、私たちはMMUを使用していません。ですので
原理的にはデフォルトのアライメントを弱めることができます。
参考までにいうと、セグメントアラインメントを16バイトに定義する
GCCの引数は`-Wl,-z -Wl,max-page-size=0x10`です。どうです、イメージが
64 KiBから200バイト以下に縮小されました。さて、ここでは数字いじりは
やめてこのバージョンのプログラムを実行しましょう。

```bash
## Starting application at 0x42000000 ...
Aye aye.
Aye aye.
Aye aye.
Aye aye.
Aye aye.
Aye aye.
Ayeaaaaaaaaaaaaaaaaaaaa...
```

文字列を見ることができましたが、ある時点で出力は再び原初の声に
逆戻りしてしまいました。これは予想されていました。新しい文字を
THRレジスタに書き込む前にTXステータスビットをまだチェックして
いないからです。興味深いことに、おそらく、UARTのTX FIFOバッファの
容量が文字列を飲み込める間しばらくは動作したのでしょう。

ところで、このようなインフラがほとんどない素朴なレベルでデバイスを
いじる場合、人工的な遅延は次ののようにして実現できます。

```c
for (i = 0; i < 1000000; i++>)
    asm volatile("nop");
```

これらの行を外側のforループの本体に追加すると確かに安定した出力を
観察することができます。ただし、もちろんこれは単なるハックに
過ぎません。実際にステータスビットを評価するようにコードを変更
しましょう。

```c
int _start()
{
  enum {
    UART_BASE = 0x1c28000,

    THR = UART_BASE,
    LSR = UART_BASE + 0x14,

    LSR_THRE = (1 << 5)
  };

  /* コンパイラがスタックフレームを作成しないようにするために
     staticが必要 */
  static char const *text = "Aye aye.";

  for (;;) {

    static char const *s;

    for (s = text; *s; s++) {

      /* 'TX Holding Register Empty' ビットをポーリング */
      while (((*(unsigned int volatile *)LSR) & LSR_THRE) == 0);

      *(unsigned int volatile *)THR = *s;
    }
  }
}
```

コードに施した口紅の量に注目してください。

- あちこちにコメントを加えました。
- 垂直空白を使ってグループ化しました。
- `enum`値を使ってマジックバリューに具体的な名前をつけました。

このような一時的なテストプログラムには少し過剰かもしれないとは
思います。しかし、これを書いたのは現在の私のためではなく、あなたや
未来の私のためであるということを心に留めてください。また、`text`
から改行を取り除いたことにも注意してください。これは、次の画面を
よりきれいに見せるため以外の理由はありません。

UARTのTXステータスビットをチェックしたおかげで出力が信頼できる
ようになりました。これで来たるべくカーネルのUARTドライバのための
最小限の動作するブループリントができました。ボードから情報を取得
するこの原始的な方法があれば、[次の記事](https://genodians.org/nfeske/2021-01-28-pine-fun-kernel-skeleton)のテーマであるカーネル移植
作業に目を向けることができます ...
