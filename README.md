# gemini-cli-template
Gemini CLI の私用テンプレートリポジトリです。  
`chrome-devtools-mcp`MCPを入れていて、`GEMINI.md`も汎用的な内容にしています。

## 技術構成
- @google/gemini-cli@0.28.2

## [Gemini モデル](https://ai.google.dev/gemini-api/docs/models?hl=ja)について
- `/model`コマンドでGeminiのモデルを選択・変更可能
- `.gemini/settings.json`で明示的にモデルを選択・変更可能

## `settings.json`でのMCP設定
- `y`オプション
`args`の先頭に "-y" を入れておくと npx 実行時に「インストールしますか？」という確認で作業が進まないを防げる。
```json
"mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": [
        "-y",
        "chrome-devtools-mcp@latest"
      ]
    }
  }
```

## Gemini Code Assist や Gemini CLI, Antigravity に対して `GEMINI.md`（憲法：`AGENTS.md`）を設定する方法

### 前提
#### 推奨ファイル名
`GEMINI.md` または `AGENTS.md`

#### 配置場所
プロジェクトのルートディレクトリ

### Gemini Code Assist（VS Code）

IDE 内でコーディング支援を受ける際（copilot用途）は、明示的に読み込ませるか、ファイルに設定する（プロジェクトを開いた時点でルールを適用）かの方法がある。

#### 1. チャットセッションごとに明示的に読み込ませる（推奨）

1. チャット欄の **「+（Add Context）」** をクリック
2. `GEMINI.md` を選択して追加

> [!NOTE]
> エディタのタブで `GEMINI.md` を開いたままにしておくと、Gemini が参照しやすくなる。

#### 2. ファイルに設定する
`.vscode/settings.json` に以下のようなシステムプロンプト設定を追加。

- `.vscode/settings.json`
```json
{
  // 注意: キー名は拡張機能のバージョンにより異なる場合があります
  // ("systemPrompt", "instructions" 等を確認)
  "google.gemini.systemInstructions": [
    "You are an agent that strictly follows the project rules defined below.",
    "---",
    "ここに GEMINI.md の内容をコピペ",
    "---"
  ]
}
```

### Gemini CLI（Terminal）
ターミナルから自律操作させる際（Claude Code のような用途）は、こちらも明示的な記述か、スクリプトに組み込むかという方法がある。 

> [!NOTE]
> ※ただし、現在（2026/02時点）ではプロジェクトルートに置いておくと自動で読み込まれているような記述が見られる。
```bash
2 GEMINI.md files | 1 MCP server
# 1 MCP server は`"chrome-devtools-mcp`を指す
```

- GEMINI.md について尋ねた内容
```bash
 GEMINI.md を読み込んでいますか？そうであれば一部を端的に要約説明して

✦ はい、GEMINI.md の内容はコンテキストとして読み込んでおり、把握しております。
  このファイルは、本プロジェクトにおける私（AI アシスタント）の行動指針であり、要約すると...
```

#### 1. 基本形（パイプ処理）
```bash
# 「資料：GEMINI.md（cat）」を渡しながら「作業（chat）」を命じる
cat GEMINI.md | gemini chat "テストコードを書いて"
```

#### 2. 自動化スクリプト（推奨）
```bash
# .zshrc または .bashrc
function gchat() {
  local prompt="$1"

  if [ -f "GEMINI.md" ]; then
    local rules=$(cat GEMINI.md)
    gemini chat "Project Rules:\n$rules\n\n---\nTask: $prompt"
  else
    gemini chat "$prompt"
  fi
}
```

**使用法**

```bash
gchat "タスク内容"
```

---

###  Google Antigravity
Antigravity はツールではなく、**「人間と AI が協働する統合環境（Workspace）」**だそう（Gemini 曰く）。

#### 立ち位置  
Code Assist、CLI、Jules（非同期エージェント）がすべて稼働する「場」

#### 適用方法  
プロジェクトルートに `GEMINI.md` を配置し、Agent モードを ON

#### 特徴  
ルートにある定義ファイルは **「プロジェクトの憲法」**として自動的に参照される傾向が最も高いです。

### ツール別役割と適用イメージ

| ツール | 役割（Role） | 実行主体 | GEMINI.md の読み込ませ方 |
|------|------------|--------|--------------------------|
| Code Assist | Copilot（副操縦士）<br />リアルタイムなコーディング支援 | 人間 + AI | Context への追加、または `.vscode` 設定 |
| CLI | Operator（実務者）<br />ターミナルでの自律操作 | AI（対話型） | `cat` で流し込み、または `gchat` 関数 |
| Jules | Worker（担当者）<br />GitHub Issue ベースの非同期作業 | AI（非同期） | リポジトリに含まれていれば自動参照 |
| Antigravity | Workspace（仕事場）<br />上記すべてが動く統合環境 | Team | ルートに置くだけ（環境全体がコンテキスト） |

## Gemini CLI セッション・履歴管理メモ
Gemini CLI (`gemini`) での会話履歴の保存・復元に関する主要コマンドです。

### 1. 対話中（インタラクティブモード）の操作
プロンプト入力時に以下のスラッシュコマンドを使用します。

| コマンド | 内容 |
| :--- | :--- |
| `/chat save <tag>` | 現在のセッションに名前（タグ）を付けて保存 |
| `/chat list` | 保存済みセッションの一覧を表示 |
| `/chat resume <tag>` | 指定したタグのセッションを読み込んで再開 |
| `/clear` | 現在の履歴をリセットし、新規セッションを開始 |
| `/stats` | 現在のトークン消費量などの統計を表示 |

### 2. 起動時のオプション
ターミナルから直接セッションを復元して起動する場合に使用します。

```bash
# 直近のセッションを再開
npx gemini --resume

# 特定のセッションIDやインデックスを指定して再開
npx gemini --resume <session_id_or_index>
