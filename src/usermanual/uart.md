# 7.3 UART

## 7.3.1 概要

UARTは周辺機器、モデム（DCE: Data Charrier Equipment）、データセットとの
シリアル通信に使用されます。データはマスタ（CPU）からAPBバスを介して
UARTに書き出され、シリアル形式に変換されて宛先デバイスに送信されます。
また、データはUARTによって受信され、マスタ（CPU）が読み戻しするために
格納されます。

UARTには文字長、ボーレート、パリティの生成とチェック、割り込み生成を
制御するレジスタがあります。UARTからの割り込み出力信号は1つだけですが、
そのアサートにはいくつかの優先割り込みタイプがあります。各割り込み
タイプは制御レジスタにより個別に有効/無効にすることができます。

UARTには16450と16550の動作モードがあり、さまざまな標準ソフトウェア
ドライバと互換性があります。16550モードでは送受信動作は両者とも
FIFOによりバッファされます。16450モードではこのFIFOは無効です。

UARTは5ビットから8ビットのデータ長、オプションのパリティビット、
1ビット、1.5ビット、2ビットのストップビットをサポートし、AMBA APB CPU
インタフェースによって完全にプログラム可能です。16ビットのプログラム
可能なボーレートジェネレータと8ビットのスクラッチレジスタが、送受信用の
個別のFIFOとともに搭載されています。8本のモデム制御ラインと診断用
ループバックモードが提供されています。

TXバッファ/FIFO、RXバッファ/FIFO、モデムステータス、ラインステータスの
条件に応じて割り込みを生成できます。

赤外線SIRシリアルデータフォーマットが必要なシステムに統合するために
UARTはソフトウェアでプログラム可能なIrDA SIRモードを持つように構成
できます。このモードが選択されていない場合はUART（RS232標準）シリアル
データフォーマットだけが使用可能です。

UARTは次の機能を持っています。

- 業界標準の16550 UARTとの互換性
- 64バイトの送受信データFIFO
- DMAコントローラインタフェース
- ソフトウェア/ハードウェアによるフロー制御
- プログラム可能な送信保持レジスタエンプティ割り込み
- FIFO、ステータスチェンジの割り込みサポート
- IrDA 1.0 SIRのサポート

## 7.3.2 UARTのタイミング図

![図7-5,6](img/fig-7.fig7-5.6.png)

## 7.3.3 UARTピンリスト

| ポート名 | 幅 | 方向 | 説明 |
|:---------|:--:|:----:|:-----|
| UART0_TX | 1 | OUT | UARTシリアルビット出力 |
| UART0_RX | 1 | IN | UARTシリアルビット入力 |
| UART1_TX | 1 | OUT | UARTシリアルビット出力 |
| UART1_RX | 1 | IN | UARTシリアルビット入力 |
| UART1_RTS | 1 | OUT | UART Request To Send. このアクティブLow出力信号はUARTがデータ送信の準備ができていることをモデムに通知する |
| UART1_CTS | 1 | IN | UART Clear To End. このアクティブLow信号はモデムがデータを受け入れる準備ができたことを示す入力である |
| UART2_TX | 1 | OUT | UARTシリアルビット出力 |
| UART2_RX | 1 | IN | UARTシリアルビット入力 |
| UART2_RTS | 1 | OUT | UART Request To Send. このアクティブLow出力信号はUARTがデータ送信の準備ができていることをモデムに通知する |
| UART2_CTS | 1 | IN | UART Clear To End. このアクティブLow信号はモデムがデータを受け入れる準備ができたことを示す入力である |
| UART3_TX | 1 | OUT | UARTシリアルビット出力 |
| UART3_RX | 1 | IN | UARTシリアルビット入力 |
| UART3_RTS | 1 | OUT | UART Request To Send. このアクティブLow出力信号はUARTがデータ送信の準備ができていることをモデムに通知する |
| UART3_CTS | 1 | IN | UART Clear To End. このアクティブLow信号はモデムがデータを受け入れる準備ができたことを示す入力である |
| UART4_TX | 1 | OUT | UARTシリアルビット出力 |
| UART4_RX | 1 | IN | UARTシリアルビット入力 |
| UART4_RTS | 1 | OUT | UART Request To Send. このアクティブLow出力信号はUARTがデータ送信の準備ができていることをモデムに通知する |
| UART4_CTS | 1 | IN | UART Clear To End. このアクティブLow信号はモデムがデータを受け入れる準備ができたことを示す入力である |
| S_UART_TX | 1 | OUT | UARTシリアルビット出力 |
| S_UART_RX | 1  | IN | UARTシリアルビット入力 |

## 7.3.4  UART制御レジスタリスト

### 基底アドレス

6つのUARTコントローラがあります。すべてのUARTコントローラは
シリアルIrDAとして構成できます。

| モジュール名 | 基底アドレス |
|:-------------|:---------------|
| UART0 | 0x01C28000 |
| UART1 | 0x01C28400 |
| UART2 | 0x01C28800 |
| UART3 | 0x01C28C00 |
| UART4 | 0x01C29000 |
| R-UART | 0x01F02800 |

###  UART制御レジスタ

| レジスタ名 | オフセット | 記述 |
|:-----------|:-----------|:-----|
| UART_RBR | 0x00 | UART Receive Buffer Register |
| UART_THR | 0x00 | UART Transmit Holding Register |
| UART_DLL | 0x00 | UART Divisor Latch Low Register |
| UART_DLH | 0x04 | UART Divisor Latch High Register |
| UART_IER | 0x04 | UART Interrupt Enable Register |
| UART_IIR | 0x08 | UART Interrupt Identity Register |
| UART_FCR | 0x08 | UART FIFO Control Register |
| UART_LCR | 0x0C | UART Line Control Register |
| UART_MCR | 0x10 | UART Modem Control Register |
| UART_LSR | 0x14 | UART Line Status Register |
| UART_MSR | 0x18 | UART Modem Status Register |
| UART_SCH | 0x1C | UART Scratch Register |
| UART_USR | 0x7C | UART Status Register |
| UART_TFL | 0x80 | UART Transmit FIFO Level |
| UART_RFL | 0x84 | UART_RFL |
| UART_HALT | 0xA4 | UART Halt TX Register |
