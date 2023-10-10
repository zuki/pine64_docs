# 3.12 GIC

GICの詳細については*GIC PL400テクニカルリファレンスマニュアル*と
*ARM GIC Architecture Specification V2.0*を参照してください。

## 3.12.1 割り込みソース

### 割り込み種別

- SRC 0-15: SGI (Software-Generated Interrupt)
- SRC 16-31: PPI (Private Peripheral Interrupt)
- SRC 32-156: SPI (Shared Peripheral Interrupt)

### デバイスツリーのinterrupts属性 = &lt;A B C&gt;

- A: 割り込み種別: 0: SPI,  1: PPI
- B: 割り込み番号: 各種別内の番号。下の表を探す際にSPIは+32, PPIは+16
- C: フラグ: 1: Lo->Highのエッジトリガ、2: High->Lowのエッジトリガ、
     4: アクティブHighのレベルセンシティブ、8: アクティブLowのレベルセンシティブ

### デバイスツリーの例

```
timer@1c20c00 {
   compatible = "allwinner,sun50i-a64-timer",
         "allwinner,sun8i-a23-timer";
   reg = <0x01c20c00 0xa0>;
   interrupts = <0 18 4>,
         <0 19 4>;
   clocks = <&osc24M>;
};
```

タイマー割り込みは2つ（Timer0, Timer1）あり、種別は0, SPI番号は18
(SRC: 50), 19 (SRC: 51), アクティブHighのレベルセンシティブ

### 割り込みソースの一部

| 割り込みソース | SRC | ベクタアドレス | 説明 |
|:---------------|----:|:---------------|:-----|
| UART 0 | 32 | 0x0080 | UART 0 割り込み |
| Timer 0 | 50 | 0x00C8 | Timer 0 割り込み |
| Timer 1 | 51 | 0x00CC | Timer 1 割り込み |
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
