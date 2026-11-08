# CERT C: C Rules / Recommendations の評価条件

## 適用と判断

C実装、Cライブラリ、ネイティブ境界、実行環境とビルド条件を対象とする。採用単位は主一覧に掲載された各元IDで、独自の規則集合へ再分類しない。下表の機能ごとの本文に、**各元ID、原典のRule/Recommendation、採否、具体的条件、前提・例外、固定原典URLと元ファイルhash**を置く。Ruleは原典で規範要求、Recommendationは安全・信頼性・セキュリティを改善する指針であり、推奨の未実施だけを必須違反に変換しない。

対象の具体的な疑問に関係するIDを選び、C規格版・API/型・環境と本文の前提/例外を確認する。全採用IDを対象へ照合する工程は設けない。他言語にC固有のAPI/規約を強制せず、共通の共有・所有権・境界リスクは対象で判断する。

失敗後の状態、長さ/単位、整数幅、寿命/所有者、同期等が結論を左右する場合は、該当本文と必要な固定原典の例を読む。原典リンクや静的解析の無警告だけで安全とせず、Recommendationとの不一致を重大リスクへ機械変換しない。

## 規範保持と採否

短い訳・タイトルだけへの圧縮で例外や環境条件を失わないため、原典の規範・例外とそれらを説明する英語本文を機能別に保持した。Risk Assessment以降の独自採点、製品別検出表、関連脆弱性/文献一覧、添付、コードブロック、図は収録しない。例の**説明文は保持**し、数値・否定・代替・言語版・型/API制約は改変しない。front matterを除き、見出し・リンクの整形と除去箇所を明示する編集だけを行った。コードを除去して本文から判断できなくなった条件は、固定原典の当該例が必要な未確認条件として残る。したがって、この抜粋だけを読んで逐条適合や全意味の無損失を保証しない。

主一覧に掲載された305 IDを新規照合した。122 Rule、183 Recommendationで、旧成果物の件数を使用していない。全305 IDのうち以下以外の300 IDを条件付き採用し、適用条件は各本文に保持する。

| 元ID | 採否 | 扱い |
|---|---|---|
| EXP14-C | 対象外（廃止） | 原典の置換先INT02-Cで評価し、旧規則を独立強制しない |
| MSC07-C | 対象外（廃止） | 原典の統合先MSC12-Cで評価する |
| ENV04-C | 未確定 | 原典がUNDER CONSTRUCTION。OpenMP環境変数の記述を残すが完成した規範に補作しない |
| CON03-C | 未確定 | 原典front matterにincomplete。記載済みの可視性条件を残すが完成を推定しない |
| MSC25-C | 未確定 | 原典がstubで例も未記入。暗号アルゴリズムの実基準をこのIDから創作しない |

FIO23-CとMEM07-CのDeprecated表示は全面廃止ではない。**C23だけに適合すればよいコードには適用されず、それ以前への適合が必要なコードには適用**される。両IDを条件付き採用として保持した。

APIとWINはカテゴリ自体がunder construction。API Rulesは掲載IDが0件でも、APIの安全性を評価不要とはしない。WINはMicrosoft Windows API / Visual C++、POSはPOSIX関数が対象で、どちらも全Cアプリに適用されるcore標準ではない。API/WIN内の掲載IDを保持しても、未掲載の規範やカテゴリ全体の完成を推定しない。標準サイト全体がwork in progressであり、未完成/誤りがあり得るという原典の留保を維持する。

## 機能別本文

下表から関係する機能ファイルの`## 元ID. ...`へ進む。同じ機能のRuleとRecommendationを一緒に収録し、各項目の種別と例外を分離した。全IDの把握や全17本文の一括読取は要求しない。既存根拠で決まるIDを再取得せず、判断が必要なIDとその前提・例外の部分だけを確認する。

