# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## リポジトリ概要

これはZMKファームウェアベースのカスタムキーボード「roBa」の設定リポジトリです。分割型キーボードで、左右それぞれSeeeduino XIAO BLEボードを使用しています。

## アーキテクチャ

### コアコンポーネント
- **build.yaml**: GitHub Actionsでのビルド設定。Seeeduino XIAO BLEボードとroBa_L/roBa_Rシールドの組み合わせを定義
- **config/roBa.keymap**: キーマップの中心となるファイル。7つのレイヤーとマクロ、カスタムビヘイビアを定義
- **config/boards/shields/Test/**: roBaキーボードのハードウェア定義
  - roBa.dtsi: 物理レイアウトとマトリックス変換の定義
  - roBa_L/R.overlay: 左右分割のピン配置
  - roBa_L/R.conf: 左右それぞれの設定（BLE、PMW3610センサー等）

### レイヤー構成
- **0: default_layer** - 基本QWERTY配列
- **1: FUNCTION** - ファンクションキーとショートカット
- **2: NUM** - 記号と特殊文字
- **3: ARROW** - 矢印キーと数字パッド
- **4: MOUSE** - マウスボタン（PMW3610センサー使用）
- **5: SCROLL** - スクロール機能
- **6: layer_6** - Bluetooth設定とブートローダー

## ビルドとデプロイ

### ローカルビルド
```bash
# west環境のセットアップが必要
west build -p -b seeeduino_xiao_ble -- -DSHIELD=roBa_R
west build -p -b seeeduino_xiao_ble -- -DSHIELD=roBa_L
```

### GitHub Actions
pushごとに自動ビルドが実行され、ファームウェアがArtifactsとして生成されます。

## 開発時の注意点

### キーマップ編集
- config/roBa.keymapを編集する際は、既存のマクロやビヘイビア定義を参照
- combos（同時押し）: TAB（0+1）、BACKSPACE（19+20）が定義済み
- カスタムビヘイビア: lt_to_layer_0（レイヤータップ）、tog_on/tog_off（トグル）

### ハードウェア設定変更
- 左側: config/boards/shields/Test/roBa_L.conf - エンコーダー有効
- 右側: config/boards/shields/Test/roBa_R.conf - PMW3610センサー有効、USB経由のZMK Studio RPC有効
- マトリックス配線: roBa.dtsiのkscan0定義を参照（4行11列）

### トラックボールモジュール（PMW3610） - 右側のみ
- **光学センサー**: PMW3610を中央に配置した基板設計
- **電源供給**: VCC/GND配線（+1V9の内部電圧レギュレーション）
- **通信インターフェース**: 
  - SPI接続（MISO/MOSI/SCLK/CS）でSeeeduino XIAOと通信
  - NCピン（MOTION）でモーション検出割り込み
- **基板特徴**:
  - 円形スルーホール×8個でボール支持構造を形成
  - 光学センサーの位置は"Optical Center"として基板中央に配置
  - 赤色（VCC/+1V9）と青色（GND）の電源ラインで明確な配線

### 依存関係
- ZMKメインブランチ
- zmk-pmw3610-driver（kumamuk-git/zmk-pmw3610-driver）

### テスト
ファームウェアの書き込み後、以下を確認:
1. 全キーの入力テスト
2. レイヤー切り替え動作
3. Bluetooth接続（左側がセントラル、右側がペリフェラル）
4. PMW3610センサーのマウス動作（左側のみ）