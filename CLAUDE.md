# CLAUDE.md — qiita-frontend-digest

フロントエンドの最新ニュースを毎日収集し、日付ごとのMarkdownとして保存しつつQiitaに自動投稿するOSSツール。

設計の全体像と背景は `docs/DESIGN.md` を必ず先に読むこと。

## 技術スタック

- TypeScript(strict)+ Node.js、パッケージマネージャは pnpm
- 実行環境: GitHub Actions(cron、JST朝 = UTC 22時)
- 外部API: Qiita API v2(投稿)、各種RSS、GitHub Releases API

## アーキテクチャ原則

- レイヤー構成は Collector → 正規化・重複排除 → Filter/Classifier → Renderer → Publisher の一方向。逆方向の依存を作らない
- Collector と Publisher はインターフェース(`src/collectors/types.ts`、`src/publish/types.ts`)を実装するプラグインとして追加する。既存実装を分岐で太らせない
- 「何を拾うか」は `TOPICS.md`、「どう見せるか」は `src/render/` のテンプレート。責務を混ぜない。TOPICS.md から render 層へ渡すのはセクション名の一覧のみ
- LLM要約・LLM分類はオプトイン。APIキーなしでもキーワードマッチで全機能が動くこと

## コーディング規約

- named export のみ。各ディレクトリに `index.ts`(バレル)を置く
- 外部I/O(fetch、fs)は各レイヤーの端に寄せ、コアロジックは純粋関数に保つ
- エラー時は throw で落とすのではなく Result 型 or 明示的な失敗ログ。1つのフィード取得失敗で全体を止めない
- テストは vitest。Collector追加時はフィクスチャ(実レスポンスの縮小版)を `tests/fixtures/` に置く

## 絶対にやらないこと

- `QIITA_TOKEN` やAPIキーをコード・config・テストに直書きしない(GitHub Secrets / 環境変数のみ)
- `config.yaml` をコミットしない(`config.yaml.example` を更新する)
- Qiita投稿のデフォルトを `private: false` に変えない(公開は人間の明示的な設定変更でのみ行う)
- `state.json` と `articles/` の手動書き換えをコードから前提にしない

## 出力ファイルの規約

- 記事アーカイブ: `articles/YYYY/MM/YYYY-MM-DD.md`
- 既出URL管理: `state.json`(ワークフロー内でコミットバック)
- スキップログ: `logs/skipped/YYYY-MM-DD.json`(TOPICS.md育成の材料)
