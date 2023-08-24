# Mainlineカーネルの手引

[オリジナル](https://linux-sunxi.org/Mainline_Kernel_Howto)

このページではLinux mainline カーネルのコンパイルと使用方法に
ついて説明します。

**警告**: mainlineのNANDサポートはAllwinnerのNANDサポートと互換性が
なく、既存のNANDが読めなくなります。

## 現状

メインライン化作業の詳細な状況、サポートされているボードとドライバの
範囲については[Linuxメインライン化作業](https://linux-sunxi.org/Linux_mainlining_effort)のページを
参照してください。

SoCやサブシステムの中にはまだサポートされていないものがあるかも
しれません。その場合は、A10/A13/A20用のAllwinnersのコードに近い、
[大幅にハックされたカーネルバージョン](https://linux-sunxi.org/Linux_Kernel)に
戻すか、BSPカーネルを使用する必要があるかもしれません。

## 前提条件

この手引は[マニュアルビルドの手引](manual_build_howto.md)の一部と
して実行する必要があります。マニュアルビルドの手引ではツールチェイン、
u-boot、rootfsなどについても説明されています。

## カーネルソース

### Linusのツリー

安定版はLinus Torvalds氏によりリリースされています。Linux 3.8 以降、
Allwinnerのサポートが徐々に追加されています。

完全なクローン(将来的に開発を行う場合)を行うには次を実行します。

```bash
$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
```

mainlineを動かすだけなら浅いクローンで十分でしょう。

```bash
$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git --depth=1
```

 ### 次の安定版リリースにマージされるパッチ

アクセプトされ、マージされ、次の安定版リリースに含まれる予定の
すべてを含んでいる _sunxi-next_ ブランチもメンテナンスされています。
何らかの開発をしたいのであれば、おそらくこちらが最適でしょう。

```bash
$ git clone [http://github.com/linux-sunxi/linux-sunxi/tree/sunxi-next git://github.com/linux-sunxi/linux-sunxi.git] -b sunxi-next --depth=1
```

## カーネル構成

### defconfig

動作するカーネルを手に入れるには構成に以下を使用するだけです。

#### armhf

```bash
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- sunxi_defconfig
```

#### arm64

```bash
$ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make defconfig
```

### oldconfig

古い関連する `.config` がある場合、たとえば、後期の debian を
インストールしていて新たにカーネルをビルドしたい場合は、.configを
コピーして `make ARCH=arm oldconfig` や `make ARCH=arm64 oldconfig` を
実行できます。

> **注意**: アーキテクチャを指定せずに `make menuconfig` を実行した
> 場合、アーキテクチャ固有のオプションは**失われます**。その場合は、
> .configを再設定して、再度、実行します。

### マニュアル構成

構成を変更したい場合は、次を実行します。

#### armhf

```bash
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```

#### arm64

```bash
$ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make menuconfig
```

コンパイルが終わると _arch/arm/boot_ (armhf)、または、
_arch/arm64/boot_ (arm64)に**Image**が生成されているはずです。

## デバイスツリー

#### armhf

```bash
$ ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j4 dtbs
```

#### arm64

```bash
$ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make -j4 dtbs
```

これで _arch/arm/boot/dts_ (armhf) か _arch/arm64/boot/dts_ (arm64) に
デバイスにマッチするデバイスツリーバイナリ (.dtb) が作成され、
デバイスのデバイスページにリストされるはずです。

sunxiのdtbの名前はすべて`<family>-<soc>-<board>.dtb`というパターンに
なっています。

## モジュール

#### armhf

```bash
$ ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j4 modules
$ ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=<any-path-you-like> make modules modules_install
```

#### arm64

```bash
$ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make -j4 modules
$ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=<any-path-you-like> make modules modules_install
```

ビルドが成功したら指定した`INSTALL_MOD_PATH` ディレクトリに
モジュールがあるはずです。

## ヘッダー

#### armhf

```bash
$ ARCH=arm INSTALL_HDR_PATH=<any-path-you-like> make headers_install
```

#### arm64

```bash
$ ARCH=arm64 INSTALL_HDR_PATH=<any-path-you-like> make headers_install
```

`INSTALL_HDR_PATH`にはヘッダーをインストールしたいパス、たとえば、
`/usr`や、クロスビルドの場合は`<some_prefix>/usr`を指定してください。

## Debianカーネル固有の注意

[省略]

## 起動

### SDカードのブートパーティション

[マニュアルビルドの手引](manual_build_howto.md)と同様に、
ブートパーティションを設定する必要があります。_script.bin_ の代わりに
ボード固有の**.dtb**をブートパーティションにインストールする必要が
あります。（上記のカーネルコンパイルでこの[デバイスツリー](https://linux-sunxi.org/Device_tree)の
バイナリ表現が自動的に生成されているはずです）。

#### boot.cmd

[マニュアルビルドの手引](manual_build_howto.md)と同様に、
以下の内容でboot.cmdを作成します。

```bash
$ fatload mmc 0 0x46000000 zImage
$ fatload mmc 0 0x49000000 <board>.dtb

$ setenv bootargs console=ttyS0,115200 earlyprintk root=/dev/mmcblk0p<partition> rootwait panic=10 ${extra}

$ bootz 0x46000000 - 0x49000000
```

> **注意**: **arm64**カーネルをブートするには、_zImage_ ではなく
> _Image_ を使うことを忘れないでください。また、_bootz_ ではなく
> _booti_ でロードする必要があります。そのため、上記の boot.cmd
> ファイルに対応する行を追加する必要があります。

必要に応じて、_fatload_ を _ext2load_ に置き換えてください。
[earlyprintk](https://linux-sunxi.org/Mainline_Kernel_Howto#Early_printk) は
オプションです。カーネルが問題なく起動するのであれば、削除して
構いません。

古いU-Bootを使っている場合（使うべきでないか？）、抽出したカーネルが
デバイスツリーの設定を上書きしないようにするために以下の行が必要に
なるかもしれません。

```bash
$ setenv fdt_high ffffffff
```

`initramfs`を使いたい場合、 _bootz_ コマンドは次のようになります。

```bash
$ bootz 0x46000000 0x<initramfs-address> 0x49000000
```

次に _boot.scr_ を生成します。_zImage_ と _<board>.dtb_ ファイルも
忘れずにコピーしてください。

### eMMCのぶーとパーティション

[省略]

### ネットワークブート設定

TODO: [mailineでのハッキングに可能なセットアップ](https://linux-sunxi.org/Possible_setups_for_hacking_on_mainline)に含める。

## 追加機能

### SYSV IPC (debianユーザ向け)

[省略]

### Early printk

カーネルが何もしない場合のみ、printkの早期サポートを有効にしてください。

まず、

```
    Kernel hacking  --->
```

に行き、**CONFIG_DEBUG_KERNEL**を有効にします。

```
[*] Kernel debugging
```

実際にearly printkを使用するには**CONFIG_DEBUG_LL**も有効にし、
**CONFIG_DEBUG_SUNXI_**オプションのいずれか1つを選択し、
**CONFIG_EARLY_PRINTK**を有効にする必要があります。

```
[*] Kernel low-level debugging functions (read help!)
      Kernel low-level debugging port (Kernel low-level debugging messages via sunXi UART0)  --->
[*] Early printk
```

デバッグポート（**CONFIG_DEBUG_SUNXI_**）の選択は使用するボードに
よって異なります。これらはすべてデバッグポートのrawアドレスへの
シンボリック名であるため、これらの選択は作為的です。

- ほとんどのSoCではまず**DEBUG_SUNXI_UART0**を選択するべきです。

  ```
  (X) Kernel low-level debugging messages via sunXi UART0
  ```

- A13ではまず**DEBUG_SUNXI_UART1**を試してください。

  ```
  (X) Kernel low-level debugging messages via sunXi UART1
  ```

- A23 ではまず**DEBUG_SUNXI_R_UART**を試してください。

  ```
  (X) Kernel low-level debugging messages via sunXi R_UART
  ```

- A80ではまず**DEBUG_SUN9I_UART0**を試してください。

  ```
  (X) Kernel low-level debugging messages via sun9i UART0
  ```

### simplefb

[省略]

## 新規デバイスの追加

主に[デバイスの説明](https://linux-sunxi.org/Device_Tree)を
書く必要があります。良い例を探している人をは透けるために
[パッチを送って](https://linux-sunxi.org/Sending_patches)ください。

## 参照

- [マニュアルビルドの手引](manual_build_howoto.md)
- [Linuxメインライン化作業](https://linux-sunxi.org/Linux_mainlining_effort)
- [mailineでのハッキングに可能なセットアップ](https://linux-sunxi.org/Possible_setups_for_hacking_on_mainline)
