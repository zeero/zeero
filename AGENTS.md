# プロジェクト概要

このリポジトリは **zeero** の GitHub プロフィールです。[`README.md`](README.md) では Swift & iOS、Flutter、Ruby、Vim Script などの興味分野と、RSSMagick や Flutter Icon Search といったプロジェクトへのリンクが紹介されています。

## GitHub ワークフロー

リポジトリには三つのワークフローがあり、いずれも **google-gemini/gemini-cli-action** を利用しています。

### Gemini Slack Summary
- [`gemini-slack-summary.yml`](.github/workflows/gemini-slack-summary.yml) に定義
- 毎日と `workflow_dispatch` で実行
- `SLACK_BOT_TOKEN`、`SLACK_TEAM_ID`、`GEMINI_API_KEY` を設定して `google-gemini/gemini-cli-action@main` を呼び出す
- `settings_json` で MCP の Slack サーバーを起動し `mcp__slack__slack_get_channel_history` を利用
- `#rss-tech_blog` `#rss-hotentry` `#rss-ai` `#rss-ios` のメッセージを要約し `#general` に投稿

### Gemini Bluesky Summary
- [`gemini-bluesky-summary.yml`](.github/workflows/gemini-bluesky-summary.yml) に定義
- 毎日と `workflow_dispatch` で実行
- Slack サーバーを起動した上で `curl` を使い、Bluesky ページ `https://bsky.app/profile/did:plc:a3vurmxtfdiehvyefas744uc/feed/aaalzlquduqha` を取得
- ページから昨日投稿された内容をいいね数の多い順に抽出し、メッセージ本文をそのまま `#general` に投稿

### Update App Store Review Guidelines Gist
- [`update-gist.yml`](.github/workflows/update-gist.yml) に定義
- 毎日と手動で実行
- 同様に `google-gemini/gemini-cli-action@main` を使用
- `#rss-apple` の履歴からガイドライン更新を検知し、見つかった場合のみ pandoc と GitHub CLI をインストールしてガイドラインを Gist に更新

## gemini-cli-action の仕様

`google-gemini/gemini-cli-action@main` は [Gemini CLI](https://github.com/google-gemini/gemini-cli) を GitHub Actions から利用するためのアクションです。`settings_json` で MCP サーバーやツールを定義することで、Slack の履歴取得やシェルコマンド実行を組み合わせた複雑な処理を行えます。主な設定項目は次のとおりです。

- **`GEMINI_API_KEY`**: Gemini API キーを入力として渡します。
- **`prompt`**: Gemini への指示文。複数行にわたる YAML テキストを指定可能です。
- **`settings_json`**: MCP サーバーや `coreTools` をまとめた JSON。ここで Slack サーバー起動コマンドや利用したい MCP ツールを定義します。
- **`project_root`**: オプション。Gemini CLI を実行する作業ディレクトリを指定できます。

### ステップ設定例

```yaml
- name: Run Gemini CLI
  uses: google-gemini/gemini-cli-action@main
  env:
    SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
    SLACK_TEAM_ID: ${{ secrets.SLACK_TEAM_ID }}
  with:
    GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
    prompt: |
      Slack の #rss-tech_blog から直近24時間の投稿を取得し、
      重要なトピックをまとめてください。
      要約は日本語で 4 行以内としてください。
    settings_json: |
      {
        "mcpServers": {
          "slack": {
            "command": "npx",
            "args": ["-y", "@modelcontextprotocol/server-slack"],
            "env": {
              "SLACK_BOT_TOKEN": "$SLACK_BOT_TOKEN",
              "SLACK_TEAM_ID": "$SLACK_TEAM_ID"
            }
          }
        },
        "coreTools": [
          "mcp__slack__slack_get_channel_history",
          "run_shell_command(echo)"
        ]
      }
```

`coreTools` に `mcp__slack__slack_get_channel_history` を指定すると Slack チャンネル履歴を取得できます。`run_shell_command(echo)` のようにシェルコマンドを許可するツールを追加すれば、Gemini の出力に応じて任意のコマンドを実行することも可能です。

## MCP の仕様

Model Context Protocol (MCP) は Gemini のコンテキストから外部サービスに安全にアクセスするための仕組みです。Slack に接続する場合、次のようなサーバーを起動します。

- `npx -y @modelcontextprotocol/server-slack`

サーバー起動時には `SLACK_BOT_TOKEN` と `SLACK_TEAM_ID` を環境変数として渡します。`settings_json` ではこのサーバー起動方法を `mcpServers.slack` として定義し、利用したい MCP ツールを `coreTools` に列挙します。たとえば、`mcp__slack__slack_get_channel_history` を指定すると Slack チャンネル履歴が取得でき、`run_shell_command(echo)` を追加するとシェルコマンドを実行できます。

