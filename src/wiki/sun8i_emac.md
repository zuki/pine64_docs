# Sun8i emac

[オリジナル](https://linux-sunxi.org/Sun8i_emac)

このページはA83T/H3/A64の統合イーサネットMACを処理する
*sun8i_emac/dwmac-sun8i*ドライバについて説明します。

## 仕様

EMACの主な機能は次のとおりです。

- 10/100/1000Mbit/s
- RX/TX CSO (CheckSum Offload)

## ステータス

Linuxドライバはdwmac-sun8iであり、これはstmmacドライバのグルーです。
最新版は <https://github.com/torvalds/linux/blob/a3c57ab79a06e333a869ae340420cb3c6f5921d3/drivers/net/ethernet/stmicro/stmmac/dwmac-sun8i.c#L3> に
あります。

もう一つのレガシーなドライバ *sun8i-emac* もあります。最新のsun8i-emacドライバは<https://github.com/montjoie/linux/tree/sun8i-emac-v5.1/drivers/net/ethernet/allwinner> です。このドライバがメインラインに到達することはなく、
開発は中止されています。

### H3

H3 SoCはよくサポートされており、すべてのMIIタイプ（Internal MII、RGMII）で
微調整は必要ありません。

### A83T

- BananaPI M3: PHY への電力供給には wens の a80-pmic ubootブランチ
  (<https://github.com/wens/u-boot-sunxi/tree/a80-pmic>) が必要です。
- H8 homlet: PHYはAC200ですがデータシートがありません。

### A64

純正のboot0もBSP U-BootもPMICがPHYに電力を供給するようにプログラムして
いません。最新の["アップストリーム"のファームウェア](https://github.com/apritzel/pine64/commit/7b5c8547c7c1988fc8b9ac5a4bc62ee4d0fd4db9)はARM Trusted Firmware
(ATF) の[コード](https://github.com/apritzel/arm-trusted-firmware/commit/23f7954665a547958379383853ad67f264661adf#diff-d0992c292503fdf9728e9409cd2af420R245)でPHYを有効にするのでEMACドライバが動作します。

### Pine64+ (ギガビット PHY)

回路図の19ページのセクションC5に電源要件が示されています。

- DC1-SWをイネーブルにする必要があります（AXP803 PMICのレジスタ 0x12の
  ビット7）
- 回路図はPMICのGPIO1出力を2.5ボルトのLDO出力として設定する必要がある
  ことを示唆しています（PMICレジスタ0x93に0x12（=2.5V）、PMICレジスタ
  0x92に0x3（=LDOを有効にする））。しかし、BSPカーネルはこのレジスタを
  無効（0x92: 0x7）のままにしているので、どうやらここでの駆動は必要
  ないようです。これはPine64+はRGMIIを3.3 ボルトで動作させていることを
  意味します。
- PHYのPHYRSTBピンはSoCのPD14 ピン（紛らわしいことにMAC-RSTと表示されて
  いる）に接続されているがプルアップされている。PD14はEMACブロック
  (RGMII-NULL) の一部ですが異なるコンフィギュレーション（ファンクション
  4ではない）が必要です: 無効(7) にするか、ハイ出力ピンとします。

## ヒント、トラブルシューティング

- EMACリセットタイムアウト

  このエラーは一般にPHYに電源が供給されていないことに関連している。

- ギガビッ EMACでリンクするが転送しない

  おそらく、RX/TX 遅延を調整する必要があります。正しい値はFEXファイルに
  あると思われます。

  今のところ、唯一の方法は /dev/mem 経由で値を書き込むことです。

  busyboxの`devmem`アプレットを使うか、`free-electrons.com/pub/mirror/ devmem2.c`ユーティリティをコンパイルしてください。

  例：BPIM3の場合
    ```bash
    devmem 0x1c00030 w 0x1806
    ```

- MACアドレスの先頭に０、３つ

  最新のubootを試してください。

- ケーブルを接続したまま起動するとリンクしない

  最新のubootを試してください。

  ## 性能

  性能はiperfで計算しました。これらの数値は決定的なものではありません。

| ボート | 送信 | 受信 | 注記 |
|:-------|-----:|-----:|:-----|
| OrangePiPC (100Mbit FullDuplex) | 94Mb/s | 94Mb/s | |
| Bananapi M3 (Gigabit FD) | 373Mb/s | 387Mb/s | no PSCI |
| Pine64 | 511Mb/s | 428Mb/s | |
