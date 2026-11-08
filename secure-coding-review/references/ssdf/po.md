# SSDF 1.1 POのTasks

[適用・証拠・出典/権利](../ssdf.md)と併読する。英文は公式Table 1のTasks列を変更せず保持。直後の日本語は対象への適用と必要な根拠の案内であり、英文条件を置き換えない。実装例列/他規格への参考対応は必須条件にしない。

出典: Souppaya M, Scarfone K, Dodson D (2022), Secure Software Development Framework (SSDF) Version 1.1: Recommendations for Mitigating the Risk of Software Vulnerabilities, NIST SP 800-218, National Institute of Standards and Technology, https://doi.org/10.6028/NIST.SP.800-218 (accessed 2026-11-01). Republished courtesy of the National Institute of Standards and Technology.

変更: 2026-11-01に公式補助xlsxからTasks列のみ抜粋し群別配置、日本語の適用/根拠案内を追加。NISTによる承認/公式訳ではない。

## PO.1.1

```text
PO.1.1: Identify and document all security requirements for the organization’s software development infrastructures and processes, and maintain the requirements over time.
```

対象の開発基盤/工程に適用。安全要求の文書と維持状況を確認する。組織共通の要求が未提供なら、その部分は未確認であり対象固有要求で代替済みとしない。

## PO.1.2

```text
PO.1.2: Identify and document all security requirements for organization-developed software to meet, and maintain the requirements over time.
```

対象softwareに適用。要求、保護対象、変更時の維持根拠を確認する。文書が無い場合は要求不明と要求未実装を区別する。

## PO.1.3

```text
PO.1.3: Communicate requirements to all third parties who will provide commercial software components to the organization for reuse by the organization’s own software. [Formerly PW.3.1]
```

再利用する商用componentの提供者がいる場合。伝達された要求と対象依存への対応を確認する。契約/調達証拠が対象外又は未提供ならその範囲と不足を明記。商用提供者が不存在なら理由付き非適用。

## PO.2.1

```text
PO.2.1: Create new roles and alter responsibilities for existing roles as needed to encompass all parts of the SDLC. Periodically review and maintain the defined roles and responsibilities, updating them as needed.
```

対象のSDLCで役割/責任と定期見直しが分かる範囲を条件付き採用。保守、release、是正の責任境界を確認する。全組織の職務配置は認証対象へ広げず、必要な組織証拠が未提供なら未確認。

## PO.2.2

```text
PO.2.2: Provide role-based training for all personnel with responsibilities that contribute to secure development. Periodically review personnel proficiency and role-based training, and update the training as needed.
```

役割別訓練と熟練度の定期見直しは組織側証拠を要する条件付き採用。コードから訓練の実施/未実施を断定しない。対象の開発/是正能力に関する不足として区別し、全社員の人事監査を追加しない。

## PO.2.3

```text
PO.2.3: Obtain upper management or authorizing official commitment to secure development, and convey that commitment to all with development-related roles and responsibilities.
```

開発のcommitmentと伝達は組織側証拠を要する条件付き採用。対象に必要な継続保守/是正の承認や責任を確認し、上層部の証拠が無いだけでコードの欠陥としない。組織認証を新目的にしない。

## PO.3.1

```text
PO.3.1: Specify which tools or tool types must or should be included in each toolchain to mitigate identified risks, as well as how the toolchain components are to be integrated with each other.
```

対象toolchainに適用。リスクに必要なtool/type、must/should区別、連携方法と受渡しを確認。特定toolの一律必須化はしない。

## PO.3.2

```text
PO.3.2: Follow recommended security practices to deploy, operate, and maintain tools and toolchains.
```

対象tool/CIに適用。導入/運用/維持の設定と実証、権限、更新/整合性に関する根拠を確認。配置ファイルだけで運用済みとしない。

## PO.3.3

```text
PO.3.3: Configure tools to generate artifacts  of their support of secure software development practices as defined by the organization.
```

対象toolchainに適用。組織が定義した安全工程の証拠をtoolが生成する設定と実成果を確認。組織定義が未提供ならその部分を未確認。

## PO.4.1

```text
PO.4.1: Define criteria for software security checks and track throughout the SDLC.
```

対象の安全確認に適用。判断基準とSDLCを通じた追跡を確認。検査実行の有無だけで基準の妥当性を代替しない。

## PO.4.2

```text
PO.4.2: Implement processes, mechanisms, etc. to gather and safeguard the necessary information in support of the criteria.
```

対象の検査/判断根拠に適用。必要情報の収集と保護、アクセス/改ざん防止の根拠を確認する。

## PO.5.1

```text
PO.5.1: Separate and protect each environment involved in software development.
```

対象の開発/build/test/配布環境に適用。環境分離と保護の構成/権限/実証を確認。未提供の外部基盤を安全扱いしない。

## PO.5.2

```text
PO.5.2: Secure and harden development endpoints (i.e., endpoints for software designers, developers, testers, builders, etc.) to perform development-related tasks using a risk-based approach.
```

対象開発に用いるendpointに条件付き適用。リスクに基づく保護/hardeningの証拠を確認。端末が範囲外なら組織側未確認として残し、依頼外の端末設定調査/変更を自動実行しない。
