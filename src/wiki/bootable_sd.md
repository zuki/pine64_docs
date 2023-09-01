# ブート可能なSDカード

[オリジナル](https://linux-sunxi.org/Bootable_SD_card#Cleaning)

## はじめに

このページではブート可能なSDカードを作成する方法を説明します。SDカードの
接続方法によりデータを書き込む場所が異なる場合があります。このドキュメント
では`${card}`はSDカード、`${p}`はパーティション（もしあれば）を指すことに
します。SDカードがUSBアダプタ経由で接続されている場合、linuxは、たとえば
`/dev/sdb`（`/dev/sda`はブートドライブ）として認識します。このデバイスは
多くの要因によって異なる可能性があるので、よくわからない場合は、デバイスを
接続した後に`dmesg`の最後の数行を確認してください（`dmesg | tail`）。
デバイスのSDスロット経由で接続されている場合、linuxは`/dev/mmcblk0`
（またはどのmmcスロットが使用されているかにより、`mmcblk1`や`mmcblk2`）と
して認識します。

データはSDカードに生のまま、または、パーティションに保存されます。`${p}`を
使用する場合は、適切なパーティションを使用する必要があります。また、これも
USBアダプタとmmcコントローラで異なります。USBアダプタを使用する場合、
`${p}`は`1`、`2`、`3`などになるのでデバイス名は`/dev/sdb1`などになります。
mmcコントローラを使う場合、`${p}`は`p1`、`p2`、`p3`などになるのでデバイス
名は`/dev/mmcblk0p1`などになります。

まとめると、`${card}`と`${card}${p}1`はUSB接続のSDカードでは`/dev/sdb`と
`/dev/sdb1`を意味し、mmcコントローラ接続のデバイスでは`/dev/mmcblk0`と
`/dev/mmcblk0p1`を意味します。

**注意**: SDカードが他の方法で接続されている場合、デバイスノードはそれに
応じて以上の説明とは異なる名前に変わる可能性があります。

## SDカードのレイアウト

AllwinnerベースのボードのデフォルトのU-Bootビルドでは(micro-)SDカード、
または、eMMCストレージ上で以下のレイアウトを使用します(v2018.05以降)。

| 開始アドレス | 開始セクタ | サイズ | 使用法 |
|-------------:|-----------:|-------:|:-------|
| 0KB | 0 | 8KB | 未使用。MBRまたは（制限付）GPTパーティションラベルで使用可能 |
| 8KB | 16 | 32KB | 初期SPLローダ |
| 40KB | 80 | - | U-Boot本体 |

通常、パーティションは1MB（ほとんどのパーティショニングツールのデフォルト
設定）から始まりますが、これにはハード上の要件はないので、U-Bootは必要に
応じて984KBより大きくすることができます。

8KBのオフセットは[BROM](https://linux-sunxi.org/BROM)によって決定され、
BROMはこの位置に有効な[eGON](https://linux-sunxi.org/EGON)/TOC0ヘッダが
あるかチェックします。U-Boot本体の40KBオフセットはデフォルトのU-Boot設定で
あり、ビルド時に`CONFIG_SYS_MMCSD_RAW_MODE_U_BOOT_SECTOR`構成変数を使用して
変更することができます。

より新しいSoC（H2+、A64、H5、H6でテスト済み）では8KBに有効なeGON/TOC0署名が
見つからなかった場合、SDカードまたはeMMCのセクタ256 (128KB) からSPLをロード
することもできます（BROMのぶーと処理順）。この場合はU-Boot本体のオフセットを
それに合わせて調整する必要があります。[これ](https://groups.google.com/forum/#!topic/linux-sunxi/BW7BKGjSTtM)や[これ](https://groups.google.com/forum/#!topic/linux-sunxi/MaiijyaAFjk)を参照してください。

メインラインのU-BootはSDカード/eMMCの最初のメガバイトのセクタに対して
もっと複雑な固定レイアウトを使用していました。

| 開始アドレス | 開始セクタ | サイズ | 使用法 |
|-------------:|-----------:|-------:|:-------|
| 0KB | 0 | 8KB | 未使用。MBR（パーティションテーブルなど）で使用可能 |
| 8KB | 16 | 32KB | 初期SPLローダ |
| 40KB | 80 | 504KB | U-Boot |
| 544KB | 1088 | 128KB | 環境 |
| 672KB | 1344 | 128KB | Falconモードのブートパラメタ |
| 800KB | 1600 | - | Falconモードのカーネルスタート |
| 1024KB | 2048 | - | パーティション用 |

時間の経過とともにU-Boot本体の機能セットが大きくなるにつれ、これでは
あまりにも制約が大きいことが判明しました。環境領域の前を完全に埋め尽くし、
破壊し始めたからです。将来の問題を避けるため、環境のデフォルト場所を
より柔軟で実際的にサイズ制限のないFATパーティションに移すことにしました。

## カードの識別

まずカードのデバイスを特定して`${card}`としてエクスポートします。

使用可能または正しいパーティション名を見つけるのには次の２つの
コマンドが役に立ちます。

```bash
cat /proc/partitions
```

```bash
blkid -c /dev/null
```

- SDカードがUSB経由で接続され、sdX（Xを正しい文字に置き換えてください）で
  ある場合は次のようにエクスポートします。

```bash
export card=/dev/sdX
export p=""
```

- SDカードがmmc経由で接続され、mmcblk0である場合は次のようにエクスポートします。

```bash
export card=/dev/mmcblk0
export p=0
```

## クリーニング

念のため、SDカードの最初の部分を消去してください（パーティションテーブルも
消去されます）。

```bash
dd if=/dev/zero of=${card} bs=1M count=1
```

パーティションテーブルを保持したい場合は以下を実行してください。

```bash
dd if=/dev/zero of=#{card} bs=1k count=1023 seek=1
```

## ブートローダ

sdカードに*u-boot-sunxi-with-spl.bin*を書き込む必要があります。
このファイルがまだない場合は[メインライン](https://linux-sunxi.org/U-Boot#Compile_U-Boot)または[レガシー](https://linux-sunxi.org/U-Boot/Legacy_U-Boot#Compile_U-Boot)のU-Bootの「コンパイル」セクションを参照
してください。

```bash
dd if=u-boot-sunxi-with-spl.bin of=${card} bs=1024 seek=8
```

U-Bootプロンプトからブートローダを更新するには、次のようにします。

```bash
mw.b 0x48000000 0x00 0x100000                 # バッファを0クリア
tftp 0x48000000 u-boot-sunxi-with-spl.bin     # または、MMCやSCSIなどから読み込無為にloadを使用する
mmc erase 0x10 0x400                          # U-Bootを含むMMC領域を消去。ここではリセットしないこと!
mmc write 0x48000000 0x10 0x400               # 更新したU-Bootを書き込む
```

U-Bootのv2013.07以前を使用する場合はオフセットとその手順は若干異なります。

*注: ブートローダをBuildrootで生成した場合は（2015.02にテスト）次のようになります。*

```bash
dd if=spl/sunxi-spl.bin of=${card} bs=1024 seek=8
dd if=u-boot.bin of=${card} bs=1024 seek=32
```

## パーティショニング

最近のU-Bootではext2/ext3をブートパーティションとして使っても問題ありません。
ルートパーティションにある他のファイルシステムも使えます。

### ブートパーティションを分ける

カードを1MBから始まる16MBをブートパーティションに、残りをルート
パーティションに分割するには次のようにしていました。

```bash
sfdisk -R ${card}
cat <<EOT | sfdisk --in-order -L -uM ${card}
1,16,c
,,L
EOT
```

v2.26以降の`sfdisk`コマンドは`-R`フラグは提供しておらず、代わりに
`blockdev`を使うことができます。また、最近、`sfdisk`は`-u`の使用を非推奨に
しました。サイズはすべてセクタで指定するようになったからです。`-L`フラグも
非推奨であり、無視されるのでスキップします。`--in-order`は廃止され、
デフォルトになりました。

これを念頭に置いてカードを次のようにパーティショニングします。

```bash
blockdev --rereadpt ${card}
cat <<EOT | sfdisk ${card}
1M,16M,c
,,L
EOT
```

これで次のコマンドで実際のファイルシステムを作成できるはずです。

```bash
mkfs.vfat ${card}${p}1
mkfs.ext4 ${card}${p}2
```

#### ブートパーティション

```bash
mount ${card}${p}1 /mnt/
cp linux-sunxi/arch/arm/boot/uImage /mnt/
cp sunxi-boards/sys_config/a10/script.bin /mnt/
umount /mnt/
```

### シングルパーティション

(experimental)

```bash
sfdisk -R ${card}
cat <<EOT | sfdisk -L --in-order -uM ${card}
1,,L
EOT
```
```bash
mkfs.ext4 ${card}${p}1
```
```bash
cardroot=${card}${p}1
```

#### ブートパーティション

```bash
mount ${card}${p}1 /mnt/
mkdir /mnt/boot
cp linux-sunxi/arch/arm/boot/uImage /mnt/boot
cp sunxi-boards/sys_config/a10/script.bin /mnt/boot
umount /mnt/
```

### GPT (expeimental)

パーティションデータ用には8kbのスペースがあります。MBRは最初のセクタしか
使用せず、4つのパーティションが使用できます。4パーティションの制限が気に
なるなら、別のパーティションスキームを試すことができます。GPT標準では
GPTは少なくとも128エントリを持つ必要があるとしていますが、`gdisk`は
GPTパーティションを56エントリにリサイズすることができ、保護すべきMBRヘッダと
GPTヘッダに続く7kbに収まります。LinuxはこのようなGPTを理解しますが、
標準に準拠していないため拒否するツールもあります。YMMV[^1]。

GPTパーティションテーブルはSPLやU-Bootの邪魔にならないように移動する
こともできます。これにはGPT標準で義務付けられている128以上のパーティション
テーブルエントリをフルに使えるという利点があります。パーティションテーブルの
開始番号はGPTヘッダ (LBA1) に保存され、通常は2に設定されます。バージョン
1.0.3以降のgdiskプログラムにはこの値を変更する機能があります（"extra
functionality"メニューのコマンド`j`）。次の表は、パーティションテーブルの
開始をLBA 2048に移したカードレイアウトを示しています。

| 開始 | サイズ | 使用法 |
|-----:| ------:|:-------|
| 0 | 0.5KB | 保護すべきMBR |
| 1 | 05KB | GPTヘッダ |
| 2 | 7KB | 未使用 |
| 8 | 32KB | 初期SPLローダ |
| 40 | 504KB | U-Boot |
| 544 | 128KB | 環境 |
| 672 | 128KB | falconモードのブートパラメタ |
| 800 | - | Falconモードのカーネルスタート |
| 1024 | 16KB | パーティションテーブル |
| 1056 | - | パーティション用 |

## ブートスクリプト

ブートスクリプトの準備については[U-Boot設定ページ](https://linux-sunxi.org/Mainline_U-Boot#Configure_U-Boot)に記載されています。

## Rootfs

これはどのディストリビューションをインストールしたいのかによって異なります。
ルートデバイスはカーネルに引数として渡されるので、どのパーティション
レイアウトを使うかはあまり重要ではありません。rootfsが期待するレイアウトと
異なる場合は、`/etc/fstab`やその他のファイルを調整する必要があるかも
しれません。この記事を書いている時点ではほとんどの利用可能なイメージは
`/boot`を分離した2つのパーティションを使用しています。

### rootfs tarballを使う

```bash
mount ${card}${p}2 /mnt/
tar -C /mnt/ -xjpf my-chosen-rootfs.tar.bz2
umount /mnt
```

#### Linaro rootfs

Linaroは[さまざまなルートファイルシステム](https://wiki.linaro.org/Platform/DevPlatform/Rootfs)を
提供しています。スナップショットサーバー上のLinaro rootfsには30日の
保持ポリシーが適用されています。リクエストに応じて新しいスナップショットを
生成することができます。最新のスナップショットは[Ubuntu Build Service](https://git.linaro.org/gitweb?p=ci/ubuntu-build-service.git)などの
ソースから作成することができます。

いずれにせよ[実際のrootfsのtarball](http://snapshots.linaro.org/ubuntu/images/)を
ここから入手できます。ALIPは最小限のLXDEベースのデスクトップ環境であり、
ほとんどのallwinnerユーザにとって有用であると思われます。

ALIP/Linaro/Ubuntuの最近（2015年、もしかしたらそれ以前）のバージョンや
*systemd*（とおそらく*upstart*）を使う他のrootfsは`CONFIG_FHANDLE=y`[^2]で
コンパイルされたカーネルでしか使えないことに注意してください。Sunxi-3.4
カーネルのデフォルト設定ではこのオプションは設定されていません（`.config`に
"# CONFIG_FHANDLE is not set"と書かれています）。そのためカーネル構成時に
（*"General Setup"の"Open by fhandle syscalls"で*）このオプションを設定する
必要があります。

そうしないと、カーネルがブートし、rootfsがマウントされても、その後に何も
起こりません。どのコンソールにもログインプロンプトが表示されません。`CONFIG_FHANDLE`のないカーネルを使わなければならない場合は、*sysvinit* を
使ってDebian rootfsを使ってみてください。

#### LinuxContainersのRootfs

LinuxContainersプロジェクトにはダウンロード可能な様々な[rootfsイメージ](https://images.linuxcontainers.org/images/)が
あります。

### debootstrapを使う - Debian/Ubuntuベースのディストリビューション

DebianのDebootstrapを使うと独自のrootfsを位置から作成することができます。
その方法は[debootstrapの手引](https://linux-sunxi.org/Debootstrap)で
説明されています。

### カーネルモジュール

rootfsをカードにコピーする際には[カーネルモジュールもコピー](https://linux-sunxi.org/Manual_build_howto#Setting_up_the_rootfs)した
方がいいだろう。

## トラブルシューティング

- **パーティショニングをチェックしてください** - 自分でパーティショニングを
  行った場合は`sfdisk`を使ってセクタ単位でレイアウトを読み込んでください。`sfdisk`と`gparted`はメガバイトを使用すると時々奇妙な丸めを適用します。

```bash
sfdisk -uS -d /dev/sdd
# partition table of /dev/sdd
unit: sectors

/dev/sdd1 : start=     2048, size= 16150528, Id=83
/dev/sdd2 : start=        0, size=        0, Id= 0
/dev/sdd3 : start=        0, size=        0, Id= 0
/dev/sdd4 : start=        0, size=        0, Id= 0
```

- **イメージが正しく書き込まれているか再確認してください** - 提供されている
  場合はイメージのチェックサムを確認してください。書き込み手順をもう一度
  よく読んでください。可能であれば別の書き込み方法: `dd` / `phoenixsuit` / `win32-diskimage` などを試してください。特にWindows上での書き込みは
  トラブルを引き起こしがちです。ボードが新しい場合は同じCPUを持つ似たような
  ボード用のイメージを試すことができます。ブートメッセージを確認するために
  [コンソールケーブル](https://linux-sunxi.org/UART)があれば使ってください。

- **起動する前にボードの電源を完全に切ってください** - コンソールケーブルを
  使用している場合、ボードの電源が完全に切れないことがあります。セルフパワーの
  USBペリフェラルやUSBハブも同様の問題を引き起こす可能性があります。
  ボードの電源を  切ると赤の電源ランプは暗くなりますが、完全には切れて
  いません。この場合、  mmcコントローラは正しくリセットされず、ボードは
  nandから起動します。ボードの電源を切り、すべてのペリフェラルを外し、
  シリアルコンソールケーブルを外してください。それから、もう一度起動して
  みてください。起動する前にペリフェリアルを再接続してもよいでしょう。
  この問題はカーネルがmmcコントローラの電源を適切に切った場合は起こらない
  ようですが、カーネルがクラッシュしたときにはよく起こります。

  - **マイクロSDカードの接触不良をチェックしてください** - マイクロSD
  ソケットを使用するボードによくある問題です。カードを抜き差ししたり、
  カードの接触部分をクリーニングしたり、SDカードソケットのホコリを落として
  みたりしてください。カードを紙片と一緒に挿入すると接触が改善され、
  ソケットが緩すぎるカードでも起動できるようになったという報告もあります。

## 参照

- [FEL](fel.md#特別なSDカードイメージ経由)
- [U-Boot](uboot.md)

## 外部情報

- [sunxiのU-Bootに関する追加情報](https://github.com/linux-sunxi/u-boot-sunxi/wiki)

## 参考資料

[^1] <https://en.wiktionary.org/wiki/YMMV>

[^2] <http://unix.stackexchange.com/questions/169935/no-login-prompt-on-serial-console>
