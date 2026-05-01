# YouTube 記事変換スキル

Hermes Agent 用の v3.2 スキル定義。YouTube 動画を構造化記事に変換する FastAPI Web アプリのワークフローをエンコードします。

## パイプライン概要

1. **抽出**: yt-dlp でメタデータと VTT 字幕を取得、9言語優先順位マッチング
2. **洗浄**: WEBVTT ヘッダー、タイムスタンプ、HTML タグを削除、重複行除去
3. **統合再構築**: Google Gemini Flash がテーマブロック構造 (H1/H2/H3) に再構成、データと引用を保持

## v3.2 の強化ポイント

- スマート URL 解析：カンマ、スペース、改行区切り対応
- マルチ結果タブ表示
- 4言語 UI（日本語、English、繁體中文、简体中文）、localStorage 永続化
- Ctrl+Enter ショートカットキー
- Map-Reduce 分割処理（>1時間の長編動画対応）
- Toast 通知（alert() 置き換え）

## リポジトリ構成

- **SKILL.md**: コア定義（トリガー条件、パイプライン手順、フロントエンド設計、i18n、API エンドポイント、落とし穴、検証基準）
- **references/common-failures.md**: トラブルシューティングガイド
- **references/apple-notes-export.md**: Apple Notes エクスポート用 osascript パターン

## 使用方法

SKILL.md を Hermes Agent のスキルライブラリにインポート。ユーザーが YouTube URL と記事変換をリクエストすると、エージェントが自動的にワークフローを実行します。

## ライセンス

MIT
