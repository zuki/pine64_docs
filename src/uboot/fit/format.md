# Flattened Image Tree (FIT)フォーマット

[オリジナル](https://u-boot.readthedocs.io/en/latest/usage/fit/source_file_format.html)

## はじめに

カーネルのブート処理で役割を果たす要素の数は時代とともに増え、
現在では通常、デバイセツリー、カーネルイメージ、おそらくは
ramdiskイメージが含まれます。一般に、すべてをシステムメモリに
配置し、一緒に起動する必要があります。

ファームウェアイメージについても同様の処理が行われ、ARMのATF、
OpenSBI、FPGA、U-Boot本体など、さまざまなバイナリが異なるアドレスに
ロードされます。

FITはこの複雑さに対処するための柔軟で拡張可能なフォーマットを提供
します。FITは複数のコンポーネントをサポートしています。また、複数の
構成もサポートしているため、同じFITを複数のボードのブートに使用し、
あるコンポーネントは共通（たとえば、カーネル）にし、あるコンポーネント
はボード固有（たとえば、デバイスツリー）にしたりすることができます。

### 用語

本文書ではFDT（Flat Device Tree）のバインディングを提供することにより
FITを定義します。これらは、FITが使用される時点での最終的な形を記述します。
ユーザの視点はよりシンプルかもしれません。なぜなら、ある種のプロパティ
（タイムスタンプやハッシュ値など）はU-Boot mkimageツールによって自動的に
入力されるからです。

カーネルFDTとの混同を避けるため、以下の命名規約を使用します。

<dl>
<dt><h4>FIT</h4></dt>
<dd>
Flattened Image Tree
</dd>
</dl>

FITは、形式的には（libfdtの意味で）このドキュメントで定義されている
バインディングに準拠している平坦化されたデバイスツリーです。

<dl>
<dt><h4>.its</h4></dt>
<dd>
Image tree source
</dd>

<dt><h4>.itb</h4></dt>
<dd>
flattened image tree blob
</dd>
</dl>

### イメージ構築手順

下図はFITの作成方法を示しています。入力はイメージソースファイル(.its)と
一連のデータファイルで構成されます。イメージは標準的なU-Boot mkimage
ツールで作成され、この中でdtc（デバイスツリーコンパイラ）を使って
イメージツリーブロブ（.itb）が作成されます。出来上がった.itbファイルは
新しいFITの実際のバイナリです。

```
tqm5200.its
+
vmlinux.bin.gz     mkimage + dtc               xfer to target
eldk-4.2-ramdisk  --------------> tqm5200.itb --------------> boot
tqm5200.dtb                          /|\
                                      |
                                 'new FIT'
```

処理手順:

1. .itsファイルを作成します。自動入力されるプロパティは省略します。
2. .itsファイルに対してmkimageツールを呼び出します。
3. mkimageがdtcを呼び出して.itbイメージを作成し、欠けていたプロパティを
   追加します。
4. .itb（新しいFIT）をターゲットにアップロードし、使用します。

### ユニーク識別子

イメージ、ハッシュ値、構成を表すFITサブノード（以下のセクションで定義
されます）を識別するために、指定されたサブノードの「ユニット名」が
識別子として使用されます。これは追加のチェックを必要とせずに一意性が
保証されるからです。

### 外部データ

FITは通常、各イメージノードの'data'プロパティで指定されたイメージデータで
初期構築されます。このデータはFITの外部に置くことも可能です。これにより、
FITの'FDT'部分を非常に小さくすることができ、大量のデータを読み込むこと
なく、ロードとスキャンをすることができます。そして、イメージが必要に
なったときに外部ソースからロードすることができます。

外部FITは'data'の代わりに'data-offset'または'data-position'を使用します。

mkimageツールは *-E* 引数を使って外部データを使うようにFITを変換でき、
オプションで *-p* を使って固定位置を指定することもできます。

各イメージをブロックサイズまたはキャッシュラインサイズ（たとえば、
512バイト）にアライメントしておくと、イメージデータを読み込む際に
アライメントされたアドレスにコピーする必要がなくなるので通常は
望ましいことです。mkimageツールにはこれをサポートする *-B* 引数が
用意されています。

## Rootノードプロパティ

FITのrootノードは次のレイアウトである必要があります。

```
/ o image-tree
    |- description = "image description"
    |- timestamp = <12399321>
    |- #address-cells = <1>
    |
    o images
    | |
    | o image-1 {...}
    | o image-2 {...}
    | ...
    |
    o configurations
      |- default = "conf-1"
      |
      o conf-1 {...}
      o conf-2 {...}
      ...
```

### オプションのプロパティ

<dl>
<dt><h4>description</h4></dt>
<dd>
FITのテキストによる記述
</dd>
</dl>

### 必須のプロパティ

<dl>
<dt><h4>timestamp</h4></dt>
<dd>
イメージの最終更新時刻で`1970-01-01 00:00:00`からの秒単位でカウント
されます。mkimageツールにより自動的に計算されます。
</dd>
</dl>