| 機能 | Rule数 | Recommendation数 | 本文 | 全元ID（R=Rule、Rec=Recommendation） |
|---|---:|---:|---|---|
| API: API設計 | 0 | 9 | [api.md](cert-c/api.md) | R:  / Rec: API00-C, API01-C, API02-C, API03-C, API04-C, API05-C, API07-C, API09-C, API10-C |
| ARR: 配列 | 6 | 3 | [arr.md](cert-c/arr.md) | R: ARR30-C, ARR32-C, ARR36-C, ARR37-C, ARR38-C, ARR39-C / Rec: ARR00-C, ARR01-C, ARR02-C |
| CON: 並行処理 | 13 | 9 | [con.md](cert-c/con.md) | R: CON30-C, CON31-C, CON32-C, CON33-C, CON34-C, CON35-C, CON36-C, CON37-C, CON38-C, CON39-C, CON40-C, CON41-C, CON43-C / Rec: CON01-C, CON02-C, CON03-C, CON04-C, CON05-C, CON06-C, CON07-C, CON08-C, CON09-C |
| DCL: 宣言・初期化 | 9 | 23 | [dcl.md](cert-c/dcl.md) | R: DCL30-C, DCL31-C, DCL36-C, DCL37-C, DCL38-C, DCL39-C, DCL40-C, DCL41-C, DCL42-C / Rec: DCL00-C, DCL01-C, DCL02-C, DCL03-C, DCL04-C, DCL05-C, DCL06-C, DCL07-C, DCL08-C, DCL09-C, DCL10-C, DCL11-C, DCL12-C, DCL13-C, DCL15-C, DCL16-C, DCL17-C, DCL18-C, DCL19-C, DCL20-C, DCL21-C, DCL22-C, DCL23-C |
| ENV: 環境 | 5 | 4 | [env.md](cert-c/env.md) | R: ENV30-C, ENV31-C, ENV32-C, ENV33-C, ENV34-C / Rec: ENV01-C, ENV02-C, ENV03-C, ENV04-C |
| ERR: エラー処理 | 4 | 7 | [err.md](cert-c/err.md) | R: ERR30-C, ERR32-C, ERR33-C, ERR34-C / Rec: ERR00-C, ERR01-C, ERR02-C, ERR04-C, ERR05-C, ERR06-C, ERR07-C |
| EXP: 式 | 15 | 16 | [exp.md](cert-c/exp.md) | R: EXP30-C, EXP32-C, EXP33-C, EXP34-C, EXP35-C, EXP36-C, EXP37-C, EXP39-C, EXP40-C, EXP42-C, EXP43-C, EXP44-C, EXP45-C, EXP46-C, EXP47-C / Rec: EXP00-C, EXP02-C, EXP03-C, EXP05-C, EXP07-C, EXP08-C, EXP09-C, EXP10-C, EXP11-C, EXP12-C, EXP13-C, EXP14-C, EXP15-C, EXP16-C, EXP19-C, EXP20-C |
| FIO: 入出力 | 13 | 20 | [fio.md](cert-c/fio.md) | R: FIO30-C, FIO32-C, FIO34-C, FIO37-C, FIO38-C, FIO39-C, FIO40-C, FIO41-C, FIO42-C, FIO44-C, FIO45-C, FIO46-C, FIO47-C / Rec: FIO01-C, FIO02-C, FIO03-C, FIO05-C, FIO06-C, FIO08-C, FIO09-C, FIO10-C, FIO11-C, FIO13-C, FIO14-C, FIO15-C, FIO17-C, FIO18-C, FIO19-C, FIO20-C, FIO21-C, FIO22-C, FIO23-C, FIO24-C |
| FLP: 浮動小数点 | 6 | 8 | [flp.md](cert-c/flp.md) | R: FLP30-C, FLP32-C, FLP34-C, FLP36-C, FLP37-C, FLP38-C / Rec: FLP00-C, FLP01-C, FLP02-C, FLP03-C, FLP04-C, FLP05-C, FLP06-C, FLP07-C |
| INT: 整数 | 7 | 16 | [int.md](cert-c/int.md) | R: INT30-C, INT31-C, INT32-C, INT33-C, INT34-C, INT35-C, INT36-C / Rec: INT00-C, INT01-C, INT02-C, INT04-C, INT05-C, INT07-C, INT08-C, INT09-C, INT10-C, INT12-C, INT13-C, INT14-C, INT15-C, INT16-C, INT17-C, INT18-C |
| MEM: メモリ管理 | 6 | 11 | [mem.md](cert-c/mem.md) | R: MEM30-C, MEM31-C, MEM33-C, MEM34-C, MEM35-C, MEM36-C / Rec: MEM00-C, MEM01-C, MEM02-C, MEM03-C, MEM04-C, MEM05-C, MEM06-C, MEM07-C, MEM10-C, MEM11-C, MEM12-C |
| MSC: その他 | 8 | 22 | [msc.md](cert-c/msc.md) | R: MSC30-C, MSC32-C, MSC33-C, MSC37-C, MSC38-C, MSC39-C, MSC40-C, MSC41-C / Rec: MSC00-C, MSC01-C, MSC04-C, MSC05-C, MSC06-C, MSC07-C, MSC09-C, MSC10-C, MSC11-C, MSC12-C, MSC13-C, MSC14-C, MSC15-C, MSC17-C, MSC18-C, MSC19-C, MSC20-C, MSC21-C, MSC22-C, MSC23-C, MSC24-C, MSC25-C |
| POS: POSIX | 16 | 4 | [pos.md](cert-c/pos.md) | R: POS30-C, POS34-C, POS35-C, POS36-C, POS37-C, POS38-C, POS39-C, POS44-C, POS47-C, POS48-C, POS49-C, POS50-C, POS51-C, POS52-C, POS53-C, POS54-C / Rec: POS01-C, POS02-C, POS04-C, POS05-C |
| PRE: プリプロセッサ | 3 | 13 | [pre.md](cert-c/pre.md) | R: PRE30-C, PRE31-C, PRE32-C / Rec: PRE00-C, PRE01-C, PRE02-C, PRE04-C, PRE05-C, PRE06-C, PRE07-C, PRE08-C, PRE09-C, PRE10-C, PRE11-C, PRE12-C, PRE13-C |
| SIG: シグナル | 4 | 3 | [sig.md](cert-c/sig.md) | R: SIG30-C, SIG31-C, SIG34-C, SIG35-C / Rec: SIG00-C, SIG01-C, SIG02-C |
| STR: 文字・文字列 | 6 | 10 | [str.md](cert-c/str.md) | R: STR30-C, STR31-C, STR32-C, STR34-C, STR37-C, STR38-C / Rec: STR00-C, STR01-C, STR02-C, STR03-C, STR04-C, STR05-C, STR06-C, STR09-C, STR10-C, STR11-C |
| WIN: Windows | 1 | 5 | [win.md](cert-c/win.md) | R: WIN30-C / Rec: WIN00-C, WIN01-C, WIN02-C, WIN03-C, WIN04-C |

