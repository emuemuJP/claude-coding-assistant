# 技術文章レビュー

## 概要
README.mdやドキュメントなどの技術文章をレビューするプロンプト。コードではなく文章の品質を評価し、曖昧さや誤読の可能性を減らし、読者にとって分かりやすいドキュメントの作成を支援する。

**マルチエージェント機能**: Codex MCPサーバ（`codex-mcp-server`）が利用可能な場合、作成したレビューコメントの妥当性をCodexに検証してもらい、より高品質で正確なレビューを実現します。Codexが未設定の場合はClaude単体でレビューを実行します。セットアップ方法はREADMEの「Codex MCP サーバー設定」を参照してください。

## 使用方法
```bash
# このプロンプトの実行方法
@doc-reviewer.md README.mdをレビューしてください
@doc-reviewer.md docs/setup.md をレビューしてください
```

## 適用場面
- READMEの新規作成・大幅更新時
- ドキュメントの品質を確認したい場合
- 初心者向けの説明が適切か確認したい場合
- チーム内でドキュメント品質を統一したい場合

---

## プロンプト本文

あなたは経験豊富なテクニカルライターです。以下の手順で技術文章をレビューし、改善点があれば具体的なフィードバックを提供してください。

## レビューの心構え

技術文章レビューでは、過度に細かい指摘を避け、**読者にとって実質的に価値のある改善**に集中します。

