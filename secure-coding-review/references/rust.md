# ANSSI Rust: Rule / Recommendation の評価条件

## 適用と判断

Rustコード、依存、ビルド/検証環境、unsafe、FFIと所有権/共有の境界に適用する。原典は高いセキュリティ要求のアプリ開発を主対象とする**unstable**ガイド。Rule/RecommendationとMUST/SHOULDの強さを保持する。高保証/安全クリティカル等の限定条件をすべてのアプリへ無条件に広げず、要求水準が未指定なら保護対象と具体的リスクへ照合する。Rust固有APIや型規約を他言語へ強制せず、型名が違っても所有権・境界・共有/変更・並行利用に同じ危険が成立すれば別途評価する。

対象の具体的な疑問に関係する規範を選び、型/API・環境・要求水準と前提/例外を確かめる。全60 IDの判定は要求しない。safe Rustだけで依存のunsafe/FFI、panic、資源の保持、論理欠陥を除外しない。

unsafe/FFIの所有権・寿命・整列・表現・aliasing・異常終了等が結論を左右するとき、該当する本文と必要な固定原典の例へ戻る。MUST/SHOULDを限定する警告も保つ。既存根拠は有効な範囲で使い、推奨を導入/実行の承認にしない。

## 保持範囲と採否

公式`src/en/SUMMARY.md`の**コメントアウトされていない公開章**を新規取得し、公開Rule/Recommendationブロック60 ID（Rule 46、Recommendation 14）とコメント内の未完成Recommendation 1 IDを区別して照合した。**公開60 IDを条件付き採用**し、例外は各本文に保持した。タイトルだけのリンク一覧ではなく、規範全文と適用を限定する本文・warning/noteを収録する。コード例・図、外部参照先本文は転載しない。ただしMEM-FORGET-LINTの規範に不可欠な1行のlint指定はinlineで保持する。削除した例へ依存する解釈は固定原典確認を要するため、抜粋だけで全意味の無損失や逐条適合を認定しない。

特にDENV-CARGO-OPTSはdev/testの`debug-assertions`・`overflow-checks`を上書きしない条件であり、全profileに拡大しない。DENV-CARGO-ENVVARSは`RUSTC`・`RUSTC_WRAPPER`・`RUSTFLAGS`、DENV-TIERSはsafety-critical systemsへのtier 1とcertified toolchainsの条件を保持する。FFIのopaque、Option-wrapped reference、foreign safe reference、platformごとの自動生成などの例外と型ごとの制約は`unsafe-ffi.md`の本文で確認する。

LIBS-UNSAFEはコメントアウト内のTODOで、**未確定**として保持し採用数には含めない。

## 機能別本文と全元ID

各IDの原典種別を明示する。ファイル内の`#### ID — Rule/Recommendation`が規範本文の入口で、同じファイルに適用条件/例外を保持する。LIBS-UNSAFEだけは未確定、それ以外の60 IDは条件付き採用で、具体的な判断に使うときは対象への適用条件を確認する。

