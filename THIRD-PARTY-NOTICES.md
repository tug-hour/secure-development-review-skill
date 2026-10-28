# 原典の出典・権利表示

本ファイルは配布候補v0.3-rc.1の原典・権利表示です。許諾者・配布者はtug-hourで、tug-hourが許諾できる整理・説明部分にCC BY-SA 4.0を適用します。対象範囲は[README](README.md)、正式条項は[LICENSE](LICENSE)を参照してください。第三者の部分には各原典の条件が残ります。原典そのものを同梱せず、原典の安全原則・技術観点を参照し、本スキル側で日本語の5分野に整理しています。選別・証拠再利用・終了・報告は本スキルの運用設計で、レビュー範囲や報告項目には原典と共通する考え方もあります。各原典の全規則・章構成・報告テンプレートを翻訳・再現したものではありません。権利表示確認は2026-10-28〜29、表現の限定比較は2026-10-29（日本時間）。最新版全体の技術精査をした日付ではありません。

本スキル独自の構成と原典の規範要件を区別し、原典の団体による認証・支持・承認は主張しません。以下は現在の一次資料に示された条件であり、過去の別版や、本原案の全表現について権利処理が完了したという宣言ではありません。

| 原典・表示対象 | 確認した権利条件と出典 | 本スキルでの参照・整理 |
|---|---|---|
| OWASP Application Security Verification Standard 5.0.0、The OWASP Foundation（© 2008–2025） | [5.0.0の権利表示](https://github.com/OWASP/ASVS/blob/180622b1329ee7ed349c9f5ce5e14fa91f42c359/5.0/en/0x01-Frontispiece.md): CC BY-SA 4.0 | 認証・認可、入力、データ、状態、実行環境の観点を選択・統合。全要件や章構成を再現せず、準拠も主張しない |
| OWASP Cheat Sheet Series、OWASP Foundationおよび各貢献者 | [公式サイト](https://cheatsheetseries.owasp.org/)・[LICENSE](https://github.com/OWASP/CheatSheetSeries/blob/5d79a8ac36f7706fc8b1ac45f84d30ce7be2c7e7/LICENSE.md): CC BY-SA 4.0 | Authorization、Input Validation、File Upload、SSRF、Logging、DoS、Supply Chain、Review、Attack Surface等の観点を要約・再構成。原典の各記事はreferences/sources.mdのURLを参照 |
| IPA『安全なウェブサイトの作り方』改訂第7版第4刷、独立行政法人情報処理推進機構（著作・制作、2021-03-31） | [原典](https://www.ipa.go.jp/security/vuln/websecurity/about.html)・[サイト利用条件](https://www.ipa.go.jp/siteinfo.html): 出所明示、個別条件があれば従う | 対策の性質を区別する考え方を参照（本文p.4、PDF第6ページの「根本的解決」「保険的対策」）。入力と処理境界に関連する第1章1.1/1.2/1.3/1.5も参照。説明文・図表・PDF・実装例・チェックリストを同梱しない。一般条件をCC再許諾の根拠にしない |
| SEI CERT Coding Standards、Carnegie Mellon University（現在のLICENSEは© 2026） | [公式サイト](https://cmu-sei.github.io/secure-coding-standards/)・[LICENSE](https://github.com/cmu-sei/secure-coding-standards/blob/4dda8e2e628a1c6f8b810ff1657b54b9995b2c18/LICENSE): 文章CC BY 4.0、コード例MIT | 数値・エラー・資源・並行性等の前提を言語共通の観点へ整理。コード例は同梱しない。別版・旧Wiki・刊行物の条件まで同一とは扱わない。CERT等の商標権は保持される |
| NIST SP 800-218 SSDF Version 1.1（2022-02）、National Institute of Standards and Technology（NIST） | [出版情報](https://csrc.nist.gov/pubs/sp/800/218/final)・[原典PDF第3ページ](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf): 米国内の著作権保護対象外、帰属表示を推奨 | 設計・検証・保護・脆弱性対応のつながりを参照。米国外まで無条件のpublic domainとは表示しない。NIST等による推奨・承認は主張しない |
| ANSSI Secure Rust Guidelines、ANSSI（現在の本文履歴参照点: 2026-04-07） | [原典](https://anssi-fr.github.io/rust-guide/print.html)・[LICENSE](https://github.com/ANSSI-FR/rust-guide/blob/b7ae8943d25a625e52a5e26fbda11454b8cc77ea/LICENSE.md): Open Licence 2.0 / Licence Ouverte 2.0 | 危険な能力の隔離、依存やメモリの前提を共通観点へ整理。出所と更新情報を示し、公式の地位・支持は示さない。Rust固有のコードは同梱しない |
| Go Security / Security Best Practices、The Go Authors | [Security](https://go.dev/doc/security/)・[Best Practices](https://go.dev/doc/security/best-practices)・[Copyright](https://go.dev/copyright): 特記を除くサイト文章CC BY 4.0、コードBSD | 依存情報・更新・検査能力の観点を参照。コード・画像は同梱しない。個別例外は全て同じライセンスとみなさない |

CCの条件は[CC BY-SA 4.0正式条項](https://creativecommons.org/licenses/by-sa/4.0/legalcode.ja)・[CC BY 4.0正式条項](https://creativecommons.org/licenses/by/4.0/legalcode.ja)を参照してください。帰属表示、ライセンス参照、変更の表示、該当する継承条件などを保持する方針です。これらの表を元のライセンス条項の代わりにはしません。

配布候補v0.3-rc.1では、15観点と共通部分を対応付けた既存記録に基づき、IPAの用語・参照位置、inputのASVS V2.2/V13.2とIPAの関連節、dataのASVS V14、原案作成時の参照版と公開準備の比較点の区別を出典表示に反映しています。レビュー手順・安全条件は変更していません。原典の権利条件・本文の比較結果は上記確認期間の記録を再利用し、今回の再調査とは扱いません。性能評価はv0.2の既存結果のみで、v0.3-rc.1の追加評価はしていません。

上記確認期間の比較では原典の段落・コード・図表・テンプレートの再掲は特定しませんでしたが、作成時の逐文出典やすべての翻案性を確定したものではありません。現在の本文比較点はASVS `180622b1329ee7ed349c9f5ce5e14fa91f42c359`、Cheat Sheet Series `9d41c054686e35fb56070b4b6271ae2829edea6a`、CERT `edf7b9081c3d18e486c5765184473066a0f8ec2a`、ANSSI `84e6ae181712c9ed797aeaf695c9965a13a1d5fa`。IPAは上記刷、SSDFは1.1、Goは2026-10-29のWeb表示です。これらを本スキル作成時の固定版とは表示しません。具体的な保護対象表現の利用が別途判明した場合は、その箇所の条件を解決してから配布します。
