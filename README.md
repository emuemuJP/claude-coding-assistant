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
- codex MCPサーバが利用可能な場合、より高品質なレビューが可能

---

## 🔧 MCP サーバー設定

このプラグインはCodex MCPサーバを使用します。設定は`.claude-plugin/plugin.json`に含まれています。

```json
{
  "name": "coding-assistant",
  "mcpServers": {
    "codex": {
      "type": "stdio",
      "command": "codex",
      "args": ["mcp-server"]
    }
  }
}
```

### 前提条件
- `codex`コマンドがインストール済みであること
- インストール方法: [Codex公式ドキュメント](https://docs.codexmcp.dev/)
- プラグインのMCPサーバーを有効にするには **Claude Codeの再起動が必要** です

---

## 📁 ファイル構成

```
claude-coding-assistant/
├── .github/
│   ├── workflows/
│   │   └── pr-review.yml    # 自動PRレビューワークフロー
│   └── copilot-instructions.md  # GitHub Copilotレビュー設定
├── .claude-plugin/
│   ├── plugin.json          # プラグイン設定（MCP設定含む）
│   └── marketplace.json     # マーケットプレイス設定
├── commands/
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

PRが作成されるたびに自動でレビューを実行できます。**GitHub Copilot**と**Claude Code**から選択可能です。

### レビュワーの選択

PRラベルでレビュワーを切り替えられます:

| ラベル | レビュワー | 特徴 |
|--------|-----------|------|
| （なし） | **GitHub Copilot**（デフォルト） | 高速・低コスト、標準的なレビュー |
| `review:claude` | **Claude Code** | 詳細な分析、カスタム観点での深いレビュー |
| `review:skip` | スキップ | 自動レビューを実行しない |

#### 各レビュワーの比較

| 観点 | GitHub Copilot | Claude Code |
|------|---------------|-------------|
| コスト | Copilotライセンスに含まれる | API従量課金 |
| レビュー速度 | 高速 | やや遅い |
| カスタマイズ | `copilot-instructions.md` | プロンプトで自由に定義 |
| 詳細度 | 標準的 | 詳細（重要度区分付き） |
| 推奨用途 | 日常的なPR、軽微な変更 | 重要な機能追加、セキュリティ関連 |

### セットアップ手順

#### 1. GitHub Copilot Code Reviewを有効化（推奨）

1. リポジトリの **Settings > General > Features** で **Copilot** を有効化
2. **Settings > Code review > Copilot** でCode Reviewを有効化

詳細: [GitHub Copilot Code Review ドキュメント](https://docs.github.com/copilot/using-github-copilot/code-review/using-copilot-code-review)

#### 2. Anthropic API Keyをシークレットに登録（Claude使用時のみ）

`review:claude` ラベルでClaudeを使用する場合は、リポジトリの Settings > Secrets and variables > Actions で以下を追加:

| シークレット名 | 値 |
|--------------|-----|
| `ANTHROPIC_API_KEY` | Anthropic APIキー |

#### 3. ワークフローファイルをコピー

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

### ラベルの作成

以下のラベルをリポジトリに作成してください:

```bash
# GitHub CLIを使用
gh label create "review:claude" --description "Claude Codeでレビュー" --color "8B5CF6"
gh label create "review:skip" --description "自動レビューをスキップ" --color "6B7280"
```

### 注意事項

- **Copilotライセンス**: GitHub Copilotを使用するには、組織またはユーザーにCopilotライセンスが必要です
- **APIコスト（Claude使用時）**: PRごとにAnthropic APIが呼び出されます。大きなPRほどトークン消費が増えます
- **タイムアウト**: デフォルトのGitHub Actionsタイムアウトは6時間です
- **大規模PR（Claude使用時）**: 10,000行または500KBを超える差分は自動レビューがスキップされます
- **フォークからのPR**: パブリックリポジトリでは、フォークからのPRでシークレットにアクセスできません。`pull_request_target`イベントは**使用しないでください**（悪意あるPRから任意コード実行のリスクがあります）。代わりに、メンテナーによる手動レビューまたはラベルベースのトリガーを検討してください

---

## 📄 ライセンス

MIT License