| 機能別本文 | 元ID（原典種別） |
|---|---|
| [devenv.md](rust/devenv.md) | `DENV-STABLE` (Rule), `DENV-TIERS` (Rule), `DENV-CARGO-LOCK` (Rule), `DENV-CARGO-OPTS` (Rule), `DENV-CARGO-ENVVARS` (Rule), `DENV-FORMAT` (Recommendation), `DENV-LINTER` (Rule), `DENV-AUTOFIX` (Rule) |
| [libraries.md](rust/libraries.md) | `LIBS-VETTING-DIRECT` (Rule), `LIBS-VETTING-TRANSITIVE` (Recommendation), `LIBS-OUTDATED` (Rule), `LIBS-AUDIT` (Rule), `LIBS-UNSAFE` (Recommendation) |
| [naming.md](rust/naming.md) | `LANG-NAMING` (Rule) |
| [integer.md](rust/integer.md) | `LANG-ARITH` (Rule) |
| [errors.md](rust/errors.md) | `LANG-ERRWRAP` (Recommendation), `LANG-LIMIT-PANIC` (Rule), `LANG-LIMIT-PANIC-SRC` (Rule), `LANG-ARRINDEXING` (Rule) |
| [unsafe/generalities.md](rust/unsafe-generalities.md) | `UNSAFE-NOUB` (Rule), `LANG-UNSAFE` (Rule), `LANG-UNSAFE-ENCP` (Rule) |
| [unsafe/memory.md](rust/unsafe-memory.md) | `MEM-NO-LEAK` (Rule), `MEM-FORGET` (Rule), `MEM-FORGET-LINT` (Recommendation), `MEM-LEAK` (Rule), `MEM-MANUALLYDROP` (Rule), `MEM-NORAWPOINTER` (Rule), `MEM-INTOFROMRAWALWAYS` (Rule), `MEM-INTOFROMRAWONLY` (Rule), `MEM-UNINIT` (Rule) |
| [unsafe/ffi.md](rust/unsafe-ffi.md) | `FFI-SAFEWRAPPING` (Recommendation), `FFI-CTYPE` (Rule), `FFI-TCONS` (Rule), `FFI-AUTOMATE` (Recommendation), `FFI-PFTYPE` (Rule), `FFI-CKNONROBUST` (Rule), `FFI-CKINRUST` (Recommendation), `FFI-CK-PTR-VALID` (Rule), `FFI-INPUT-PTR` (Recommendation), `FFI-CK-INPUT-REF-VALID` (Rule), `FFI-MARKEDFUNPTR` (Rule), `FFI-CKFUNPTR` (Rule), `FFI-NOENUM` (Rule), `FFI-R-OPAQUE` (Recommendation), `FFI-C-OPAQUE` (Recommendation), `FFI-CK-REF-MODEL` (Rule), `FFI-MEM-NODROP` (Rule), `FFI-MEM-OWNER` (Rule), `FFI-MEM-WRAPPING` (Recommendation), `FFI-NOPANIC` (Recommendation), `FFI-CAPI` (Rule) |
| [standard.md](rust/standard.md) | `LANG-SYNC-TRAITS` (Rule), `LANG-CMP-INV` (Rule), `LANG-CMP-DEFAULTS` (Recommendation), `LANG-CMP-DERIVE` (Recommendation), `LANG-DROP` (Rule), `LANG-DROP-NO-PANIC` (Rule), `LANG-DROP-NO-CYCLE` (Rule), `LANG-DROP-SEC` (Rule), `MEM-MUT-REC-RC` (Rule) |

前提の本文: [Introduction](rust/introduction.md)、[Language guarantees](rust/guarantees.md)、[Unsafe Rust](rust/unsafe.md)。IDがないこれらの章も保証の限界と対象範囲の解釈に使う。

## 未完成・範囲外

- 原典はasync Rustを扱わない。asyncの安全性をこのガイドから充足認定しない。
- SUMMARYでコメントアウトされているmacros/typesystem/testfuzzは公開章集合に含まれず、内容もTODO段階でRule/Recommendationは0件。これらを完成した基準に創作しない。必要なリスクを他の採用条件で確認できない場合は未確認として残す。
- 公開章にもerrorsのpanic回避策、memoryの所有/解放の説明、FFIの例などにTODOがある。libraries内のLIBS-UNSAFEはコメントアウトされたTODOのため未確定。記載済みの60規範を保持したことは、それらの未完成部分の解消を意味しない。補足が不足して判断できない部分は原典依存の未確認とする。
- バージョン依存のAPI/ツール機能・安定化状況は固定原典時点の記述であり、現行対象に合わせる根拠がなければ未確認にする。原典の一般説明（例: target tierのABI記述）をRustのあらゆるABIの保証へ広げない。

## 固定点・権利・網羅確認

