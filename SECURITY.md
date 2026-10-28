# 安全上の問題の報告

管理者・配布者: **tug-hour**

## 受付状況と窓口

公開先は[tug-hour/secure-development-review-skill](https://github.com/tug-hour/secure-development-review-skill)です。一般質問・改善提案は公開先のIssues／PRで受け付けます。

**Private vulnerability reportingは未設定で、非公開の報告受付はまだ開始していません。** 公開IssueやPRへ機密性のある詳細を投稿せず、利用可能な非公開窓口の案内を待ってください。代替のメールアドレスやフォームは指定していません。

非公開受付を開始した場合は、この文書で案内します。有効化後の報告場所は公開先のSecurity → Advisories → Report a vulnerabilityです。[GitHub公式説明](https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/configure-vulnerability-reporting/configure-for-a-repository)も参照してください。

## 対象と報告内容

対象は、このスキルの指示・参照資料・配布物にある、安全でない判断を誘発する記述や情報露出につながる問題です。一般的な文章修正・機能提案は、公開後のIssues／PRを利用してください。

報告には、対象の版またはcommit、該当ファイル・箇所、期待する判断と実際の問題、成立条件・影響を含めてください。再現資料は架空の値や権限のある最小の例で示し、実トークン・秘密鍵・認証情報・個人情報・機密コードやログは送らないでください。第三者のシステムへの攻撃や無断の検証を求めるものではありません。

## 対応範囲

tug-hourが個人運営の範囲で報告を確認し、必要に応じて修正や注意事項を整理します。固定の回答・修正期限、継続サポート、報奨金は約束しません。この文書は、見逃しゼロやスキルの安全性を保証するものではありません。

## ライセンス

本ファイルのうちtug-hourが権利を許諾できる部分にはCC BY-SA 4.0を適用します。対象範囲・第三者の条件は[README](README.md)・[LICENSE](LICENSE)・[THIRD-PARTY-NOTICES](THIRD-PARTY-NOTICES.md)を参照してください。報告時の情報の扱いや作者の運用方針を、ライセンスへ追加の利用制限として課すものではありません。
