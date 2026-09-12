# 3.22 ポートコントローラ (CPUs-PORT) : p.410

チップには多機能の入出力ピン用ポートが1つあります。

- ポート L(PL):13 入力/出力ポート

各種システム構成において、これらのポートはソフトウェアで簡単に構成できます。
マルチプレックス関数を使用しない場合、これらすべてのポートをGPIOとして構成できます。
外部PIO割り込みソースが1つサポートされており、割り込みモードはソフトウェアで
構成可能です。

### 基底アドレス

| モジュール名 | 基底アドレス |
|:-------------|:---------------|
| PIO | 0x01F02C00 |

### レジスタ

| レジスタ名 | オフセット | 記述 |
|:-----------|:-----------|:-----|
| PL_CFG0 | 0*0x24+0x00 | Port L Configure Register 0 |
| PL_CFG1 | 0*0x24+0x04 | Port L Configure Register 1 |
| PL_CFG2 | 0*0x24+0x08 | Port L Configure Register 2 |
| PL_CFG3 | 0*0x24+0x0C | Port L Configure Register 3 |
| PL_DAT | 0*0x24+0x10 | Port L Data Register |
| PL_DRV0 | 0*0x24+0x14 | Port L Multi-Driving Register 0 |
| PL_DRV1 | 0*0x24+0x18 | Port L Multi-Driving Register 1 |
| PL_PUL0 | 0*0x24+0x1C | Port L Pull Register 0 |
| PL_PUL1 | 0*0x24+0x20 | Port L Pull Register 1 |
| PL_EINT_CFG0 | 0x200+0*0x20+0x00 | PIO External Interrupt Configure Register 0 |
| PL_EINT_CFG1 | 0x200+0*0x20+0x04 | PIO External Interrupt Configure Register 1 |
| PL_EINT_CFG2 | 0x200+0*0x20+0x08 | PIO External Interrupt Configure Register 2 |
| PL_EINT_CFG3 | 0x200+0*0x20+0x0C | PIO External Interrupt Configure Register 3 |
| PL_EINT_CTL | 0x200+0*0x20+0x10 | PIO External Interrupt Control Register |
| PL_EINT_STA | 0x200+0*0x20+0x14 | PIO External Interrupt Status Register |
| PL_EINT_DEB | 0x200+0*0x20+0x18 | PIO External Interrupt Debounce Register |