- 原典: **ANSSI-FR, Secure Rust Guidelines (unstable)**。[公開本文](https://anssi-fr.github.io/rust-guide/print.html)。本編集はANSSIの承認を表さない。
- [公式固定commit](https://github.com/ANSSI-FR/rust-guide/tree/3f9e2e28095d0eeb35ed61e212a82be12fde4884): `3f9e2e28095d0eeb35ed61e212a82be12fde4884`。commit日時2026-05-18T15:58:16Z、取得日2026-11-01（JST）。帰属の最終更新点はこの公式commitに固定する。
- [公式LICENSE](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/LICENSE.md): **OPEN LICENCE 2.0 / LICENCE OUVERTE 2.0**。[取得したライセンス全文](rust/LICENSE.md)を保持。出所/最終更新点と固定原典へのリンクを各本文へ表示する。
- 編集: YAMLを参照リンクへ整形し、Rule/RecommendationのHTMLをID付き見出しに変更。コード/図/コメントを除去し（規範の1行lint指定は保持）、本文・警告・例外・規範の強さは英語のまま保持した。外部参照先そのものの再配布はしない。
- 静的照合: 公開SUMMARYの12章/文脈ファイルと本文から公開60 IDとコメント内の未確定1 IDを取得し、元種別と規範本文/未完成状態を対応。未取得0、ID重複0。非公開のTODO章にもRule/Recommendationがないことを確認。旧成果物の規則数・要約・権利調査は使用していない。
- 件数/文字一致は規範の保持確認であり、実対象の安全性・各条件の充足の証明ではない。実対象のレビュー/実行試験/性能比較は未実施。

元ファイルSHA-256（未加工bytes）と保持先:

| 公式元パス | 保持先 | SHA-256 | 公開Rule/Recommendation数 |
|---|---|---|---:|
| `src/en/introduction.md` | [introduction.md](rust/introduction.md) | `9271537038e079b04155cfc20c9c87e68e533a526c81815886dd2c9be988b989` | 0 |
| `src/en/devenv.md` | [devenv.md](rust/devenv.md) | `ea3d76c12bf1add44a56f50b873a5ab03b28d9570031f28c95d03859b0c162dd` | 8 |
| `src/en/libraries.md` | [libraries.md](rust/libraries.md) | `5606cdf4df83f874267c0512540b4606554cf452b12ef1f00ad6d2b1cba805d1` | 4 |
| `src/en/naming.md` | [naming.md](rust/naming.md) | `58ad9c75257ffb5601086a669616c084a73c71bad2702b2c5ce2234657297d60` | 1 |
| `src/en/integer.md` | [integer.md](rust/integer.md) | `e2e96c52b5c3af89fc060694c7191249f85878bce87a86fa6eaf1fae058bcc23` | 1 |
| `src/en/errors.md` | [errors.md](rust/errors.md) | `dbaad11513deb2c10294f18c9d94c82102f9ebaf006f96ea87b2b5447014e91c` | 4 |
| `src/en/guarantees.md` | [guarantees.md](rust/guarantees.md) | `04fd02f6139f998d4b73fc6a6f468ec3d47941fc6733f03812df7ba83ef5b15e` | 0 |
| `src/en/unsafe.md` | [unsafe.md](rust/unsafe.md) | `45bfaaca652ab59720e6fac0f6dbc508950517ed70c02997552a9075936ac2a2` | 0 |
| `src/en/unsafe/generalities.md` | [unsafe-generalities.md](rust/unsafe-generalities.md) | `44aefd158dabff1e859913b143f99612013cd54a5dc112859b7240e8fee02ae3` | 3 |
| `src/en/unsafe/memory.md` | [unsafe-memory.md](rust/unsafe-memory.md) | `5ea8d8d1a8e258565a2c7723e86fd52bff89930f9e06aef5b1b0869592f9789e` | 9 |
| `src/en/unsafe/ffi.md` | [unsafe-ffi.md](rust/unsafe-ffi.md) | `f457e1e418996b3c6702918c25a84e31129dbc8dee1fc118572903db46e38b1a` | 21 |
| `src/en/standard.md` | [standard.md](rust/standard.md) | `0472a3ebd7d93326e152891666fba7df8f26f90d0452812192b3610285657b06` | 9 |
| `src/en/SUMMARY.md` | 範囲/未完成の照合のみ | `a32a9a4e0f6f84272ab5b35161a98e09d288a6cef08504a7eef0d25bbb518e35` | 0 |
| `LICENSE.md` | [LICENSE.md](rust/LICENSE.md) | `8cfe0ca1246eb6043211add2a0ce7ac344c3fd95a958e2c894493b5fa69efb68` | 0 |
| `src/en/macros.md` | 範囲/未完成の照合のみ | `f9cfa3747ef75829e5c9f348c4526bdaa748a6ed6e89e9142a2a41366997e1ec` | 0 |
| `src/en/typesystem.md` | 範囲/未完成の照合のみ | `8da108576ff5148648073d4dfb05b3050f46a9eb8065818efd532c3fa3d749e1` | 0 |
| `src/en/testfuzz.md` | 範囲/未完成の照合のみ | `e15f62660cb14a50abd5874fa999b2cae3f36e89341462b3b0a355e776e18471` | 0 |

案内の変更: 2026-11-03、全件評価の手順から必要時参照へ変更。原典の具体条件・帰属・許諾は保持。
