## Pine64 Allwinnerファームウェアの起動プロセス

[オリジナル](https://github.com/apritzel/pine64/blob/master/Booting.md)

ほとんどのAllwinner SoCでは、リセットされると最初のARMコアがアドレス0に
マッピングされているSoC内蔵のマスクROMのコードの実行を開始します。
このコードはPine64にSDカードがないというFEL条件をチェックします。
FEL条件を満たした場合はUSB-OTGポート（USBの上段ソケット）からの要求を
待ちます。FEL条件が満たされなかった場合、ROMコードはSDカードのセクタ16
から32KBをSRAMのある場所（TODO: どこ？）にロードし、そのコードを実行します。

Allwinnerが提供するバージョンのSPL（boot0と呼ばれる）はSoCをさらに
初期化します。その中で最も興味深いのはDRAMコントローラーの設定です。
その後、SDカードからさらにデータをロードします。

* セクタ38,192 (19,096KByte) からU-BootバイナリをDRAMにロードします
  （おそらくアドレス 0x4a00_0000に）。このブロブのサイズはこのアドレスに
  あるヘッダーにあり、チェックサムも含まれています。既存のboot0は期待される
  チェックサムと計算されたチェックサムを提示するので必要に応じてハック
  することができます。

* BL3-1と呼ばれるARM Trusted Firmware (ATF) ベースのイメージが
  セクタ39,664 (19,832KByte) からアドレス 0x4000_0000（DRAMの先頭）に
  ロードされます。これはPSCIファームウェアのランタイム部分を含んでいる
  ことが最も重要であり、後でAArch64のEL3モードで実行されます。カーネルに
  SMPサポート（`CPU_ON`と`CPU_OFF`）を提供する他に、ボードのリセット機能と
  シャットダウン機能も含まれています。

* SoC搭載の管理コントローラ（arisc、OpenRISCベースのCPUコア）用のSCP
  ファームウェアはセクタ39,752 (19,876 KByte) からアドレス 0x40_000の
  SRAMにロードされます（メモリマップには記載されていない？）。 この
  ファームウェアはARMコアとは独立に動作し、電源管理IC（PMIC）を制御して
  いるようです。おそらく、ARMコアがディープスリープ状態になっている間、
  システム（バッテリーと充電）を監視していると思われます。また、OS
  （またはU-Boot）からコマンドを送信して実行させることもできます。
  どうやらこれは、PMICが接続されているRSBバスへのアクセスをラップして
  いるだけで、現時点ではそれ以上の抽象化は提供されていないようです。

* ちょうど20MB（セクタ40,960）には、U-Boot（とおそらくはAndroid）が
  使用する名前付きパーティションのパーティションテーブルがあります。
  これはAllwinnersのNANDパーティションスキームに従っており
  [sunxi-tools](http://linux-sunxi.org/Sunxi-tools)のコードに文書化
  されています。このパーティションテーブルはちょうど64KByteを占め、
  実際のテーブルの4つのチェックサム付きのコピーが含まれています。
  このテーブルのオフセットはテーブルの先頭からの相対値なので、
  パーティションの実際の開始アドレスを得るには20MB（すなわち、40,960
  セクタ）を足します。

U-Bootバイナリの前にあるヘッダーはATFとSCPのブロブもカバーしているので、
ブートログは個別にロードしていると主張していますが、ほとんどの場合は
これら2つはU-Bootと一緒にロードされます。

すべてのファームウェアがロードされると、BL3-1イメージに制御が移り、
CPUを（再度）初期化して、U-Bootに制御を渡します。これらすべては32ビット
モードで行われます。A64 SoCはAArch32モードで開始するようにハードワイヤ
されているからです。そのため、BROMとboot0はどちらも32ビットARMコードです。
その後、U-Boot（完全に32ビットのままです）が引き継ぎ、タイムアウト前に
中断されなければ、デフォルトの"bootcmd"環境変数のコマンドを実行を続けます。