---
title: "GitのTag・Release・CI/CD・ブランチ管理の基本"
emoji: "🔖"
type: "tech"
topics: ["git", "github", "versioning", "branch", "release"]
published: false
---

GitのPull Request、tag、Release、バージョン管理、ブランチ管理、GitHub Actions、CI/CD自動化を、チーム開発で使う前提で整理します。

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
| GitHub Actions | GitHub上でテスト・ビルド・デプロイを自動実行する仕組み |
| CI/CD | テスト・ビルド・デプロイの自動化 |

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

## 10. どのcommitにtagを付けるのか

結論として、tagを付けるcommitは、基本的には `main` に入った後のcommitです。

理由は、バージョンtagは「リリース版」「本番版」「公開版」を固定するための印だからです。レビュー前や統合前のdev branch / feature branch上のcommitは、まだ正式版ではありません。

一般的な流れは次の通りです。

```text
feature / dev branch
  ↓ commitを積む
  ↓ Pull Request
  ↓ review
main
  ↓ merge後のcommit
  ↓ tagを付ける
release / deploy
```

つまり、次のように分けて考えます。

```text
dev branch内のcommit = 作業履歴
main上のcommit = 安定版候補
tag付きcommit = リリース版
```

例えば、feature branchで3つのcommitを作ったとします。

```text
feature/login
  A: ログインAPIを追加
  B: ログイン画面を追加
  C: テストを追加
```

Pull Requestでmainへmergeされた後、mainにmerge commitができます。

```text
main
  M: Merge pull request #10 from feature/login
```

この場合、基本的にtagを付けるのは `A`、`B`、`C` ではなく、mainに入った後の `M` です。

```bash
git switch main
git pull
git tag -a v1.1.0 -m "Release v1.1.0"
git push origin v1.1.0
```

Squash mergeを使うチームでは、feature branch上の複数commitがmain上で1つのcommitにまとめられます。

```text
feature/login
  A
  B
  C

main
  S: Add login feature (#10)
```

この場合、tagを付けるのはmain上の `S` です。

Rebase mergeの場合は、feature branchのcommitがmain上に並び直します。

```text
main
  A'
  B'
  C'
```

この場合は、リリースしたい最後のcommit、例えば `C'` にtagを付けます。

dev branchやstaging branchにtagを付けるケースもあります。

例えば、QA用ビルドやリリース候補を固定したい場合です。

```text
v1.1.0-rc.1
v1.1.0-beta.1
staging-20260708
qa-20260708
```

ただし、これは正式リリースtagとは分けて考えます。

基本は次の理解で十分です。

```text
Dev branch内のcommitには通常tagを付けない
mainに入って、リリースすると決めたcommitにtagを付ける
```

## 11. バージョン管理の基本

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

## 12. GitHub Actionsとは

結論として、GitHub Actionsは、GitHub上で自動処理を実行する仕組みです。

理由は、Pull Request作成時やmainへのpush時など、GitHub上のイベントをきっかけにテスト、Lint、ビルド、デプロイなどを自動実行できるからです。

Git自体には「Action」という機能は基本的にありません。一般的に「Git Action」と言われているものは、GitHubが提供しているGitHub Actionsを指していることが多いです。

整理すると次の通りです。

```text
Git = バージョン管理ツール
GitHub = Gitリポジトリを置くサービス
GitHub Actions = GitHub上で自動処理を動かす仕組み
```

GitHub Actionsでよくやることは次のような処理です。

- Pull Requestが作られたら自動テストを実行する
- Pull Requestが作られたらLintや型チェックを実行する
- mainにmergeされたらビルドする
- mainにmergeされたら検証環境へデプロイする
- tagを作ったら本番デプロイする
- tagを作ったらGitHub Releaseを作る
- 毎日決まった時間にバッチ処理を実行する

企業開発でもGitHub ActionsのようなCI/CD自動化はよく使われます。

ただし、会社によって使うツールは違います。

| 利用サービス | よく使われるCI/CD |
| --- | --- |
| GitHub | GitHub Actions |
| GitLab | GitLab CI/CD |
| Bitbucket | Bitbucket Pipelines |
| 既存の社内基盤 | Jenkins |
| AWS中心の構成 | CodePipeline / CodeBuild |

名前は違っても、考え方は同じです。

```text
コード変更
  ↓
自動テスト
  ↓
自動ビルド
  ↓
自動デプロイ
```

## 13. CI/CD自動化とは

結論として、CI/CD自動化とは、コード変更後のテスト・ビルド・デプロイを自動で実行する仕組みです。

理由は、人が毎回手動で確認すると、手順漏れや環境差分によるミスが起きやすいからです。

CIはContinuous Integrationの略です。日本語では「継続的インテグレーション」と呼ばれます。

