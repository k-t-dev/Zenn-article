---
title: "CodexでZenn投稿用リポジトリを作る"
emoji: "📝"
type: "tech"
topics: ["zenn", "git", "github", "codex"]
published: false
---

## 結論

Codexを使って、ZennとGitHub連携用のリポジトリをローカルに作成できました。

## 理由

ZennのGitHub連携では、記事をMarkdownファイルとしてGitリポジトリで管理します。  
そのため、最初に `articles/` ディレクトリと記事ファイルを用意しておくと、次の流れで投稿作業を進められます。

1. ローカルで記事を書く
2. Gitで変更履歴を管理する
3. GitHubへpushする
4. Zenn側でGitHubリポジトリと連携する

## 今回作ったもの

今回作成した主な構成は次の通りです。

```text
.
├── README.md
├── articles/
│   └── 20260708-codex-zenn-setup.md
└── books/
```

`articles/` にはZennの記事を置きます。  
`books/` にはZennの本を作る場合のファイルを置きます。

## 記事ファイルの基本形

Zennの記事は、Markdownの先頭にFront Matterを書きます。

```markdown
---
title: "記事タイトル"
emoji: "📝"
type: "tech"
topics: ["zenn", "git"]
published: false
---
```

`published: false` にしておくと、下書きとして扱えます。  
公開する準備ができたら `published: true` に変更します。

## 次にやること

GitHubに空のリポジトリを作成し、このローカルリポジトリをpushします。  
その後、Zennの管理画面からGitHubリポジトリを連携すると、Zenn上で記事を管理できます。

