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

`ツール` → `ボード` → `UIAP_HID` → 使用するボードを選択します。

---

## 対応ボード

### HID ProMicro CH32V003

USB HID 経由で PC とデータを送受信する汎用ボードです。  
`HIDuiap` ライブラリを使って PC との通信ができます。

### HID ProMicro CH32V003 KBD+Mouse

USB キーボード・マウスとして PC を操作するボードです。  
`Tools` → `Board Version Select` で機能を選択します。

| Board Version | 機能 |
|--------------|------|
| V1.4 | キーボード ＋ マウス |
| V1.4 + WebHID (EP3) | キーボード ＋ マウス ＋ WebHID 双方向通信 |

WebHID を選択すると、Chrome / Edge の WebHID API を使って  
ブラウザと UIAPduino がリアルタイムで双方向通信できます。

---

## バージョン履歴

### 1.0.0（最新）

- **HID ProMicro CH32V003 KBD+Mouse** ボードを追加
- `Keyboard` / `Mouse` ライブラリを追加
- **WebHID ライブラリ**を追加（EP3 双方向通信、Chrome / Edge 対応）
- Board Version Select メニューで V1.4 / V1.4 + WebHID を切り替え可能に
- USB デバイス名: `UIAPduino KBD+Mouse` / `UIAPduino KBD+Mouse+Web`

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
