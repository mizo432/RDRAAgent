# AGENTS.md

このファイルは、Junieがこのリポジトリのコードを扱う際のガイドラインを提供します。

## プロジェクト概要

RDRAAgentは、**RDRA（Relationship Driven Requirement Analysis：関係駆動要件分析）**手法を実装したNode.jsベースのシステムです。4つのフェーズを通じて、初期の概念から詳細なシステム仕様まで、ビジネス要件を段階的に精査するプロセスをユーザーに案内します。

**主な特徴:**
- 外部依存なし（ネイティブモジュールのみを使用した純粋なNode.js）
- テキスト/TSVベースのデータフォーマット（データベース不使用）
- LLM統合ワークフロー（Junie/Claude/Gemini/Codex）によるAI支援分析
- マルチプラットフォーム対応（Windows、macOS）
- モジュール化されたフェーズベースのアーキテクチャ

## アーキテクチャ概要

### コアコンポーネント

1. **menu.js**（エントリポイント）
   - Node.js `readline`を使用したインタラクティブなCLIメニューシステム
   - 全操作とフェーズ管理のオーケストレーション
   - 並行サーバープロセス（UIエディタ）の管理
   - クロスプラットフォームコマンド実行（Windows cmd/PowerShell、macOS Terminal）

2. **3つの主要処理ストリーム**

   **ストリーム1: ZeroOne（0_RDRAZeroOne/** - 要件定義）
   - Phase1: 初期要素の抽出（アクター、システム、ポリシー、ワークフロー、情報、状態）
   - Phase2: 要素の精査と詳細条件の定義
   - Phase3: コンテキストマッピングとビジネスユースケース（BUC）の定義
   - Phase4: 関係モデルの変換
   - Phase5: 統一RDRAフォーマットへの統合（1_RDRA/）

   **ストリーム2: RDRA（1_RDRA/** - 要件の可視化と管理）
   - 全フェーズの出力を統合
   - RDRAGraph可視化用の関係データを生成
   - Googleスプレッドシートエクスポートをサポート
   - 定義の一貫性を検証

   **ストリーム3: RDRASpec（2_RDRASpec/** - 仕様生成）
   - 論理データモデル（TSV/Markdown）
   - UI定義（JSONフォーマット）
   - ビジネスルールのドキュメント化
   - 仕様の検証

3. **ヘルパーツール**（RDRA_Knowledge/helper_tools/**）
   - `makeGraphData.js` - RDRAGraph可視化用の関係データを生成
   - `makeZeroOneData.js` - Googleスプレッドシートエクスポート用のデータを準備
   - `copyToClipboard.js` - データをクリップボードにコピー
   - `convertUI2ActorUI.js` - アクターロールごとにUI定義を変換
   - `deleteFiles.js` - クリーンアップユーティリティ
   - `rdraFileCopy.js` - フェーズからの出力を統合
   - `drawioExtractor.js` - draw.ioダイアグラムを処理
   - `makeActorUI.js` - アクター固有のUI生成
   - Webサーバー（`ui_server.js`、`actorUI_server.js`）- ブラウザベース編集用のHTTPサーバー（ポート3000、3001）

4. **ナレッジベース**（RDRA_Knowledge/**）
   - **0_ZeroOne指示.md** - フェーズごとの要件抽出指示
   - **1_RDRA指示.md** - （非推奨）`_no_use/deprecated_menu_13_14_22/`下に移動済み
   - **2_Spec指示.md** - 仕様作成指示（Phase1はメニュー21経由；Phase2は非推奨）
   - **0_RDRAZeroOne/**、**1_RDRA/**、**2_RDRASpec/** - ドメイン知識ドキュメント

### データフローモデル

```
初期要望.txt
         ↓
    menu.js セレクター
         ↓
    ├─→ Phase1-5（Junie/ClaudeによるLLM駆動）
    │    （RDRA_Knowledge/0_ZeroOne指示.md）
    │         ↓
    │    0_RDRAZeroOne/phase1~4/（TSV出力）
    │         ↓
    │    1_RDRA/（統合済み）
    │
    ├─→ 可視化
    │    makeGraphData.js → RDRAGraph（外部ツール）
    │
    ├─→ 検証
    │    _no_use/deprecated_menu_13_14_22/RDRA_Knowledge/1_RDRA指示.md
    │    （妥当性検証環境.csvとの照合）
    │
    └─→ 仕様生成
         RDRA_Knowledge/2_Spec指示.md
              ↓
         2_RDRASpec/
         ├─ 論理データモデル.md
         ├─ 画面照会.json
         └─ ビジネスルール.md
```

## エージェントルール

`.claude/rules/rdrarules`からの主要ルール:

1. **フォルダ管理**: 作成前にフォルダの存在を確認；既存のディレクトリを再利用
2. **ファイルフォーマット**:
   - TSVファイルでは `、` をタブ区切り文字として使用（出力時に実際のタブに変換）
   - UTF-8エンコーディング必須
   - TSVファイルには常に列名を含むヘッダー行が必要
3. **プログラミング**: JavaScriptのみ（HTML/CSSはHTMLに埋め込み）
4. **UIライブラリ**: Webベースエディタではshadcn UIコンポーネントを使用
5. **実行フロー**:
   - 「return」= メッセージ付き関数終了
   - 「StepN:」= 順次実行
   - 「start process」= コマンド/ブラウザ/JavaScript実行
   - 「Goal:」= 関数の目的を示す
6. **ファイル生成**: テンプレート/プレースホルダーファイルを作成しない；LLMに完全なコンテンツを生成させる

## 一般的な開発タスク

### アプリケーションの起動
```bash
node menu.js
```
9つの主要オプションを持つインタラクティブメニューを開きます。

### シングルファイル操作
メニュー内では、「Phase 1」や「仕様作成」などの操作が以下のスクリプトを介してLLM実行をトリガーします:
- **Windows**: `RDRA_Knowledge/helper_tools/command/commandRun.bat`
- **macOS**: `RDRA_Knowledge/helper_tools/command/commandRun.sh`

これらのスクリプトは新しいターミナルウィンドウを開き、指定された指示でLLMを実行します。

### テスト/開発ワークフロー
1. LLMの動作変更には `RDRA_Knowledge/*指示.md` ファイルを修正
2. UI/メニューの変更には `menu.js` を更新
3. データ変換には `RDRA_Knowledge/helper_tools/` のヘルパーツールを更新
4. `Samples/` のサンプルプロジェクトで素早くテスト可能:
   ```bash
   copy "Samples/図書館システム/初期要望.txt" "初期要望.txt"
   node menu.js
   ```

### 検証/テストデータ
- `妥当性検証環境.csv` - 要件検証用のテストシナリオを定義
- 構造: アクターロール、人数、場所ごとの制約
- Phase2の検証ロジックで使用（メニューオプション14）

## TSVファイルフォーマットルール

- **区切り文字**: ソースコード内の `、` 文字はファイルに書き出す際に実際のTABに変換
- **1行目**: ヘッダー（列名）必須
- **エンコーディング**: 常にUTF-8
- **出力先**: フェーズ出力は `0_RDRAZeroOne/phase#/` へ、最終出力は `1_RDRA/` または `2_RDRASpec/` へ

ヘッダー変換の例:
```javascript
"Column1、Column2、Column3"  →  "Column1\tColumn2\tColumn3"
```

## LLM統合ポイント

システムはターミナル実行を介してLLMを以下に使用します:

1. **Phase1-4 コンテンツ生成**（`menu.js:1,2` - オプション1,2）
   - 読み込み: `RDRA_Knowledge/0_ZeroOne指示.md`
   - 生成: `0_RDRAZeroOne/phase#/` 内のTSVファイル

2. **RDRA説明と検証**（オプション13,14は非推奨/メニューから削除済み）
   - 読み込み: `_no_use/deprecated_menu_13_14_22/RDRA_Knowledge/1_RDRA指示.md`（非推奨）
   - 検証: `妥当性検証環境.csv` に対して

3. **仕様作成**（`menu.js:21` - オプション21；オプション22は非推奨/メニューから削除済み）
   - 読み込み: `RDRA_Knowledge/2_Spec指示.md`
   - 生成: データモデル、UI定義、ビジネスルール

**重要**: LLM指示にはファイルパスのプレフィックス（例:「パス「RDRA_Knowledge/0_ZeroOne指示.md」」）と、作業ディレクトリに対するファイルの場所に関する注記が含まれています。

## プラットフォーム固有の詳細

### Windows
- LLMターミナル: `start cmd` で呼び出される `cmd.exe`
- パス処理: PowerShell用フォワードスラッシュをバックスラッシュに変換
- クリップボード: UTF-8のPowerShell `Set-Clipboard`
- ブラウザ: PowerShell `Start-Process`

### macOS
- LLMターミナル: `osascript` 経由の `Terminal.app`
- クリップボード: `pbcopy`
- ブラウザ: `open` コマンド

### ポート管理
- **ポート3000**: UIエディタサーバー（オプション23）
- **ポート3001**: アクター固有UIサーバー（オプション24）
- 既に実行中の既存プロセスを再利用；EADDRINUSEエラーをチェック

## 重要な実装ノート

1. **npm依存なし**: 意図的な設計です。`package.json` の依存関係を追加しないでください。

2. **ファイル上書き**: Menu.jsは再生成時に同名の既存ファイルを自動的に上書きします。

3. **フェーズ間の依存関係**:
   - Phase1の出力はPhase2のユーザーレビューの入力となる
   - Phase3,4の出力は1_RDRAフォルダに統合される
   - 仕様生成には完全な1_RDRAフォルダが必要

4. **プロセス管理**:
   - グローバル変数が生成されたプロセスを格納: `global.serverProcess`、`global.actorScreenServer`
   - メニュー終了（オプション0）時にすべてのプロセスを優雅にシャットダウン

5. **ファイル構成**:
   - 入力: `初期要望.txt`、`妥当性検証環境.csv`
   - 出力フェーズ: `0_RDRAZeroOne/`、`1_RDRA/`、`2_RDRASpec/`
   - ナレッジ/ツール: `RDRA_Knowledge/`
   - サンプル: `Samples/`

## 外部リソース

- **RDRAGraph可視化**: https://vsa.co.jp/rdratool/graph/v0.94/
- **Googleスプレッドシートテンプレート**: https://docs.google.com/spreadsheets/d/1h7J70l6DyXcuG0FKYqIpXXfdvsaqjdVFwc6jQXSh9fM/
- **RDRA手法**: `RDRA_Knowledge/1_RDRA/RDRA.md` を参照

## 主要ファイルの概要

| ファイル | 目的 |
|------|---------|
| menu.js | メインCLIオーケストレーター |
| RDRA_Knowledge/0_ZeroOne指示.md | 要件生成指示 |
| _no_use/deprecated_menu_13_14_22/RDRA_Knowledge/1_RDRA指示.md | （非推奨）RDRA検証/説明 |
| RDRA_Knowledge/2_Spec指示.md | 仕様生成 |
| 初期要望.txt | ユーザー入力: 初期要件 |
| 妥当性検証環境.csv | ユーザー入力: 検証シナリオ |
| Samples/ | 参照プロジェクトテンプレート |
| helper_tools/ | データ変換ユーティリティ |
