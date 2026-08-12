# UIAPduino HID Board Manager Files

Arduino IDE のボードマネージャー用インデックスファイルです。  
UIAPduino (HID ProMicro CH32V003) シリーズを Arduino IDE で使えるようにします。

---

## インストール方法

### 1. ボードマネージャーの URL を追加

Arduino IDE を開き、メニューから  
`ファイル` → `環境設定`（macOS: `Arduino IDE` → `設定`）を開きます。

「追加のボードマネージャのURL」欄に以下を入力して **OK** を押します:

```
https://github.com/tarosay/board_manager_files/raw/main/package_uiap_hid_index.json
```

### 2. ボードをインストール

`ツール` → `ボード` → `ボードマネージャ` を開きます。  
検索欄に `UIAPduino` と入力し、**UIAPduino HID** をインストールします。

### 3. ボードを選択

`ツール` → `ボード` → `UIAP_HID` → **HID ProMicro CH32V003** を選択します。

---

## 対応ボード

### HID ProMicro CH32V003

1 種類のボードに統合されています。  
`ツール` メニューで USB モード・UART・最適化レベルを選択します。

#### USB メニュー

| 設定 | 機能 |
|------|------|
| WebHID Only（デフォルト） | WebHID 双方向通信（Chrome / Edge 対応） |
| Keyboard+Mouse | USB キーボード ＋ マウス |
| Keyboard+Mouse+WebHID | USB キーボード ＋ マウス ＋ WebHID 双方向通信 |
| Terminal HID | HID シリアルターミナル |
| No USB (SD log / UART only) | USB スタックを除外（約 484B 節約）SD ログ・UART 専用 |

#### U(S)ART support メニュー

| 設定 | 機能 |
|------|------|
| None — use UIAPSerial（デフォルト） | 軽量 UART ラッパー。Flash 消費を最小に抑える |
| HardwareSerial (Serial / USART1) | Arduino 標準の `Serial` を使用（+約 4.7 KB） |

#### PWM メニュー（v1.2.6 以降）

| 設定 | TIM2 使用ピン |
|------|--------------|
| TIM2 Default（デフォルト） | pin 2 (PC0 / TIM2-CH3) |
| TIM2 Remap3 | pin 3 (PC1) / 9 (PC7) / 15 (PD5) / 16 (PD6) |

TIM1 ピン（pin 0 / 5 / 6 / 12）はどちらの設定でも使用できます。  
PWMmin ライブラリと組み合わせて使います。

#### Optimize メニュー

| 設定 | フラグ |
|------|--------|
| Smallest (-Os) with LTO（デフォルト） | `-Os -flto` |
| Fast (-O1) with LTO | `-O1 -flto` |
| Faster (-O2) with LTO | `-O2 -flto` |
| Fastest (-O3) with LTO | `-O3 -flto` |

---

## バージョン履歴

### 1.2.11（最新）

- **NeoPixelmin ライブラリを追加** — WS2812B（NeoPixel）を SPI1 で駆動する最小限のドライバ。波形を GPIO のサイクル数え打ちではなく SPI で生成するため、`show()` 中に割り込みを止める必要がない（Adafruit_NeoPixel の CH32 実装は 12 個で 7.7ms 割り込みを停止するため、ソフトウェア USB と両立しない）
- **LED 12 個で Flash 8,500 → 5,932 バイト（▲2,568）**、実 RAM 276 → 220 バイト。`begin` / `show` / `clear` / `setPixelColor` / `fill` / `setBrightness` / `Color` は Adafruit_NeoPixel と同じ意味で動く
- **DIN は pin 8（PC6 / SPI1 MOSI）固定** — `SPI.h` / `SDmin` とは併用不可。`NEOPIXELMIN_MAX_LEDS` は定義必須（未定義はコンパイルエラー）
- **スケッチ例 `NeoPixelmin_ring` を追加**

### 1.2.10

- **Wiremin に 16bit アドレス版の API を追加** — `Wiremin_write_reg16()` / `Wiremin_read_reg16()`。メモリアドレスが 16bit の I2C EEPROM（24FC256 など）を、スケッチ側でアドレス 2 バイトを組み立てずに読み書きできる
- **Wiremin のマスタ転送を 1 本の実装に統合** — スケッチ例 9 本の合計で ▲40 バイト。最も Flash が厳しい `Wiremin_bmi270` は 15,696 → 15,616 バイト（95%）。**挙動変更**: `Wiremin_read(addr, buf, 0)` の戻り値が `true` → `false`
- **スケッチ例 `Wiremin_EEPROM_24FC256` を追加** — 24FC256（32KB）の読み書きテスト。16bit アドレス、ページ境界をまたぐ分割、ACK ポーリングによる書き込み待ち、電源断をまたぐ保持までを実機で確認
- **README に「書き込み方法」を追記** — 基板のボタンを押しながら USB を接続してブートローダで起動し、そのまま IDE から書き込む手順（書き込み器は不要）

