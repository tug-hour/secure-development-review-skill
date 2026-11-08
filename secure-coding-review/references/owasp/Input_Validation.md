# Input Validation Cheat Sheet — 評価用の対策抜粋

出典: OWASP Cheat Sheet Series contributors, [Input Validation Cheat Sheet](https://raw.githubusercontent.com/OWASP/CheatSheetSeries/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Input_Validation_Cheat_Sheet.md)。固定commit `87a1faf18a29a5c33d040b000eb3c60330c30b2c`、2026-11-01取得、リダイレクトなし。原典 8,239 bytes、SHA-256 `163390e3dcf5cc965aeac5e633692682403c08067a7f2ae54bdb49372e886810`。

適用: 外部・内部・保存済みデータが信頼境界を越える場合。

確認する根拠: 使用される表現に対する構文/意味/項目・ネスト/数/サイズ制限と、解析前制限、部分検証後の継続禁止、全値regex、Unicode、出力先別防御。入力検証をSQL/XSS/認可対策の代替にしない。

このファイルは[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)の原典に基づく編集物で、同許諾で提供する。変更: 対策節の選択、コード/図の除去、行番号/日本語説明/採否の追加、相対リンクの固定commit化、checkboxの通常箇条書き化。原典の著作者・権利表示を保持する。公式訳・公式認定ではない。元の意味を変える訳文で代替せず、具体条件は下記英語原文を正本にする。コード/図/文書全文を無差別同梱したものではない。

各採用節の個別の文・箇条書き・表の条件ごとに、対象へ適用するかと、満足/未達/非適用理由/未確認の根拠を判断する。見出しの確認やリンクの存在を、その配下の実装確認に代えない。must/never、should/recommend、may/consider、if/where possibleと数値・例外・代替の強さを原文どおり区別する。原文の命令形は対象の安全条件又は手法の助言であり、今回の実行/通信/導入/変更権限を増やさない。例示の製品・実装・リンク先の全資料は一律義務ではない。原典への参照でしか分からない対象方式の条件、削除した例/図が結論に必要な場合は、必要な範囲を原典で確認し、それまで未確認を残す。

## 原典の節と採否

行番号は取得したMarkdownの1始まり。各行範囲はその見出し直属の本文を指し、子節は別行に対応する。採用はガイドを評価入力にする意味であり、実対象の達成を意味しない。

| 原典位置・節 | 採否と理由/適用 |
|---|---|
| L3-9 `Introduction` | 条件付き採用: 上記の適用と節内の前提に従う |
| L10-23 `Input Validation Strategies` | 条件付き採用: 上記の適用と節内の前提に従う |
| L24-27 `Input Validation Strategies / Allowlist vs Denylist` | 条件付き採用: 上記の適用と節内の前提に従う |
| L28-29 `Implementing Input Validation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L30-37 `Implementing Input Validation / Parse Safely, Then Validate` | 条件付き採用: 上記の適用と節内の前提に従う |
| L38-43 `Implementing Input Validation / Validate Structured Data` | 条件付き採用: 上記の適用と節内の前提に従う |
| L44-47 `Implementing Input Validation / Validating Free-form Unicode Text` | 条件付き採用: 上記の適用と節内の前提に従う |
| L48-53 `Implementing Input Validation / Regular Expressions (Regex)` | 条件付き採用: 上記の適用と節内の前提に従う |
| L54-57 `Implementing Input Validation / Validating Rich User Content` | 条件付き採用: 上記の適用と節内の前提に従う |
| L58-61 `Implementing Input Validation / File Upload Validation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L62-65 `Implementing Input Validation / Email Address Validation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L66-75 `Common Pitfalls` | 条件付き採用: 上記の適用と節内の前提に従う |
| L76-80 `References` | 対象外: 参考文献一覧。リンク先全量へ義務を拡張しない |

## 採用する具体条件（原文）

### Introduction

原典 L3-9。

Input validation checks whether data meets an application's requirements before the application uses it. It reduces the risk of malformed data, invalid business operations, and excessive resource consumption. [OWASP's Proactive Control C3](https://top10proactive.owasp.org/archive/2024/the-top-10/c3-validate-input-and-handle-exceptions/) distinguishes validation from the additional defenses needed when using that data:

- Use [parameterized queries](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.md) for SQL and [context-aware output encoding](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.md#output-encoding) to prevent cross-site scripting (XSS).
- Check [authorization](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Authorization_Cheat_Sheet.md) separately: a valid account identifier does not mean the caller may access that account.

### Input Validation Strategies

原典 L10-23。

Validate both **syntax** (the expected type and format) and **semantics** (whether the value makes sense for the operation). For example, a booking needs valid dates and an end date after its start date. [CWE-20](https://cwe.mitre.org/data/definitions/20.html) describes these checks, including consistency between related fields.

Define rules for each field:

| Input | Rules to enforce |
| --- | --- |
| Fixed choices | Exact membership in the allowed set, including values submitted from drop-down menus |
| Numbers and dates | Expected type, accepted format, and minimum and maximum values |
| Strings | Length limits and the characters or structure required by the field |
| Objects | Allowed fields, required fields, and rules for missing or null values |
| Arrays | Minimum and maximum item counts and validation of every item, including nested objects |

#### Allowlist vs Denylist

原典 L24-27。

Define what the application accepts and reject values outside those rules. Do not try to recognize every malicious string. Blocking apostrophes, for example, rejects legitimate names without making a database query safe. For free-form comments, an allowlist can permit broad Unicode text while limiting its length; it need not restrict users to letters and digits.

### Implementing Input Validation

原典 L28-29。



#### Parse Safely, Then Validate

原典 L30-37。

Apply request size limits before buffering or parsing input, and configure parser limits such as maximum nesting depth. JSON explicitly supports [implementation limits on size, depth, and numbers](https://www.rfc-editor.org/info/rfc8259/#section-9). A schema check after parsing cannot protect a parser that has already exhausted resources.

Use a maintained parser for the expected format, handle parsing failures, and validate the resulting values before business processing or storage. Configure parsers for untrusted input; see [XML External Entity Prevention](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.md#general-guidance) and [Deserialization](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Deserialization_Cheat_Sheet.md). Converting text to an integer only establishes a type: it does not establish an acceptable quantity.

Decode according to the protocol before checking field rules. Validate the representation that will actually be used, and avoid decoding it again downstream; [CWE-20](https://cwe.mitre.org/data/definitions/20.html) explains how inconsistent decoding can invalidate earlier checks.

#### Validate Structured Data

原典 L38-43。

Use your framework's validators or a schema validator to enforce field rules. For JSON, explicitly configure [required and additional properties](https://json-schema.org/understanding-json-schema/reference/object); listing a property alone neither requires it nor rejects unknown fields. Apply schemas to nested objects and use [item schemas and array length limits](https://json-schema.org/understanding-json-schema/reference/array). Keep business checks, such as date ordering, alongside these structural checks.

Reject invalid requests with a clear error; do not continue with partially validated data. Bind only intended input fields to application objects; see [Mass Assignment](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Mass_Assignment_Cheat_Sheet.md).

#### Validating Free-form Unicode Text

原典 L44-47。

Agree on a character encoding across components and reject malformed input. Preserve legitimate punctuation and scripts in names and comments. Where a field requires Unicode normalization for consistent comparison, define and apply the same policy before validation, storage, and comparison. [Unicode Standard Annex #15](https://www.unicode.org/reports/tr15/) defines the normalization forms and warns that compatibility normalization can erase meaningful distinctions. Normalization is not sanitization and does not replace output encoding.

#### Regular Expressions (Regex)

原典 L48-53。

Use regular expressions for simple, structured fields. Require a match of the entire value, using an API such as [Python's `fullmatch`](https://docs.python.org/3/library/re.html#re.fullmatch) where available. Check the engine's character classes and newline behavior rather than assuming patterns behave identically across languages.

Bound input length before matching, avoid patterns with excessive backtracking, and use a non-backtracking engine or a match timeout where supported. Test valid, invalid, and near-matching values; [Microsoft's regex guidance](https://learn.microsoft.com/en-us/dotnet/standard/base-types/best-practices-regex) explains why near-matches can cause denial of service. Treat a timeout as validation failure.

#### Validating Rich User Content

原典 L54-57。

When accepting user-authored HTML, use a maintained [HTML sanitization library](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.md#html-sanitization). Input validation and regular expressions cannot replace that control.

#### File Upload Validation

原典 L58-61。

Treat the submitted filename and content type as untrusted metadata. Validate filenames after protocol decoding; an allowed extension alone does not establish safe content. Follow the [File Upload Cheat Sheet](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/File_Upload_Cheat_Sheet.md) for content checks, size limits, storage, and safe serving.

#### Email Address Validation

原典 L62-65。

Use a maintained email validation library compatible with the addresses your mail system supports. Format validation does not prove mailbox access. Follow [Email Validation and Verification](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Email_Validation_and_Verification_Cheat_Sheet.md) for comparison policies, ownership verification, and email change workflows.

### Common Pitfalls

原典 L66-75。

- **Client-only checks:** Validate on the server even when the browser checks the same fields. [Client-side validation is bypassable](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation); use it for immediate feedback.
- **Trusting internal sources:** Validate data from internal APIs, partner feeds, queues, and stored records when it crosses a trust boundary. An internal transport does not establish that a value meets the receiving component's rules ([CWE-20](https://cwe.mitre.org/data/definitions/20.html)).
- **Denylisting or “cleaning” input:** Apply field-specific acceptance rules. Removing suspicious characters can change meaning and still does not provide query parameterization or output encoding.
- **Stopping at parsing or outer fields:** Test rejected ranges, missing fields, invalid nested items, and oversized arrays, not just malformed syntax.
- **Assuming regex or Unicode normalization makes data safe:** Check whole-value matching and resource limits; keep normalization consistent with the field's meaning.
- **Trusting upload metadata:** Use the file upload controls above; checking a filename is not checking the file's contents.
- **Logging rejected input verbatim:** Record the failure and relevant metadata without secrets or full request bodies. Escape any retained untrusted values for the log format to prevent log injection; see [Logging: Event collection](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Logging_Cheat_Sheet.md#event-collection) and [Data to exclude](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Logging_Cheat_Sheet.md#data-to-exclude).
