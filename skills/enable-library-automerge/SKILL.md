---
name: enable-library-automerge
description: >-
  Assesses whether existing tests and CI sufficiently protect upgrades of a
  specific npm library, stopping without changes when sufficient. When coverage
  is insufficient, adds product-focused regression tests and CI coverage, then
  enables Renovate minor/patch automerge. Use when the user asks to automerge a dependency safely, add tests
  so a library can be auto-merged, enable Renovate automerge for a package, or
  harden dependency upgrades with regression coverage.
---

# Enable library automerge

任意のライブラリについて、まず既存のテストと CI が**本プロダクト上の挙動**を十分に保護しているか調査する。十分なら変更せず終了する。不足がある場合だけ必要な回帰テスト・CI を補強し、CI 通過を条件に Renovate の minor/patch 自動マージを有効化する。

## 手順

### 作業ブランチとコミットの方針

- ユーザーから対象の Renovate PR が提示された場合、別途指示がなければ、その PR の head リポジトリとブランチを確認し、当該ブランチで調査・対応する。追加の作業ブランチや別 PR は作成しない
- 調査で不足が見つかり変更した場合は、必要な検証を通したうえで、その PR ブランチに新規コミットを追加する。既存の Renovate コミットは書き換えない
- 既存のテストと CI が十分なら、従来どおり変更・コミットを行わず終了する
- push はコミットと区別し、ユーザーの依頼・許可範囲に従う

### 1. 調査：既存 CI の十分性を判断する

このフェーズではファイルを変更しない。

#### プロダクト上の利用を特定する

- import / 呼び出し箇所をすべて洗う
- 入力の実体を特定する（DB の保存データ、API レスポンス、設定ファイル、ユーザー入力など）
- 出力の消費者を特定する（画面描画、ファイル出力、別モジュールへの引き渡しなど）
- 壊れたときにユーザー影響が出る境界だけをテスト対象にする

#### 既存テストと CI を確認する

- 適用先の `package.json` の scripts、テストランナー設定、既存テスト、CI のワークフロー定義を読む
- 対象ライブラリを使う本番経路と重要な入出力が既存テストで検証されているか確認する。ライブラリをモックしているだけのテストは、その更新による破壊を検出できる根拠にしない
- 対象依存の更新 PR で必要な検証が実行され、失敗がマージを阻止するか確認する。実行条件・パスフィルター・失敗の無視設定と、確認可能なマージ条件も調べる
- 必要に応じて既存の検証を実行し、確認した範囲と未確認事項を区別する。CI が成功していることだけで十分とは判断しない

#### 判定して分岐する

- **十分**：重要な製品挙動の破壊を既存の検証で検出でき、更新 PR のマージ前にその成功が要求される。根拠となるテスト・CI 設定を報告して、ここで終了する。テスト追加、リファクタリング、CI 変更、Renovate 設定変更は行わない。自動マージが未設定でも、この分岐では設定しない
- **不足**：保護されていない製品挙動や CI 上の具体的な不足を報告し、その不足を解消するために必要な後続手順だけ実施する
- **判断材料が不足**：未確認事項を報告する。未確認であることを理由にテストや設定を追加せず、判断に必要な情報を確認する

### 2. 不足するテストを設計する（製品契約）

ライブラリ単体の仕様・API 確認で終わらせない。プロダクトの入出力を通す。

優先順:

1. **実データ / 本番相当フィクスチャ**を通し、プロダクトが依存する出力を固定する
2. 出力がテキストや大きな構造体なら **スナップショット**を使う（アップグレード差分が製品影響として読める）
3. 破壊時コストが高い境界があるなら、製品文脈で最小限の断言を足す

避ける:

- ライブラリのドキュメント例や公開 API カタログをなぞるだけのケース
- プロダクトが使っていないオプション・機能の網羅

呼び出しがプロダクト固有コードに埋もれている場合は、**本番と同じ経路**を叩ける薄い関数へ切り出してからテストする。

### 3. 必要な場合だけ CI に載せる

- 先に適用先リポジトリのテストランナーと CI 設定を確認する（`package.json` の scripts、テストランナーの設定ファイル、CI のワークフロー定義）。確認した構成に合わせてテストを書く
- 追加したテストが CI で必ず走る状態にする
- テスト基盤が無ければ、対象の製品契約を固定できる最小構成だけ導入する

### 4. Renovate automerge

まず適用先の Renovate 設定ファイルを読む（`renovate.json`、`renovate.json5`、`.github/renovate.json`、`package.json` の `renovate` キーなど、置き場はリポジトリによる）。

- 共有 Renovate 設定を `extends` しているリポジトリなら、その allowlist に対象パッケージが含まれているかを確認する。含まれていなければリポジトリ側の設定に packageRule を足す（例: `github>ttt3pu/renovate-settings` のような共有プリセットを使っている場合）
- 共有設定を使っていないリポジトリでも、同じ packageRule をそのまま足せばよい
- `extends` の値は既存のものを踏襲する。ここに書かれた例を写して既存の `extends` を書き換えない

既存の設定を残したまま `packageRules` を追記する:

```json
{
  "packageRules": [
    {
      "description": "Automerge <package> minor/patch when tests pass",
      "matchPackageNames": ["<package>"],
      "matchUpdateTypes": ["minor", "patch"],
      "automerge": true
    }
  ]
}
```

テストが無い他リポジトリまで巻き込む場合を除き、共有 allowlist へ安易に追加しない。

### 5. 完了条件（不足を補強した場合）

調査で十分と判断した場合は、根拠の報告のみで完了とし、以下の条件を満たすための変更は行わない。

- [ ] プロダクト経路の回帰テストがあり、通る
- [ ] CI がそのテストを実行する
- [ ] 対象パッケージの minor/patch が automerge 対象になっている

## 追加資料

- 具体例（markdown-it）: [examples.md](examples.md)
