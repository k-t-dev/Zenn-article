# Zenn Content

ZennとGitHubを連携して記事を管理するためのリポジトリです。

## 構成

- `articles/`: Zennの記事Markdownを置く場所
- `books/`: Zennの本を置く場所

## 使い方

1. `articles/` に記事Markdownを追加します。
2. `published: false` のままGitHubへpushすると下書きとして管理できます。
3. 公開する準備ができたら、記事のFront Matterを `published: true` に変更します。

## ローカルプレビュー

Zenn CLIを使える環境では、次のコマンドでプレビューできます。

```bash
npx zenn preview
```

