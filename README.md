# RDRAAgent - 使い方ガイド

## 概要

RDRAAgentは、RDRA（Relationship Driven Requirement Analysis）手法を用いてLLM（大規模言語モデル）を使って要件定義を段階的に行うシステムです。初期要望から5つのフェーズを通じて体系的に要件を定義し、仕様のたたき台まで生成します。
初期要望から要件を定義し、妥当性検証環境で要件を検証します。これらはメニュー(node menu)を使ってRDRA定義や検証、仕様化、ツール利用を行います。

**対応LLM**: Junie (default) / Claude Code / Gemini CLI / Codex CLI

## インストール方法

### 方法: GitHubから直接ダウンロード（推奨）

Zipでダウンロードする：
「Download Zip」を選択する

### 部分的にJavaScriptを使っているので、Node.jsのインストールが必要です

### 方法2: リポジトリをクローンして直接使用（開発・カスタマイズ向け）

開発やカスタマイズを行う場合は、リポジトリを直接クローンします：

```bash
git clone https://github.com/kanzaki/RDRAAgent0.6.git
cd RDRAAgent0.6
node menu.js
```

**この方法の特徴：**
- `npm install` は不要（外部依存関係がないため）
- ダウンロードかクローンするだけですぐに使用可能

**含まれるフォルダー構造：**
```
RDRAAgent/
├── RDRA_Knowledge/       # ナレッジベース
├── .junie/               # Junie設定
├── .claude/              # Claude Code設定
├── .cursor/              # Cursor設定
├── Samples/              # サンプルプロジェクト
├── 初期要望.txt
└── 妥当性検証環境.csv
```

## 「初期要望.txt」と「妥当性検証環境.csv」の二つを入力する
### 初期要望.txt：RDRA定義のための初期の要望のリスト
- システム名
- 要求や課題
- 業務概要や業務の背景
- アクターや外部システムを限定したい場合は記述
- ビジネスポリシーなど分かる範囲で記述

### 妥当性検証環境.csv
- 要件の妥当性を検証するために記述する
- システムを導入する環境の制約を記述
- 拠点毎のアクター別、人数と制約で記述する

### Samplesの活用
- 入力内容はSamples内のファイルを参照する
- 最初は思いつくものを記入し、実行しながら何度も壁打ちする

## プロジェクトの構築ポリシー
- 様々な環境、モデルに対応できるように他のライブラリーへの依存を無くす
- 外部ライブラリの依存を無くすために、テキストベースのファイルで構成する
- `モデル設定.json`でLLMプロバイダーを切り替え可能（Junie/Claude Code/Gemini/Codex）
- 新規プロバイダーを追加するには、`モデル設定.json`に設定を追加するのみ

### 対応環境
- **OS**: Windows 11 / Mac
- **Node.js**: バージョン18以上推奨
- **LLM**: Junie (default) / Claude Code / Gemini CLI / Codex CLI

### LLMの切り替え
`モデル設定.json`の`default.provider`を変更することで、使用するLLMを切り替えることができます。

```json
{
  "default": {
    "provider": "junie"  // junie, claude, gemini, codex から選択
  }
}
```

**Junieを使用する場合**:
- JetBrains IDEでJunieを有効にする
- `junie`コマンドが利用可能であることを確認
- `モデル設定.json`の`default.provider`を`"junie"`に設定（デフォルト）

**Claude Codeを使用する場合**:
- Claude Codeをインストールし、`claude`コマンドが利用可能であることを確認
- `モデル設定.json`の`default.provider`を`"claude"`に設定

## メニュー起動 Terminal
```bash/command
node menu.js
```
## メニュー起動 Cursor
「/」スラッシュコマンドでメニュー表示

### 主な特徴
- **段階的な要件定義**: 5つのフェーズで段階的にRDRA定義を作成
- **マルチLLM対応**: Junie（デフォルト）/ Claude Code / Gemini / Codex から選択可能
- **可視化機能**: RDRAGraphによる要件の関連性の可視化
- **画面プロトタイプ**: ブラウザでBUC・アクター別画面を確認
- **Spreadsheet展開**: Google SpreadsheetにRDRA定義を展開可能

## メニュー構成

### ■ZeroOne（要件定義）