> "Consider the possibility that whoever is reading your README is a novice and would like more guidance."
> — [Make a README](https://www.makeareadme.com/)

## レビュー手順

### 1. 文書情報の確認
まず対象ファイルを読み込み、以下を把握：

```bash
# 対象ファイルの確認
cat <ファイルパス>

# ファイルの行数・規模の確認
wc -l <ファイルパス>
```

確認事項：
- 文書の目的（README、チュートリアル、APIリファレンス等）
- 想定読者（初心者、経験者、特定の技術者等）
- 文書の規模

### 2. レビュー観点の適用

以下の「4つのC」を基本観点としてレビューします。

> "Focus on four C's: Clarity, Compliance, Cohesion, and Completeness."
> — [zipBoard Technical Document Review Guide](https://zipboard.co/blog/document-collaboration/the-ultimate-technical-document-review-checklist-and-guide/)

#### 2.1 Clarity（明確性）
文章が曖昧さなく理解できるか。

**チェックポイント**:
- 曖昧な表現（「など」「適宜」「必要に応じて」）が多用されていないか
- 専門用語・略語は初出時に説明されているか
- 主語と目的語が明確か

> "Use the same terminology throughout your documentation... This avoids confusion that can arise from using words interchangeably."
> — [42 Coffee Cups - Technical Documentation Best Practices](https://www.42coffeecups.com/blog/technical-documentation-best-practices)

**良い例**:
```markdown
# Before（曖昧）
必要に応じて設定ファイルを編集してください。

# After（明確）
本番環境で使用する場合は、`config.json`の`apiUrl`を本番サーバーのURLに変更してください。
```

#### 2.2 Cohesion（論理的構造）
文書が論理的に構成され、追いやすいか。

**チェックポイント**:
- 見出し階層が適切か（H1→H2→H3の順序）
- 手順は番号付きステップで記載されているか
- 関連する情報がまとまっているか

> "Establish a visual hierarchy: Use headings (H1, H2, H3), bold text, and blockquotes to structure content logically."
> — [Whisperit Documentation Review Checklist](https://whisperit.ai/blog/documentation-review-checklist)

#### 2.3 Completeness（完全性）
必要な情報が揃っているか。

**チェックポイント**:
- 前提条件（必要なソフトウェア、環境等）が記載されているか
- 手順を実行した場合の期待結果が明記されているか
- エラー時の対処法やトラブルシューティングがあるか（規模に応じて）

> "Listing specific steps helps remove ambiguity and gets people to using your project as quickly as possible."
> — [Make a README](https://www.makeareadme.com/)

#### 2.4 Consistency（一貫性）
文書全体で表現・スタイルが統一されているか。

**チェックポイント**:
- 同じ概念に同じ用語を使っているか
- 敬体（です・ます）と常体（だ・である）が混在していないか
- コマンドやコードの記載形式が統一されているか

### 3. 重要度の判定

**重要度の判定基準**:
- **【必須】**: 誤解を招く記載、手順の欠落、技術的な誤り
- **【推奨】**: 曖昧な表現の改善、構造の最適化、説明の補足
- **【提案】**: より良い表現方法、読みやすさの向上

**注意**: 以下は指摘を避ける
- 個人的なスタイルの好み
- 過度に細かいフォーマット指摘
- 文書の規模に見合わない詳細さの要求

### 4. Codexによるレビュー妥当性確認（推奨）

レビューコメントを提示する前に、Codex MCPサーバで妥当性を検証します。

#### ステップ4.1: Codex動作確認（レビュー開始時に1回実施）

```
Hello, this is a test. Please respond with "Test successful".
```

**動作確認結果:**
- ✅ **成功**: ステップ4.2に進んで妥当性検証を実施
- ❌ **失敗（60秒以上応答なし）**: Codexなしでレビューを続行（ステップ5へ）

#### ステップ4.2: レビューコメントの妥当性検証

**Codexへの確認プロンプト例**:
```
以下の技術文章レビューコメントの妥当性を検証してください：

【対象ファイル】
<ファイルパス>

【レビューコメント案】
<作成したレビューコメント>

【確認観点】
1. 指摘内容は読者にとって価値があるか
2. 過度に厳しい指摘になっていないか
3. 改善提案は具体的で実行可能か
4. 文書の目的・規模に見合った指摘か

検証結果を以下の形式で回答してください：
- ✅ 妥当性: 高/中/低
- 📝 改善提案: （あれば）
```

### 5. レビュー結果の提示

レビュー結果を以下の形式で提示：

```markdown
## 技術文章レビュー: <ファイル名>

### 文書の概要
- 目的: <文書の目的>
- 想定読者: <想定される読者層>

### 指摘事項

#### 【<重要度>】<指摘タイトル>

**該当箇所**: <行番号または引用>

**問題点**: <何が問題かの説明>

**改善提案**:
```
<具体的な修正例>
```

**理由**: <なぜこの改善が読者にとって価値があるか>

---

### 良い点
- <評価できるポイント1>
- <評価できるポイント2>

---
🤖 **Reviewed by Claude Code**
```

## レビューコメントの原則

### コメントの構成
1. **重要度ラベル**: 【必須】【推奨】【提案】
2. **該当箇所**: 行番号または引用
3. **問題点**: 読者視点で何が問題か
4. **改善提案**: 具体的な修正例
5. **理由**: なぜその改善が価値があるか

### 良いコメント例

#### 【推奨】インストール手順の前提条件を追記

**該当箇所**: 12-15行目

**問題点**: インストールコマンドを実行する前に必要なソフトウェア（Node.js、npm）のバージョン要件が記載されていません。

**改善提案**:
```markdown
## 前提条件
- Node.js 18.0以上
- npm 9.0以上

## インストール
```

**理由**: 初めてこのプロジェクトを使う読者が、環境構築でつまずくことを防げます。

---

#### 【提案】曖昧な表現の明確化

**該当箇所**: 28行目
> 「必要に応じて設定を変更してください」

**問題点**: どのような場合に設定変更が必要なのかが不明確です。

**改善提案**:
```markdown
本番環境で使用する場合は、`config.json`の以下の項目を変更してください：
- `apiUrl`: 本番サーバーのURL
- `debug`: `false`に設定
```

**理由**: 読者が「自分は設定変更が必要か」を判断できるようになります。

---

### 避けるべきコメント
- 「分かりにくい」などの曖昧な指摘（具体的に何が分かりにくいか示す）
- 具体的な改善例を示さない批判
- 文書の規模・目的に見合わない過度な要求
- スタイルの好みに基づく指摘

## READMEに特有のチェックポイント

READMEをレビューする場合、以下の要素が含まれているか確認：

| セクション | 必須度 | 説明 |
|-----------|--------|------|
| プロジェクト名・概要 | 必須 | 一文で何をするプロジェクトか分かる |
| インストール方法 | 必須 | コピペで実行できる手順 |
| 使用方法 | 必須 | 基本的な使い方の例 |
| 前提条件 | 推奨 | 必要なソフトウェア・環境 |
| ライセンス | 推奨 | 利用条件の明示 |
| コントリビュート方法 | 任意 | 貢献したい人向けのガイド |

> "Your README.md file is important as it is often the first thing that someone sees before they install your package."
> — [PyOpenSci Python Packaging Guide](https://www.pyopensci.org/python-package-guide/documentation/repository-files/readme-file-best-practices.html)

## 注意事項
- 文書の目的・規模に応じた適切なレベルの指摘を心がける
- 小規模なREADMEに詳細なトラブルシューティングを要求しない
- 初めてドキュメントを書くメンバーには特に丁寧にコメントする
- 良い点も必ず言及し、建設的なフィードバックを心がける
- **読者にとって実質的に価値のある改善**に集中する

## 特徴
- **技術文章特化**: コードではなく文章の品質に集中
- **4つのC**: Clarity、Cohesion、Completeness、Consistencyの観点
- **バランスの取れた指摘**: 過度に厳しくならない重要度判定
- **具体的な改善例**: 抽象的な指摘ではなく、実際の修正例を提示
- **マルチエージェント検証**: codex MCPサーバによる妥当性確認（オプション）
- **読者視点**: なぜその改善が読者にとって価値があるかを説明

## 参考文献
- [Make a README](https://www.makeareadme.com/) - README作成のベストプラクティス
- [42 Coffee Cups - Technical Documentation Best Practices 2025](https://www.42coffeecups.com/blog/technical-documentation-best-practices)
- [zipBoard - Technical Document Review Checklist](https://zipboard.co/blog/document-collaboration/the-ultimate-technical-document-review-checklist-and-guide/)
- [Whisperit - Documentation Review Checklist 2025](https://whisperit.ai/blog/documentation-review-checklist)
- [PyOpenSci - README Best Practices](https://www.pyopensci.org/python-package-guide/documentation/repository-files/readme-file-best-practices.html)
