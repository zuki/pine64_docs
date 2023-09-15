# TPL/SPLからのブート

[オリジナル](https://u-boot.readthedocs.io/en/latest/usage/spl_boot.html)

U-BootバイナリはおそらくBoot ROMが直接ロードするには大きすぎます。
これがU-Bootを複数のブートステージに分割することを促した元々の
要因でした。

通常、U-Bootは以下のブートフェーズをたどります。ここで、TPL、VPL、SPLは
オプションです。多くのボードはSPLを使用し、TPLを使用するボードはほとんど
ありません。

**TPL**

    Tertiary Program Loader: ごく初期の可能な限り小さいinitです。
    SPL（有効な場合はVPL）をロードします。

**VPL**

    Verifying Program Loader: オプションの検証ステップで、A/B検証
    ブートが有効な場合、複数のSPLバイナリの中から1つを選択できます。
    VPLロジックの実装は作業中です。今のところ、SPLをブートするだけです。

**SPL**

    Secondary Program Loader: SDRAMをセットアップし、U-Boot本体を
    ロードします。他のファームウェアコンポーネントをロードすることも
    できます。

**U-Boot**

    U-Boot本体: コマンドを含む唯一のステージです。オペレーティング
    システムをロードするロジック（UEFI経由など）も実装しています。

**注記** PowerPCアーキテクチャの命名規約は他のアーキテクチャとは
異なっており、ブートシーケンスはSPL->TPL->U-Bootとなります。

UブートSPLのその他の用途は次のとおりです。

- U-BootをBL33として呼び出すARM Trusted FirmwareのBL31の起動
- EDK IIの起動
- Linuxの起動（[ファルコンモード](https://u-boot.readthedocs.io/en/latest/develop/falcon.html)など）
- U-Boot本体を呼び出すRISC-V OpenSBIの起動

## 対象のバイナリ

SPL/TPLにより以下のバイナリがロードできます。

- エントリアドレスが開始アドレスに等しい生のバイナリ。これはTPLが
  サポートする唯一のバイナリ形式です。
- FIT image
- レガシーU-Boot image

### 構成

SPLでraw imageがサポートされるのは`CONFIG_SPL_RAW_IMAGE_SUPPORT=y`の
場合だけです。

FIT imageをロードするには`CONFIG_SPL_FIT=y`と`CONFIG_SPL_LOAD_FIT=y`が
必要です。

レガシー U-Boot imageをロードするには`CONFIG_SPL_LEGACY_IMAGE_FORMAT=y`が
必要です。`CONFIG_SPL_LEGACY_IMAGE_CRC_CHECK=y`はレガシー U-Boot imageの
CRC32のチェック機能を有効にします。

## Imageロードメソッド

ボードが利用できるImageのブートメソッドは2箇所で定義する必要があります。

ボードコードは最大5つのブートメソッドとそれらを試行する順序を列挙する
関数`board_boot_order()`を実装します (ブートメソッドの最大数は現在の
ところ変数`spl_boot_list[]`としてハードコーディングされています)。
ブートメソッド関数が1つしかない場合は、代わりに`spl_boot_device()`を
実装することができます。

構成はこれらのブートメソッドのうちどれが実際に利用可能かを制御します。

### ブロックデバイスからのロード

#### MMC1, MMC2, MMC2_2

これらのメソッドはSDカードまたはeMMCからimageを読み込みます。
"MMC"の後の最初の数字はデバイス番号を示します。必須の構成設定は次の
とおりです。

- `CONFIG_SPL_MMC=y` または `CONFIG_TPL_MMC=y`

PCI接続のMMCコントローラを使用する場合はさらに以下を指定する必要が
あります。

- `CONFIG_SPL_PCI=y`
- `CONFIG_SPL_PCI_PNP=y`
- `CONFIG_MMC=y`
- `CONFIG_MMC_PCI=y`
- `CONFIG_MMC_SDHCI=y`

ファイルシステムからロードするには以下を設定します。

- `CONFIG_SPL_FS_FAT=y` または `CONFIG_SPL_FS_EXT=y`
- `CONFIG_SPL_FS_LOAD_PAYLOAD_NAME=”<filepath>”`

[以下省略]