**1. フェーズ単位実行**
- 成果物が無い最初のPhaseを実行
- Phase4が揃っていればPhase5を実行
- Phase1→Phase2→Phase3→Phase4→Phase5 の順で対話的に進める
- 各フェーズで生成されるファイル：
  - Phase1: アクター、外部システム、ビジネスポリシー、業務、情報、状態
  - Phase2: 業務詳細化、条件、バリエーション、情報・状態の構造化
  - Phase3: コンテキスト化、BUC定義、関連付け
  - Phase4: 関係モデル変換、最終形式の定義
  - Phase5: システム概要生成、1_RDRAフォルダへの統合

**2. 一括要件定義**
- RDRA定義を一括で実行
- 全フェーズを自動で連続実行
- Phase別のフォルダーに成果物が無いフォルダー（Phase）から実行する

### ■RDRA（RDRA定義の可視化・管理）

**11. RDRAGraphを表示**
- 関連データを生成し、RDRAGraphで可視化
- 処理内容：
  1. `makeGraphData.js` で関連データ生成
  2. `copyToClipboard.js` でデータをクリップボードにコピー
  3. ブラウザで https://vsa.co.jp/rdratool/graph/v0.94/index.html?clipboard を開く
  4. 自動的にクリップボードのデータを読み込み、グラフ表示

**12. Spreadsheetに展開**
- RDRA定義をクリップボードにコピー
- 処理内容：
  1. `makeZeroOneData.js` で関連データ生成
  2. `copyToClipboard.js` でデータをクリップボードにコピー
  3. ブラウザでGoogle Spreadsheetを開く
  4. SpreadsheetのZeroOneシートに貼り付けて活用

### ■RDRASpec（仕様）

**21. 仕様の作成**
- 論理データ構造、画面定義、ビジネスルールを生成
- 出力ファイル：
  - `2_RDRASpec/論理データモデル.md` - 論理データモデル（Markdown形式）
  - `2_RDRASpec/画面照会.json` - 画面定義
  - `2_RDRASpec/ビジネスルール.md` - ビジネスルール

**22. BUC・アクター別画面を表示する**
- BUC（ビジネスユースケース）とアクター別に画面を整理して表示
- `bucActorUI.js` でHTTPサーバーを起動（ポート3002）
- ブラウザで http://localhost:3002/ を開く
- 画面の「閉じる」ボタンでサーバーを停止可能

### ■全般

**0. メニュー終了**
- プログラムを終了
- 起動中のサーバーも自動停止

**99. 生成した成果物の削除**
- 以下のフォルダーの成果物を削除：
  - `0_RDRAZeroOne/`
  - `1_RDRA/`
  - `2_RDRASpec/`

## ワークフロー

### 基本的な作業の流れ

1. **初期準備**
   - `初期要望.txt` にシステム要求を記述
   - `Samples/` フォルダーのサンプルを参考にする

2. **要件定義（ZeroOne）**
   - メニュー「1」または「2」で要件定義を実行
   - 5つのフェーズで段階的に要件を洗い出し
   - Phase1→Phase2→Phase3→Phase4→Phase5の順で実行

3. **RDRA可視化**
   - メニュー「11」でRDRAGraphを表示
   - 要件の関連性を図で確認

4. **仕様作成**
   - メニュー「21」で仕様を生成
   - データモデル、画面、ビジネスルールを作成

5. **画面の確認**
   - メニュー「22」でBUC・アクター別画面を表示
   - ブラウザで画面定義を確認（ポート3002）

## 使用例

### 例1: 図書館システムの要件定義

```bash
# 1. サンプルの初期要望をコピー
copy Samples\図書館システム\初期要望.txt 初期要望.txt

# 2. メニューシステムを起動
node menu.js

# 3. メニューで「2」を選択（一括要件定義）
# → 5つのPhaseが順次実行される

# 4. メニューで「11」を選択（RDRAGraph表示）
# → ブラウザでRDRAGraphが表示される

# 5. メニューで「21」を選択（仕様作成）
# → 論理データモデル、画面定義、ビジネスルールが生成される

# 6. メニューで「22」を選択（BUC・アクター別画面表示）
# → ブラウザで画面が表示される（ポート3002）
```

### 例2: 段階的な要件定義