主にやることは次の通りです。

```text
コード変更
  ↓
自動テスト
  ↓
Lint
  ↓
型チェック
  ↓
ビルド確認
```

CIの目的は、壊れたコードをmainに入れにくくすることです。

CDは文脈によって2つの意味で使われます。

| 用語 | 意味 |
| --- | --- |
| Continuous Delivery | 本番に出せる状態まで自動化し、最後の本番反映は人が承認する |
| Continuous Deployment | テスト成功後、本番反映まで完全自動で行う |

企業の重要な本番環境では、Continuous Deliveryの形がよく使われます。

つまり、テスト、ビルド、検証環境へのデプロイまでは自動化し、本番デプロイだけ人が承認する形です。

一般的な企業開発の流れは次のようになります。

```text
開発者がfeature branchで作業
  ↓
commit
  ↓
push
  ↓
Pull Request
  ↓
CI: 自動テスト・Lint・ビルド
  ↓
レビュー
  ↓
mainへmerge
  ↓
CD: ステージングへ自動デプロイ
  ↓
本番リリース承認
  ↓
tag作成
  ↓
本番デプロイ
```

短く言うと、次のように整理できます。

```text
CI = merge前に壊れていないか自動チェック
CD = merge後に環境へ自動反映
```

## 14. GitHub Actionsのコード例

GitHub Actionsは、リポジトリ内の `.github/workflows/` にYAMLファイルを置いて設定します。

例えば、Pull Requestとmainへのpushでチェックを動かす場合は、次のようなファイルを作ります。

```yaml
name: CI

on:
  pull_request:
    branches:
      - main
  push:
    branches:
      - main

jobs:
  check:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Run build
        run: npm run build
```

この例では、Pull Requestまたはmainへのpushをきっかけに、依存関係のインストール、テスト、ビルドを実行します。

Zenn記事リポジトリのようにアプリのテストがない場合でも、MarkdownやZenn CLIのチェックを入れることがあります。

例えば、Zenn CLIで記事構成を確認する場合は次のようなイメージです。

```yaml
name: Zenn Check

on:
  pull_request:
    branches:
      - main
  push:
    branches:
      - main

jobs:
  zenn-check:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Validate Zenn content
        run: npx zenn preview --no-open
```

実際のコマンドはプロジェクト構成やZenn CLIの仕様に合わせて調整します。

重要なのは、手動で確認していた作業を、Pull Requestやmainへのpushをきっかけに自動化することです。

## 15. tagとGitHub Actionsを組み合わせる例

tagを作った時にだけ本番リリース処理を動かすこともできます。

例えば、`v1.0.0` や `v1.1.0` のようなtagがpushされた時だけ実行する場合は、次のように書きます。

```yaml
name: Release

on:
  push:
    tags:
      - "v*.*.*"

jobs:
  release:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Show release tag
        run: echo "Release tag is ${{ github.ref_name }}"
```

このようにすると、通常のcommitやPull Requestでは動かず、バージョンtagをpushした時だけリリース用の処理が動きます。

企業開発では、次のような使い方がよくあります。

```text
mainにmerge
  ↓
動作確認
  ↓
git tag -a v1.2.0 -m "Release v1.2.0"
  ↓
git push origin v1.2.0
  ↓
GitHub Actionsがtagを検知
  ↓
本番デプロイ or GitHub Release作成
```

この流れにすると、「どのcommitを本番に出したか」と「どのバージョンとして出したか」が追いやすくなります。

## 16. よく使う確認コマンド

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

GitHub Actionsの実行状況を見る場合は、GitHubの画面で次を確認します。

```text
GitHub repository
  ↓
Actions tab
  ↓
対象のworkflow
```

GitHub CLIを使える場合は、次のような確認もできます。

```bash
gh run list
gh run view
```

## 17. まとめ

Git運用は、次のように覚えると整理しやすいです。

```text
commit = 作業履歴
branch = 作業場所
Pull Request = レビュー依頼
main = 安定版
tag = リリースするコードの固定点
Release = 変更内容を説明するページ
GitHub Actions = GitHub上で自動処理を動かす仕組み
CI/CD = テスト・ビルド・デプロイの自動化
```

企業内の開発では、Pull Requestだけでなく、tagやReleaseを使って「どのコードがいつ本番に出たか」を追跡できるようにします。

特に重要なのは、次の考え方です。

```text
PRは変更内容の説明単位
本番デプロイの正確な基準はcommitまたはtag
Releaseは人間向けの説明ページ
CIはmerge前の自動チェック
CDはmerge後の自動反映
```

これらを分けて理解すると、Gitのバージョン管理、CI/CD自動化、チーム開発の流れがかなり分かりやすくなります。