### 1.2.9

- **HardwareSerial に USART 受信割り込みとリングバッファを追加** — `Serial.available()` / `read()` / `peek()` が使えるようになった（既定 64 バイト、`SERIAL_RX_BUFFER_SIZE` で変更可）
- **`realloc()` が内容を保持しない問題を修正** — `String` の連結（`s += ...`）でバッファが伸びるたびに先頭がゴミになっていた。あわせて `free()` が領域を回収するようになり、`loop()` 内で `String` を使ってもヒープが枯渇しない
- **`dtostrf()` の修正** — `strdup` 未定義によるリンクエラー、小数の先頭ゼロ落ち（`1.05` → `"1.5"`）、負値の符号消失を解消。**挙動変更**: `prec == 0` で小数点を付けなくなった（avr-libc 準拠）
- **`pwm_start()` / `pwm_stop()` の NULL 参照を修正**
- **CH32V003 で `analogWrite()` を非推奨に** — 理由と代替を README に明記し、スケッチ例 `PwmAndToneTest` を PWMmin (`Pwm_write`) に書き換え

### 1.2.8

- **Zicsr 拡張の明示指定に対応** — `noInterrupts()` / `interrupts()` を使うライブラリでビルドが通らない問題を修正
  - 現行の RISC-V 仕様では `csrr` / `csrw` などの CSR 命令が `Zicsr` 拡張に分離されており、GCC 11 以降は `-march=rv32ec` のままだとアセンブラが認識できず `extension 'zicsr' required` エラーになっていた
  - 各 CSR 命令の直前に `.option arch, +zicsr` を挿入するマクロを追加。**`-march` は変更していない**ため、既存の割り込み設定・ABI・コード生成には影響なし
  - Adafruit NeoPixel の `show()` が `noInterrupts()` を呼ぶため表面化していた。CH32V003 + Adafruit NeoPixel 1.15.5 で動作確認済み
- 全スケッチ例 41 本のビルド確認済み（40/41 OK、SPIFlash のみ本家由来の既知エラーで従来からビルド不可）

### 1.2.7

- **フォーク元（YuukiUmeta-UIAP/arduino_core_ch32）の main をマージ** — openwch 本家の以下の修正を取り込み
  - **Print: `print(0)` が `"0"` を出力するように修正**（本家 #189）
  - **platform.txt: Arduino IDE 1.8.x 互換の修正** — minichlink の Windows パス区切り修正、`upload.params.verbose` / `upload.params.quiet` 追加
  - **PeripheralPins.c（CH32V003F4）: ADC1_IN6 / ADC1_IN7 のピン割り当てを PA6 / PA4 → PD6 / PD4 に修正**（本家 #160）
  - **tools/platformio-build.py 更新**（本家 #147）
- マージ後、メンテ対象の全スケッチ例 40 本のビルド確認済み（退行なし）

### 1.2.6

- **PWMmin ライブラリ追加** — CH32V003 専用の軽量 PWM ライブラリ（ヘッダーオンリー、未使用関数は LTO で自動削除）
  - `Pwm_write` / `Pwm_freq` / `Pwm_freq_TIM1` / `Pwm_freq_TIM2` / `Pwm_stop`
  - `Pwm_tone` / `Pwm_tone_update` — ノンブロッキング tone 相当
  - `Pwm_servo_begin` / `Pwm_servo` — サーボ制御（0〜180°）
  - 誤設定コンパイルエラーマクロ: `PWMMIN_REQUIRE_DEFAULT()` / `PWMMIN_REQUIRE_REMAP3()`
  - サンプルスケッチ 5 つ（Basic / Remap3 / Tone / Servo / ServoRemap3）
- **Tools > PWM メニュー追加** — TIM2 Default（pin 2）/ TIM2 Remap3（pin 9/15/16）を切り替え
- **USB VID/PID 注記追加** — `usb_config.h` と README に「開発・評価用」と明記。製造・配布・販売時は正規 VID/PID を設定するよう案内

### 1.2.5

