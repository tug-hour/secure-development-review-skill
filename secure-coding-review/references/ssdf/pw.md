# SSDF 1.1 PWのTasks

[適用・証拠・出典/権利](../ssdf.md)と併読する。英文は公式Table 1のTasks列を変更せず保持。直後の日本語は対象への適用と必要な根拠の案内であり、英文条件を置き換えない。実装例列/他規格への参考対応は必須条件にしない。

出典: Souppaya M, Scarfone K, Dodson D (2022), Secure Software Development Framework (SSDF) Version 1.1: Recommendations for Mitigating the Risk of Software Vulnerabilities, NIST SP 800-218, National Institute of Standards and Technology, https://doi.org/10.6028/NIST.SP.800-218 (accessed 2026-11-01). Republished courtesy of the National Institute of Standards and Technology.

変更: 2026-11-01に公式補助xlsxからTasks列のみ抜粋し群別配置、日本語の適用/根拠案内を追加。NISTによる承認/公式訳ではない。

## PW.1.1

```text
PW.1.1: Use forms of risk modeling – such as threat modeling, attack modeling, or attack surface mapping – to help assess the security risk for the software.
```

対象の設計/リスクに適用。脅威・攻撃・攻撃面等のrisk modelingと実対象の対応を確認。図の存在だけで全境界評価済みとしない。

## PW.1.2

```text
PW.1.2: Track and maintain the software’s security requirements, risks, and design decisions.
```

対象に適用。安全要求・リスク・設計判断の追跡と維持、現在版への適用を確認。

## PW.1.3

```text
PW.1.3: Where appropriate, build in support for using standardized security features and services (e.g., enabling software to integrate with existing log management, identity management, access control, and vulnerability management systems) instead of creating proprietary implementations of security features and services. [Formerly PW.4.3]
```

標準化されたsecurity機能/serviceを利用することが適切な場合。連携の対応と選択根拠を確認。原文のwhere appropriateを残し、独自実装を一律禁止しない。

## PW.2.1

```text
PW.2.1: Have 1) a qualified person (or people) who were not involved with the design and/or 2) automated processes instantiated in the toolchain review the software design to confirm and enforce that it meets all of the security requirements and satisfactorily addresses the identified risk information.
```

対象の設計に適用。設計へ不参加のqualified reviewer、又はtoolchainの自動process、又は両方という代替を保持し、安全要求/特定リスクの確認と強制の根拠を確認。別人と自動化の両方を一律必須にしない。

## PW.4.1

```text
PW.4.1: Acquire and maintain well-secured software components (e.g., software libraries, modules, middleware, frameworks) from commercial, open-source, and other third-party developers for use by the organization’s software.
```

外部componentを使う場合。取得元、保守、安全性と対象での利用の根拠を確認。commercial/open-source等を除外しない。

## PW.4.2

```text
PW.4.2: Create and maintain well-secured software components in-house following SDLC processes to meet common internal software development needs that cannot be better met by third-party software components.
```

外部componentでより良く満たせない共通の内部需要を内製componentで満たす場合。内製する判断とSDLCに従う保守/安全性を確認。

## PW.4.4

```text
PW.4.4: Verify that acquired commercial, open-source, and all other third-party software components comply with the requirements, as defined by the organization, throughout their life cycles.
```

取得した全外部componentに適用。組織で定義した要求をlifecycleを通じて満たす確認を調べる。要求/外部証拠未提供は未確認、現在の版の検査だけで将来の運用済みとしない。

## PW.5.1

```text
PW.5.1: Follow all secure coding practices that are appropriate to the development languages and environment to meet the organization’s requirements.
```

対象の言語/環境に適用。適切な全secure coding条件と組織要求を確認。他系統の具体条件と共通根拠を使い、この抽象taskで言語条件を代替しない。

## PW.6.1

```text
PW.6.1: Use compiler, interpreter, and build tools that offer features to improve executable security.
```

compiler/interpreter/buildを使う場合。実際のtoolと実行形式の保護機能を確認。tool名だけで保護が有効としない。

## PW.6.2

```text
PW.6.2: Determine which compiler, interpreter, and build tool features should be used and how each should be configured, then implement and use the approved configurations.
```

対象のbuildに適用。採用する保護機能/設定の判断、承認済み設定、実際の使用を確認。未承認/未使用と設定不明を分ける。

## PW.7.1

```text
PW.7.1: Determine whether code review (a person looks directly at the code to find issues) and/or code analysis (tools are used to find issues in code, either in a fully automated way or in conjunction with a person) should be used, as defined by the organization.
```

人が読めるcodeの確認方法の決定に適用。組織定義に基づく人のreview/自動analysis/組合せの選択と理由を確認。両方一律必須ではない。

## PW.7.2

```text
PW.7.2: Perform the code review and/or code analysis based on the organization’s secure coding standards, and record and triage all discovered issues and recommended remediations in the development team’s workflow or issue tracking system.
```

決定されたcode review/analysisに適用。secure coding基準に沿う実施、発見事項と推奨是正の全記録/triageを確認。検査定義だけでは実施/是正済みとしない。

## PW.8.1

```text
PW.8.1: Determine whether executable code testing should be performed to find vulnerabilities not identified by previous reviews, analysis, or testing and, if so, which types of testing should be used.
```

実行可能codeの検査要否決定に適用。既存review等で未発見の脆弱性を見つける検査の要否と、必要時の種類を確認。検査不要という決定にも根拠を要し、不明を不要へ移さない。

## PW.8.2

```text
PW.8.2: Scope the testing, design the tests, perform the testing, and document the results, including recording and triaging all discovered issues and recommended remediations in the development team’s workflow or issue tracking system.
```

必要と判断された実行可能code検査に適用。範囲/設計/実施/結果、発見事項と推奨是正の全記録/triageを確認。未実施を成功としない。

## PW.9.1

```text
PW.9.1: Define a secure baseline by determining how to configure each setting that has an effect on security or a security-related setting so that the default settings are secure and do not weaken the security functions provided by the platform, network infrastructure, or services.
```

安全に影響する設定に適用。安全なdefault baselineとplatform/network/serviceの保護を弱めない根拠を確認。

## PW.9.2

```text
PW.9.2: Implement the default settings (or groups of default settings, if applicable), and document each setting for software administrators.
```

default設定に適用。実装された設定又は設定groupと管理者向けの各設定文書を確認。sampleと実際のdefaultを区別。
