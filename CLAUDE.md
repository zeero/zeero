# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

このリポジトリは zeero の GitHub プロフィールページです。主な機能は以下の GitHub Actions ワークフローです。

## GitHub Actions ワークフロー

### 利用可能なワークフロー

1. **Gemini Slack Summary** (`.github/workflows/gemini-slack-summary.yml`)
   - 複数の RSS チャンネルのメッセージを要約して #general に投稿
   - 必要なシークレット: `SLACK_BOT_TOKEN`, `SLACK_TEAM_ID`, `GEMINI_API_KEY`

2. **Gemini Bluesky Summary** (`.github/workflows/gemini-bluesky-summary.yml`)
   - Bluesky の投稿を取得・要約して Slack に投稿
   - 必要なシークレット: `SLACK_BOT_TOKEN`, `SLACK_TEAM_ID`, `GEMINI_API_KEY`

3. **Update App Store Review Guidelines Gist** (`.github/workflows/update-app-store-review-guidelines-gist.yml`)
   - Apple ガイドラインの変更を自動検知して Gist を更新
   - 必要なシークレット: `GH_TOKEN`, `GIST_ID`

### ワークフロー実行方法

```bash
# 手動実行
gh workflow run gemini-slack-summary.yml
gh workflow run gemini-bluesky-summary.yml
gh workflow run update-app-store-review-guidelines-gist.yml
```

## アーキテクチャ

このプロジェクトは以下の2つのアプローチを使用：

1. **Slack 連携ワークフロー** (Slack Summary, Bluesky Summary)
   - `google-gemini/gemini-cli-action` を中核として使用
   - MCP (Model Context Protocol) を通じて Slack と連携
   - **Slack サーバー**: `@modelcontextprotocol/server-slack`

2. **直接 API アクセスワークフロー** (App Store Guidelines)
   - GitHub Actions の標準機能のみを使用
   - `actions/cache` でハッシュ値をキャッシュして変更検知
   - pandoc と GitHub CLI で変換・更新処理

### 主要なツール

**Slack 連携用:**
- `mcp__slack__slack_get_channel_history`: Slack チャンネル履歴取得
- `mcp__slack__slack_post_message`: Slack メッセージ投稿
- `run_shell_command`: シェルコマンド実行

**App Store Guidelines 用:**
- `actions/cache`: ハッシュ値のキャッシュ
- `pandoc`: HTML から Markdown への変換
- `gh`: GitHub CLI による Gist 更新

## 開発時の注意点

- ワークフローファイルを編集する際は AGENTS.md の詳細仕様を参照
- Slack 連携ワークフローでは Gemini API の制限とコスト効率を考慮してプロンプトを最適化
- MCP サーバーの起動パラメータは settings_json 内で定義
- App Store Guidelines ワークフローはハッシュ比較による変更検知で API コストを削減

## コミュニケーション

- 受け答えは日本語で行う
  - 途中の思考は英語でもOK
- 各文の末尾に感情を表す絵文字をつけてください
