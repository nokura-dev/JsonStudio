# JsonStudio デスクトップ版

Neutralinojsで、ミニファイ済みのJsonStudioをデスクトップアプリとして実行する構成です。Windows向けEXEとMac向けアプリの生成に対応します。

## 起動

この`neu`フォルダで実行します。

```sh
neu run
```

設定を変更した後は、起動中のアプリを終了して再実行してください。

## ファイル構成

| ファイル | 用途 |
| --- | --- |
| `neutralino.config.json` | アプリ名、タイトル、サイズ、アイコン、API権限、事前読み込みの設定 |
| `resources/index.html` | ミニファイ済みHTMLのコピー |
| `resources/js/desktop-adapter.js` | ネイティブ操作と既存のJsonStudioを接続 |
| `resources/icons/jsonstudio.png` | ウィンドウ・Windows EXEのアイコン |
| `assets/icons/candidates/` | アイコン候補と生成プロンプト |
| `bin/` | Neutralinojs実行ファイル |
| `dist/` | ビルド結果 |
| `build_common.py` | 共通のビルド入力・配布設定 |
| `build-windows.py` | Windows向けEXEの生成・検証 |
| `build-macos.py` | Mac向けアプリの生成・検証 |

`injectClientLibrary`と`injectScript`でNeutralinojsのクライアントと接続コードを読み込みます。`resources/index.html`にscriptタグを追加する必要はありません。テンプレートの`resources/js/main.js`は使用しません。

## 接続している機能

- フォルダ選択と、直下のJSONファイル一覧取得
- JSONの読み込み・作成・保存・名前変更・削除
- 自動保存、外部アプリによる変更の検出、設定ファイルの保存
- 2種類のCSV読み込みで使用するファイル選択
- SQL・CSV・PNGの保存先選択とファイル保存
- SQL・CSV全文のクリップボードへのコピー
- 終了時の未保存確認と、保存処理中の終了防止

Macではアプリケーションメニューの「JsonStudioを終了」または`⌘Q`、Windowsでは「ファイル → 終了」から終了できます。ウィンドウを閉じる操作と同じ未保存確認を行い、保存処理中は終了を中止します。Macにはテキスト編集用の「編集」メニューも用意します。

フォルダ操作には、ブラウザのFile System Access APIと互換性のあるハンドルを用意し、既存の保存処理へ接続しています。APIの利用許可と、OSの読み書き権限は別です。実際にアクセスできない場所では操作がエラーになります。

JSONと出力ファイルは、同じフォルダの一時ファイルへ書き込んでから置き換えます。書き込み・置き換えに失敗した場合は元ファイルを削除せず、一時ファイルを削除します。CSVはUTF-8・BOM付き・CRLF、SQLはUTF-8・BOMなし・CRLF、PNGはバイナリのまま保存します。

## 動作確認

テスト用フォルダにJSONをコピーして、次の順で確認してください。

1. ウィンドウタイトル、最大化、アイコンを確認する。
2. 「フォルダを開く」でテスト用フォルダを選び、一覧を確認する。ドットで始まるファイルとサブフォルダ内のファイルは対象外。
3. ファイルを選択し、テーブル定義・データ・JSONエディタを確認する。
4. 編集して保存し、再読み込み後も内容が残っていることを確認する。
5. 新規作成・名前変更・削除を確認する。
6. CSV読み込みの正常・エラー・キャンセルを確認する。
7. SQL・CSV・PNGの保存と、クリップボードへのコピーを確認する。
8. 未保存の編集がある状態でウィンドウを閉じ、終了のキャンセルを確認する。

## Windows向けビルド

この`neu`フォルダで、次のコマンドを実行します。

```sh
python3 build-windows.py
```

インストール済みの`neu`をPATHまたはMacのnvm配下から探します。見つからない場合は`--neu /path/to/bin/neu`で指定できます。

開発設定を保ったまま、一時的なビルド用フォルダでWindows x64バイナリへリソースを埋め込みます。配布版では開発者ツールとログを無効にし、テンプレートのサンプルコード・テスト変数・認証情報・作業中のJSONは含めません。ミニファイ済みHTMLは再生成せず、`resources/index.html`をそのまま使用します。

| 生成物 | 用途 |
| --- | --- |
| `dist/windows/JsonStudio.exe` | 単独実行ファイル |
| `dist/JsonStudio-windows-x64.zip` | EXE・利用説明・Neutralinojsライセンス・検証情報をまとめた配布用ZIP |
| `dist/windows/build-info.json` | バージョン、検証結果、SHA-256 |

ビルド時にWindows x64形式、埋め込みリソースの一致、アイコン、製品名、タイトル、開発者ツール・ログの無効化を検証します。Windows上での起動・操作の確認は別途必要です。

WindowsではZIPを展開し、`JsonStudio.exe`を実行して上記の操作を確認してください。[Microsoft Edge WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/)が必要です。ランタイムはEXEに同梱していません。

基になる公式CLIコマンドは`neu build --embed-resources`です。`build-windows.py`はこれをビルド用フォルダで実行し、検証・配布ファイルの作成まで行います。

## Mac向けビルド

Mac上で、この`neu`フォルダから実行します。

```sh
python3 build-macos.py
```

| 生成物 | 用途 |
| --- | --- |
| `dist/macos/JsonStudio.app` | Apple Silicon・Intel両対応のUniversalアプリ |
| `dist/JsonStudio-macos-universal.zip` | アプリをまとめた配布用ZIP |
| `dist/macos/build-info.json` | バージョン、署名方式、SHA-256 |

ZIPを展開して`JsonStudio.app`を開きます。必要に応じてアプリケーションフォルダへコピーしてください。アプリケーションメニューの名前は`JsonStudio`です。

既存のミニファイ済みHTMLをそのまま埋め込み、アイコンとInfo.plistを設定します。配布版では開発者ツールとログを無効にし、Neutralinojsのデータ保存先をアプリ外のユーザー用データ領域に設定します。ビルド時に両アーキテクチャ、埋め込みリソース、アイコン、署名を検証します。

署名はローカル実行用のアドホック署名です。Apple Developer IDによる署名・公証は行っていないため、他のMacへ配布した場合、macOSのセキュリティ確認が必要になる場合があります。

## 自動テスト

プロジェクト直下で実行します。ネイティブAPIとDOMを模擬し、実際のユーザーファイルは変更しません。

```sh
/System/Library/Frameworks/JavaScriptCore.framework/Versions/A/Helpers/jsc tests/desktop-adapter.test.js
```

テスト対象は、読み込みとフィルター、保存失敗時の保持、一時ファイルの後片付け、外部変更の検出、Windows形式のパス、出力形式、CSV選択、クリップボードと終了処理です。

## テンプレートについて

Neutralinojsの初期テンプレートを使用しています。以下はテンプレート由来のライセンス・アイコンクレジットです。

## License

[MIT](LICENSE)

## Icon credits

- `trayIcon.png` - Made by [Freepik](https://www.freepik.com) and downloaded from [Flaticon](https://www.flaticon.com)