```bash
# 1. 初期要望を記述
notepad 初期要望.txt

# 2. メニューシステムを起動
node menu.js

# 3. メニューで「1」を選択（フェーズ単位の要件定義）
# → Phase1が実行される

# 4. Phase1の結果を確認
# → 0_RDRAZeroOne/phase1/ フォルダー内のTSVファイルを確認

# 5. 再度メニューで「1」を選択
# → Phase2が実行される

# （Phase3、Phase4、Phase5も同様に実行）
```

### 例3: BUC・アクター別画面の確認

```bash
# 1. 仕様作成後（メニュー21実行済み）

# 2. メニューで「22」を選択（BUC・アクター別画面表示）
# → ブラウザでBUC・アクター別画面が表示される（ポート3002）

# 3. ブラウザでBUCやアクターを切り替えて画面を確認
# → 画面の「閉じる」ボタンでサーバーを停止可能
```

## ファイル構成

### 入力ファイル
- `初期要望.txt` - システムの初期要求
- `妥当性検証環境.csv` - 検証用データ（オプション）

### サンプルフォルダー
- `Samples/` - 参考となるサンプルプロジェクト
  - `介護事業者向けシステム/` - 介護事業者向けシステムのサンプル
  - `図書館システム/` - 図書館システムのサンプル
  - `貸し会議室SaaS/` - 貸し会議室SaaSのサンプル
  - 各サンプルには `初期要望.txt` が含まれています

### 出力フォルダー
- `0_RDRAZeroOne/phase1~5/` - フェーズ別のTSVファイル
- `1_RDRA/` - RDRA定義ファイル（統合版、TSV/JSON形式）
- `2_RDRASpec/` - 仕様ファイル（JSON/TSV/Markdown形式）

### ヘルパーツール
- `RDRA_Knowledge/helper_tools/` - 各種支援スクリプト
  - `makeGraphData.js` - グラフデータ生成
  - `makeZeroOneData.js` - RDRASheetに生成
  - `copyToClipboard.js` - データをクリップボードにコピー
  - `rdraFileCopy.js` - Phase5での1_RDRAフォルダへのコピー
  - `deleteFiles.js` - 成果物削除
  - `validate_screen_shokai.js` - 画面照会の検証
  - `web_tool/bucActorUI.js` - BUC・アクター別画面表示サーバー

### ナレッジベース
- `RDRA_Knowledge/0_RDRAZeroOne/` - ZeroOneフェーズの説明
  - Phase:1-5 - フェーズ毎の成果物生成Prompt

- `RDRA_Knowledge/1_RDRA/` - RDRA定義の説明
  - `RDRA.md` - RDRAの基本概念
  - `RDRAGraph.md` - RDRAGraphの説明
  - `RDRASheet.md` - RDRASheetの説明
- `RDRA_Knowledge/2_RDRASpec/` - 仕様作成の説明
  - `phase1/` - 仕様Phase1のプロンプト
    - `21_論理データ生成.md` - 論理データモデル作成方法
    - `22_ビジネスルール生成.md` - ビジネスルール作成方法
    - `23_画面一覧生成.md` - 画面一覧作成方法
    - `24_BUC画面生成.md` - BUC画面作成方法
    - `25_アクター画面生成.md` - アクター画面作成方法
  - `phase2/` - 仕様Phase2のプロンプト
    - `26_画面照会生成.md` - 画面照会作成方法
- `RDRA_Knowledge/utilityPrompt/` - ユーティリティプロンプト集

### 設定ファイル
- `.claude/settings.local.json` - Cursor/Claudeのアクセス権限設定

## ファイルフォーマット

### TSVファイル規則
- UTF-8エンコーディング
- 1行目：ヘッダー行（カラム名）

### 主要なTSVファイル

#### Phase1で生成されるファイル
- `アクター.tsv` - システム利用者の役割
- `外部システム.tsv` - 連携する外部システム
- `ビジネスポリシー.tsv` - ビジネスポリシー
- `業務.tsv` - 業務フロー
- `情報.tsv` - システムが扱う情報
- `状態.tsv` - 状態遷移モデル

#### Phase2で生成されるファイル
- `業務.tsv` - 業務詳細化
- `条件.tsv` - ビジネスルール
- `バリエーション.tsv` - 業務パラメータ
- `情報.tsv` - 情報の構造化
- `状態.tsv` - 状態の詳細化

