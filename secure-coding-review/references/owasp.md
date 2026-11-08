# OWASP Cheat Sheetの評価基準

固定範囲はOWASP Cheat Sheet Seriesのcommit `87a1faf18a29a5c33d040b000eb3c60330c30b2c`にある下記10記事。2026-11-01に公式raw URLから新規取得し、リダイレクトは追っていない。旧スキル・旧条件・過去評価を使用していない。対策節を原文で保持し、日本語の適用/根拠/採否と、原典見出し・直属本文の行番号へ対応付けた。数値、例外、代替、前提、規範の強さを短い要約で失わないための構成である。

これは他系統と組み合わせる基準部品であり、単独で完成したレビュー用スキルではない。具体的な疑問に関係する条件を選んで使う。「読み終えた」「一覧がある」を「実装が満たす」の根拠にしない。

## 対象に応じて読む条件

下表から具体的な疑問に関係する記事・節へ進む。全10記事の適用判定や全量読取りを先行工程にしない。対象の役割・境界と原典の前提を照合し、必要な部分だけ読む。

| 読む基準 | 適用と主要な根拠 |
|---|---|
| [Secure Code Review](owasp/Secure_Code_Review.md) | 範囲・変更影響・安全なデータ流・状態/並行処理・暗号・メモリ・設定・是正の確認。差分/全体レビューを依頼と既存根拠に合わせる |
| [Attack Surface Analysis](owasp/Attack_Surface_Analysis.md) | 入口/出口/保護対象/役割/変更、公開/内部、proxy背後、backup/旧版/未使用機能、実配備との差 |
| [Authorization](owasp/Authorization.md) | 最小権限、既定拒否、全要求・objectごとのサーバ側検査、静的資源、拒否/例外、監査と回帰 |
| [Input Validation](owasp/Input_Validation.md) | 解析前の資源制限、使用表現の構文/意味、項目/深さ/配列、Unicode/regex、信頼境界、部分検証後の継続禁止 |
| [File Upload](owasp/File_Upload.md) | ファイル受信/一時処理/保存/配信の拡張子・内容・名前・権限・隔離・容量・変換の合成 |
| [SSRF Prevention](owasp/Server_Side_Request_Forgery_Prevention.md) | 限定宛先/任意外部宛先の区別、IPv4/IPv6・DNS・実接続固定・再試行/redirect・parser差・network/metadata |
| [Logging](owasp/Logging.md) | アプリ内/関連ログの一貫性、目的に合うイベント/属性、秘密除外、形式別encode、枯渇/失敗、改変/アクセス、監視と保持/廃棄。用途の列挙は全項目必須ではなく、目的別に分離するという原典のusuallyも保持 |
| [HTTP Headers](owasp/HTTP_Headers.md) | 応答/クライアントに対する効果・例外、CORS/cache/HSTS/frame/CSP/cookie、廃止機能、配信file、重複とerror等の実効応答 |
| [Denial of Service](owasp/Denial_of_Service.md) | CPU/メモリ/容量/帯域/接続/セッション/依存先、検査順序、縮退/隔壁、slow client/timeout/rate、lockout悪用 |
| [Software Supply Chain Security](owasp/Software_Supply_Chain_Security.md) | source/依存/開発/ビルド/配布/実行の権限・秘密・監視・供給者・固定/更新・署名/来歴・隔離・成果物 |

## 適用と判断

具体的な判断に使う条件の前提と根拠を確認する。共通根拠は再利用し、重要な判断に足りない証拠を同じ経路ごとに補う。全条件の適用/非適用や充足判定は要求しない。原典の推奨未採用を重大脆弱性とせず、成立条件・影響と規範の位置付けを分ける。

原典のmust/never、should/recommend、may/consider、if/where possibleはそれぞれの強さのまま使う。例や製品名、リンク先全量を新たな義務にしない。SBOMは手段の候補であり独自の提出義務へしない。レビュー工程のチーム/ツール例は、別担当の起動やインストール/実行/通信の権限を与えない。

ファイル内の採否表は原典の全見出しを列挙し、採用節と背景/参考文献/ツール例等の対象外を区別する。コード/図を除去しても、本文にある方式固有の条件や制約は残す。例・参照先にしかない挙動が対象の判断に必要なら原典の必要範囲へ戻り、確認前は未確認とする。数値例はその方式/契約での意味を保ち、他方式の普遍閾値へ広げない。

HTTP HeadersのSet-Cookie/CSP詳細案内は別記事を参照しており、この10記事だけで詳細条件まで具体化済みではない。該当2箇所を未確定として表示した。他系統の具体条件と既存根拠で必要な条件を判定できなければ、必要な一次資料を確認するまで未確認を残す。固定範囲をリンク先全量へ再帰拡張しない。

## 原典固定点と権利

各記事冒頭に公式URL、原典bytes、SHA-256、取得日を記録した。採用本文はCC BY-SA 4.0の編集物として原典の著作者「OWASP Cheat Sheet Series contributors」、出所、許諾、変更を表示し、同許諾で提供する。[許諾本文](owasp/LICENSE-CC-BY-SA-4.0.md)は同じcommitの[LICENSE.md](https://raw.githubusercontent.com/OWASP/CheatSheetSeries/87a1faf18a29a5c33d040b000eb3c60330c30b2c/LICENSE.md)から取得した（18,637 bytes、SHA-256 `2b47ecdd5078e948395bc5980226933854228ff688f10d1dd059208be4956084`）。この索引も同編集物としてCC BY-SA 4.0で提供する。IPA等の別原典へこの許諾を広げない。

変更は、必要な対策節の選択、図/コードの除去、見出し/行対応・日本語説明・採否の追加、相対リンクの公式固定commit化、checkboxの通常箇条書き化、単一行URLのコード記法の正規化。AuthorizationのURL例は内容を変えず、残り本文がcode blockにならないようinline codeへ整形した。リンクされた外部作品・コード・図は同梱しない。原文は英語のまま保持し、公式訳/認定と表示しない。

整合確認結果: 全249見出しの直属範囲が採否表と一致し、本文保持212範囲（条件付き採用210・上記未確定2）を確認した。残る37範囲は理由付き対象外。23資料（OWASP10記事/許諾とIPA案内/11節）の取得hashも再照合した。

整合確認: 元のMarkdownをfence外の見出しで分け、全見出しの直属行範囲が各採否表に1対1対応すること、採用した全範囲が本文にあること、元の対策本文との相違が表示した編集だけであることを静的に確認する。これは原典対応の検査であり、実対象の検出性能・実装達成・セキュリティ保証の検証ではない。

案内の変更: 2026-11-03、全件評価の手順から必要時参照へ変更。原典の具体条件・帰属・許諾は保持。
