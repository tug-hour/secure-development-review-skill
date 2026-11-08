# SSDF 1.1 RVのTasks

[適用・証拠・出典/権利](../ssdf.md)と併読する。英文は公式Table 1のTasks列を変更せず保持。直後の日本語は対象への適用と必要な根拠の案内であり、英文条件を置き換えない。実装例列/他規格への参考対応は必須条件にしない。

出典: Souppaya M, Scarfone K, Dodson D (2022), Secure Software Development Framework (SSDF) Version 1.1: Recommendations for Mitigating the Risk of Software Vulnerabilities, NIST SP 800-218, National Institute of Standards and Technology, https://doi.org/10.6028/NIST.SP.800-218 (accessed 2026-11-01). Republished courtesy of the National Institute of Standards and Technology.

変更: 2026-11-01に公式補助xlsxからTasks列のみ抜粋し群別配置、日本語の適用/根拠案内を追加。NISTによる承認/公式訳ではない。

## RV.1.1

```text
RV.1.1: Gather information from software acquirers, users, and public sources on potential vulnerabilities in the software and third-party components that the software uses, and investigate all credible reports.
```

対象と利用外部componentに適用。取得者/利用者/公開情報からの情報収集とcredibleな報告を全て調査する根拠を確認。購読手段の名前だけでは足りない。

## RV.1.2

```text
RV.1.2: Review, analyze, and/or test the software’s code to identify or confirm the presence of previously undetected vulnerabilities.
```

対象の継続確認に適用。未発見の脆弱性を識別/確認するreview/analysis/testの実施と対象版/範囲を確認。

## RV.1.3

```text
RV.1.3: Have a policy that addresses vulnerability disclosure and remediation, and implement the roles, responsibilities, and processes needed to support that policy.
```

対象の脆弱性受付/是正に適用。開示/是正方針とそれを支える役割/責任/processの実装を確認。組織共通方針が未提供なら区別して未確認。

## RV.2.1

```text
RV.2.1: Analyze each vulnerability to gather sufficient information about risk to plan its remediation or other risk response.
```

脆弱性が識別された場合。各件のriskと対応計画に必要な情報/根拠を確認。件数やseverityラベルだけで影響分析済みとしない。

## RV.2.2

```text
RV.2.2: Plan and implement risk responses for vulnerabilities.
```

脆弱性が識別された場合。各risk responseの計画と実施を確認。修正以外の対応も原典の範囲で根拠を評価し、risk受容を修正済みにしない。

## RV.3.1

```text
RV.3.1: Analyze identified vulnerabilities to determine their root causes.
```

識別済み脆弱性に適用。根本原因の分析と根拠を確認し、表面的なpatchだけで原因解消としない。

## RV.3.2

```text
RV.3.2: Analyze the root causes over time to identify patterns, such as a particular secure coding practice not being followed consistently.
```

根本原因の履歴がある場合。時系列のpattern分析を確認。履歴未提供なら未確認、まだ対象履歴が無い場合はその条件と制約を明記。

## RV.3.3

```text
RV.3.3: Review the software for similar vulnerabilities to eradicate a class of vulnerabilities, and proactively fix them rather than waiting for external reports.
```

類似脆弱性の可能性がある場合。対象software内の同種箇所をreviewし、外部報告を待たない是正の根拠を確認。

## RV.3.4

```text
RV.3.4: Review the SDLC process, and update it if appropriate to prevent (or reduce the likelihood of) the root cause recurring in updates to the software or in new software that is created.
```

原因分析を受けるSDLCに適用。再発防止のprocess見直しと、適切な場合の更新を確認。原文のif appropriateを無条件のprocess変更へ変えない。