#### Phase3で生成されるファイル
- `アクター.tsv` - アクターのコンテキスト化
- `外部システム.tsv` - 外部システムのコンテキスト化
- `条件.tsv` - 条件の関連付け
- `バリエーション.tsv` - バリエーションの関連付け
- `情報.tsv` - 情報の関連付け
- `状態.tsv` - 状態の関連付け
- `BUC.tsv` - ビジネスユースケース定義

#### Phase4で生成されるファイル
- `BUC.tsv` - BUCの関係モデル化
- `条件.tsv` - 条件の最終形式
- `情報.tsv` - 情報の最終形式
- `状態.tsv` - 状態の最終形式

#### 1_RDRAフォルダーで生成されるファイル
- `システム概要.json` - システム全体の概要
- `BUC.tsv` - ビジネスユースケース（統合版）
- `アクター.tsv` - アクター（統合版）
- `外部システム.tsv` - 外部システム（統合版）
- `情報.tsv` - 情報（統合版）
- `状態.tsv` - 状態（統合版）
- `条件.tsv` - 条件（統合版）
- `バリエーション.tsv` - バリエーション（統合版）
- `関連データ.txt` - RDRAGraph用の関連データ

## LLM連携

メニューの一部機能（1, 2, 21）はLLM（大規模言語モデル）を使用します：
- `モデル設定.json`で指定されたLLMが自動的に実行されます
- 実行される指示ファイル：
  - メニュー1, 2: `RDRA_Knowledge/0_ZeroOne指示.md`
  - メニュー21: `RDRA_Knowledge/2_Spec指示.md`
- `RDRA_Knowledge/helper_tools/parallelRun/parallel-runner.js` を経由して実行

### 対応LLM
- **Junie** (デフォルト): JetBrains IDEに組み込みのコーディングエージェント
- **Claude Code**: AnthropicのCLIツール
- **Gemini CLI**: GoogleのCLIツール
- **Codex CLI**: OpenAIのCLIツール

LLMの切り替え方法は「対応環境」セクションを参照してください。

## トラブルシューティング

### ファイルが生成されない
- **原因**: LLMの実行が完了していない、またはエラーが発生
- **対処**: 各フェーズの実行確認機能が自動でチェック
  - 不足ファイルは自動再生成される
  - ターミナルウィンドウのエラーメッセージを確認

### サーバーが起動しない
- **原因**: ポート競合（3002）
- **対処**: 
  - メニュー「0」で一度終了し、再起動
  - タスクマネージャーでNode.jsプロセスを確認
  - 別のポートを使用している場合は手動でブラウザを開く

### データが表示されない
- **原因**: 必要なファイルが未生成
- **対処**:
  - 事前にメニュー「21」で仕様ファイル生成が必要
  - `2_RDRASpec/画面照会.json` の存在を確認
  - メニュー「99」で一度クリアして再実行

### PowerShellの実行ポリシーエラー
- **原因**: PowerShellの実行ポリシーが制限されている
- **対処**: 管理者権限で以下を実行
  ```powershell
  Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
  ```

## 参考情報

### RDRA構造の基本概念
- **アクター** - システムを利用する人の役割
- **外部システム** - 連携する外部システム
- **業務** - 会社の機能単位
- **BUC** - ビジネスユースケース（業務の価値実現単位）
- **情報** - システムが扱うビジネス情報
- **状態** - 情報の変化を管理する状態モデル
- **UC** - ユースケース（システムとの接点）
- **条件** - ビジネスルールや制約
- **バリエーション** - 業務のバリエーションパラメータ

詳細は `RDRA_Knowledge/1_RDRA/RDRA.md` を参照してください。

### Phase別の作業内容

#### Phase1: 要素の洗い出し
- システムの基本要素を特定
- アクター、外部システム、ビジネスポリシー、業務、情報、状態を定義

#### Phase2: 要素の詳細化
- 業務の詳細化とBUC候補の抽出
- 条件（ビジネスルール）の定義
- バリエーションの定義
- 情報と状態の構造化

#### Phase3: コンテキスト化とBUC定義
- 各要素のコンテキスト（文脈）を明確化
- BUC（ビジネスユースケース）の正式定義
- 要素間の関連付け

#### Phase4: 関係モデル化
- RDRAの関係モデルへの変換
- 最終的なRDRA定義の完成

#### Phase5: システム概要生成と統合
- システム概要の生成
- 1_RDRAフォルダーへの統合
- 全フェーズの成果物を統合した最終出力

