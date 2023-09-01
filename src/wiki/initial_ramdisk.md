# 初期Ramdisk

[オリジナル](https://linux-sunxi.org/Initial_Ramdisk)

sunxiで初期ramdisk (initramfs/initrd) を使用するには、たとえば、
モジュラーsataがサポートされておりrootファイルシステムがsata上に
ある場合、`mkimage`を使用して通常の初期ramdiskをU-Bootのフォーマットに
変換し、U-Bootの構成を調整する必要があります。

初期ramdiskをU-Bootのフォーマットに変換するには次のコマンドを使用します。

```bash
mkimage -A arm -T ramdisk -C none -n uInitrd -d /path/to/initrd.img /path/to/uInitrd
```

U-Bootの構成ではuInitrdファイルをロードし、そのアドレスをbootmに追加します。

- `fex/script.bin`カーネル（sunxiの3.0や3.4など）の場合は次のようにします。

```bash
fatload mmc 0 0x43000000 script.bin
fatload mmc 0 0x41000000 uImage
fatload mmc 0 0x50000000 uInitrd
bootm 0x41000000 0x50000000
```

uInitrdはメモリ割り当てが競合するため、`0x43000000-0x4FFFFFF`の範囲に
入れてはいけません。詳細は[こちら](https://groups.google.com/d/msg/linux-sunxi/Itt3Bko0bVA/Mqt5zTj1qaIJ)を参照してください。

（これは今でも有効ですか？私は`kernel-3.4.113`を`0x48000000`にロードし、
`initrd`は`0x47200000`にロードし、cmaを`0x4d400000`から300MBから予約
することができました。）

- mainline/devicetreeを使うカーネル(たとえば、sunxi-currentやmainline
  3.8+)の場合は以下を使ったほうが良いかもしれません。

```bash
fatload mmc 0 0x43000000 board.dtb
fatload mmc 0 0x41000000 uImage
fatload mmc 0 0x45000000 uInitrd
bootm 0x41000000 0x45000000 0x43000000
```

少なくともcubieboardとmainline/devicetreeカーネルの場合はbootmを
呼び出す前にinitrd_highを次のように設定した方が良いでしょう。

```bash
setenv initrd_high 0xffffffff
```

これをsata上のrootfsを使用するために行っている場合は初期ramdiskが
sataモジュールをロードするようにする必要があります。`initramfs-tools`を
使用しているDebianでは`/etc/initramfs-tools/modules`に`sw_ahci_platform`の
¹行を追加するだけです。

`/etc/initramfs-tools/modules`を編集した後、使用したい初期ramdskを以下の
コマンドで更新する必要があります（これはinitramfs-toolsが知っている
すべての利用可能なカーネルのinitramfsを更新します）。

```bash
update-initramfs -u -k all
```

また、早い段階で初期ramdiskに表示モジュールをロードさせ、何か問題が
発生した場合に出力を見たり、シェルと対話したりできるようにしておくと
よいでしょう。これを行うには以下のモジュールを`/etc/initramfs-tools/modules`
にこの順で追加します（そして、再度初期ramdiskを更新します）。

```bash
lcd
hdmi
ump
disp
mali
mali_drm
```

初期ramdiskがfatパーティションのファイルにアクセスする必要があるような
特殊な状況では、nls_asciiやnls_cp437などの関連するcharsetモジュールも
早期に初期ramdiskにロードされるモジュールリストに追加したらよいでしょう。
