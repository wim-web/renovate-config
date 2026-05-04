---
name: renovate-automerge
description: このリポジトリの Renovate PR を調査し、repo固有ルールに従ってマージする時に使う。
---

# Renovate Automerge

この skill は、`wim-web/renovate-config` で Renovate が作成した open PR を確認し、以下のルールに従ってマージまたは報告する。

## 対象PR

- 作成者が Renovate の open PR のみを対象にする。
- このリポジトリで観測した Renovate author: `app/renovate`
- base branch が `main` の PR のみを対象にする。
- Dependabot や人間が作成した PR は対象外。

## リポジトリ前提

- このリポジトリは共有 Renovate config repo。
- package manager manifest や lockfile は存在しない。
- Renovate 設定ファイルは `default.json` と `renovate.json`。
- CI は `.github/workflows/renovate_config_validate.yaml` の `validate` job。
- Docker、infra、deploy、database migration は存在しない。

## 必ず確認すること

明らかなブロック条件があっても、そこで早期終了しない。影響範囲調査を完了してから、マージしてよい / マージしてはいけない / 人間確認が必要、のいずれかを判断する。

- PR title/body
- changed files
- update type
- Renovate がPR本文に載せた release notes / changelog / compatibility notes
- upstream changelog / release notes / migration guide
- 破壊的変更、deprecated API、設定変更、peer dependency変更、runtime要件変更の有無
- 影響範囲: Renovate config / GitHub Actions / CI
- check status
- merge conflict の有無
- requested changes / 未解決の人間 review comment の有無

## 調査手順

対象 PR ごとに、必ずこの順序で調査してから判定する。

1. `gh pr list --state open --author app/renovate --json number,title,author,baseRefName,headRefName,isDraft,url` で対象候補を確認する。
2. `gh pr view PR番号 --json title,body,author,baseRefName,headRefName,isDraft,mergeable,reviewDecision,files,statusCheckRollup,commits,reviews,comments,url` で PR metadata を確認する。
3. `gh pr diff PR番号 --patch` で差分を確認する。
4. Renovate PR 本文の release notes / changelog / compatibility notes を読む。
5. upstream 公式 release notes / changelog / upgrade guide / migration guide を読む。PR 本文だけで済ませない。
6. repo 内で変更対象の参照箇所を `rg` で検索し、この repo での影響範囲を確認する。
7. ここまで完了してから判定する。

## マージしてよいもの

以下をすべて満たす PR だけ自動マージしてよい。

- 作成者が `app/renovate`。
- base branch が `main`。
- draft ではない。
- merge conflict がない。
- GitHub checks がすべて成功している。
- requested changes や未解決の人間 review comment がない。
- changed files が `default.json`、`renovate.json`、`.github/workflows/renovate_config_validate.yaml` の範囲に収まる。
- `default.json` または `renovate.json` の Renovate 設定更新で、patch/minor update か digest/pin update。
- `.github/workflows/renovate_config_validate.yaml` の GitHub Actions 更新で、patch/minor update か digest/pin update。
- 対象 dependency は、この repo で観測済みの `aquaproj/aqua-renovate-config`、`actions/checkout`、`rinchsan/renovate-config-validator`。
- upstream release notes / changelog で breaking change、設定変更必須、runtime 要件変更、deprecated API の影響がないと確認できる。

## マージしてはいけないもの

以下のいずれかに該当する PR はマージしない。必要なら理由を報告する。

- 作成者が `app/renovate` ではない。
- base branch が `main` ではない。
- major update。
- `config:best-practices`、`schedule:daily`、`:timezone(Asia/Tokyo)` など Renovate preset の意味を変える可能性がある変更。
- `default.json`、`renovate.json`、`.github/workflows/renovate_config_validate.yaml` 以外を変更する PR。
- package manager manifest、lockfile、source code、Docker、infra、deploy、database migration を追加または変更する PR。
- breaking changes、peer dependency変更、runtime要件変更、設定変更の可能性が残る PR。
- changelog / release notes / migration guide を確認できず、影響範囲を判断できない PR。
- failed / pending / missing checks がある PR。
- requested changes や未解決の人間コメントがある PR。
- merge conflict がある PR。

## check の扱い

- `statusCheckRollup` を確認し、すべて成功している場合だけマージしてよい。
- pending、failed、cancelled、skipped、timed out、missing の check がある場合はマージしない。
- check が一つも報告されていない PR は missing checks として扱い、マージしない。
- branch protection や required check は推測しない。

## マージ方法

- 自動マージする場合は squash merge を使う。
- コマンドは `gh pr merge PR番号 --squash` を使う。
- Renovate branch は削除しない。

## post-merge action

- Renovate PR のマージ後に release、deploy、local tool update、GitHub comment などの追加 action は実行しない。
- 複数 PR をマージした場合も、最後にまとめて実行する追加 action はない。

## 報告

- マージした PR は PR number、title、URL、merge method を報告する。
- マージしなかった PR は PR number、title、URL、理由、人間確認ポイントを報告する。
- 対象 PR がない場合は、open Renovate PR がないことを報告して終了する。

## PR コメント

- マージしなかった Renovate PR には、通常コメントしない。
- 人間確認が必要で、同じ調査結果を PR 上に残すと有用な場合だけコメントする。
- コメントする場合は、調査した release notes / changelog、影響範囲、マージしなかった具体理由、人間に確認してほしい点を簡潔に含める。

## 禁止操作

- Renovate branch に commit や push をしない。
- PR を close しない。
- この skill に明記されていない条件の PR はマージしない。
