# キーボードショートカットで Claude Code を起動する方法

VSDCraft（マクロキーボード設定ソフト）と Mac の Automator を組み合わせて、ワンボタンで Claude Code を起動するセットアップガイドです。

## 概要

VSDCraft の「マクロ（キーストロークの連続入力）」機能で直接キー入力を送る方法もありますが、アプリの起動待ち時間（ラグ）によって入力漏れが発生しやすい問題があります。

そのため、**Automator で「一連の動作を行うアプリ」を作成し、VSDCraft のボタンでそのアプリを起動させる**方法を採用しています。

## ステップ 1：Automator で起動用アプリを作成する

1. Mac の「アプリケーション」フォルダ、または Spotlight 検索（`Cmd + Space`）から **Automator** を開く
2. 「新規書類」をクリックし、書類の種類として **「アプリケーション」** を選択して「選択」を押す
3. 左側の検索窓に `AppleScript` と入力し、**「AppleScript を実行」** アクションを右側のスペースにドラッグ＆ドロップ
4. 表示された入力欄に以下のコードを貼り付ける：

```applescript
on run {input, parameters}
    tell application "iTerm"
        activate
        if not (exists window 1) then
            create window with default profile
        end if
        tell current session of current window
            write text "cd ~/ws/MHoldings/skyjob && claude"
        end tell
    end tell
    return input
end run
```

> **Note:** `cd` のパスは自分のプロジェクトディレクトリに合わせて変更してください。

5. メニューバーの「ファイル」>「保存」をクリックし、名前を付けて保存する（例：`Launch_Claude.app`）。保存場所は「アプリケーション」フォルダが分かりやすい。

## ステップ 2：VSDCraft でボタンに割り当てる

1. **VSDCraft** アプリを起動し、設定したいボタンを選択
2. アクションの選択メニューから、**「ファイルを開く」** や **「アプリケーションの起動」** を選択
3. 実行するファイルとして、ステップ 1 で作成した `Launch_Claude.app` を選択
4. 必要に応じてアイコンを Claude のロゴなどに変更し、設定を保存（同期）

## 初回起動時の注意点

VSDCraft のボタンを初めて押したとき、Mac のセキュリティ機能により以下の警告が出ることがあります：

> "Launch_Claude.app" が "iTerm" を制御しようとしています

**「OK」または「許可」** をクリックしてください。2 回目以降はスムーズに立ち上がります。

## 動作仕様

- iTerm2 が**すでに開いている場合**：現在のウィンドウ（または新しいウィンドウ）で実行
- iTerm2 が**閉じている場合**：新しく起動して実行

## カスタマイズ例

常に新しいタブを開いて実行したい場合は、AppleScript を以下のように書き換えます：

```applescript
on run {input, parameters}
    tell application "iTerm"
        activate
        tell current window
            create tab with default profile
            tell current session
                write text "cd ~/ws/MHoldings/skyjob && claude"
            end tell
        end tell
    end tell
    return input
end run
```
