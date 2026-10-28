# 出典・採否登録 — セキュア開発レビュー

確認日: 2026-10-28（日本時間）。これは7系統の主要原則を統合した原案であり、各言語版・全記事・全規則の逐条精査、ASVS準拠、SSDF準拠を主張しない。原案作成時のライブ資料のコミットと逐文出典は固定していない。公開準備では現在の比較点と本スキルの観点を対応付けたが、それは作成時の版の復元や全原典の逐条対応を意味しない。現在の権利表示と本スキルでの整理は配布物のTHIRD-PARTY-NOTICES.mdに示す。普段のレビューで原典全量を再取得しない。仕様・対応版が判断を変え得る箇所だけ一次資料で確認する。

| 出典 | 版・位置付け | 採用・変換 | 共通必須から外したもの |
|---|---|---|---|
| [ASVS](https://asvs.owasp.org/) / [5.0.0要件CSV](https://raw.githubusercontent.com/OWASP/ASVS/v5.0.0/5.0/docs_en/OWASP_Application_Security_Verification_Standard_5.0.0_en.csv) | 安定版5.0.0 | identity: V6–V10、input: V1/V2.2/V3/V4/V5/V13.2、data: V11–V14/V16、state: V2/V15.4、supply: V13/V15.2 | 全レベル・Web固有機能の一律適用。記載した章参照は範囲の案内で、全要件採用ではない |
| [Cheat Sheets](https://cheatsheetseries.owasp.org/) | ライブWeb、コミット未固定 | 下記の実装・検証観点 | すべての記事・ツール・製品を毎回使用すること |
| [IPA安全なウェブサイト](https://www.ipa.go.jp/security/vuln/websecurity/about.html) | 改訂第7版第4刷、2021-03-31 | 対策の性質を区別する考え方を参照（本文p.4の「根本的解決」「保険的対策」）。入力と処理境界について第1章1.1/1.2/1.3/1.5を参照。原典の説明文・図表・実装例・チェックリストは同梱しない | 古いブラウザ対策の継承。X-XSS-Protection有効化等は現行指針と照合し不採用 |
| [SEI CERT](https://cmu-sei.github.io/secure-coding-standards/) | C等のオンライン規則。公開版と作業版を区別 | 数値範囲、エラー、資源、並行処理の安全性の前提 | 特定言語のAPI・構文禁止や優先度を他言語へ一律適用すること |
| [SSDF](https://csrc.nist.gov/pubs/sp/800/218/final) | 1.1正式版。1.2ドラフトは規範に使わない | 設計・コード検証・保護・脆弱性対応の連携 | 組織全活動の毎回のコードレビュー化。実装例の一律必須化 |
| [ANSSI Rust](https://anssi-fr.github.io/rust-guide/print.html) | Webはunstable | 危険な能力の隔離、依存検証、安全性の前提 | Rust固有のツール・設定・書式を共通要件にすること |
| [Go Security](https://go.dev/doc/security/) / [Best Practices](https://go.dev/doc/security/best-practices) | ライブWeb | 依存情報、更新の吟味、自動検査の能力と限界 | Go固有コマンドの必須化、未実行経路まで検証済みとすること |

詳細の主要参照:
- identity: [Authorization](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- input: [Input Validation](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)、[File Upload](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)、[SSRF](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- data: [Logging](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)、[HTTP Headers](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html)（旧対策の照合）
- state: [DoS](https://cheatsheetseries.owasp.org/cheatsheets/Denial_of_Service_Cheat_Sheet.html)、ASVS V2.3/V15.4
- supply: [Software Supply Chain Security](https://cheatsheetseries.owasp.org/cheatsheets/Software_Supply_Chain_Security_Cheat_Sheet.html)
- 選別・影響範囲: [Secure Code Review](https://cheatsheetseries.owasp.org/cheatsheets/Secure_Code_Review_Cheat_Sheet.html)、[Attack Surface Analysis](https://cheatsheetseries.owasp.org/cheatsheets/Attack_Surface_Analysis_Cheat_Sheet.html)。外部攻撃面だけに限定しない。

この原案の5分野、読み分け、証拠再利用・終了条件・報告規約は独自の設計であり、原典の規範要件と区別する。原典の特定条件での強い要求を、意味の違う一般的推奨へ書き換えない。採用を追加するときは適用条件と必要証拠も更新する。
