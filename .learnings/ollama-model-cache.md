# Ollama モデルキャッシュ

### 巨大な Actions キャッシュ `Linux-ollama-gemma4-e4b` を削除しない

**学び**: リポジトリの Actions キャッシュ一覧に約 7.2GB の `Linux-ollama-gemma4-e4b` が存在するが、これはゴミではなく `.github/workflows/gemini-summarize.yml` が使う gemma4:e4b モデルの実体である。削除すると次回の gemini-summarize 実行時にモデル全体を再ダウンロードすることになる。

**根拠**: 2026-07-27、キャッシュ復元エラーの調査中に「リポジトリのキャッシュ使用量 10GB のうち約 7.2GB を1エントリが占有している」ことに気づき、不要物と誤認しかけた。`grep -rn 'ollama' .github/workflows/` で `gemini-summarize.yml` の `Cache Ollama model` ステップ（`path: ~/.ollama` / `key: ${{ runner.os }}-ollama-gemma4-e4b`）が作成元と判明した。

**適用範囲**: このリポジトリの Actions キャッシュを整理・削除しようとするとき。サイズが大きいエントリを見つけた場合は、削除前に必ず `grep` でキー名の作成元ワークフローを特定する。

**例外・前提**: このキャッシュは最終アクセスから7日でいずれ自動削除され、次回実行時に再生成される。したがって「容量を空けたい」という理由だけで手動削除する意味は薄い。gemini-summarize.yml から Ollama 利用がなくなった場合はこの学びは無効になる。
