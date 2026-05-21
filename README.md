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

#### Optimize メニュー

| 設定 | フラグ |
|------|--------|
| Smallest (-Os) with LTO（デフォルト） | `-Os -flto` |
| Fast (-O1) with LTO | `-O1 -flto` |
| Faster (-O2) with LTO | `-O2 -flto` |
| Fastest (-O3) with LTO | `-O3 -flto` |

---

## バージョン履歴

### 1.2.2（最新）

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
