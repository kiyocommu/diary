# Diary 変更履歴

---

## 2026-07-23

### 初版作成（Dropbox 連携 日記アプリ）

**ファイル**: `output/diary/index.html`、`output/diary/manifest.webmanifest`、`output/diary/icons/`、`output/diary/diary-取扱説明書.html`

スマホの Chrome での利用を主とした、Dropbox 連携の 1日1記事 Markdown 日記アプリを新規作成した。

#### 機能

- **Dropbox 公式 API 連携（OAuth 2 + PKCE、SDK 不使用の fetch 直叩き）**: 自分用アプリの App Key を設定 → 認可 → `token_access_type=offline` でリフレッシュトークンを取得し、アクセストークンは期限前・401 時に自動更新。`Dropbox-API-Arg` ヘッダーの日本語パスは `\uXXXX` エスケープで対応。
- **フォルダ選択**: `files/list_folder` によるフォルダブラウズ（パンくず付き）とパス直接入力の併用。
- **1日1記事**: 「＋ 今日の日記」で `YYYY-MM-DD.md` を新規作成（既にあればその記事を開く）。一覧は日付降順、タグチップで絞り込み。
- **フロントマター自動管理**: `created`（新規時に当日を自動入力）/ `tags`（チップ入力）/ `author`（設定値）。編集画面には本文のみ表示し、保存時にフロントマターを組み立てて書き込む。未知キーは raw 行のまま保持して書き戻す（Obsidian 互換）。インライン `tags: [a, b]` 形式・フロントマターなしファイルも読み込み可。
- **競合安全**: 保存は `files/upload` の `mode: update`（rev 指定）。Dropbox 側が先に変更されていた場合は競合ダイアログ（「Dropbox 側を確認」/「自分の編集で上書き」）を表示し、無警告上書きはしない。
- **下書き自動保存**: 編集中の本文・タグを 1 秒 debounce で localStorage に保存。保存成功でクリア、次回起動・再編集時に復元を提案。
- **キャッシュ差分同期**: `diary_cache` に rev + 本文を保持し、起動時は rev が変わったファイルだけを再取得（並列 4 に制限）。
- **UI**: モバイルファーストの単一カラム画面遷移型（一覧 / 閲覧 / 編集）。ブラウザバック対応（history API）。デスクトップ幅では中央寄せ。cyberpunk-design-spec 準拠のダークテーマ + `body.light` のライトテーマ。閲覧は `DOMPurify.sanitize(marked.parse())` + highlight.js。タグ入力に IME 変換中ガードあり。
- **PWA**: manifest + アイコン一式（Service Worker なし。ai-blog と同じ構成）。

#### 設定・データ

- localStorage キー: `diary_dbx_app_key` / `diary_dbx_tokens` / `diary_folder_path` / `diary_author`（初期値 kiyohito）/ `diary_default_tags`（初期値 ["日記"]）/ `diary_theme` / `diary_cache` / `diary_draft`。すべて try/catch + 型正規化付きで読み込む。
- `APP_VERSION = '2026-07-23.initial'`

#### リポジトリ側の変更

- `.claude/launch.json` に `diary`（ポート 8909、Node インライン静的サーバー方式）を追加。
- `tools/check-js.mjs` の `DEFAULT_TARGETS` に `output/diary/index.html` を追加。

#### 検証

- `node tools/check-js.mjs` 全 48 ファイル構文エラーなし。
- プレビュー（http://localhost:8909）+ モバイル幅 375×812 で確認: 未連携時の空状態と設定導線、モックデータでの一覧（TODAY バッジ・タグフィルタ・抜粋）、閲覧（メタバッジ・見出し・コード・引用の描画）、編集（フロントマター非表示・タグチップの Enter 追加・× 削除）、下書き自動保存と復元ダイアログ、ブラウザバックでの画面遷移（編集→閲覧→一覧）、ダーク / ライト両テーマ、デスクトップ幅の中央寄せ。コンソールエラーなし。
- フロントマター parse/serialize は実ファイル（2026-07-22.md 相当の形式）でラウンドトリップ一致・未知キー保持・日本語パスのヘッダーエスケープをコンソールで確認。
- **未確認**: 実際の Dropbox App Key での OAuth 認可〜実フォルダの読み書き・競合ダイアログ・トークン自動リフレッシュ（ユーザーの App Console 設定が必要なため）。手順は取扱説明書 2〜3 章を参照。
