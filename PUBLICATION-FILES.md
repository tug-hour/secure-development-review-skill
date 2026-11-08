# 公開対象と差分

対象は下記99ファイルのみ。現行スキル84ファイル、旧スキル9ファイル、root文書6ファイル。非公開リポジトリの履歴・設計書・サンプル・生報告・計測・セッション情報・ローカル環境は取り込まない。2担当方式は未実装の状態説明だけをREADMEに記載する。

## 導入済み版との対応

2026-11-09に導入済み84ファイルと開発版の一致を確認。SKILL.mdを含む82ファイルは公開コピーでも同一。sources.mdは内部リンク/公開前の記述、cert-c/err.mdは原典由来ローカルリンクだけを公開用に整理した。原典の条件・数値・例外とスキルの動作指示は変えていない。全84ファイルはMarkdown/textで、実行物・symlink・追加依存なし。

## 権利・情報の確認

7系統の帰属・版・変更・条件はTHIRD-PARTY-NOTICES.mdと各参照に記載。OWASP/CERT/ANSSIの同梱LICENSE3件は公式固定版とbyte一致を確認。IPAのサイト条件/FAQ、ASVSの表示、GoのCopyright、NIST再掲載方針も確認。原典由来の著作者名・公開書誌情報は保持し、個人情報の削除を理由に必須の帰属を消さない。

配布ファイル全体を対象にローカルパス・秘密鍵/トークン等の代表的表現とリンクを確認し、該当するローカルパス1件は公開用コピーから除去。ユーザーの実データ・認証情報・連絡先を取り込まず、許諾者/配布者表示はtug-hourのみ。すべての情報漏えいや第三者の権利不存在を保証するものではない。旧版フォルダ・LICENSE・SECURITY.mdは変更せず、旧READMEはLEGACY.mdにbyte一致で保持する。

性能比較・新しいレビュー試験・スキル導入・課金API・マージは実施しない。

## ファイル一覧

- `LEGACY.md`
- `LICENSE`
- `PUBLICATION-FILES.md`
- `README.md`
- `SECURITY.md`
- `THIRD-PARTY-NOTICES.md`
- `secure-coding-review/SKILL.md`
- `secure-coding-review/references/asvs.md`
- `secure-coding-review/references/asvs/v1.md`
- `secure-coding-review/references/asvs/v10.md`
- `secure-coding-review/references/asvs/v11.md`
- `secure-coding-review/references/asvs/v12.md`
- `secure-coding-review/references/asvs/v13.md`
- `secure-coding-review/references/asvs/v14.md`
- `secure-coding-review/references/asvs/v15.md`
- `secure-coding-review/references/asvs/v16.md`
- `secure-coding-review/references/asvs/v17.md`
- `secure-coding-review/references/asvs/v2.md`
- `secure-coding-review/references/asvs/v3.md`
- `secure-coding-review/references/asvs/v4.md`
- `secure-coding-review/references/asvs/v5.md`
- `secure-coding-review/references/asvs/v6.md`
- `secure-coding-review/references/asvs/v7.md`
- `secure-coding-review/references/asvs/v8.md`
- `secure-coding-review/references/asvs/v9.md`
- `secure-coding-review/references/cert-c.md`
- `secure-coding-review/references/cert-c/LICENSE.txt`
- `secure-coding-review/references/cert-c/api.md`
- `secure-coding-review/references/cert-c/arr.md`
- `secure-coding-review/references/cert-c/con.md`
- `secure-coding-review/references/cert-c/dcl.md`
- `secure-coding-review/references/cert-c/env.md`
- `secure-coding-review/references/cert-c/err.md`
- `secure-coding-review/references/cert-c/exp.md`
- `secure-coding-review/references/cert-c/fio.md`
- `secure-coding-review/references/cert-c/flp.md`
- `secure-coding-review/references/cert-c/int.md`
- `secure-coding-review/references/cert-c/mem.md`
- `secure-coding-review/references/cert-c/msc.md`
- `secure-coding-review/references/cert-c/pos.md`
- `secure-coding-review/references/cert-c/pre.md`
- `secure-coding-review/references/cert-c/sig.md`
- `secure-coding-review/references/cert-c/str.md`
- `secure-coding-review/references/cert-c/win.md`
- `secure-coding-review/references/criteria.md`
- `secure-coding-review/references/go.md`
- `secure-coding-review/references/ipa.md`
- `secure-coding-review/references/ipa/access-control.md`
- `secure-coding-review/references/ipa/bach-overflow.md`
- `secure-coding-review/references/ipa/clickjacking.md`
- `secure-coding-review/references/ipa/cross-site-scripting.md`
- `secure-coding-review/references/ipa/csrf.md`
- `secure-coding-review/references/ipa/http-header.md`
- `secure-coding-review/references/ipa/mail-header.md`
- `secure-coding-review/references/ipa/os-command.md`
- `secure-coding-review/references/ipa/parameter.md`
- `secure-coding-review/references/ipa/session-management.md`
- `secure-coding-review/references/ipa/sql.md`
- `secure-coding-review/references/owasp.md`
- `secure-coding-review/references/owasp/Attack_Surface_Analysis.md`
- `secure-coding-review/references/owasp/Authorization.md`
- `secure-coding-review/references/owasp/Denial_of_Service.md`
- `secure-coding-review/references/owasp/File_Upload.md`
- `secure-coding-review/references/owasp/HTTP_Headers.md`
- `secure-coding-review/references/owasp/Input_Validation.md`
- `secure-coding-review/references/owasp/LICENSE-CC-BY-SA-4.0.md`
- `secure-coding-review/references/owasp/Logging.md`
- `secure-coding-review/references/owasp/Secure_Code_Review.md`
- `secure-coding-review/references/owasp/Server_Side_Request_Forgery_Prevention.md`
- `secure-coding-review/references/owasp/Software_Supply_Chain_Security.md`
- `secure-coding-review/references/rust.md`
- `secure-coding-review/references/rust/LICENSE.md`
- `secure-coding-review/references/rust/devenv.md`
- `secure-coding-review/references/rust/errors.md`
- `secure-coding-review/references/rust/guarantees.md`
- `secure-coding-review/references/rust/integer.md`
- `secure-coding-review/references/rust/introduction.md`
- `secure-coding-review/references/rust/libraries.md`
- `secure-coding-review/references/rust/naming.md`
- `secure-coding-review/references/rust/standard.md`
- `secure-coding-review/references/rust/unsafe-ffi.md`
- `secure-coding-review/references/rust/unsafe-generalities.md`
- `secure-coding-review/references/rust/unsafe-memory.md`
- `secure-coding-review/references/rust/unsafe.md`
- `secure-coding-review/references/sources.md`
- `secure-coding-review/references/ssdf.md`
- `secure-coding-review/references/ssdf/po.md`
- `secure-coding-review/references/ssdf/ps.md`
- `secure-coding-review/references/ssdf/pw.md`
- `secure-coding-review/references/ssdf/rv.md`
- `secure-development-review/SKILL.md`
- `secure-development-review/references/data.md`
- `secure-development-review/references/identity.md`
- `secure-development-review/references/index.md`
- `secure-development-review/references/input.md`
- `secure-development-review/references/selection-contract.md`
- `secure-development-review/references/sources.md`
- `secure-development-review/references/state.md`
- `secure-development-review/references/supply.md`
