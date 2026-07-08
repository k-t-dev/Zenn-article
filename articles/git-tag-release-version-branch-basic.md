---
title: "GitのTag・Release・バージョン管理・ブランチ管理の基本"
emoji: "🔖"
type: "tech"
topics: ["git", "github", "versioning", "branch", "release"]
published: false
---

GitのPull Request、tag、Release、バージョン管理、ブランチ管理を、チーム開発で使う前提で整理します。

## 結論

Git運用は、まず次の役割で整理すると分かりやすいです。

| 用語 | 役割 |
| --- | --- |
| commit | 作業履歴を保存する単位 |
| branch | 作業場所 |
| Pull Request | レビュー依頼・mainへ取り込む相談 |
| main | 安定版のコード |
| tag | 特定のcommitに付けるバージョンの印 |
| Release | tagに説明文や配布物を付けた公開ページ |

理由は、チーム開発では「誰が何を変更したか」だけでなく、「どのコードを本番に出したか」「どの変更がどのバージョンに含まれるか」を追える必要があるからです。

## 1. Gitの基本的な流れ

まずは、基本の流れです。

```bash
git status
git add .
git commit -m "変更内容を説明するメッセージ"
git push
```

それぞれの意味は次の通りです。

| コマンド | 意味 |
| --- | --- |
| `git status` | 現在の変更状態を確認する |
| `git add .` | 変更をcommit対象に追加する |
| `git commit -m "..."` | 変更を履歴として保存する |
| `git push` | ローカルのcommitをGitHubなどへ送る |

commitは「作業の保存」です。

Pull Requestは「チームに見てもらう依頼」です。

この2つは役割が違います。

## 2. commitのタイミング

commitは、作業の小さな区切りで作ります。

良い例:

```bash
git commit -m "ログインAPIを追加"
git commit -m "ログイン画面を追加"
git commit -m "ログイン処理のテストを追加"
```

悪い例:

```bash
git commit -m "いろいろ修正"
```

commitメッセージは、あとから見た時に「何をしたか」が分かる内容にします。

## 3. ブランチ管理の基本

結論として、ブランチは「開発者ごと」ではなく「作業内容ごと」に作ることが多いです。

理由は、Pull Requestでレビューする単位が「誰が作ったか」ではなく「何を変更したか」だからです。

よく使うブランチ名:

| ブランチ | 用途 |
| --- | --- |
| `feature/...` | 新機能追加 |
| `fix/...` | バグ修正 |
| `docs/...` | ドキュメント修正 |
| `refactor/...` | 動作を変えないコード整理 |
| `test/...` | テスト追加・修正 |
| `chore/...` | 設定や依存関係の更新 |
| `hotfix/...` | 本番障害などの緊急修正 |

例:

```bash
git switch -c feature/user-login
git switch -c fix/password-error
git switch -c docs/readme-update
```

同じ開発者でも、作業内容が違えばブランチを分けます。

```text
開発者A -> feature/user-login
開発者A -> docs/readme-update
開発者B -> fix/payment-error
```

## 4. Pull Requestのタイミング

Pull Requestは、レビューしてもらえるまとまりになったタイミングで出します。

一般的な流れ:

```bash
git switch main
git pull
git switch -c feature/user-login
```

作業してcommitします。

```bash
git add .
git commit -m "ログイン機能を追加"
```

GitHubへpushします。

```bash
git push -u origin feature/user-login
```

その後、GitHub上でPull Requestを作ります。

```text
base: main
compare: feature/user-login
```

意味は「`feature/user-login` の変更を `main` に取り込みたい」です。

## 5. mainとは何か

`main` は、基本的に安定版のコードを置くブランチです。

チーム開発では、直接 `main` にcommitするのではなく、次のような流れにします。

```text
featureブランチ
  ↓
Pull Request
  ↓
レビュー
  ↓
mainへmerge
```

理由は、mainに直接作業すると、壊れたコードがそのまま本番や他の開発者に影響する可能性があるからです。

## 6. tagとは何か

tagは、特定のcommitに名前を付ける機能です。

例えば:

```bash
git tag v1.0.0
```

これは「今いるcommitを `v1.0.0` と呼ぶ」という意味です。

GitHubへ送るには:

```bash
git push origin v1.0.0
```

チーム開発や正式リリースでは、注釈付きタグを使うこともあります。

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

tagは「このコードを本番に出した」という固定点になります。

## 7. Releaseとは何か

Releaseは、GitHubなどが提供する機能です。