### よくある質問

**Q: Phase1で生成されるファイルとPhase4で生成されるファイルで同じ名前のものがありますが、違いは何ですか？**
A: 各Phaseで同じ名前のファイルが生成されることがありますが、内容は段階的に詳細化・構造化されていきます。最終的にPhase4までの内容が1_RDRAフォルダーに統合されます。

**Q: RDRAGraphが表示されない場合は？**
A: 1_RDRA/関連データ.txtが生成されていることを確認し、ブラウザのポップアップブロックが有効になっていないか確認してください。

**Q: 途中でエラーが出た場合は？**
A: メニュー「99」で成果物を削除し、最初からやり直すことをお勧めします。または、エラーが出たPhaseのみを再実行することもできます。

**Q: サンプルプロジェクトを試すには？**
A: Samplesフォルダー内のサンプルの初期要望.txtを、プロジェクトルートの初期要望.txtにコピーして使用してください。

### 関連リンク
- **RDRAツール公式サイト**: https://vsa.co.jp/rdratool/
- **RDRAGraph**: https://vsa.co.jp/rdratool/graph/
- **RDRA手法について**: `RDRA_Knowledge/1_RDRA/RDRA.md` を参照

---

## バージョン情報

**現在のバージョン**: RDRAAgent v0.65

### 変更履歴
- v0.65: 現行バージョン
  - 5フェーズのRDRA定義プロセス実装（Phase5: システム概要生成と統合）
  - BUC・アクター別画面表示機能（メニュー22、ポート3002）
  - **Junie対応**: Junie liteで動作可能に
  - **マルチLLM対応**: モデル設定.jsonでLLMを切り替え可能（Junie/Claude/Gemini/Codex）
  - **Junie設定**: .junie/AGENTS.mdを追加
  - メニュー簡素化: 廃止機能（13, 14, 23, 24）を削除

## 別ディレクトリでRDRAAgentを動作させる

RDRAAgentは、インストールしたディレクトリとは異なる場所でプロジェクトを実行できます。例えば、`../modulith-learning/` などの別ディレクトリで要件定義を行う場合の手順を説明します。

### 方法1: 入力ファイルを別ディレクトリに配置（推奨）

RDRAAgent本体はそのままにし、入力ファイルのみを別ディレクトリに配置します。

```bash
# 1. 作業ディレクトリを準備
cd ../modulith-learning/

# 2. 初期要望.txtを作成
cat > 初期要望.txt << 'EOF'
システム名: Modulith Learning System
要求や課題: 学習管理システムの要件定義
業務概要: 受講者の学習進捗を管理するシステム
EOF

# 3. RDRA_Knowledgeフォルダをコピーまたはリンク
# オプションA: コピー（独立した環境）
cp -r /Users/mizoguchikenichi/IdeaProjects/RDRAAgent/RDRA_Knowledge ./

# オプションB: シンボリックリンク（本体と共有）
ln -s /Users/mizoguchikenichi/IdeaProjects/RDRAAgent/RDRA_Knowledge ./

# 4. RDRAAgentのmenu.jsを指定して実行
node /Users/mizoguchikenichi/IdeaProjects/RDRAAgent/menu.js
```

**注意点:**
- menu.jsは実行時のカレントディレクトリを作業ディレクトリとして使用します
- 出力フォルダ（`0_RDRAZeroOne/`、`1_RDRA/`、`2_RDRASpec/`）はカレントディレクトリに生成されます
- **必須**: `RDRA_Knowledge/`フォルダはmenu.jsと同じディレクトリ（カレントディレクトリ）に配置する必要があります
- `RDRA_Knowledge/`フォルダには、要件定義の指示ファイルや処理スクリプトが含まれています

### 方法2: シンボリックリンクを使用

頻繁に別ディレクトリでRDRAAgentを使用する場合、シンボリックリンクを設定すると便利です。

```bash
# 1. 作業ディレクトリに移動
cd ../modulith-learning/

# 2. RDRAAgent本体へのシンボリックリンクを作成
ln -s /Users/mizoguchikenichi/IdeaProjects/RDRAAgent/RDRA_Knowledge RDRA_Knowledge
ln -s /Users/mizoguchikenichi/IdeaProjects/RDRAAgent/Samples Samples
ln -s /Users/mizoguchikenichi/IdeaProjects/RDRAAgent/menu.js menu.js
ln -s /Users/mizoguchikenichi/IdeaProjects/RDRAAgent/モデル設定.json モデル設定.json

# 3. 初期要望.txtを作成
cat > 初期要望.txt << 'EOF'
システム名: Modulith Learning System
要求や課題: 学習管理システムの要件定義
EOF

# 4. 通常通り実行
node menu.js
```

