# SSDF 1.1の評価条件

NIST SP 800-218正式版(2022年2月)、Table 1のTasks全42件を採用候補として保持する。対象はコード、設計、設定、依存、開発/検査/CI/配布、更新と欠陥是正に関係する範囲。全組織の成熟度や規格認証を目的に追加しない。原典は推奨frameworkであり、この文書だけで法的義務や認証要件を作らない。対象の契約/方針等が要求化している場合は、その根拠を区別する。

## 読み先と適用

対象の開発・保護・検査・配布・更新是正について、具体的な疑問に関係するtaskを選ぶ。全42 taskの判定や全4群の非適用説明は要求しない。組織/外部側の証拠とコードの根拠を分け、必要な確認だけを対象のリスクへ結び付ける。

| 読み先 | 対象の証拠と適用条件 | 元Tasks数 |
|---|---|---|
| [PO: Prepare the Organization](ssdf/po.md) | 対象の安全要求、責任境界、toolchain、検査基準、環境保護。PO.2.1–PO.2.3の組織役割/訓練/上層部commitmentは対象の継続開発能力に関係する部分を条件付き採用し、組織側の証拠を区別する。PO.1.3は商用component提供者、PO.5.2は開発endpointがある場合 | 13 |
| [PS: Protect the Software](ssdf/ps.md) | 全形態のcode保存、release整合性、保存、component由来。releaseに関するtaskは配布/引渡しがある場合。未releaseの作業で既存releaseの証拠を要求せず、将来の準備と区別する | 4 |
| [PW: Produce Well-Secured Software](ssdf/pw.md) | リスク/設計、内外のcomponent、言語固有coding、build保護、code確認、実行検査、default。内製/外部componentや必要な検査の種類に応じて適用 | 16 |
| [RV: Respond to Vulnerabilities](ssdf/rv.md) | 対象と依存の継続確認、受付/開示/是正、risk response、原因/類似箇所/再発防止。個別原因や是正のtaskは該当する脆弱性/履歴がある場合 | 9 |

全42件の英語Tasksを省略せず保持し、安全条件を削ったIDや原典未取得のIDはない。各taskの条件・where appropriate・if so・and/or・must/shouldの差を保つ。判断に使うtaskの前提を対象の構成と照合する。組織証拠が無いことを、対象外/実装欠陥/充足のいずれにも自動変換しない。例えば訓練記録や上層部のcommitmentが対象のcodeからは分からない場合は、必要な証拠と影響範囲を示して組織側未確認とする。

既存の設計/検査結果/CI設定/依存情報/是正履歴等は対象版と前提を照合して再利用し、不足する根拠だけを追加確認する。方針の宣言、設定ファイル、検査ツールの存在だけで実施/安全な結果/是正を満たしたとしない。全42件を保持したことは対象の評価済みを意味しない。

## Tasksと実装例の区別

具体条件の本体は分割先の英文Tasksであり、日本語は適用と根拠の案内。Table 1のNotional Implementation Examples列は実現手段の例なので同梱・義務化しない。例示にだけある頻度、製品、文書/署名/管理方式等をtask自体の必須条件へ昇格させない。Tasks自体の括弧内の例は原文のまま保持する。References列の他規格も再帰的な必須対象へ追加しない。

特にPW.2.1は「設計に不参加の有資格者」及び/又は「toolchainの自動process」という代替を保つ。PW.7.1の人review/自動analysis、PW.8.1の実行検査の要否、PW.1.3とRV.3.4の適切な場合という条件を省かない。実際に必要な調査や検査を省く理由にはせず、その決定の根拠を確認する。採用toolchainの導入や組織設定変更をこの参照だけで許可しない。

## 出典、固定点、権利

Souppaya M, Scarfone K, Dodson D (2022), *Secure Software Development Framework (SSDF) Version 1.1: Recommendations for Mitigating the Risk of Software Vulnerabilities*, NIST Special Publication 800-218, National Institute of Standards and Technology, [DOI: 10.6028/NIST.SP.800-218](https://doi.org/10.6028/NIST.SP.800-218), accessed 2026-11-01. Republished courtesy of the National Institute of Standards and Technology.

[正式出版情報](https://csrc.nist.gov/pubs/sp/800/218/final)から案内される[公式Table 1補助xlsx](https://csrc.nist.gov/files/pubs/sp/800/218/final/docs/nist.sp.800-218.ssdf-table.xlsx)を2026-11-01に新規取得。50,039 bytes、SHA-256 `f5729c4c6c792cbf6cfbea74eee7cc84c579b2109006fe3cbfa8934eb460bd55`。版1.1とこの生バイトハッシュを固定点とし、可変URLからの再取得を同一内容と仮定しない。正式版以外のpotential updates/draftは混ぜない。

[NISTのTechnical Series Publications権利方針](https://www.nist.gov/open/copyright-fair-use-and-licensing-statements-srd-data-software-and-technical-series-publications)は米国政府職員の著作の米国内保護の扱いと、NISTが国外で権利を主張できる範囲の全世界再掲載/派生物許諾、帰属表示を案内している。[一般方針](https://www.nist.gov/copyrights-disclaimers)も、別途著作権表示があるものを除く公開情報の配布/複写を認める。第三者の引用規格/図/コード/実装例は転載せず、NIST自身のTasks列だけを抜粋する。他系統へこの許諾を拡張しない。

変更: 2026-11-01に補助表のTasks全42件を4群へ配置し、対象への適用/根拠案内を日本語で追加。Tasks原文とIDは変更せず、公式訳/認証/NISTの承認ではない。出典/変更表示を保持する。

静的確認: 補助xlsxのTable 1相当sheetのTasks列にある42件を一意な元IDごとに照合し、分割先で各1回・本文完全一致を確認した。正式版の欠番（PW.3、PW.4.3等）を埋めず、Formerly表示を保持した。これは構造/転記の確認であり、実対象での安全性/レビュー性能の検証ではない。状態DB、catalog、実行用readerは作成しない。

案内の変更: 2026-11-03、全件評価の手順から必要時参照へ変更。原典の具体条件・帰属・許諾は保持。