- **SDmin: `sm_seek(pos)` 追加** — 読み取り位置を任意位置へ移動（ランダムアクセス読み）
- **SDmin: `sm_write_at(path, pos, buf, len)` 追加** — 既存ファイルの途中を部分上書き（1 セクタ内限定・サイズ拡張なし）
- **SDmin: SeekWriteAt サンプルスケッチ追加**

### 1.2.4

- **Wiremin ライブラリ追加** — Wire.h の代替となる最小 I2C ドライバ。Flash を **▲6,176 バイト**削減。BMI270（6軸 IMU）が 16KB Flash 内で動作確認済み（15,728 バイト）
- **HcSr04 ライブラリ追加** — HC-SR04 超音波距離センサ対応。`pulseIn` で ECHO パルス幅を計測し距離（cm）を算出。計測範囲 約 2〜400 cm

### 1.2.3

- **SDmin: `sm_open_a(path)` 追加** — 既存ファイルへの追記オープン（ファイルが存在しない場合は新規作成）
- **SDmin: `sm_sync_w()` 追加** — ファイルを開いたまま現在のセクタをフラッシュしディレクトリのファイルサイズを更新（電源断対策）
- **SDmin: SDLog サンプルスケッチ追加** — UART 受信データを microSD に記録する OpenLog 互換ロガー

### 1.2.2

- **SDmin: `sm_rmdir(path)` 追加** — 空ディレクトリを削除する関数を追加  
  `sm_del()` と共通実装 `_sm_del_entry()` に統合し、Flash 増加を約 20 バイトに最小化

### 1.2.1

- **Wire (I2C) ライブラリを完全動作させた**（マスター・スレーブともに v1.1.5 以前は完全に動作不可）
- **根本原因修正**: I2C ISR の `WCH-Interrupt-fast` 属性を標準割り込みに変更  
  `WCH-Interrupt-fast` は MIE=0 でも割り込みを発火させる WCH PFIC HPE 機構を使用しており、  
  rv003usb（ソフトウェア USB）のビットサンプリング処理を横取りして USB HID を切断していた
- **追加修正** — v1.2.0 で残っていた以下の不具合をすべて解消:
  - `variant_CH32V003F4.h` に `PIN_WIRE_SDA=PC1` / `PIN_WIRE_SCL=PC2` を追加  
    （未定義のまま `Wire.begin()` を呼ぶと誤ったピンを使いハングしていた）
  - `variant_CH32V003F4.h` で `I2C_MODULE_ENABLED` を有効化  
    （コメントアウトされていたため `PinMap_I2C_SDA/SCL` 未定義でリンクエラーが発生していた）
  - `boards.txt` に `-DCPLUSPLUS` フラグを追加  
    （`TwoWire` グローバルコンストラクタが動作せず `Wire.begin()` が確実に初期化されなかった）
  - `cores/arduino/uiapusb.c` に `GetTick()` オーバーライドを追加  
    （SysTick 割り込みなしで `millis()` と I2C タイムアウトを正常動作させる）
  - `Wire.h` / `Wire.cpp` に `begin(int, int)` オーバーロードを追加  
    （`Wire.begin(PC1, PC2)` が `begin(int addr, bool)` に誤解決されスレーブモードになっていた）
- **Wire examples 追加**: `i2c_scanner` / `i2c_slave_test` / `i2c_master_test` / `i2c_BMP280_test`  
  BMP280（温度・気圧センサ）読み取りサンプルを新規追加

> **v1.2.0 について**: v1.2.0 は Wire が初めて動作したバージョンですが、上記の追加修正が含まれておらず、
> 多くの環境で Wire が正常に動作しません。v1.2.1 へのアップグレードを強く推奨します。
> そのため、ボードマネージャーの選択肢から v1.2.0 は除外しています。

### 1.1.5

- **SDmin ライブラリを追加**
  - CH32V003 (16KB Flash) 向けに最適化した FAT32 ミニマル SD ドライバ
  - LFN・サブディレクトリ対応、SPI1 直接レジスタ制御で 6MHz 転送
  - Flash 使用率を 77% 以下に抑えつつ SD spec 準拠の動作を実現
- **WebHID Feature Report を 16 → 32 バイトに拡張**
  - `usb_config.h`: `HID_REPORT_COUNT` を 32 に変更（2 箇所）
  - `uiapusb.c`: `webhid_rx_buf` を 32 バイトに拡張（WebHID Only・複合モード両方）