### 方法3: RDRAAgentをコピーして独立したプロジェクトとして使用

完全に独立したプロジェクトとして使用する場合、RDRAAgent全体をコピーします。

```bash
# 1. RDRAAgentをコピー
cp -r /Users/mizoguchikenichi/IdeaProjects/RDRAAgent ../modulith-learning/rdra-agent

# 2. 作業ディレクトリに移動
cd ../modulith-learning/rdra-agent

# 3. 初期要望.txtを編集
cat > 初期要望.txt << 'EOF'
システム名: Modulith Learning System
要求や課題: 学習管理システムの要件定義
EOF

# 4. 通常通り実行
node menu.js
```

**注意点:**
- コピーしたディレクトリは独立するため、本体の更新が反映されません
- Git管理している場合、コピーしたディレクトリは別のリポジトリとして扱われます

### 別ディレクトリ実行時のファイル構成

```
../modulith-learning/
├── 初期要望.txt          # ユーザーが入力
├── RDRA_Knowledge/       # 必須: ナレッジベース（コピーまたはシンボリックリンク）
├── 0_RDRAZeroOne/        # 出力: フェーズ別のTSVファイル
├── 1_RDRA/               # 出力: 統合RDRA定義
├── 2_RDRASpec/           # 出力: 仕様ファイル
└── (menu.jsは絶対パスまたはシンボリックリンクで参照)
```

**ドキュメントの配置場所について:**
- `RDRA_Knowledge/`フォルダは、menu.jsと同じディレクトリ（RDRAAgent本体がある場所）に配置する必要があります
- 別ディレクトリで実行する場合、以下の2つの方法があります：
  1. **シンボリックリンク**: 作業ディレクトリからRDRAAgent本体の`RDRA_Knowledge/`へリンクを作成（方法2参照）
  2. **コピー**: `RDRA_Knowledge/`フォルダを作業ディレクトリにコピー（方法3参照）
- `RDRA_Knowledge/`フォルダには以下の重要なドキュメントが含まれています：
  - `0_RDRAZeroOne/` - 要件定義フェーズの指示ファイル
  - `1_RDRA/` - RDRA定義の説明ファイル
  - `2_RDRASpec/` - 仕様生成の指示ファイル
  - `helper_tools/` - 各種処理スクリプト

### サンプルプロジェクトの活用

別ディレクトリでサンプルプロジェクトを試す場合:

```bash
# 1. 作業ディレクトリに移動
cd ../modulith-learning/

# 2. RDRA_Knowledgeフォルダをコピーまたはリンク
ln -s /Users/mizoguchikenichi/IdeaProjects/RDRAAgent/RDRA_Knowledge ./

# 3. サンプルの初期要望をコピー
cp /Users/mizoguchikenichi/IdeaProjects/RDRAAgent/Samples/図書館システム/初期要望.txt 初期要望.txt

# 4. RDRAAgentを実行
node /Users/mizoguchikenichi/IdeaProjects/RDRAAgent/menu.js
```

### Windowsでの別ディレクトリ実行

Windows環境では以下のコマンドを使用します:

```cmd
REM 1. 作業ディレクトリに移動
cd ..\modulith-learning\

REM 2. RDRA_Knowledgeフォルダへのシンボリックリンクを作成
mklink /D RDRA_Knowledge C:\path\to\RDRAAgent\RDRA_Knowledge

REM 3. 初期要望.txtを作成
echo システム名: Modulith Learning System > 初期要望.txt

REM 4. RDRAAgentを実行（絶対パス指定）
node C:\path\to\RDRAAgent\menu.js
```

または、シンボリックリンク（mklink）を使用:

```cmd
REM シンボリックリンクを作成
mklink /D RDRA_Knowledge C:\path\to\RDRAAgent\RDRA_Knowledge
mklink menu.js C:\path\to\RDRAAgent\menu.js

REM 通常通り実行
node menu.js
```

---

## ライセンス
MIT License
