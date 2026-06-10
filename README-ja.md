# gh-security-audit-skill

[English](./README.md)

`gh api` で GitHub リポジトリ1つのセキュリティ設定を読み取り専用で監査する
Agent Skill です。

## なにをするか

`OWNER/REPO` を渡すと、決まった読み取り専用の REST API を一通り実行し、
観測された設定・アラート件数・確認できなかった事項を、同一のデータから
Markdown レポートと JSON にまとめます。

出力は現状のインベントリであって、助言ではありません。ステータス
(`PASS` / `WARN` / `MANUAL` / `SKIP`)は skill 内の決定表から機械的に
付与され、`PASS` は「レビュー対象として引っかからなかった」以上の意味を
持ちません。その状態でよいか、何を変えるかの判断は利用者側の仕事です。
自分の要件と、その時点でのベストプラクティスに照らして読んでください。
`FAIL` は、失敗条件を定義したポリシーを自分で渡さない限り使われません。

## 確認する項目

- リポジトリのメタデータ、`security_and_analysis`、マージ/フォーク設定
- Actions: リポジトリ権限、ワークフロートークンのデフォルト、fork PR の
  承認ポリシー、secrets / variables のメタデータ(名前のみ)、OIDC
  subject claim、セルフホストランナー
- ブランチ保護: branch / tag / push ルールセット、デフォルトブランチの
  アクティブルール、レガシーブランチ保護
- code security configuration、CodeQL デフォルトセットアップ
- Dependabot: 設定ファイル、security updates、vulnerability alerts、
  オープンアラート件数
- secret scanning の設定とオープンアラート件数
- SECURITY.md の有無、CODEOWNERS のエラー、private vulnerability reporting
- environments、リリース、immutable releases、artifact attestations
  (digest の指定が必要)

v0.1 の対象外: ワークフローファイルの中身(zizmor、Scorecard)、ローカル
clone、クラウド側の trust policy、SBOM の中身、修正作業。リポジトリ設定の
変更は一切行いません。

## 前提

- `gh` がインストール・認証済みで、対象リポジトリの read 権限があること。
  admin 等のスコープがあると確認できる項目が少し増えます。届かない範囲は
  失敗ではなく limitation として記録されます。
- `jq`
- POSIX シェル前提です。Windows の場合は WSL か Git Bash を使ってください。

## インストール

`gh skill` は現時点で GitHub CLI のプレビュー機能です。

```sh
gh skill install K-Oxon/gh-security-audit-skill gh-security-audit --agent claude-code --scope user
gh skill install K-Oxon/gh-security-audit-skill gh-security-audit --agent codex --scope user
```

ローカルの checkout から入れる場合:

```sh
gh skill install . gh-security-audit --from-local --agent claude-code --scope project
```

## 使い方

Claude Code:

```text
> gh-security-audit で OWNER/REPO のセキュリティ設定を監査して
```

Codex:

```sh
codex "gh-security-audit で OWNER/REPO を監査して。Markdown と JSON で出力して。"
```

エージェントは `references/command_recipe.md` を読み、読み取り専用の
`gh api` コマンドを実行し、生データは一時ディレクトリに保存した上で、
`references/finding_model.md` に従ってレポートを組み立てます。出力例は
`skills/gh-security-audit/examples/` にあります。

## レポートの読み方

- `PASS`: 証拠が取得でき、レビューフラグに該当しなかった。安全という
  判定ではありません。
- `WARN`: 観測された状態が決定表のレビューフラグに該当した。自分の
  ポリシーと突き合わせて確認してください。
- `MANUAL`: 人の判断か、API では取れない証拠が必要。
- `SKIP`: 対象外、または該当しない。

API エラー・権限不足・プランの違いは limitation として報告されます。
シークレットの値、アラートの位置情報、Actions variables の値が出力に
含まれることはありません。

## 構成

```text
skills/gh-security-audit/
├── SKILL.md
├── references/   # コマンドレシピ、finding model、API ソース
└── examples/     # レポートと findings のサンプル
```

License: Apache-2.0
