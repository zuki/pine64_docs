# 3.8 RTC

## 3.8.1 概要

リアルタイムクロック（RTC）はカレンダー用です。30ビットのカウンターで
構成され、YY-MM-DDおよびHH-MM-SSの経過時間をカウントします。
システム電源がオフの間はバックアップバッテリーで動作させることが
できます。うるう年ジェネレータと独立した電源端子（RTC_VIO）を備えて
います。

アラームは電源オフモード/通常動作モードにおいて指定した時刻にアラーム
信号を発生します。通常動作モードではアラーム割り込みと電源管理ウェイク
アップがアクティブになります。電源オフモードでは電源管理ウェイクアップ
信号がアクティブになります。2種類のアラームがあります。アラーム0は
一般的なアラームでカウンタは秒単位です。アラーム1は週アラームであり、
カウンタはリアルタイムに基づいています。

32,768Hzの発振器はRTCに低電力で正確なリファレンスを提供するためにのみ
使用されます。

General Purposeレジスタはフラグレジスタにすることができ、VDD_RTCが
パワーオフでない時は常に値を保存します。

## 3.8.2 RTCレジスタリスト

### 基底アドレス

| モジュール名 | 基底アドレス |
|:-------------|:---------------|
| RTC | 0x01F00000 |


### レジスタ

| レジスタ名 | オフセット | 記述 |
|:-----------|:-----------|:-----|
| LOSC_CTRL_REG | 0x0 | Low Oscillator Control Register |
| LOSC_AUTO_SWT_STA_REG | 0x4 | LOSC Auto Switch Status Register |
| INTOSC_CLK_PRESCAL_REG | 0x8 | Internal OSC Clock Prescalar Register |
| RTC_YY_MM_DD_REG | 0x10 | RTC Year-Month-Day Register |
| RTC_HH_MM_SS_REG | 0x14 | RTC Hour-Minute-Second Register |
| ALARM0_COUNTER_REG | 0x20 | Alarm 0 Counter Register |
| ALARM0_CUR_VLU_REG | 0x24 | Alarm 0 Counter Current Value Register |
| ALARM0_ENABLE_REG | 0x28 | Alarm 0 Enable Register |
| ALARM0_IRQ_EN | 0x2C | Alarm 0 IRQ Enable Register |
| ALARM0_IRQ_STA_REG | 0x30 | Alarm 0 IRQ Status Register |
| ALARM1_WK_HH_MM-SS | 0x40 | Alarm 1 Week HMS Register |
| ALARM1_ENABLE_REG | 0x44 | Alarm 1 Enable Register |
| ALARM1_IRQ_EN | 0x48 | Alarm 1 IRQ Enable Register |
| ALARM1_IRQ_STA_REG | 0x4C | Alarm 1 IRQ Status Register |
| ALARM_CONFIG_REG | 0x50 | Alarm Config Register |
| LOSC_OUT_GATING_REG | 0x60 | LOSC output gating Register |
| GP_DATA_REG | 0x100 + N*0x4 | General Purpose Register (N=0~3) |
| GPL_HOLD_OUTPUT_REG | 0x180 | GPL Hold Output Register |
| VDD_RTC_REG | 0x190 | VDD RTC Regulate Register |
| IC_CHARA_REG | 0x1F0 | IC Characteristic Register |
| CRY_CONFIG_REG | 0x210 | Crypt Configuration Register |
| CRY_KEY_REG | 0x214 | Crypt Key Register |
| CRY_CONFIG_REG | 0x218 | Crypt Enable Register |
