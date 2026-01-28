# Claude Code コーディングアシスタント

Claude Codeでコードレビューを自動化するプラグインです。

## 特徴
- **自動コードレビュー**: GitHub CLIを使用してPRを分析し、体系的なレビューを実行
- **PR記載レビュー**: PRタイトル・説明文・テンプレート準拠をチェックし、ドキュメント品質を向上
- **技術文章レビュー**: README等のドキュメントの曖昧さ・誤読の可能性をチェック
- **Codex連携**: Codex MCPサーバによるマルチエージェント検証で高品質なレビューを実現
- **体系的分析**: PRテンプレートに基づく統一されたレビュー観点
- **GitHub統合**: レビューコメントを直接GitHubに投稿

---

## 🚀 インストール

### 1. マーケットプレイスを追加
```
/plugin marketplace add https://github.com/emuemuJP/claude-coding-assistant 
```

### 2. プラグインをインストール
```
/plugin install coding-assistant@emuemuJP/claude-coding-assistant
```

### 3. Claude Code を再起動

### 4. テスト
```
/coding-assistant:code-reviewer PR#123をレビューしてください
```

---

## 📝 使い方

### コードレビュー
```
/coding-assistant:code-reviewer PR#<番号>をレビューしてください
```

プラグインは以下の処理を自動実行：
1. PR情報の取得と分析
2. ファイル別の詳細レビュー
3. Codexによる妥当性検証（オプション）
4. GitHubへのコメント投稿
5. レビューサマリーの作成

### PR記載レビュー
```
/coding-assistant:pr-description-reviewer PR#<番号>をレビューしてください
```

PRのコードではなく、記載内容（タイトル、説明文、テンプレート準拠）をレビュー：
1. PRタイトルの明確さ・簡潔さをチェック
2. 説明文の目的・背景・影響範囲を評価
3. PRテンプレート必須項目の記入漏れを検出
4. テスト計画・関連Issue・スクリーンショットの有無を確認
5. 具体的な改善例を提示してGitHubにコメント投稿

### 技術文章レビュー
```
/coding-assistant:doc-reviewer README.mdをレビューしてください
```

README等の技術文章の品質をレビュー（4つのCの観点）：
1. **Clarity（明確性）**: 曖昧な表現がないか、用語が統一されているか
2. **Cohesion（論理的構造）**: 見出し階層、手順の記載が適切か
3. **Completeness（完全性）**: 必要な情報（前提条件、手順、期待結果）が揃っているか
4. **Consistency（一貫性）**: 表現・スタイルが統一されているか

### 前提条件
- GitHub CLI（gh）がインストール・認証済みであること
- **Codex 連携（オプション）**: Codex MCP サーバが利用可能な場合、マルチエージェント検証により高品質なレビューが可能（セットアップ方法は後述の「Codex MCP サーバー設定」を参照）

---

## 🔧 Codex MCP サーバー設定

