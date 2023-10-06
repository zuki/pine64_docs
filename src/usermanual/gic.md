# 3.12 GIC

GICの詳細については*GIC PL400テクニカルリファレンスマニュアル*と
*ARM GIC Architecture Specification V2.0*を参照してください。

## 3.12.1 割り込みソース

| 割り込みソース | SRC | ベクタアドレス | 説明 |
|:---------------|----:|:---------------|:-----|
| UART 0 | 32 | 0x0080 | UART 0 割り込み |
| Timer 0 | 50 | 0x00C8 | Timer 0 割り込み |
| MSGBOX | 81 | 0x0144 | MSGBOX 割り込み |
| DMA | 82 | 0x0148 | DMAチャネル 割り込み |
| HS TIMER | 83 | 0x14C | 高速タイマー 割り込み |
| SD/MMC 0 | 92 | 0x170 | SD/MMCホストコントローラ 0 割り込み |
| SD/MMC 1 | 93 | 0x174 | SD/MMCホストコントローラ 1 割り込み |
| SD/MMC 2 | 94 | 0x178 | SD/MMCホストコントローラ 2 割り込み |
| DRAM_MDFS | 101 | 0x0194 | DRAM MDFS 割り込み |
| USB-OTG | 103 | 0x019C | USB-OTG 割り込み |
| USB-OTG-EHCI | 104 | 0x01A0 | USB-OTG-EHCI 割り込み |
| USB-OTG-OHCI | 105 | 0x01A4 | USB-OTG-OHCI 割り込み |
| USB-EHCI | 106 | 0x01A8 | USB-EHCI 割り込み |
| USB-OHCI | 107 | 0x01AC | USB-OHCI 割り込み |
| EMAC | 114 | 0x01C8 | EMAC 割り込み |