## 固定点・帰属・網羅確認

- 原典: Carnegie Mellon University, **SEI CERT C Coding Standard**。© 2026 Carnegie Mellon University。Carnegie Mellon / CERTは同大学の登録商標。本編集は同大学による承認を表さない。
- [公開主一覧](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/)のRules/Recommendationsを範囲とし、Admin、他言語標準、外部参照先へ拡大しない。
- [公式固定commit](https://github.com/cmu-sei/secure-coding-standards/tree/4a2cb5ce988376f2bae03b1b269e7deb9179bf47): `4a2cb5ce988376f2bae03b1b269e7deb9179bf47`、commit日時2026-10-30T13:40:00Z、取得日2026-11-01（JST）。公開ページの無固定内容よりこの固定本文を再現点とする。
- [公式LICENSE](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/LICENSE): 本文CC BY 4.0、コード例MIT。[取得した権利表示](cert-c/LICENSE.txt)をそのまま保持。コード例は本成果物へ転載しない。[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)に基づく本文の機能別編集/抜粋であり、原文の改訂ではない。
- 元ファイルのSHA-256は各ID冒頭に置く。hashは未加工の取得bytesに対する値で、編集後本文の値ではない。
- 静的照合: categoryの箇条書き主一覧122/183 IDと規範ページ見出し122/183 IDが完全一致し、未取得0、重複0。関連規則テーブルに他カテゴリ/他種別として現れるリンクは新規項目に数えない。全IDは各本文の採否で説明可能。
- この照合は**IDの取得/対応漏れ**の確認。実対象の全条件適用・意味の解釈・安全性は別のレビュー判断であり、実行試験や性能比較は行っていない。

一覧の取得点（各IDの本文hashとは別）:

| 公式元パス | SHA-256 |
|---|---|
| `content/4.sei-cert-c-coding-standard/01.index.md` | `aa21db71670392159d3aa56739b64870cb5f6fafadb1ae79af88b1f918f08157` |
| `content/4.sei-cert-c-coding-standard/03.rules/01.index.md` | `d90918cce2d81cac085e2f5b3c39afc0d99f214c1d914cf529da7c9098f1f25e` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/01.index.md` | `93473ec3dd0b8ca7a593a9a6975d7cdff1e03a2219784536372ccdf0892c4854` |
| `content/4.sei-cert-c-coding-standard/03.rules/02.application-programming-interfaces-api.md` | `9dca2bffb97e7ea3cfb26e729a268b696097af16ac5c551915be42acd808e138` |
| `content/4.sei-cert-c-coding-standard/03.rules/03.arrays-arr/1.index.md` | `537f10fb6158fee925797a64ddee7ce37a548f1ee4748251d9b3fb0aec58659e` |
| `content/4.sei-cert-c-coding-standard/03.rules/04.characters-and-strings-str/1.index.md` | `2b38780b51227dc262a0d48a2cd1b4890a8e37a501644441679cbdc0c8e16ebc` |
| `content/4.sei-cert-c-coding-standard/03.rules/05.concurrency-con/01.index.md` | `5e170319a95702d6eddf1d5195e8a1a76ffeff601064b2bad600397da23d70a5` |
| `content/4.sei-cert-c-coding-standard/03.rules/06.declarations-and-initialization-dcl/01.index.md` | `2207eb69dfb43998eaeafba8f458ff54e122b45bf9db1d02462dfde9bda1aa14` |
| `content/4.sei-cert-c-coding-standard/03.rules/07.environment-env/1.index.md` | `e938606acae8b29253f5fa01ff353549eee184a11b657ffd0a1ec8d1ba3edfe5` |
| `content/4.sei-cert-c-coding-standard/03.rules/08.error-handling-err/1.index.md` | `507e0f2811b6edc1cd362da3f7e852e15b0692fed67e53243fd71fa4205e1e99` |
| `content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/01.index.md` | `59fb807a4867218393c7619abe4e179e461cfaaff384ccd959e2a0bcd6e0e3b9` |
| `content/4.sei-cert-c-coding-standard/03.rules/10.floating-point-flp/1.index.md` | `0c77ed99b7c84ff7e46fdd266590f0d47059a24b918a509f40928e796c23a3fa` |
| `content/4.sei-cert-c-coding-standard/03.rules/11.input-output-fio/01.index.md` | `2d237f5d4f4e433f89408a6c39202c9a01b761142a55bbaa9a0fd02790589003` |
| `content/4.sei-cert-c-coding-standard/03.rules/12.integers-int/1.index.md` | `914aef59e5a7cbfa8b01180916feb684be368663d6eeea4acebb6223aebeb965` |
| `content/4.sei-cert-c-coding-standard/03.rules/13.memory-management-mem/1.index.md` | `436a9df9e5fea7f5645535caa9ce5ab3840acd408bf60b9f9c62847e728dd211` |
| `content/4.sei-cert-c-coding-standard/03.rules/14.microsoft-windows-win/1.index.md` | `d64b76300dde6934ce37c595a2929a8e12cae2dd9f15d82c45ef9ed3b40b014e` |
| `content/4.sei-cert-c-coding-standard/03.rules/15.miscellaneous-msc/1.index.md` | `4ee98d18cebb7febcd6bd8a4d9ef323a1681225d3b1a49cb48bdcf0da743a31f` |
| `content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/01.index.md` | `341d7536a812c83ef0fd93d42f9082da4c25b7e35b73709941b1c6687518081a` |
| `content/4.sei-cert-c-coding-standard/03.rules/17.preprocessor-pre/1.index.md` | `df90a3764cc11f85155cee7fba3b15a3cad9fa1d05e82e3e5189cc5336472c85` |
| `content/4.sei-cert-c-coding-standard/03.rules/18.signals-sig/1.index.md` | `ebcd2a994deed0dc025651564392b782b185a400f58d70d6c5bb0eb1af71086e` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/02.application-programming-interfaces-api/01.index.md` | `806e30c469291fceb0f9d05546620252e5b705b5f685dc766eb3f4695b1b09c4` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/03.arrays-arr/1.index.md` | `b15036fdafff76a1f81bf511dc4227577b5ecc0420685ead29f0b3a8cfac98d7` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/04.characters-and-strings-str/01.index.md` | `6b303edc0c266e6847dc4034bf0bdb3283f3f4045bddb61c2456dd44946375a9` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/05.concurrency-con/01.index.md` | `819b365ce748068ad7413c7683ccec9ecc476cf98b559abe1ef95f98b8388cbd` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/01.index.md` | `a6bd825cff47fe56c9b2e17a4ee294ca49751e79c40870ebd44a5bf4e62f8550` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/07.environment-env/1.index.md` | `86a0eb60828d9362ea75c6d9d6d4422c26e8a04708c9810d3404849a426da708` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/08.error-handling-err/1.index.md` | `a2b3856f91fc8b0ff509684d5a6f7b992b8f79384dbe0740d3d8c471130724fa` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/01.index.md` | `c4b9b8f59603c9539d6a7a488b98f2d0ed563cda1f9707058ab1ebfaee5249b6` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/10.floating-point-flp/1.index.md` | `26bf4f978e9a84833bd322bcc17710887a80c41642ad7accaaed2458a4afbf66` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/01.index.md` | `94ee9c4ba2cd72a5e5c36d1f57244ca66fc1f0defe26feb92aa5f17f7698600c` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/01.index.md` | `8abe10f265db4ccf19a8f22643ae73598aac3b18b97e51495fe78174b7300853` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/13.memory-management-mem/01.index.md` | `c83e6ad836a56865b12f052675ef943f54c73e615ff154fa2851d6c0a7121e1f` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/14.microsoft-windows-win/1.index.md` | `c69204b0ab130617c85293632c9fcd762147ba0bd2ac74acaf80bcbb30b7a90f` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/01.index.md` | `f6851e969fe3a42d0d988103cd2fbbc2ffe7bff43a93a5b2b26ec861a14ed718` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/16.posix-pos/1.index.md` | `090f330aa97fa5911fb9fe5bfd58a84d8699df2f1709ef3af39525f5997bdb05` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/17.preprocessor-pre/01.index.md` | `bddde27595100e79234c57f6fed50dbf3222070de3759f940dfb5be94adb8899` |
| `content/4.sei-cert-c-coding-standard/08.recommendations/18.signals-sig/1.index.md` | `34b078edc6774fda81c17654b8b31804787f832d5bd2bf5e1a435746ff434a81` |
| `LICENSE` | `c8f6c90cf3545cc62fc2e0f08ee0aa345b82c00c60528b61116861fc0cc31aac` |

案内の変更: 2026-11-03、全件評価の手順から必要時参照へ変更。原典の具体条件・帰属・許諾は保持。
