# マニュアルビルドの手引

[オリジナル](https://linux-sunxi.org/Manual_build_howto)

このページではsunxi U-Boot、linuxカーネル、その他を組み合わせて、
さらなるハッキングの基礎となる便利なSDカードをゼロから作成する
プロセスを説明します。

このページはA10、A13/A10s、A20ベースの各デバイスのビルドを扱って
います。[これ以外のSoC用のマニュアルビルド](#参照)の
方法はこのページの最後にあります。

もちろん、ディストリビューション全体をビルドするわけではありません。
U-Bootとカーネル、そして一握りのツールだけをビルドし、既存のrootfsを
使って有用なシステムを構築します。rootfsのサイズにもよりますが
2GB以上のSDカードを使った方がいいでしょう。SDカードのパーティション
分割とフォーマットは後で行います。

## クロスツールチェーンの取得

これについては[ツールチェーンのページ](https://linux-sunxi.org/Toolchain)を
参照する必要があります。

カーネルをビルドするには他にも追加パッケージが必要になるかも
しれません。

```bash
$ apt install flex bison
```

## U-Bootのビルド

### Sunxi/Legacy U-Boot

[U-Bootのコンパイル](https://linux-sunxi.org/U-Boot#Compile_U-Boot)を
参照してください。ビルドが完了するとU-Bootツリーに _spl/sunxi-spl.bin_ と _u-boot.img_ ができます。

SDカードへのインストールは[Bootable_SD_card#Bootloader](https://linux-sunxi.org/Bootable_SD_card#Bootloader)にあります。

### Upstream/Mainline U-Boot

あなたのデバイスがUpstream/MainlineのU-Bootでサポートされている
場合はおそらくそれを代わりに使うべきです。

[これらの手順](https://linux-sunxi.org/Mainline_U-Boot#Compile_U-Boot)に
従って _u-boot-sunxi-with-spl.bin_ を作成してください。

**注意**: 古い3.4.xカーネルをmainline U-Bootでブートする場合は
その[ブート設定](https://linux-sunxi.org/Mainline_U-Boot#Boot)も
理解しておいてください。このページで紹介する情報の一部は調整する
必要があります。特に **bootm_boot_mode** が重要になります。

## カーネルのビルド

### Sunxi/Legacyカーネル

#### script.binのビルド

sunxiのカーネルは[script.bin](https://linux-sunxi.org/Script.bin)と
呼ばれるAllwinners独自のハードウェア記述ファイルに依存しており、
カーネルがアクセスできるようにブートローダによってメモリにロード
される必要があります。

_script.bin_ のビルドは[script.binのビルドガイド](https://linux-sunxi.org/Script.bin#Build_script.bin)に従ってください。

#### 実際のカーネルのビルド

[カーネルコンパイルガイド](https://linux-sunxi.org/Linux_Kernel#Compilation)を参照してください。

### Upstream/Mainlineカーネル

[Upstreamカーネルの手引](https://linux-sunxi.org/Mainline_Kernel_Howto)を参照してください。

デバイスツリーバイナリのビルドも忘れないでください。

## ブート用のSDカードの設定

### SDカードの初期化

[SDカード初期化の手引](https://linux-sunxi.org/Bootable_SD_card#Cleaning)を参照してください。

### SDカードのパーティショニング

[SDカードパーティショニングの手引](https://linux-sunxi.org/Bootable_SD_card#Cleaning)を参照してください。

### SDカードへのブートローダのインストール

[SDカードへのブートローダのインストールの手引](https://linux-sunxi.org/Bootable_SD_card#Bootloader)を参照してください。

### ブートパーティションまたはルートパーティションのマウント

ブートパーティションまたはルートパーティションを再度マウントして
ください。

```bash
$ mount /dev/${card}1 /mnt
```

### ブーロローダconfigurationのインストール

 mainline U-Bootまたはlegacy sunxi U-Bootの設定方法に従ってください。

 - [mainline U-Boot](https://linux-sunxi.org/Mainline_U-Boot#Boot)
 - [legacy U-Boot](https://linux-sunxi.org/U-Boot#Boot)

### カーネルバイナリのインストール

ようやくビルドしたカーネルをブートパーティションにインストールできる
ようになりました。

```bash
# cp /home/user/dir/linux-sunxi/arch/arm/boot/uImage /mnt
```

### ボード記述ファイルのインストール

#### script.bin (sunxiカーネルの場合)

sunxiカーネルを使用する場合は[script.bin](https://linux-sunxi.org/Manual_build_howto#Building_script.bin)を
ブートパーティションにインストールする必要があります。

```bash
$ cp /home/user/dir/sunxi-boards/sys_config/aXX/script.bin /mnt
```

#### デバイスツリーバイナリ（upstreamカーネルの場合）

upstream/mainlineカーネルを使用する場合は コンパイルした
[デバイスツリー](https://linux-sunxi.org/Device_tree)をブート
パーティションにインストールする必要があります。

```bash
$ cp /home/user/dir/linux/arch/arm/boot/dtbs/<your_board>.dtb /mnt
```

(カーネルソースが _/home/user/dir/linux_ にあると仮定します)

利用可能なコンパイル済みツリーはカーネルディレクトリ _arch/arm/boot/dts/_ を
一覧することで見つけることができます。

```bash
$ ls /home/user/dir/linux/arch/arm/boot/dts/*.dtb
```

カーネルコンパイル時にどのツリーがコンパイルされるかはカーネル構成の
際に "System Type" menuconfigで何を選択したかに依存します。ただし、
カーネルを再コンパイルすることなく[任意のツリーをコンパイルする](https://linux-sunxi.org/Device_tree#Compiling_the_Device_Tree)
ことができます。

U-Bootはデフォルトでは[U-Bootのconfigオプション](https://linux-sunxi.org/Mainline_U-Boot#Build) `CONFIG_DEFAULT_DEVICE_TREE=<your_board>`
で指定したツリーで起動します。

### ブートパーティションまたはルートパーティションのアンマウント

再度、これらのパーティションをアンマウントします。

```bash
$ umount /mnt
```

### rootfsの設定

[Bootable_SD_card#Rootfs](https://linux-sunxi.org/Bootable_SD_card#Rootfs)を
参照してください。

### カーネルモジュールのインストール

最後のステップとして、新しく作成したrootfsにカーネルモジュールを
コピーする必要があります。新しく作成したrootfsの最上位ディレクトリに
移動して以下を実行してください。

```bash
$ mount ${cardroot} /mnt
$ mkdir -p /mnt/lib/modules
$ rm -rf /mnt/lib/modules
$ cp -r <PATH_TO_KERNEL_TREE>/output/lib /mnt/
$ umount /mnt
```

(ここで<PATH_TO_KERNLE_TREE>は[上で記述した](https://linux-sunxi.org/Manual_build_howto#Building_the_kernel)
カーネルを微ツドしたディレクトリに置き換えてください)

### 起動!

これでSDCardのファイルシステムをアンマウントし、新しいインストールを
起動できるはずです。

### 参照

すべてのSoC用のマニュアルビルドの手引があります。

- [H3マニュアルビルドの手引](https://linux-sunxi.org/H3_Manual_build_howto)
- [マニュアルビルドの手引](manual_build_howto.md)
- [Mainlineカーネルの手引](mianline_kernel_howoto.md)