- **`uiapwebhid_send()` にタイムアウトを追加（スタンドアロン動作修正）**
  - USB ホスト未接続時（モバイルバッテリー駆動など）に `while (webhid_tx_pending) {}` が
    無限ループしてフリーズする問題を修正
  - `while` → タイムアウト付き `for` ループに変更、ホスト接続時の動作は変わらない

### 1.1.3

- **USB デバイス名を修正（WebHID Only モード）**
  - `usb_config.h`: `STR_PRODUCT` を `"UIAPduino SD+WebHID"` → `"UIAPduino WebHID"` に変更
    （"SD" は開発時の内部コード名の名残で、公開名として不正確だった）
  - `STR_SERIAL` を `"SDHID000"` → `"WOHID000"` に変更
  - コメントも合わせて更新

### 1.1.2

- **`Tools > USB > WebHID Only` でのビルドエラーを修正**
  - `boards.txt`: WebHID Only モードに `-DUIAP_WEBHID_ONLY` フラグを追加
    （以前はフラグが空で `WebHID.h` インクルード時にコンパイルエラーになっていた）
  - `WebHID.h`: `#error` ガードを `UIAP_WEBHID_ONLY` にも対応させ、
    エラーメッセージを現行メニュー名に更新
  - `uiapusb.c`: WebHID-only デフォルト分岐のコメントを補強（保守ガイド追記）

### 1.1.1

- **USB メニューに「No USB」を追加**
  - `No USB (SD log / UART only)` — USB スタックを除外して約 484B Flash を節約
  - SD ログ・UART 専用スケッチ（UIAPLog など）向け

### 1.1.0

- **ボードを 1 種類に統合**
  - `HID ProMicro CH32V003`・`HID ProMicro CH32V003 KBD+Mouse` など 3 ボードを  
    `HID ProMicro CH32V003` 1 ボードに統合
  - FQBN: `UIAP_HID:ch32v:CH32V003:opt=oslto`
- **USB メニューを追加**
  - WebHID Only（デフォルト）/ Keyboard+Mouse / Keyboard+Mouse+WebHID / Terminal HID
  - デフォルトを Terminal HID → **WebHID Only** に変更
- **U(S)ART support メニューを追加**
  - None/UIAPSerial（デフォルト）/ HardwareSerial を切り替え可能に
- **Optimize メニューを整理**
  - LTO なしオプションと `-Og` / `-O0` を削除
  - すべての最適化レベルで LTO (`-flto`) を有効化

### 1.0.4

- **Tone ライブラリ**を追加
  - `tone(pin, freq)` がハードウェア PWM による真のノンブロッキング無限再生に対応
  - `tone(pin, freq, duration)` で指定時間だけ鳴らして自動停止
  - タイマーチャネルがないピンはソフトウェアビットバンフォールバック
  - スケッチ例: ToneBasic / ToneDuration / ToneNoTone

### 1.0.3

- **Mouse.moveLarge()** を追加
  - `-127〜127` の制限を超える大きなマウス移動を複数ステップに分割して送信
  - `Mouse.moveLarge(x, y, wheel, steps)` — wheel・steps は省略可能

### 1.0.2

- **Keyboard.write()** の信頼性を改善
  - 特殊キー（矢印・BackSpace・Enter など）の取りこぼしを修正
  - press/release 間の待機時間を調整し USB ポーリングに確実に乗せるよう改善
- スケッチ例を追加: KeyboardPractice / KeyboardSwitch

### 1.0.1

- **WebHID** の送受信を 16 バイト対応に拡張
- EP3 の余分なポーリング送信を抑制（NAK で無送信）

### 1.0.0

- **HID ProMicro CH32V003 KBD+Mouse** ボードを追加
- `Keyboard` / `Mouse` ライブラリを追加
- **WebHID ライブラリ**を追加（EP3 双方向通信、Chrome / Edge 対応）
- Board Version Select メニューで V1.4 / V1.4 + WebHID を切り替え可能に

### 0.1.1

- HID ProMicro CH32V003 の安定版

### 0.1.0

- 初回リリース

---

## 関連リンク

| リンク | 内容 |
|-------|------|
| [arduino_core_ch32](https://github.com/tarosay/arduino_core_ch32) | Arduino コア本体・ライブラリ・サンプル |
| [GitHub Issues](https://github.com/tarosay/arduino_core_ch32/issues/new) | バグ報告・要望 |

---

## 対応 OS

| OS | 状況 |
|----|------|
| Windows | 動作確認済み（Arduino IDE 2.0 以上） |
| Linux | 動作確認中 |
| macOS | 動作確認中 |