Gitの機能そのものはtagです。

GitHub Releaseは、そのtagに対して説明文やファイルを付けた公開ページです。

| 項目 | tag | Release |
| --- | --- | --- |
| 機能の場所 | Git | GitHubなど |
| 目的 | commitに名前を付ける | 変更内容を説明する |
| 説明文 | 基本なし | 書ける |
| ファイル添付 | なし | できる |
| 例 | `v1.0.0` | `v1.0.0` のリリースページ |

覚え方は次の通りです。

```text
tag = バージョンの印
Release = そのバージョンの説明ページ
```

## 8. 本番デプロイとtag / Release

本番デプロイ時に、必ずGitHub Releaseを作る必要はありません。

ただし企業では、tagやReleaseを使うことが多いです。

理由は、次の情報を追えるようにするためです。

- どのcommitを本番に出したか
- どのバージョンで障害が起きたか
- どのバージョンに戻せばよいか
- そのリリースにどのPull Requestが含まれているか

よくある流れ:

```text
Pull Requestをmainへmerge
  ↓
mainを最新化
  ↓
tagを付ける
  ↓
そのtagを本番へデプロイ
  ↓
Release notesを書く
```

コマンド例:

```bash
git switch main
git pull
git tag -a v1.1.0 -m "Release v1.1.0"
git push origin v1.1.0
```

## 9. 「どのPull Requestを本番に出したか」ではないのか

結論として、「どのPull Requestが本番に含まれるか」も見ます。

ただし、最終的には「どのcommit / tagを本番に出したか」で管理します。

理由は、本番にデプロイされる実体はPull Requestそのものではなく、Pull Requestがmergeされた後のcommitだからです。

例えば:

```text
PR #10 ログイン機能追加
PR #11 README更新
PR #12 決済バグ修正
```

これらがmainにmergeされ、`v1.2.0` のtagが付いたとします。

```text
v1.2.0 = commit D
commit D までに PR #10, #11, #12 が含まれる
```

つまり、管理の考え方はこうです。

```text
本番に出したもの: v1.2.0 tag / commit D
その中に含まれる変更: PR #10, PR #11, PR #12
```

Pull Requestは「何が入ったか」を説明する単位です。

tagやcommitは「実際にどのコードを出したか」を固定する単位です。

## 10. バージョン管理の基本

よく使われるバージョン番号は `v1.2.3` のような形式です。

```text
vメジャー.マイナー.パッチ
```

| 種類 | 例 | いつ上げるか |
| --- | --- | --- |
| メジャー | `v2.0.0` | 大きな変更、互換性が壊れる変更 |
| マイナー | `v1.3.0` | 新機能追加 |
| パッチ | `v1.2.1` | バグ修正、小さな修正 |

例:

```text
v1.0.0  最初の正式版
v1.1.0  新機能追加
v1.1.1  バグ修正
v2.0.0  大きな仕様変更
```

mainにmergeするたびに必ずバージョンを上げるわけではありません。

多くの場合、バージョンを上げるのはリリース単位です。

## 11. よく使う確認コマンド

現在の状態を見る:

```bash
git status
```

変更内容を見る:

```bash
git diff
```

commit対象に追加済みの差分を見る:

```bash
git diff --staged
```

履歴を見る:

```bash
git log --oneline
```

現在のブランチを見る:

```bash
git branch
```

リモートリポジトリを確認する:

```bash
git remote -v
```

tag一覧を見る:

```bash
git tag
```

tagの内容を見る:

```bash
git show v1.0.0
```

前回リリースから今回リリースまでのcommitを見る:

```bash
git log v1.0.0..v1.1.0 --oneline
```

前回リリースから今回リリースまでの差分を見る:

```bash
git diff v1.0.0..v1.1.0
```

## 12. まとめ

Git運用は、次のように覚えると整理しやすいです。

```text
commit = 作業履歴
branch = 作業場所
Pull Request = レビュー依頼
main = 安定版
tag = リリースするコードの固定点
Release = 変更内容を説明するページ
```

企業内の開発では、Pull Requestだけでなく、tagやReleaseを使って「どのコードがいつ本番に出たか」を追跡できるようにします。

特に重要なのは、次の考え方です。

```text
PRは変更内容の説明単位
本番デプロイの正確な基準はcommitまたはtag
Releaseは人間向けの説明ページ
```

この3つを分けて理解すると、Gitのバージョン管理とチーム開発の流れがかなり分かりやすくなります。
