# bootm コマンド

## 書式

```bash
bootm [fit_addr]#<conf>[#extra-conf]
bootm [[fit_addr]:<os_subimg>] [[<fit_addr2>]:<rd_subimg2>] [[<fit_addr3>]:<fdt_subimg>]

bootm <addr1> [[<addr2> [<addr3>]]    # Legacy boot
```

## 説明

bootmコマンドはオペレーティングシステムの起動に使用します。
起動に何が必要かにより多くのオプションも持ちます。

2番目の書式ではハイフン `-` を使って第1引数と第2引数を省略できる
ことに注意してください。

### `fit_addr / fit_addr2 / fit_addr3`

> 起動するFITのアドレス。デフォルトはCONFIG_SYS_LOAD_ADDR。
> 以下の注記を参照してください。

### `conf`

> 起動に必要な構成ユニット（前にハッシュ '#' をつける必要が
> あります）

### `extra-conf`

> 起動に必要な追加の構成。これは、最初の構成ユニットによって
> 提供されたベースデバイスツリーに追加のデバイスツリーオーバーレイを
> 適用する場合にのみサポートされます。

### `os_subimg`

> 起動に必要なOSサブイメージ（前にコロン ':' をつける必要が
> あります）

### `rd_subimg`

> 起動に必要なramdiskのサブイメージ。ramdiskはないがFDTが必要な
> 場合はハイフン '-' を使用する。

### `fdt_subimg`

> 起動に必要なFDTサブイメージ。