このプラグインは [OpenAI Codex CLI](https://github.com/openai/codex) の MCP サーバー機能を使用し、マルチエージェントレビュー（Claude + Codex による二重検証）を実現します。Codex が利用できない場合でも、Claude 単体でレビューは正常に動作します（Codex 検証はオプション機能）。

### Codex とは

[OpenAI Codex CLI](https://github.com/openai/codex) は、OpenAI が提供するオープンソースのコーディングエージェントです。ターミナル上で動作し、コードの読み書き・実行が可能です。このプラグインでは、Codex を **MCP（Model Context Protocol）サーバー** として起動し、Claude からのレビュー検証リクエストに応答させます。

### セットアップ手順

#### 1. Codex CLI のインストール

以下のいずれかの方法でインストールしてください。

```bash
# 方法1: npm（推奨）
npm install -g @openai/codex

# 方法2: Homebrew（macOS）
brew install --cask codex

# 方法3: GitHub Releases からバイナリをダウンロード
# https://github.com/openai/codex/releases
# ダウンロード後、バイナリを PATH の通ったディレクトリに配置
```

インストール確認:
```bash
codex --version
# codex-mcp-server バイナリも同梱されていることを確認
codex-mcp-server --help
```

#### 2. OpenAI API キーの設定

Codex の動作には OpenAI API キーが必要です。

**方法A: ChatGPT アカウントでログイン（Plus/Pro/Team/Edu/Enterprise プランの場合、追加コスト不要）**
```bash
codex
# 起動後、"Sign in with ChatGPT" を選択してブラウザ認証
```

**方法B: API キーを直接設定**
```bash
# 環境変数として設定
export OPENAI_API_KEY="sk-your-api-key-here"

# または Codex CLI にログイン
printenv OPENAI_API_KEY | codex login --with-api-key
```

API キーは [OpenAI Platform](https://platform.openai.com/api-keys) で取得できます。

#### 3. Codex 動作確認

```bash
# 対話モードで起動テスト
codex

# 非対話モード（CI/CD向け）で簡単なテスト
codex exec "echo hello"
```

#### 4. MCP サーバーの動作確認

```bash
# codex-mcp-server が起動するか確認（Ctrl+C で終了）
codex-mcp-server
```

`codex-mcp-server` は `codex` CLI に同梱されるバイナリで、Codex エージェントを MCP ツールとして外部から利用可能にします。JSON-RPC 2.0 over stdin/stdout で通信し、以下の 2 つの MCP ツールを公開します:

| ツール名 | 説明 |
|---------|------|
| `codex` | 新しい Codex セッションを開始 |
| `codex-reply` | 既存のセッションに続けてメッセージを送信 |

#### 5. プラグインの MCP 設定

プラグインの MCP サーバー設定は `.claude-plugin/plugin.json` に含まれています:

```json
{
  "name": "coding-assistant",
  "mcpServers": {
    "codex": {
      "type": "stdio",
      "command": "codex-mcp-server",
      "args": []
    }
  }
}
```

#### 6. Claude Code の再起動

プラグインの MCP サーバーを有効にするには **Claude Code の再起動が必要** です。

### トラブルシューティング

| 症状 | 原因 | 対処法 |
|------|------|--------|
| `codex: command not found` | Codex CLI 未インストール | `npm install -g @openai/codex` を実行 |
| `codex-mcp-server: command not found` | npm 版に未同梱の場合あり | [GitHub Releases](https://github.com/openai/codex/releases) からバイナリを取得 |
| Codex テストで 60 秒以上応答なし | API キー未設定 or ネットワーク問題 | `OPENAI_API_KEY` が設定済みか確認。`codex exec "hello"` で単体テスト |
| MCP サーバー接続失敗 | Claude Code 未再起動 | Claude Code を再起動 |
| レビュー中に Codex 検証がスキップされる | Codex MCP が利用不可 | 上記手順で Codex の動作を確認。レビュー自体は Claude 単体で正常動作 |

### 公式ドキュメント

- [Codex CLI 概要](https://developers.openai.com/codex/cli/)
- [Codex クイックスタート](https://developers.openai.com/codex/quickstart/)
- [Codex MCP 設定](https://developers.openai.com/codex/mcp/)
- [Codex 設定リファレンス](https://developers.openai.com/codex/config-reference/)
- [Codex GitHub リポジトリ](https://github.com/openai/codex)

---

## 📁 ファイル構成

```
claude-coding-assistant/
├── .claude-plugin/
│   ├── plugin.json          # プラグイン設定（MCP設定含む）
│   └── marketplace.json     # マーケットプレイス設定
├── commands/
│   ├── pr-reviewer.md       # PR一括レビューコマンド定義（統合）
│   ├── code-reviewer.md     # コードレビューコマンド定義
│   ├── pr-description-reviewer.md  # PR記載レビューコマンド定義
│   └── doc-reviewer.md      # 技術文章レビューコマンド定義
├── README.md
└── LICENSE
```

---

## 🎯 レビュー観点

- **機能性**: 実装が要件を満たしているか
- **可読性**: コードが理解しやすいか、命名は適切か
- **保守性**: 将来の変更に対して柔軟性があるか
- **パフォーマンス**: 効率的な実装になっているか
- **セキュリティ**: セキュリティリスクはないか
- **テスト**: 適切なテストが含まれているか
- **ドキュメント**: 必要な文書化がされているか

---

## GitHub Actions による自動PRレビュー

PRが作成されるたびに自動でClaudeによるレビューを実行できます。

### セットアップ手順

#### 1. Anthropic API Keyをシークレットに登録

リポジトリの Settings > Secrets and variables > Actions で以下を追加:

| シークレット名 | 値 |
|--------------|-----|
| `ANTHROPIC_API_KEY` | Anthropic APIキー |

#### 2. ワークフローファイルをコピー

`.github/workflows/pr-review.yml` をあなたのリポジトリにコピーしてください。

```bash
# リポジトリのルートで実行
mkdir -p .github/workflows
curl -o .github/workflows/pr-review.yml \
  https://raw.githubusercontent.com/emuemuJP/claude-coding-assistant/main/.github/workflows/pr-review.yml
```

#### 3. 動作確認

PRを作成すると、自動的にClaude Codeがレビューを実行し、コメントを投稿します。

### カスタマイズ

ワークフローファイル内のレビュープロンプトを編集することで、レビュー観点をカスタマイズできます。

### 注意事項

- **APIコスト**: PRごとにAnthropic APIが呼び出されます。大きなPRほどトークン消費が増えます
- **タイムアウト**: デフォルトのGitHub Actionsタイムアウトは6時間です
- **大規模PR**: 10,000行または500KBを超える差分は自動レビューがスキップされます
- **フォークからのPR**: パブリックリポジトリでは、フォークからのPRでシークレットにアクセスできません。`pull_request_target`イベントは**使用しないでください**（悪意あるPRから任意コード実行のリスクがあります）。代わりに、メンテナーによる手動レビューまたはラベルベースのトリガーを検討してください

---

## 📄 ライセンス

MIT License
