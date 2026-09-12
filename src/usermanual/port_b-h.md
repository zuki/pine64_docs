# 3.21 ポートコントローラ (CPUx-PORT) : p.376

このチップには多機能の入出力ピン用ポートが7つあります。以下のとおりです。

- ポート B(PB)：10 入出力ポート
- ポート C(PC): 17 入出力ポート
- ポート D(PD): 25 入出力ポート
- ポート E(PE): 18 入出力ポート
- ポート F(PF): 7 入出力ポート
- ポート G(PG): 14 入出力ポート
- ポート H(PH): 12 入出力ポート

各種システム構成において、これらのポートはソフトウェアで簡単に構成できます。
マルチプレックス関数を使用しない場合、これらすべてのポートをGPIOとして構成できます。
合計で3つの外部PIO割り込みソースがサポートされており、割り込みモードはソフトウェアで
構成可能です。

**訳注 1**: デフォルト値はいずれも`IO Disable`になっている。使用する場合は構成が必要。
<br/>
**訳注 2**: [Pine A64ピン配置](https://files.pine64.org/doc/Pine%20A64%20Schematic/Pine%20A64%20Pin%20Assignment%20160215.pdf)
<br/>
**訳注 3**: 外部PIO割り込みソースはB (m=0), G (m=1), H (m=2)の3つである。

## 3.21.1 ポートコントローラレジスタ一覧

### 基底アドレス

| モジュール名 | 基底アドレス |
|:-------------|:---------------|
| PIO | 0x01C20800 |


### レジスタ

- nはB-Hを表し、B=1, C=2, ..., H=7 である
- たとえば、ポートBについては
    - PB_CFG0 : 0x24
    - PB_CFG1 : 0x28
    - PB_CFG2 : 0x2C
    - PB_CFG3 : 0x30
    - PB_DAT  : 0x34
    - PB_DRV0 : 0x38
    - PB_DRV1 : 0x3C
    - PB_PUL0 : 0x40
    - PB_PUL1 : 0x44
    - PB_INT_CFG0 :

| レジスタ名 | オフセット | 記述 |
|:-----------|:-----------|:-----|
| Pn_CFG0 | n * 0x24 + 0x00 | Port n Configure Register 0 (n from 1 to 7) |
| Pn_CFG1 | n*0x24+0x04 | Port n Configure Register 1 (n from 1 to 7) |
| Pn_CFG2 | n*0x24+0x08 | Port n Configure Register 2 (n from 1 to 7) |
| Pn_CFG3 | n*0x24+0x0C | Port n Configure Register 3 (n from 1 to 7) |
| Pn_DAT | n*0x24+0x10 | Port n Data Register (n from 1 to 7) |
| Pn_DRV0 | n*0x24+0x14 | Port n Multi-Driving Register 0 (n from 1 to 7) |
| Pn_DRV1 | n*0x24+0x18 | Port n Multi-Driving Register 1 (n from 1 to 7) |
| Pn_PUL0 | n*0x24+0x1C | Port n Pull Register 0 (n from 1 to 7) |
| Pn_PUL1 | n*0x24+0x20 | Port n Pull Register 1 (n from 1 to 7) |
| Pn_EINT_CFG0 | 0x200+m*0x20+0x00 | PIO External Interrupt Configure Register 0 |
| Pn_EINT_CFG1 | 0x200+m*0x20+0x04 | PIO External Interrupt Configure Register 1 |
| Pn_EINT_CFG2 | 0x200+m*0x20+0x08 | PIO External Interrupt Configure Register 2 |
| Pn_EINT_CFG3 | 0x200+m*0x20+0x0C | PIO External Interrupt Configure Register 3 |
| Pn_EINT_CTL | 0x200+m*0x20+0x10 | PIO External Interrupt Control Register |
| Pn_EINT_STA | 0x200+m*0x20+0x14 | PIO External Interrupt Status Register |
| Pn_EINT_DEB | 0x200+m*0x20+0x18 | PIO External Interrupt Debounce Registe |

## 3.21.2 ポートコントローラレジスタの説明
