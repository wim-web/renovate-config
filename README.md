# renovate-config

Renovate の共有設定です。利用するリポジトリの `renovate.json` から参照します。

```json
{
  "extends": ["local>wim-web/renovate-config"]
}
```

## 脆弱性の修正を Renovate に集約する

GitHub の Dependabot alerts で脆弱性を検知し、そのアラートを Renovate が読み取って修正 PR を作成します。このプリセットでは、Renovate が標準で有効にしている `vulnerabilityAlerts` の設定を明示し、脆弱性修正 PR に `security` ラベルを追加します。

共有設定を利用する各リポジトリでは、GitHub 側を次の状態にします。

| 設定 | 状態 | 役割 |
| --- | --- | --- |
| Dependency graph | ON | 依存関係の把握 |
| Dependabot alerts | ON | 脆弱性の検知・通知 |
| Dependabot security updates | OFF | Dependabot による脆弱性修正 PR の作成を停止 |
| Dependabot version updates | OFF | Dependabot による通常の更新 PR の作成を停止 |

Dependabot の自動トリアージルールで PR を作成する設定がある場合は、その PR 作成ルールも無効にします。Renovate App には Dependabot alerts の読み取り権限が必要です。

これらは GitHub 側の設定であり、このプリセットでは変更できません。個人アカウントでは **Settings → Code security** から、既存リポジトリの Dependency graph と Dependabot alerts を一括で有効にし、新規リポジトリでも自動的に有効にする設定ができます。

脆弱性修正 PR は、通常の更新に設定している `schedule:daily` の時間帯を待たず、Renovate がアラートを処理した時点で作成対象になります。

- [Renovate: vulnerabilityAlerts](https://docs.renovatebot.com/configuration-options/#vulnerabilityalerts)
- [GitHub: アカウント全体のセキュリティ設定](https://docs.github.com/en/account-and-profile/how-tos/account-settings/managing-security-and-analysis-features)
- [GitHub: Dependabot security updates の設定](https://docs.github.com/en/code-security/how-tos/secure-your-supply-chain/secure-your-dependencies/configure-security-updates)
- [GitHub: Dependabot version updates の設定](https://docs.github.com/en/code-security/how-tos/secure-your-supply-chain/secure-your-dependencies/configure-version-updates)
