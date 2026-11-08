# SSDF 1.1 PSのTasks

[適用・証拠・出典/権利](../ssdf.md)と併読する。英文は公式Table 1のTasks列を変更せず保持。直後の日本語は対象への適用と必要な根拠の案内であり、英文条件を置き換えない。実装例列/他規格への参考対応は必須条件にしない。

出典: Souppaya M, Scarfone K, Dodson D (2022), Secure Software Development Framework (SSDF) Version 1.1: Recommendations for Mitigating the Risk of Software Vulnerabilities, NIST SP 800-218, National Institute of Standards and Technology, https://doi.org/10.6028/NIST.SP.800-218 (accessed 2026-11-01). Republished courtesy of the National Institute of Standards and Technology.

変更: 2026-11-01に公式補助xlsxからTasks列のみ抜粋し群別配置、日本語の適用/根拠案内を追加。NISTによる承認/公式訳ではない。

## PS.1.1

```text
PS.1.1: Store all forms of code – including source code, executable code, and configuration-as-code –  based on the principle of least privilege so that only authorized personnel, tools, services, etc. have access.
```

source、実行形式、configuration-as-codeの保存先に適用。人/tool/serviceを含む最小権限の実設定とアクセス経路を確認。

## PS.2.1

```text
PS.2.1: Make software integrity verification information available to software acquirers.
```

release又は利用者への引渡しがある場合。取得者が利用できる整合性検証情報と対象成果物の結び付きを確認。特定署名方式を唯一の方法にしない。

## PS.3.1

```text
PS.3.1: Securely archive the necessary files and supporting data (e.g., integrity verification information, provenance data) to be retained for each software release.
```

releaseがある場合。必要fileと整合性/由来等の補助dataが安全に保存される根拠を確認。必要保持範囲が不明なら未確認。

## PS.3.2

```text
PS.3.2: Collect, safeguard, maintain, and share provenance data for all components of each software release (e.g., in a software bill of materials [SBOM]).
```

releaseがある場合。全componentのprovenance収集/保護/維持/共有を確認。SBOMは例であり、特定形式を義務化しない。
