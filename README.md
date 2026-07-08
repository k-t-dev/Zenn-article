# Zenn Content

ZennとGitHubを連携して記事を管理するためのリポジトリです。

このプロジェクトでは、Zennに投稿する記事Markdownに加えて、記事化に使えるチャット、記事アイデア、公開状況も管理します。

## 構成

- `articles/`: Zennの記事Markdownを置く場所
- `books/`: Zennの本を置く場所
- `chats/`: CodexやChatGPTで話したZenn関連チャットの要約、参照元、次アクション
- `notes/`: 調査メモ、参考リンク、記事化前の断片メモ

## Zenn記事の使い方

1. `articles/` に記事Markdownを追加します。
2. `published: false` のままGitHubへpushすると下書きとして管理できます。
3. 公開する準備ができたら、記事のFront Matterを `published: true` に変更します。

## ローカルプレビュー

Zenn CLIを使える環境では、次のコマンドでプレビューできます。

```bash
npx zenn preview
```

## 管理フロー

1. Zennに関するチャットを見つけたら `chats/zenn-chat-index.md` に記録する
2. 記事化できそうな内容は `articles/backlog.md` に追加する
3. Zenn記事として扱うものは `articles/<slug>.md` に保存する
4. 公開済みになったら `articles/published.md` にも記録する

## 現在の確認状況

2026-07-08時点で、Codexスレッド一覧とローカルセッションログから `Zenn` / `Zeen` / `zenn.dev` を検索しました。

確認できたZenn関連スレッドは、現在のこのプロジェクト作成スレッドのみです。過去アーカイブ由来の具体的なZenn関連チャットは、今回の検索範囲では見つかっていません。
