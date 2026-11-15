# JsonStudio — 開発者向けガイド

JsonStudioは、ローカルのJSONでテーブル定義とデータを編集し、CSV変換、PostgreSQL向けDDL生成、ER図による参照編集、SELECTによるデータ抽出を行うアプリケーションです。ブラウザー版とNeutralinojsによるWindows・macOS版で、同じ画面と編集処理を使います。データベース接続やデータのサーバアップロードは行いません。

このREADMEは開発・保守・検証・配布のための資料です。利用者向けの操作説明は、アプリ右上の3本線メニュー →「ヘルプ」にまとめています。「設定」は未実装で選択できません。

## GitHub掲載物と開発用ソース

[GitHubの公開リポジトリ](https://github.com/nokura-dev/JsonStudio)に掲載するアプリケーションは、**ミニファイ済みの単独HTMLと、ビルド済みのデスクトップアプリ**です。開発用ソースをそのまま配置したものではありません。

| GitHubルートの掲載物 | 内容 |
| --- | --- |
| `index.html` | 開発用HTML・JavaScript・CSSを結合・ミニファイした単独HTML。追加の`js/`・`css/`なしで動作 |
| `JsonStudio-windows-x64.zip` | ビルド済みWindows x64版EXEと配布用の説明・ライセンス・検証情報 |
| `JsonStudio-macos-universal.zip` | ビルド済みmacOS Universal版APP（Apple Silicon・Intel対応） |
| `README.md` | この開発者向け資料のコピー |

以下のパスとコマンドは、開発用ファイル一式があるローカル作業環境を前提にしています。GitHub掲載の単独HTMLを取得するだけでは、ここに記載したソース編集やテスト・再ビルドはできません。掲載済みの既存`neu/`等が残っていても、このローカル作業環境一式と同じ内容であるとは限りません。

## 開発環境と起動

- ブラウザー版：`showDirectoryPicker`などのFile System Access APIを使えるPC版EdgeまたはChrome。安全なコンテキストとフォルダへの読み書き許可が必要です。
- 単体テスト：Node.js。追加のnpmパッケージは不要です。
- ブラウザー統合テスト：Python 3とChromeまたは互換ブラウザー。
- 単独HTML生成：Python 3と`tools/requirements-minify.txt`の依存パッケージ。
- デスクトップ開発：Neutralinojs CLI（`neu`）。macOS版ビルドにはmacOSと署名などに使うシステムツールが必要です。

開発用のプロジェクト直下の`index.html`をブラウザーで開きます。`js/`と`css/`は相対パスで参照されるため、同じディレクトリー構成を保ってください。通常のscriptタグを使っており、npmインストールやJavaScriptのバンドルは不要です。ローカルHTTPで確認する場合も、File System Access APIを利用できるコンテキストで開いてください。

編集動作の確認には、実データから独立した作業フォルダを選びます。画面の編集・参照編集は選択フォルダへ直接保存されます。デスクトップ開発版は`neu/`で`neu run`を実行しますが、読み込むのは`neu/resources/index.html`です。ルートのソース変更は、HTMLの再生成とコピーを行うまでデスクトップ版へ反映されません。

## ディレクトリー構成

| パス | 役割 |
| --- | --- |
| `index.html` | 開発用の画面構造、利用者向けヘルプ、script・CSSの読み込み順 |
| `js/` | アプリケーション処理。各ファイルは`window.JsonStudio`へ機能を登録 |
| `css/` | テーマ、共通部品、各画面のスタイル |
| `tests/` | DOM・ファイル操作を模擬する単体テストとブラウザー統合テスト |
| `tools/` | 手動ミニファイとブラウザーテストのランナー |
| `json/テスト/` | 5業種のJSON・CSVサンプル。使い方と件数は同フォルダのREADMEを参照 |
| `neu/` | Neutralinojsの設定、接続コード、ビルドスクリプト |
| `neu/resources/index.html` | デスクトップへ埋め込む生成HTMLのコピー |
| `neu/dist/` | 単独HTML、READMEのコピー、EXE・APP・配布ZIPなどの生成物 |
| `HISTORY.md` | 日本語のローカル作業履歴と最終掲載チェックポイント。公開対象外 |
| `AGENTS.md` | 生成物・ビルド・履歴・GitHub掲載についての作業規約 |
| `work/` | 調査記録や提案資料。実装済みの仕様とは区別 |

## モジュールの責務

| 領域 | 主なファイルと役割 |
| --- | --- |
| 起動・状態・永続化 | `base.js`：画面連携と保存の調停、`state.js`：選択・ドラフト・保存状態、`filesystem.js`：ファイルアクセスと外部変更検出 |
| メイン画面 | `titlebar.js`、`navigator.js`、`workspace.js`、`buttons.js`、`statusbar.js`：メニュー、一覧検索、概要・表描画、活性条件、通知 |
| テーブル編集 | `designer.js`：操作と保存連携、`designer-document.js`：JSONの変更と値変換、`designer-suggestions.js`：入力候補、`designer-headers.js`：固定ヘッダー、`designer-rows.js`：行ドラッグ、`datagrid-layout.js`：列幅調整 |
| JSONエディタ | `editor.js`：入力・パンくず・折りたたみ、`editor-search.js`：検索、`json-analysis.js`：構文色とエラー解析、`json-locations.js`：要素の位置対応 |
| 共通処理 | `utils.js`：DOM・エスケープ・コピー・出力・名前検査、`table.js`：解析・検証・集計・整形、`dialogs.js`・`modals.js`：確認画面、`icons.js`：動的ボタンのSVG定義・生成 |
| CSV | `csv-parser.js`：解析・推定・型変換、`csv-import.js`：選択・プレビュー・保存、`csv.js`：CSV生成・出力 |
| DDL | `ddl.js`：PostgreSQL向け生成・診断、`ddl-values.js`：型検査とSQL値の表現、`sql-highlight.js`：DDLとSELECTの構文表示 |
| ER図 | `er.js`：操作と保存・PNG出力、`er-model.js`：モデル・寸法・SVG、`er-layout.js`：参照関係を考慮した自動配置 |
| SQLエンジン | `sql-syntax.js`：トークン化・構文木・条件整形、`sql-runtime.js`：式評価・同期/非同期実行、`sql-engine.js`：カタログとの照合・生成・グリッド変換と従来API |
| 名称・保存 | `names.js`：従来APIの窓口、`name-transform.js`：名称差分・SQL/モデル変換、`name-persistence.js`：診断・連鎖保存と失敗時の復旧 |
| バックアップ | `backup.js`：対象収集・日付保持・定期実行、`zip.js`：DEFLATE ZIP生成・CRC32 |
| SQL画面 | `sql-designer.js`：セッション・編集履歴・保存、`sql-store.js`：設定の読み書きと外部変更検出、`sql-dialog.js`：SQL名入力・確認と応答の管理 |
| SQL表示 | `sql-panels.js`：分割、`sql-diagram.js`：カード・線、`sql-grid.js`：抽出条件・列幅、`sql-results.js`：結果の仮想スクロール |
| レイアウト | `layout.js`：メイン分割、`zoom.js`：倍率と座標補正、`interactions.js`：編集領域の移動監視 |

上表のJavaScriptはすべて`js/`配下です。読み込み順は`index.html`を正とし、最後に`base.js`が初期化します。DOMを使わない解析・モデル・ドキュメント操作と、表示・入力・保存の調停を分けています。SQL表示モジュールは`create()`で組み立て、現在のモデルを返す関数などを受け取ります。SQL切り替えやUndoでモデルが置き換わるため、古いオブジェクトを保持し続けないようにします。

CSSは`theme.css`の共通配色、`syntax.css`の構文色、`base.css`・`modals.css`・`dialogs.css`などの共通部品、`previews.css`のDDL/CSV出力・取込プレビュー、`designer-layout.css`のテーブル配置・固定ヘッダー・スクロール、各画面用ファイルに分かれます。通常ERとSQL図は`er-model.js`のカード寸法を共有します。サイズ監視からの再描画は次のアニメーションフレームへ送り、監視中の連続再描画を避けています。

SQLエンジンは`sql-syntax.js` → `sql-runtime.js` → `sql-engine.js`の順で読み込みます。構文解析と実行はDOMやカタログ生成に依存せず、`sql-engine.js`が従来の`app.sqlEngine` APIを公開します。同期・非同期の実行は同じジェネレーターを使い、行数・比較回数の上限とSQLエラー型を共有します。

DDLプレビューとSQLエディタは`sql-highlight.js`の安全なHTML生成を共有します。DDLの型名表示とSQLエディタの行ごとの表示は別のトークン規則として維持しています。動的モーダルボタンのSVGパスは`icons.js`にまとめ、コピー・保存・チェック・中止・ER配置・PNG出力で採用済みの形を参照します。画面内の静的SVGは引き続き`index.html`にあります。

## JSONデータ形式

新規作成する標準形式は、物理テーブル名`table`、カラム定義`cols`、レコード`data`を持つオブジェクトです。

```json
{
    "table": "orders",
    "cols": [
        { "column": "id", "type": "integer", "pk": true, "not null": true },
        { "column": "customer_id", "type": "integer", "reftable": "顧客", "refcolumn": "id" },
        { "column": "amount", "type": "numeric(10,2)", "default": 0, "comment": "受注金額" }
    ],
    "data": [
        { "id": 1, "customer_id": 101, "amount": 1500 }
    ]
}
```

| フィールド | 意味・注意点 |
| --- | --- |
| `table` | DDL／SELECTで使う物理テーブル名 |
| `cols[].column` / `type` | カラム名と型。新規行の型は`string` |
| `pk` / `not null` | 主キーとNULL不可。主キーON時はNULL不可もON。主キーOFFだけではNULL不可を解除しない |
| `default` | JSONの値として保持する初期値。行追加では式を評価せず値を複製 |
| `reftable` / `refcolumn` | 参照先JSONの物理テーブル名（table値）と物理カラム名（column値）。自己参照は禁止 |
| `comment` | 論理カラム名と呼称するコメント。DDLのCOMMENT ON COLUMNへ出力し、画面の名称ラベルには使用しない |
| `data` | レコードオブジェクトの配列。キー省略、`null`、空文字を区別 |

JSONファイル名（拡張子なし）は論理名として画面表示に使い、DDL／SELECT・参照の識別子には物理名を使います。ファイル名だけの変更ではtable値と物理名のSQLは変わりません。table欠落・null・空文字・空白のみの標準形式JSONを選択すると、論理名で補完して即時保存します。補完値の重複・自己参照があればエラー行を示し、保存を拒否してドラフトを残します。旧形式tableオブジェクトは上書きしません。

名称変更は`js/names.js`を共通の窓口とし、`name-transform.js`が状態やファイル操作に依存しない差分・SQL/モデル変換、`name-persistence.js`が状態を使う診断・連鎖保存・復旧を担当します。入力確定後の保存時に検査します。table重複・自己参照は保存拒否。table変更では保存SQLの本文・モデルと参照元reftable、column変更ではdataキー・保存SQLと該当テーブルのrefcolumnを連鎖更新します。JSON原文の改名は、同じ配列位置で消えた旧名と新規名を対応付ける方式のため、複数列の追加削除・並べ替えを伴う編集では対応を推定できない場合があります。SQLの文字列・コメント・明示AS別名は保持します。対象名を含む未完成／未対応SQLを安全に解析できなければ保存を中止します。

関連ファイルは書込前に外部変更を検査し、途中失敗は変更前へ復元します。復元失敗時は同フォルダの`.jsonstudio-recovery-時刻.json`へ変更前後の情報を保存します。フォルダへ書けない場合は同名ファイルをダウンロードします。復元情報は自動適用せず、内容と現在のファイルを照合して復旧してください。失敗時のドラフトは保持します。開いているSQLの編集中内容・保存基準を同期し、旧名を復活させないようSQLのUndo/Redoを消去します。配置・列幅ファイルの名称キーは自動移行しません。

DDL生成時だけtableが欠落していた場合は、警告して出力内だけ論理名を使用します（JSON保存なし）。外部キーreftableは記載された物理名を直接出力します。SQL一覧では物理名欠落テーブルも論理名で表示しますが選択不可です。参照候補は物理名で、欠落候補は論理名表示・選択不可、自己テーブルも選択不可です。旧論理名reftableは自動移行しないため、物理名が異なる既存JSONは修正が必要です。

旧形式の`table.cols`・`colName`・`primaryKey`も編集用の読み込みで扱いますが、DDL生成は標準形式を前提とします。`table.js`の検証と`designer-document.js`の編集用読み込みは目的が異なり、編集中の不完全な定義を扱えることと、DDL・SQLで利用できることを同一視しないでください。

セルを空欄にするとキーを削除します。`null`はNULL、`"null"`は文字列、`""`は明示的な空文字です。行追加は定義にある初期値を複製し、未定義の列のキーは作りません。カラム名変更は既存データのキーにも反映しますが、定義行の削除ではデータのキーを残します。JSONの整形・設定保存は空白4文字と末尾改行を使います。

## 編集状態と保存

- `state.editorDrafts`が編集中テキストを保持し、`state.textFor(entry)`がドラフトを優先します。保存済みテキストは`entry.text`に残します。
- 通常保存とCSV追加は`base.js`の`persistEntry()`を共用します。書き込み成功後に解析結果と保存基準を更新し、対象のドラフトだけを解除します。保存中に生じた新しい編集は残します。
- 表は行外へのフォーカス移動、エディタは領域外のクリックで自動保存します。行追加・削除・並べ替え・主キー変更は即時保存です。構文エラーがあるJSONは保存・整形できません。
- `filesystem.saveFile()`は現在のファイル本文と`entry.text`を比較し、外部変更時は上書きを中止します。`sql-store.js`もSQL設定・レイアウトの基準テキストを比較します。
- JSON構文検査・パンくず・検索の入力待機は約200msです。構文エラーは最大200行を表示します。折りたたみ表示は保存する本文と分離します。
- デザイナーの操作箇所は`json-locations.js`とエディタの`revealLocation()`で対応付け、薄緑で表示します。エディタで手動操作すると解除します。
- ボタンの活性条件は`buttons.js`、処理中のロックは`buttons.lock()`で管理します。保存失敗時は編集を保持し、処理終了後に再試行できる状態へ戻します。

フォルダ一覧の対象は選択フォルダ直下のドットで始まらない`.json`です。サブフォルダは走査しません。一覧の再読み込みはJSONドラフトの保存を先に行い、保存失敗時は中止します。削除操作に連動した個別バックアップは作成しません。

### 自動バックアップ

- フォルダを開いた時点と、以降10分ごと（前回処理完了から）にバックアップします。再読み込みボタンでは追加のバックアップを行いません。
- 対象は選択フォルダ直下に保存済みの全JSONです。先頭ドットの設定・復元ファイルも含みます。サブフォルダ内のファイルと未保存の編集内容は含みません。
- 保存先は選択フォルダ内の`backup`サブフォルダです。存在しなければ作成し、`backup_YYYYMMDDHHmmss.zip`の名前で圧縮保存します。日時は端末のローカル日時で、同秒の既存ZIPは上書きしません。
- 新しいZIPの保存成功後、ファイル名の`YYYYMMDD`が今日より4日以上前のバックアップを自動削除します。例えば11月15日なら11月11日以前が対象です。正しい形式・有効な日時のバックアップ名を持つファイルだけを削除し、他の名前のファイルやフォルダは削除しません。
- フォルダを切り替えると、進行中のバックアップ完了後、新しいフォルダだけを定期保存します。アプリ／ページを閉じている間は動作せず、ブラウザー休止中は実行が遅れる場合があります。
- 失敗はステータス欄へ表示し、次回の定期実行で再試行します。ZIP保存に失敗した場合は古いバックアップを削除しません。

復元する場合は、ZIPを別フォルダへ展開して内容を確認し、必要なJSONを作業フォルダへ戻してから開き直してください。設定も戻す場合は、先頭ドットの非表示ファイルを含めてコピーしてください。自動復元機能はありません。

`backup.js`は対象収集・ZIP保存・保持期限・定期実行を担当し、バイナリ形式の構築は`zip.js`に分離しています。CRC32の変換表は初回に作成して共有し、ファイルごとの計算ではバイト単位に参照します。バックアップの公開API・タイマー間隔・日付規則は維持しています。

### フォルダ内の設定ファイル

| ファイル | 保存内容・契機 |
| --- | --- |
| `.jsonstudio-layout.json` | ナビゲーター幅、デザイナーとエディタの分割。自動保存 |
| `.jsonstudio-er-layout.json` | 通常ER図のカード位置・サイズ。自動保存 |
| `.jsonstudio-sql.json` | SQL一覧。`version: 1`、`queries`に`id`・`name`・`sql`・`model` |
| `.jsonstudio-sql-layout.json` | `version: 1`、共通のペイン配分、SQL IDごとのカード・抽出条件列・結果列の幅。自動保存 |

設定ファイルは一覧・件数集計から除外します。SQL本文・テーブル選択・抽出条件は「SQL保存」で全SQLを一括保存します。SQL一覧の新規作成・名称変更・コピー・削除でも全SQLを保存するため、別のSQLの編集中内容も保存対象になります。

SQLの参照編集は元JSONへ、配置はレイアウトへ自動保存します。「保存せずに閉じる」はSQLを最終保存状態へ戻しますが、自動保存済みのJSON参照・配置は戻しません。SQLのUndo/Redoは直近100操作を保持し、参照変更の取り消しでもJSONを保存します。「SQL保存」は成功・失敗にかかわらずUndo/Redo履歴を消去します。同じフォルダで画面を閉じても履歴を保持し、フォルダ変更・ページ再読み込みで初期化します。

## 変換・生成・内部実行の仕様

### CSV

新規JSON生成では先頭行をカラム名とし、先頭10データレコードから`date → datetime → boolean → integer → float → string`の順に型を推定します。以降のデータも推定型で検査します。既存JSONへの追加はヘッダー名の完全一致で対応付け、列順は問わず、定義外のCSV列は警告表示して除外します。UTF-8・Shift_JIS・BOM付きUTF-16に対応し、上限はヘッダーを含む10,000レコードです。引用符内の改行は別レコードに数えません。

CSVにない列は`default`を適用し、初期値もなければキーを省略します。CSVに列があり値が空欄なら、文字列型は空文字、対応する数値・日付・真偽値型は`null`です。引用なしの`null`と引用付きの`"null"`を区別します。型変換・ヘッダー・列数などのエラーは全件中止とし、保存・取り込みボタンを無効にします。取り込み先変更や外部変更も確認し、失敗時は追加前の編集状態を保ちます。

出力はUTF-8・BOM付き・CRLFです。コピーにはBOMを含めません。定義CSVは行番号と現在の簡易／詳細項目、データCSVは定義のカラム順とレコード順を使います。削除ボタンは出力せず、データCSVには行番号・定義外キーも含めません。SQL結果も全件をCSVに変換し、表示中の仮想スクロール範囲に限定しません。

### PostgreSQL向けDDL

`ddl.js`はDROP TABLE、CREATE TABLE、主キー・外部キー・NULL不可・初期値、コメント、INSERTを生成します。`string → varchar`、`integer → int`、`datetime → timestamp`、`bool → boolean`の別名変換を行います。対応外の型は定義エラーです。入力候補の型・別名は`ddl-values.js`で初期値を検査し、それ以外の型では初期値の型検査を省略します。

`serial`は主キーを必須とし、初期値は未定義または`null`のみ許容し、明示的なシーケンス操作は生成しません。日付は`YYYY-MM-DD`、日時は`YYYY-MM-DD HH:mm:ss`（T区切り・小数秒1〜6桁可）を検査します。`current_timestamp`は日付・日時型でキャスト付きSQL式になりますが、デザイナーの行追加時には式評価しません。

定義エラーは赤、INSERTの型・NULL不可との不一致は黄色です。INSERTでは省略キーを対象外とし、定義外キーを警告付きで除外し、空の`{}`からは生成しません。値は可能な限り維持するため、出力できることとDBで実行できることは別です。生成不能時はコピー・保存を無効にします。保存名は`DDL(ファイル名).sql`、UTF-8・CRLFです。

### ER・SQL

通常ERとSQL図のカラム間ドラッグは、ドラッグ元JSONの`reftable`・`refcolumn`を書き換えて保存します。線削除は参照キーを削除し、同じカラム対の相互参照は両側を削除します。SQL図のカード削除はSQL対象から外す操作であり、JSONファイルは残します。

SQLのJOIN自動生成は、同じカラム対の相互参照をINNER JOIN、片方向を参照先が左側のLEFT JOIN、無関係なテーブルをCROSS JOINとします。循環などで生成できなければSQLでJOINを明示します。グリッドの同じ条件行はAND、条件行同士はORです。条件欄のNULL判定は`IS NULL`・`IS NOT NULL`のみを受け付けます。条件欄の日付は引用なし・二重引用符付きも受け付けて文字列化しますが、SQL本文では単一引用符を使います。

内部実行はJSONの`data`に対する読み取り専用処理です。対応範囲はSELECT列・別名・`*`・DISTINCT、INNER/LEFT/CROSS JOIN、WHERE（比較・AND/OR/NOT・IS NULL・LIKE・IN・BETWEEN）、ORDER BYです。更新SQL、集計、関数、サブクエリ、UNIONなどには対応しません。同じテーブルを1回ずつ使い、OR条件を3組以内に展開できるSQLをグリッドへ反映します。変換できないSQLは理由を表示して保持し、エンジンの対応構文なら直接実行できます。

識別子はPostgreSQL形式の二重引用符で生成します。DISTINCT時は並び順の列も表示対象にする必要があります。NULLの並びは昇順で末尾、降順で先頭です。通常文字列はPostgreSQLの標準設定`standard_conforming_strings=on`を前提にしますが、内部実行はDBの型変換や照合順序すべてを再現しません。SELECTのテーブル名は物理名です。SQL側で`table`が未指定・空欄・文字列以外の場合は対象を無効にし、上部へエラーを表示します。重複する物理名も対象を一意に特定できないためエラーになります。

対象・中間結果の上限は100,000行、処理回数は2,000,000回です。超過時は途中結果を表示せずエラーにします。SQL出力は選択フォルダの`SQL名.sql`、結果出力は`SQL名.csv`です。

## テスト

プロジェクトルートで実行します。

```sh
node tests/run.cjs
```

`tests/*.test.js`と`tests/*.test.cjs`を自動検出し、スイートごとに独立したプロセスで実行します。`.test.js`は互換VM、`.test.cjs`はNode.jsの標準APIを使います。追加パッケージは不要です。JavaScriptCore用の`load`・`readFile`・`drainMicrotasks`をランナーが用意し、失敗時は終了コード1になります。保存・復旧・外部変更、JSON解析と検索、CSV、DDL、ER配置、SQL解析・実行、デスクトップ接続などを検証します。`sql-highlight.test.js`で文字列・コメント・HTMLエスケープと行境界、`sql-runtime.test.js`で非同期の待機・同期結果との一致・処理上限を確認します。`backup.test.cjs`も一括実行の対象で、圧縮ZIPの展開一致、CRC32とヘッダー、保持日付、故障時の古いZIP保持、定期実行・フォルダ切替を検証します。実ユーザーファイルは変更しません。

単独スイートは次のように実行できます。`.test.js`はmacOSのJavaScriptCoreによる直接実行も可能です。バックアップ試験はWeb Streamsを使うためNode.jsで実行します。

```sh
node tests/run.cjs --suite tests/app.test.js
node tests/run.cjs --suite tests/backup.test.cjs
/System/Library/Frameworks/JavaScriptCore.framework/Versions/A/Helpers/jsc tests/app.test.js
```

実ブラウザーの統合テストは次のコマンドです。

```sh
python3 tools/test_browser.py
python3 tools/test_browser.py --window-size=2560,1440
```

`--chrome`で実行ファイルを指定できます。ランナーは開発用HTMLの一時コピーに`tests/sql-designer.browser.js`を追加し、一時プロファイルで起動・終了後に削除します。メモリー内のファイルで画面連携、Undo/Redo、保存競合、参照編集、列幅、SQL実行、大量結果の描画、固定ヘッダーの寸法・独立スクロール、行ドラッグの保存、サイズ監視の再描画停止などを確認します。成功・失敗を終了コードで返します。ヘッドレス試験は実行ファイルの起動、OSのカーソル表示、PNGの外観を保証しません。

手動確認用データは[5業種のテストデータ説明](json/テスト/README.md)を参照してください。操作するとJSONや設定が更新されるため、検証用コピーを使います。

## 単独HTMLの手動生成

生成HTMLは凍結したスナップショットです。通常のソース変更・文書更新・テストで自動再生成しません。明示的なミニファイ・実行ファイル作成・GitHub掲載の依頼時に更新します。

```sh
python3 -m pip install --target /private/tmp/jsonstudio-minify-deps -r tools/requirements-minify.txt
PYTHONPATH=/private/tmp/jsonstudio-minify-deps python3 tools/build_minified.py
```

`tools/build_minified.py`は`index.html`と参照するJS・CSSを結合・ミニファイし、必要なら出力ディレクトリーを作成して`neu/dist/index.html`へ保存します。成功後にルートのREADMEを内容を変えず`neu/dist/README.md`へコピー・上書きします。旧`index.min.html`は生成・更新しません。

ミニファイのみでは`neu/resources/index.html`やEXE・APP・ZIPは更新しません。生成HTMLを直接編集せず、元の`index.html`・`js/`・`css/`を編集します。`neu build --clean`で`neu/dist/index.html`が削除される場合があります。

## デスクトップ版のビルド

実行ファイル作成の依頼では、両プラットフォームを対象に次の順で実施します。上の依存導入が済んでいる前提です。

```sh
PYTHONPATH=/private/tmp/jsonstudio-minify-deps python3 tools/build_minified.py
cp neu/dist/index.html neu/resources/index.html
python3 neu/build-windows.py
python3 neu/build-macos.py
```

コピー後、`neu/resources/js/desktop-adapter.js`と`neu/neutralino.config.json`の互換性を確認してからビルドします。各ビルドスクリプトは既存の`resources/index.html`を埋め込み、HTMLのミニファイは行いません。通常のソース修正ではこの手順を自動実行しません。

| 出力 | 内容 |
| --- | --- |
| `neu/dist/windows/JsonStudio.exe` | Windows x64単独実行ファイル |
| `neu/dist/macos/JsonStudio.app` | macOS Universalアプリ |
| `neu/dist/JsonStudio-windows-x64.zip` | Windows配布用ZIP |
| `neu/dist/JsonStudio-macos-universal.zip` | macOS配布用ZIP |
| 各プラットフォームの`build-info.json` | ビルド入力や検証結果、ハッシュなど |

デスクトップ接続はFile System Access API互換のハンドルを提供し、既存の保存処理へ接続します。ファイル保存は同じフォルダの一時ファイルからの置き換えを使います。配布ビルドでは開発者ツール・ログを無効にし、テンプレートのテスト変数や作業用JSONを含めません。

WindowsはWebView2 Runtimeを必要とし、ZIPに同梱しません。macOSはアドホック署名で、Developer ID署名・公証は行いません。ビルド内の形式・埋め込みリソース・アイコン・署名などの検証と、実OSでの起動・読み書き・出力・終了確認は区別して記録します。詳細は[デスクトップ版の開発資料](neu/README.md)を参照してください。

## GitHub掲載と履歴

掲載は明示的な依頼がある場合のみ、`nokura-dev/JsonStudio`の`main`へ行います。`neu/dist/`の`index.html`・`README.md`・両OSの配布ZIPの4ファイルだけをリポジトリルートへ対応させ、既存の他ファイルを維持します。

掲載前は単独HTMLとREADMEコピーを更新し、ZIP内HTMLと現在のデスクトップ入力の一致を確認します。欠落・古いZIPは両OSのビルド手順で更新し、入力が変わらないZIPは再利用します。リモート最新`main`と内容で比較して変更ファイルだけをコミットし、全件同一ならコミット・pushを行いません。ローカルがGit管理外なら隔離したチェックアウトを使います。

コミットメッセージは英語とし、前回の掲載成功以降の履歴を実際の差分と照合します。push後にリモートを確認し、`HISTORY.md`へSHA・URL・正確なメッセージ・変更ファイル・対象履歴IDを記録します。`HISTORY.md`はローカル記録であり、公開や`neu/dist/`へのコピーに含めません。作業規約の詳細は`AGENTS.md`を参照してください。

CSVから新規JSONを作成するとき、同名回避の連番はJSONファイル名だけに付けます。JSON内の`table`は元CSVの拡張子なし名称を維持し、物理名が重複してもファイル作成は可能です。
