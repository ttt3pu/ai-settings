---
name: commit
description: コード変更を適切なgitコミット戦略でgit commitします。基本的には新しくgitコミットを作成し、必要な場合にのみ履歴整理を行います。実装完了時やユーザーがgit commitを依頼した時に使用します。
---

# Commit and Push Code Changes

コード変更を適切なgitコミット戦略でgit commitするためのスキルです。
現在の依頼・許可範囲でコミットする変更だけを選び、無関係な変更や秘密値を含めないでください。非公開情報や移植元の名前・URLなど、公開を許可されていない情報をコミットメッセージに記載しないでください。
このスキルが呼び出された際には、Instructionsに従って、コード変更のgit commitを行ってください。

# Instructions

## 実行ステップ

以下のステップでコード変更のgit commitを行ってください。

### ステップ1: ブランチとgitコミット履歴の確認

以下のコマンドで現在の状態を確認：

```bash
git status
git symbolic-ref refs/remotes/origin/HEAD
git log --oneline --graph <remote>/<default-branch>..HEAD
```

`<remote>/<default-branch>` は確認したリモートとデフォルトブランチに置き換える。`origin/HEAD` が未設定ならリモートの情報や利用側の規約から確認し、`main` と決めつけない。

確認事項：

- 現在のブランチ名
- `origin/HEAD` が指すデフォルトブランチ名
- デフォルトブランチから何gitコミット進んでいるか
- 各gitコミットの内容と粒度

### ステップ2: ブランチ方針を確認

現在のブランチがデフォルトブランチの場合、ユーザーが直接pushを明示していなければ作業ブランチを作成してください。ユーザーが`main`への直接commit・pushを明示した場合は、その指示を優先してください。

ブランチ名は `feature/` 固定ではなく、変更内容に応じて
gitコミットメッセージのtypeと揃えます。

**推奨prefix:**

- `feat/`: 新機能
- `fix/`: バグ修正
- `refactor/`: リファクタリング
- `test/`: テスト追加・修正
- `docs/`: ドキュメント変更
- `chore/`: ビルドプロセスやツールの変更
- `misc/`: 上記に当てはまらない変更

**ブランチ名の形式:**

```text
<type>/<short-description>
```

例：

- `feat/add-data-import`
- `fix/handle-empty-list`
- `refactor/simplify-worker-retry`

**実行方法：**

```bash
git switch -c <type>/<short-description>
```

すでにデフォルトブランチ以外で作業している場合は、このステップを
スキップして次に進んでください。

### ステップ2.5: コミット前の変更範囲別検証（必須）

利用側の `AGENTS.md` と既存の検証コマンドを確認し、変更したmodule・packageと直接依存先に必要なテスト、型チェック、Lint、buildを選んで実行してください。全体検証は、利用側の規約で定める場合、影響範囲を限定できない場合、またはユーザーが明示した場合に実行します。実行した検証と結果を報告してください。

選定した検証が1つでも失敗した場合は、コミットを作成せずその場で原因を修正してください。

### ステップ3: gitコミット戦略の判断

以下の基準でgitコミット戦略を選択：

#### 戦略A: 新規gitコミット（基本戦略）

以下の場合は新規gitコミットを作成：

- ブランチに初めてのgitコミット
- 既存のgitコミットに関連する変更だが、追加の作業履歴として残したい
- 既存のgitコミットとは異なる独立した変更
- gitコミットを分けることで履歴がより理解しやすくなる

**実行方法：**

```bash
git add -- <対象ファイル>
git commit
```

#### 戦略B: AmendまたはSquash（例外対応）

ユーザーが明示的に `--amend` やsquashを希望している場合のみ、既存のgitコミットを書き換えます。軽微な修正やPR作成前の整理だけを理由に履歴を書き換えないでください。

**実行方法：**

```bash
git add -- <対象ファイル>
git commit --amend
```

amendは原則として避け、新規gitコミットで履歴を積み上げることを優先してください。
gitコミットメッセージを更新する必要がある場合も、ユーザーの明示的な希望がない限り
`git commit --amend` を選ばないでください。

#### 戦略C: Interactive Rebase（gitコミット再構成）

ユーザーが履歴の再構成を明示的に希望している場合に限り、以下の目的でブランチ全体のgitコミットを再構成：

- 複数の小さなgitコミットを論理的なまとまりに整理したい
- gitコミットの順序を変更したい
- 不要なgitコミットを削除したい
- gitコミット履歴を意味のある単位に再編成したい

**実行方法：**

```bash
git rebase -i <remote>/<default-branch>
```

エディタで以下の操作を実行：

- `pick`: gitコミットをそのまま維持
- `squash`または`s`: 前のgitコミットと統合
- `reword`または`r`: gitコミットメッセージを変更
- 行の順序を変更してgitコミット順を変更

### ステップ4: gitコミットメッセージのガイドライン

日本語でメッセージを作成する。
gitコミットメッセージは以下の形式で記述：

```
<type>: <subject>

<body>

<footer>
```

**Type:**

- `feat`: 新機能
- `fix`: バグ修正
- `refactor`: リファクタリング
- `test`: テスト追加・修正
- `docs`: ドキュメント変更
- `chore`: ビルドプロセスやツールの変更

**Subject:**

- 50文字以内
- 命令形で記述（例: "add"ではなく"Add"）
- 末尾にピリオドを付けない

**Body（オプション）:**

- 変更の理由と背景を説明
- 何を変更したかではなく、なぜ変更したかを記述
- 72文字で折り返す

**Footer（オプション）:**

- Issue番号への参照（例: `Closes #123`）
- Breaking changesの記述

### ステップ5: git commit後の確認

git commit後、以下を確認：

```bash
git log -1 --stat
git status
```

- gitコミットが正しく作成されたか
- 意図したファイルがすべて含まれているか
- gitコミットメッセージが適切か

### ステップ6: push前のデプロイ判断

pushはcommitとは別の操作です。pushを求められた場合だけ、利用側の `AGENTS.md` のpush・デプロイ規約を確認してください。

- 現在のworkflowとdeploy scriptから、pushがCIだけを起動するか、本番デプロイも起動するかを確認する。
- GitHub上のworkflow有効状態を確認できない場合は`不明`と報告し、過去の会話から推測しない。
- 本番deployを起動し得るpushは、既存のユーザー指示がそのデプロイを含むか確認する。commitのみの承認をpush・deployの承認と扱わず、許可範囲外なら実行前に確認する。
- push後にCI/CDをポーリングしない。手動deployを行った場合も、Gitへの記録やCI/CD反映とは区別して報告する。

## 戦略選択のフローチャート

```
現在のブランチはデフォルトブランチ？
  ├─ Yes → 直接commit・pushをユーザーが明示？
  │          ├─ Yes → そのまま継続
  │          └─ No → <type>/<short-description> で作業ブランチを作成
  └─ No → そのまま継続

ブランチにgitコミットがある？
  ├─ No → 新規gitコミット作成
  └─ Yes → 変更は既存のgitコミットと同じテーマ？
      ├─ Yes → 原則は新規gitコミット作成
      └─ No → 新規gitコミット作成

amendやsquashを明示的に求められている？
  ├─ Yes → git commit --amend を検討
  └─ No → 新規gitコミットを維持

履歴全体の整理をユーザーが明示している？
  ├─ Yes → Interactive Rebase
  └─ No → そのまま継続
```
