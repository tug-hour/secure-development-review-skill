# ASVS 5.0.0の評価条件

ASVSはweb application / serviceを主対象とする。全345要求を採用候補として保持し、下の機能別参照で要求本文を読む。分割先が具体的な評価条件であり、章名だけでレビューしない。非web対象でも入力、認可、暗号、依存、並行性等に関係する条件を検討し、HTTP/browser固有機能の不存在は根拠付きで非適用にできる。言語や構成が不明なだけでは除外しない。

## 判断に使うとき

対象の具体的な疑問に関係する要求を選び、条件・数値・例外・代替・水準を原文で確かめる。全345要求の適用/充足判定や、全章の非適用説明は要求しない。選んだ要求も、その要求が対象に適用されるかを確認して使う。根拠不足を安全の証明へ変えない。

Lは最低要求水準で、重大度ではない。指定水準・契約がなければ創作せず、保護対象とリスクに応じて参照する。例示を唯一の手段にせず、文書/運用要求をコードだけで充足としない。版と元IDを区別して参照する。

## 機能別の読み先

下表から具体的な疑問に関係する章・節へ進む。全章を順番に読む工程ではない。表の根拠例は原文条件を狭めない。

| 原典章・読み先 | 適用を確かめる機能 / 共通根拠の例 | 要求数 |
|---|---|---|
| [V1 Encoding and Sanitization](asvs/v1.md) | 復号・出力・パーサー・メモリー / 入力から解釈器までの変換順序、sink、解放、異常入力 | 30 |
| [V2 Validation and Business Logic](asvs/v2.md) | 入力検証・業務・有限資源 / 文書と信頼される検証層、状態遷移/原子性/排他、個人別と全体の制限 | 13 |
| [V3 Web Frontend Security](asvs/v3.md) | ブラウザー・cookie・外部asset / 実応答、DOM、cross-origin、非対応時の動作 | 31 |
| [V4 API and Web Service](asvs/v4.md) | HTTP・GraphQL・WebSocket / proxyからbackendの境界とheader、query負荷、WSS/session | 16 |
| [V5 File Handling](asvs/v5.md) | upload・展開・保存・download / 容量と展開前制限、内容/型、symlink/path、公開設定 | 13 |
| [V6 Authentication](asvs/v6.md) | 認証・登録・password・MFA・回復・IdP / 全経路、数値、生成/保存/期限/失効、assertion | 47 |
| [V7 Session Management](asvs/v7.md) | session・SSO・再認証・logout / token生成/更新、idle/absolute期限、失効後の再利用拒否 | 19 |
| [V8 Authorization](asvs/v8.md) | 認可・tenant・管理 / 主体とresourceの規則、強制層、変更反映、代理呼出し、拒否経路 | 13 |
| [V9 Self-contained Tokens](asvs/v9.md) | self-contained token / 検証前利用、algorithm/key信頼元、purpose/audience/期限 | 7 |
| [V10 OAuth and OIDC](asvs/v10.md) | OAuth/OIDC各役割 / flowとtransaction束縛、再利用拒否、PKCE、claim、consent/失効 | 36 |
| [V11 Cryptography](asvs/v11.md) | 暗号・鍵・password保存・乱数 / inventory、lifecycle、bit/parameter、nonce、MAC/署名、移行 | 24 |
| [V12 Secure Communication](asvs/v12.md) | 外部/内部通信・TLS・mTLS / protocol/cipher、証明書/信頼store、検証失敗/fallback | 12 |
| [V13 Configuration](asvs/v13.md) | 構成・backend・secret・本番 / 接続資源/timeout/retry、service権限、rotation、配布物 | 21 |
| [V14 Data Protection](asvs/v14.md) | 機微情報・client storage / 保護区分、URL/log/cache/外部送信、retention、消去 | 13 |
| [V15 Secure Coding and Architecture](asvs/v15.md) | 依存・危険機能・並行処理 / 版/更新期限、推移依存取得元、field/型、race/TOCTOU/lock | 21 |
| [V16 Security Logging and Error Handling](asvs/v16.md) | log・監視・error / metadata/時間、機微値/injection、保護、拒否/例外/障害時の状態 | 17 |
| [V17 WebRTC](asvs/v17.md) | WebRTC / TURN IP/port、DTLS鍵/fingerprint、SRTP認証、異常messageとflood | 12 |

## 出典、固定点、権利

OWASP Foundation / OWASP ASVS Project contributors, *OWASP Application Security Verification Standard 5.0.0* (May 2025)。[公式CSV](https://raw.githubusercontent.com/OWASP/ASVS/v5.0.0/5.0/docs_en/OWASP_Application_Security_Verification_Standard_5.0.0_en.csv)を2026-11-01に新規取得。版タグ`v5.0.0`と生バイトSHA-256 `98c8fe911b9edb403af8ee05d3ce8201ecac2659e313b053890a62847cdcf680`で固定、105,100 bytes。タグの不変性を仮定せず、このハッシュの内容を基準とする。

[同タグ公式READMEのLicense](https://github.com/OWASP/ASVS/blob/v5.0.0/README.md#license)はプロジェクト全体を[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)と明示。[同タグLICENSE.md](https://github.com/OWASP/ASVS/blob/v5.0.0/LICENSE.md)で確認。要求文だけを評価のため保持し、解説章・図・実行コード・外部規格は同梱しない。要求内の短い例やliteralは条件保持に必要な部分としてそのまま残した。

変更: 2026-11-01にCSVのchapter/sectionでMarkdownへ配置し、適用判断・根拠の日本語案内を追加。要求ID/文/Lは翻訳/改変なし。`asvs.md`と`asvs/*.md`の翻案はCC BY-SA 4.0で提供する。この許諾を他系統やスキル全体へ自動拡張しない。著作者・出典・ライセンス・変更表示を保持する。OWASPの承認・公式訳・適合認証ではない。

静的確認: 公式CSVの345個の一意な元IDが17章の本文へ各1回存在し、各IDの要求文とLが元データと完全一致することを照合した。これは構造と転記の確認であり、実対象のレビュー性能・安全性・ASVS適合性の検証ではない。専用reader、状態DB、生成metadataは持たせない。

案内の変更: 2026-11-03、全件評価の手順から必要時参照へ変更。原典の具体条件・帰属・許諾は保持。