レガシーブートについては以下を参照してください。[FIT](https://u-boot.readthedocs.io/en/latest/usage/fit/index.html)
(Flat Image Tree) を使った起動を推奨します。

## カレントイメージアドレスに関する注記

bootmを引数なしで呼び出すとカレントイメージアドレスのイメージが
起動されます。カレントイメージアドレスとは、loadコマンドなどで直近に
設定されたアドレスのことであり、デフォルトはCONFIG_SYS_LOAD_ADDR
です。たとえば、以下のコマンドを考えます。

```bash
ftp 200000 /tftpboot/kernel
bootm
# このコマンドは次のコマンドに相当します
# bootm 200000
```

上で示したように、FITでは引数のアドレス部分を省略することができます。
`<addr3>`が省略された場合は`<addr2>`のイメージが使用されるものとみなされ
ます。同様に、`<addr2>`が省略された場合は`<addr1>`のイメージが使用される
ものとみなされます。`<addr1>`が省略された場合はカレントイメージ
アドレスが使用されるものとみなされます。たとえば、以下のコマンドを
考えます。

```bash
tftp 200000 /tftpboot/uImage
bootm :kernel-1
# このコマンドは次のコマンドに相当します
# bootm 200000:kernel-1

tftp 200000 /tftpboot/uImage
bootm 400000:kernel-1 :ramdisk-1
# このコマンドは次のコマンドに相当します
# bootm 400000:kernel-1 400000:ramdisk-1

tftp 200000 /tftpboot/uImage
bootm :kernel-1 400000:ramdisk-1 :fdt-1
# このコマンドは次のコマンドに相当します
# bootm 200000:kernel-1 400000:ramdisk-1 400000:fdt-1
```

## レガシーブート

U-BootはCONFIG_LEGACY_IMAGE_FORMATにより有効になるレガシー
イメージフォーマットをサポートしています。これはかなり限定的で
安全ではないので推奨されません。代わりにFIT (Flat Image Tree)を
使用してください。ここでは、依然としてレガシーフォーマット使用
している古いボードのために文書化しています。

引数は以下の通りです。

### `addr1`

> 起動するレガシーイメージのアドレス。イメージが2つ目のコンポーネント
> （ramdisk）を含んでいる場合は2つ目のパラメータがハイフン '-' で
> ない限り、それも使用されます。

### `addr2`

> ramdiskとして使用するレガシーイメージのアドレス。

### `addr3`

> FDTとして使用するレガシーイメージのアドレス

## 構文例

このセクションでは指定可能なさまざまな使用例を紹介します。

```bash
1. boot   /* カレントアドレスのイメージを起動します。2,3,8に相当 */
```

現在のイメージアドレスのイメージのタイプに応じてケース 2、3、8に
相当します。

起動方法：ケース2,3,8を参照

### レガシーuImage構文

```bash
2. bootm <addr1>        /* <addr1>にある単一イメージ */
```

&lt;addr1&gt;にあるカーネルイメージを起動します。

起動方法: 非FDT

```bash
3. bootm <addr1>        /* <addr1>にある複数イメージ */
```

&lt;addr1&gt;にあるイメージの1番目と2番目のコンポーネントは各々
カーネルとramdiskであると仮定されます。カーネルはイメージに含まれる
initrdをramdiskにロードして起動されます。

起動方法: &lt;addr1&gt;にあるコンポーネントの数とU-BootがOFサポート付きで
コンパイルされているかどうかに依存し、次の通り。

| 構成 | 2コンポーネント(kernel, initrd) | 3コンポーネント(kernel, initrd, fdt) |
|:-----|:--------|:-------------|
| #ifdef CONFIG_OF_* | 非FDT | FDT |
| #ifmdef CONFIG_OF_* | 非FDT | 非FDT |

```bash
4. bootm <addr1> -      /* <addr1>にある複数イメージ */
```

ケース3と同じですがカーネルはinitrdなしで起動されます。マルチイメージの
2番目のコンポーネントは無視されます（ダミーの1バイトファイルで構いません）。

起動方法: ケース3を参照

```bash
5. bootm <addr1> <addr2>  /* <addr1>にある単一イメージ */
```

&lt;addr1&gt;にあるカーネルイメージが&lt;addr2&gt;にあるイメージに含まれる
initrdをramdiskにロードして起動されます。

起動方法: 非FDT

```bash
6. bootm <addr1> <addr2> <addr3>    /* <addr1>にある単一イメージ */
```

&lt;addr1&gt;はカーネルイメージの、&lt;addr2&gt;はramdiskイメージの、
&lt;addr3&gt;はFDTバイナリブロブのアドレス。カーネルイメージは&lt;addr2&gt;に
あるイメージに含まれるinitrdをramdiskにロードして起動されます。

起動方法: FDT

```bash
7. bootm <addr1> - <addr3>    /* <addr1>にある単一イメージ */
```

&lt;addr1&gt;はカーネルイメージの、&lt;addr3&gt;はFDTバイナリブロブの
アドレス。カーネルはinitrdなしで起動されます。

起動方法: FDT

### FIT構文

```bash
8. bootm <addr1>
```

&lt;addr1&gt;にあるイメージにはデフォルト構成が含まれていると仮定し、
それが起動されます。

起動方法: デフォルト構成がFDTを定義しているか否かによりFDTか非FDT

```bash
9. bootm [<addr1>]:<subimg1>
```

ケース2と同様です。アドレス&lt;addr1&gt;にあるイメージの&lt;subimg1&gt;>に
格納されているカーネルを起動します。

起動方法: 非FDT

```bash
10. bootm [<addr1>]#<conf>[#extra-conf[#...]]
```

アドレス&lt;addr1&gt;にあるイメージの構成&lt;conf&gt;を起動します。

起動方法: 指定された構成がFDTを定義しているか否かによりFDTか非FDT

```bash
12. bootm [<addr1>]:<subimg1> [<addr2>]:<subimg2>
```

ケース5と同じです。アドレス&lt;addr1&gt;にあるイメージの&lt;subimg1&gt;>に
格納されているカーネルをアドレス&lt;addr2&gt;にあるイメージの
&lt;subimg2&gt;>に格納されているinitrdをramdiskにロードして起動します。

起動方法: 非FDT

```bash
12. bootm [<addr1>]:<subimg1> [<addr2>]:<subimg2> [<addr3>]:<subimg3>
```

ケース6と同じです。アドレス&lt;addr1&gt;にあるイメージの&lt;subimg1&gt;>に
格納されているカーネルを、アドレス&lt;addr2&gt;にあるイメージの
&lt;subimg2&gt;>に格納されているinitrdをramdiskにロードし,
アドレス&lt;addr3&gt;にあるイメージの&lt;subimg3&gt;>に格納されている
FDTブロブを渡して起動します。

起動方法: FDT

```bash
13. bootm [<addr1>]:<subimg1> [<addr2>]:<subimg2> <addr3>
```

ケース12と同じですが、&lt;addr3&gt;はカーネルに渡されるFDTブロブのアドレス
です。

起動方法: FDT

```bash
14. bootm [<addr1>]:<subimg1> - [<addr3>]:<subimg3>
```

ケース7と同じです。アドレス&lt;addr1&gt;にあるイメージの&lt;subimg1&gt;>に
格納されているカーネルを、initrdなしで、アドレス&lt;addr3&gt;にあるイメージの
&lt;subimg3&gt;>に格納されているFDTブロブを渡して起動します。

起動方法: FDT

```bash
15. bootm [<addr1>]:<subimg1> - <addr3>
```

ケース14と同じですが、&lt;addr3&gt;はカーネルに渡されるFDTブロブのアドレス
です。

起動方法: FDT

## 指定例

200000にある新uImageに格納されているカーネル"kernel-1"を起動する。

```bash
bootm 200000:kernel-1
```

200000にある新uImageに格納されている構成"cfg-1"を起動する。

```bash
bootm 200000#cfg-1
```

200000にある新uImageに格納されている構成"cfg-1"を補助"cfg-2"とともに起動する。

```bash
bootm 200000#cfg-1#cfg-2
```

200000にある新uImageに格納されているカーネル"kernel-1"をアドレス800000に
ある別のuImageに格納されているinitrd "ramdisk-2"付きで起動する。

```bash
bootm 200000:kernel-1 800000:ramdisk-2
```

200000にある新uImageに格納されているカーネル"kernel-2"をアドレス800000に
ある別の新uImageに格納されているinitrd "ramdisk-1"とFDT "fdt-1" 付きで
起動する。

```bash
bootm 200000:kernel-2 800000:ramdisk-1 800000:fdt-1
```

200000にある新uImageに格納されているカーネル"kernel-2"を、同じuImageに
格納されているとinitrd "ramdisk-2" と、アドレス800000に格納されている
生のFDTブロブ "fdt-1" 付きで起動する。

```bash
bootm 200000:kernel-2 200000:ramdisk-1 800000
```

200000にある新uImageに格納されているカーネル"kernel-2"を、同じuImageに
格納されているFDT "fdt-1" 付きで起動する。

```bash
bootm 200000:kernel-2 - 200000:fdt-1
```
