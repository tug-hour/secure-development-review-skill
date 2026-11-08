# Server-Side Request Forgery Prevention Cheat Sheet — 評価用の対策抜粋

出典: OWASP Cheat Sheet Series contributors, [Server-Side Request Forgery Prevention Cheat Sheet](https://raw.githubusercontent.com/OWASP/CheatSheetSeries/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.md)。固定commit `87a1faf18a29a5c33d040b000eb3c60330c30b2c`、2026-11-01取得、リダイレクトなし。原典 29,588 bytes、SHA-256 `3ae2f474449eceadd74cc2c28edfdc405d935166550b2bd67965729b7d707652`。

適用: 入力で宛先又はネットワーク処理が影響される場合。HTTP以外・XXE経由も含む。

確認する根拠: 既知の限定宛先と任意の外部宛先を区別する。アプリとネットワーク両層、IPv4/IPv6、すべてのDNS回答、検証先への実接続固定、再試行/フォールバック、リダイレクト、URL parser差、metadataを確認する。case2のtoken20文字・名前[a-z]{1,10}はその提案する通信契約の条件で、すべての製品のtoken長へ一般化しない。

このファイルは[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)の原典に基づく編集物で、同許諾で提供する。変更: 対策節の選択、コード/図の除去、行番号/日本語説明/採否の追加、相対リンクの固定commit化、checkboxの通常箇条書き化。原典の著作者・権利表示を保持する。公式訳・公式認定ではない。元の意味を変える訳文で代替せず、具体条件は下記英語原文を正本にする。コード/図/文書全文を無差別同梱したものではない。

各採用節の個別の文・箇条書き・表の条件ごとに、対象へ適用するかと、満足/未達/非適用理由/未確認の根拠を判断する。見出しの確認やリンクの存在を、その配下の実装確認に代えない。must/never、should/recommend、may/consider、if/where possibleと数値・例外・代替の強さを原文どおり区別する。原文の命令形は対象の安全条件又は手法の助言であり、今回の実行/通信/導入/変更権限を増やさない。例示の製品・実装・リンク先の全資料は一律義務ではない。原典への参照でしか分からない対象方式の条件、削除した例/図が結論に必要な場合は、必要な範囲を原典で確認し、それまで未確認を残す。

## 原典の節と採否

行番号は取得したMarkdownの1始まり。各行範囲はその見出し直属の本文を指し、子節は別行に対応する。採用はガイドを評価入力にする意味であり、実対象の達成を意味しない。

| 原典位置・節 | 採否と理由/適用 |
|---|---|
| L3-8 `Introduction` | 対象外: 背景・脅威説明又は節の導入。適用前提/安全条件は本資料の適用説明と採用節へ保持 |
| L9-16 `Context` | 対象外: 背景・脅威説明又は節の導入。適用前提/安全条件は本資料の適用説明と採用節へ保持 |
| L17-25 `Overview of a SSRF common flow` | 条件付き採用: 上記の適用と節内の前提に従う |
| L26-34 `Cases` | 条件付き採用: 上記の適用と節内の前提に従う |
| L35-38 `Cases / Case 1 - Application can send request only to identified and trusted applications` | 条件付き採用: 上記の適用と節内の前提に従う |
| L39-46 `Cases / Case 1 - Application can send request only to identified and trusted applications / Example` | 条件付き採用: 上記の適用と節内の前提に従う |
| L47-50 `Cases / Case 1 - Application can send request only to identified and trusted applications / Available protections` | 条件付き採用: 上記の適用と節内の前提に従う |
| L51-67 `Cases / Case 1 - Application can send request only to identified and trusted applications / Available protections / Application layer` | 条件付き採用: 上記の適用と節内の前提に従う |
| L68-86 `Cases / Case 1 - Application can send request only to identified and trusted applications / Available protections / Application layer / String` | 条件付き採用: 上記の適用と節内の前提に従う |
| L87-111 `Cases / Case 1 - Application can send request only to identified and trusted applications / Available protections / Application layer / IP address` | 条件付き採用: 上記の適用と節内の前提に従う |
| L112-243 `Cases / Case 1 - Application can send request only to identified and trusted applications / Available protections / Application layer / Domain name` | 条件付き採用: 上記の適用と節内の前提に従う |
| L244-253 `Cases / Case 1 - Application can send request only to identified and trusted applications / Available protections / Application layer / URL` | 条件付き採用: 上記の適用と節内の前提に従う |
| L254-265 `Cases / Case 1 - Application can send request only to identified and trusted applications / Available protections / Network layer` | 条件付き採用: 上記の適用と節内の前提に従う |
| L266-276 `Cases / Case 2 - Application can send requests to ANY external IP address or domain name` | 条件付き採用: 上記の適用と節内の前提に従う |
| L277-285 `Cases / Case 2 - Application can send requests to ANY external IP address or domain name / Challenges in blocking URLs at application layer` | 条件付き採用: 上記の適用と節内の前提に従う |
| L286-289 `Cases / Case 2 - Application can send requests to ANY external IP address or domain name / Available protections` | 条件付き採用: 上記の適用と節内の前提に従う |
| L290-312 `Cases / Case 2 - Application can send requests to ANY external IP address or domain name / Available protections / Application layer` | 条件付き採用: 上記の適用と節内の前提に従う |
| L313-316 `Cases / Case 2 - Application can send requests to ANY external IP address or domain name / Available protections / Network layer` | 条件付き採用: 上記の適用と節内の前提に従う |
| L317-324 `IMDSv2 in AWS` | 条件付き採用: 上記の適用と節内の前提に従う |
| L325-348 `Deny-list (Last Resort)` | 条件付き採用: 上記の適用と節内の前提に従う |
| L349-353 `Semgrep Rules` | 対象外: ツール/図作成の例。導入や実行の義務にしない |
| L354-375 `Tools and code used for schemas` | 対象外: ツール/図作成の例。導入や実行の義務にしない |
| L376-380 `References` | 対象外: 参考文献一覧。リンク先全量へ義務を拡張しない |

## 採用する具体条件（原文）

### Overview of a SSRF common flow

原典 L17-25。

[説明図を省略。図中の構成が判断に必要なら原典へ戻る。]
*Notes:*

- SSRF is not limited to the HTTP protocol. Generally, the first request is HTTP, but in cases where the application itself performs the second request, it could use different protocols (*e.g.* FTP, SMB, SMTP, etc.) and schemes (*e.g.* `file://`, `phar://`, `gopher://`, `data://`, `dict://`, etc.).
- If the application is vulnerable to [XML eXternal Entity (XXE) injection](https://portswigger.net/web-security/xxe) then it can be exploited to perform a [SSRF attack](https://portswigger.net/web-security/xxe#exploiting-xxe-to-perform-ssrf-attacks), take a look at the [XXE cheat sheet](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.md) to learn how to prevent the exposure to XXE.

### Cases

原典 L26-34。

Depending on the application's functionality and requirements, there are two basic cases in which SSRF can happen:

- Application can send request only to **identified and trusted applications**: Case when [allowlist](https://en.wikipedia.org/wiki/Whitelisting) approach is available.
- Application can send requests to **ANY external IP address or domain name**: Case when [allowlist](https://en.wikipedia.org/wiki/Whitelisting) approach is unavailable.

Because these two cases are very different, this cheat sheet will describe defenses against them separately.

#### Case 1 - Application can send request only to identified and trusted applications

原典 L35-38。

Sometimes, an application needs to perform a request to another application, often located on another network, to perform a specific task. Depending on the business case, user input is required for the functionality to work.

##### Example

原典 L39-46。

> Take the example of a web application that receives and uses personal information from a user, such as their first name, last name, birth date etc. to create a profile in an internal HR system. By design, that web application will have to communicate using a protocol that the HR system understands to process that data.
 > Basically, the user cannot reach the HR system directly, but, if the web application in charge of receiving user information is vulnerable to SSRF, the user can leverage it to access the HR system.
 > The user leverages the web application as a proxy to the HR system.

The allowlist approach is a viable option since the internal application called by the *VulnerableApplication* is clearly identified in the technical/business flow. It can be stated that the required calls will only be targeted between those identified and trusted applications.

##### Available protections

原典 L47-50。

Several protective measures are possible at the **Application** and **Network** layers. To apply the **defense in depth** principle, both layers will be hardened against such attacks.

###### Application layer

原典 L51-67。

The first level of protection that comes to mind is [Input validation](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Input_Validation_Cheat_Sheet.md).

Based on that point, the following question comes to mind: *How to perform this input validation?*

As [Orange Tsai](https://twitter.com/orange_8361) shows in his [talk](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/assets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet_Orange_Tsai_Talk.pdf), depending on the programming language used, parsers can be abused. One possible countermeasure is to apply the [allowlist approach](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Input_Validation_Cheat_Sheet.md#allowlist-vs-denylist) when input validation is used because, most of the time, the format of the information expected from the user is globally known.

The request sent to the internal application will be based on the following information:

- String containing business data.
- IP address (V4 or V6).
- Domain name.
- URL.

**Note:** Disable the support for the following of the [redirection](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections) in your web client in order to prevent the bypass of the input validation described in the section `Exploitation tricks > Bypassing restrictions > Input validation > Unsafe redirect` of this [document](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/assets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet_SSRF_Bible.pdf).

###### String

原典 L68-86。

In the context of SSRF, validations can be added to ensure that the input string respects the business/technical format expected.

A [regex](https://www.regular-expressions.info/) can be used to ensure that data received is valid from a security point of view if the input data have a simple format (*e.g.* token, zip code, etc.). Otherwise, validation should be conducted using the libraries available from the `string` object because regex for complex formats are difficult to maintain and are highly error-prone.

User input is assumed to be non-network related and consists of the user's personal information.

Example:

[実装コード例を省略。方式固有の挙動に依存する判断では原典の当該範囲へ戻る。]

###### IP address

原典 L87-111。

In the context of SSRF, there are 2 possible validations to perform:

1. Ensure that the data provided is a valid IP V4 or V6 address.
2. Ensure that the IP address provided belongs to one of the IP addresses of the identified and trusted applications.

The first layer of validation can be applied using libraries that ensure the security of the IP address format, based on the technology used (library option is proposed here to delegate the managing of the IP address format and leverage battle-tested validation function):

> Verification of the proposed libraries has been performed regarding the exposure to bypasses (Hex, Octal, Dword, URL and Mixed encoding) described in this [article](https://medium.com/@vickieli/bypassing-ssrf-protection-e111ae70727b).

- **JAVA:** Method [InetAddressValidator.isValid](http://commons.apache.org/proper/commons-validator/apidocs/org/apache/commons/validator/routines/InetAddressValidator.html#isValid(java.lang.String)) from the [Apache Commons Validator](http://commons.apache.org/proper/commons-validator/) library.
    - **It is NOT exposed** to bypass using Hex, Octal, Dword, URL and Mixed encoding.
- **.NET**: Method [IPAddress.TryParse](https://docs.microsoft.com/en-us/dotnet/api/system.net.ipaddress.tryparse?view=netframework-4.8) from the SDK.
    - **It is exposed** to bypass using Hex, Octal, Dword and Mixed encoding but **NOT** the URL encoding.
    - As allowlisting is used here, any bypass tentative will be blocked during the comparison against the allowed list of IP addresses.
- **JavaScript**: Library [ip-address](https://www.npmjs.com/package/ip-address).
    - **It is NOT exposed** to bypass using Hex, Octal, Dword, URL and Mixed encoding.
- **Ruby**: Class [IPAddr](https://ruby-doc.org/stdlib-2.0.0/libdoc/ipaddr/rdoc/IPAddr.html) from the SDK.
    - **It is NOT exposed** to bypass using Hex, Octal, Dword, URL and Mixed encoding.

> **Use the output value of the method/library as the IP address to compare against the allowlist.**

After ensuring the validity of the incoming IP address, the second layer of validation is applied. An allowlist is created after determining all the IP addresses (v4 and v6 to avoid bypasses) of the identified and trusted applications. The valid IP is cross-checked with that list to ensure its communication with the internal application (string strict comparison with case sensitive).

###### Domain name

原典 L112-243。

In the attempt of validate domain names, it is apparent to do a DNS resolution to verify the existence of the domain. In general, it is not a bad idea, yet it opens up the application to attacks depending on the configuration used regarding the DNS servers used for the domain name resolution:

- It can disclose information to external DNS resolvers.
- It can be used by an attacker to bind a legit domain name to an internal IP address. See the section `Exploitation tricks > Bypassing restrictions > Input validation > DNS pinning` of this [document](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/assets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet_SSRF_Bible.pdf).
- An attacker can use it to deliver a malicious payload to the internal DNS resolvers and the API (SDK or third-party) used by the application to handle the DNS communication and then, potentially, trigger a vulnerability in one of these components.

In the context of SSRF, there are two validations to perform:

1. Ensure that the data provided is a valid domain name.
2. Ensure that the domain name provided belongs to one of the domain names of the identified and trusted applications (the allowlisting comes to action here).

Similar to the IP address validation, the first layer of validation can be applied using libraries that ensure the security of the domain name format, based on the technology used (library option is proposed here in order to delegate the managing of the domain name format and leverage battle tested validation function):

> Verification of the proposed libraries has been performed to ensure that the proposed functions do not perform any DNS resolution query.

- **JAVA:** Method [DomainValidator.isValid](https://commons.apache.org/proper/commons-validator/apidocs/org/apache/commons/validator/routines/DomainValidator.html#isValid(java.lang.String)) from the [Apache Commons Validator](http://commons.apache.org/proper/commons-validator/) library.
- **.NET**: Method [Uri.CheckHostName](https://docs.microsoft.com/en-us/dotnet/api/system.uri.checkhostname?view=netframework-4.8) from the SDK.
- **JavaScript**: Library [is-valid-domain](https://www.npmjs.com/package/is-valid-domain).
- **Python**: Module [validators.domain](https://validators.readthedocs.io/en/latest/#module-validators.domain).
- **Ruby**: No valid dedicated gem has been found.
    - [domainator](https://github.com/mhuggins/domainator), [public_suffix](https://github.com/weppos/publicsuffix-ruby) and [addressable](https://github.com/sporkmonger/addressable) has been tested but unfortunately they all consider `<script>alert(1)</script>.owasp.org` as a valid domain name.
    - This regex, taken from [here](https://stackoverflow.com/a/26987741), can be used: `^(((?!-))(xn--|_{1,1})?[a-z0-9-]{0,61}[a-z0-9]{1,1}\.)*(xn--)?([a-z0-9][a-z0-9\-]{0,60}|[a-z0-9-]{1,30}\.[a-z]{2,})$`

Example of execution of the proposed regex for Ruby:

[実装コード例を省略。方式固有の挙動に依存する判断では原典の当該範囲へ戻る。]

[実装コード例を省略。方式固有の挙動に依存する判断では原典の当該範囲へ戻る。]

After ensuring the validity of the incoming domain name, the second layer of validation is applied:

1. Build an allowlist with all the domain names of every identified and trusted applications.
2. Verify that the domain name received is part of this allowlist (string strict comparison with case sensitive).

Domain allowlisting alone does not prevent DNS rebinding. Validate the resolved destination IP addresses against the application's permitted networks, then ensure the HTTP client connects only to validated addresses. A second, unchecked DNS lookup between validation and connection can bypass these checks; see [GitLab's URL blocker guidance](https://docs.gitlab.com/development/secure_coding_guidelines/ruby/#url-blocker--validation-libraries).

Use an HTTP client mechanism that connects to the validated IP while preserving the original hostname for the HTTP `Host` header, TLS Server Name Indication (SNI), and certificate verification, as illustrated by [curl's custom address resolution](https://everything.curl.dev/usingcurl/connections/name.html#provide-a-custom-ip-address-for-a-name). Apply the destination policy to retries and fallback connections as well. The following DNS configuration and monitoring provide additional detection, not a substitute for connection-time enforcement:

1. Ensure that the domains that are part of your organization are resolved by your internal DNS server first in the chains of DNS resolvers.
2. Monitor the domains allowlist in order to detect when any of them resolves to a/an:
   - Local IP address (V4 + V6).
   - Internal IP of your organization (expected to be in private IP ranges) for the domain that are not part of your organization.

The following Python3 script can be used, as a starting point, for the monitoring mentioned above:

[実装コード例を省略。方式固有の挙動に依存する判断では原典の当該範囲へ戻る。]

###### URL

原典 L244-253。

Do not accept complete URLs from the user because URL are difficult to validate and the parser can be abused depending on the technology used as showcased by the following [talk](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/assets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet_Orange_Tsai_Talk.pdf) of [Orange Tsai](https://twitter.com/orange_8361).

If network related information is really needed then only accept a valid IP address or domain name.

**Match the host against an allowlist, and build the request yourself.** The input here is an IP address or a domain name rather than a whole URL, so compare that value against an explicit [allowlist](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html#allowlist-vs-denylist) of permitted destinations, then build the request from the entry that matched, together with a scheme, port and path the application fixes itself. Where that host was extracted from a URL the user supplied, do not copy the other components of that URL across either: carrying its path or query through, rather than rebuilding them, hands the next component something it has to parse again.

**Treat parser disagreement as a rejection.** Where a URL does cross a service boundary as a string and is parsed again at the other end, two implementations can read different hosts from the same bytes. `http://example.com\@evil.com` is such a string: a parser following the [WHATWG URL Standard](https://url.spec.whatwg.org/#url-parsing) treats the backslash under a special scheme as a path separator and reads the host as `example.com`, while the `userinfo` grammar in [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986#section-3.2.1) admits no backslash at all, so the string is not a URI that RFC defines. An implementation that does not enforce the grammar still returns a host, because it takes everything before the final `@` to be userinfo: CPython's [`urllib.parse`](https://docs.python.org/3/library/urllib.parse.html#urllib.parse.urlsplit) derives the host with [`netloc.rpartition('@')`](https://github.com/python/cpython/blob/v3.11.15/Lib/urllib/parse.py#L208), so `urlsplit("http://example.com\@evil.com").hostname` returns `evil.com`. Reject a URL whose host is not read identically by every parser in play, rather than reconciling the readings.

###### Network layer

原典 L254-265。

The objective of the Network layer security is to prevent the *VulnerableApplication* from performing calls to arbitrary applications. Only allowed *routes* will be available for this application in order to limit its network access to only those that it should communicate with.

The Firewall component, as a specific device or using the one provided within the operating system, will be used here to define the legitimate flows.

In the schema below, a Firewall component is leveraged to limit the application's access, and in turn, limit the impact of an application vulnerable to SSRF:

[説明図を省略。図中の構成が判断に必要なら原典へ戻る。]
[Network segregation](https://www.mwrinfosecurity.com/our-thinking/making-the-case-for-network-segregation) (see this set of [implementation advice](https://www.cyber.gov.au/acsc/view-all-content/publications/implementing-network-segmentation-and-segregation) can also be leveraged and **is highly recommended in order to block illegitimate calls directly at network level itself**.

#### Case 2 - Application can send requests to ANY external IP address or domain name

原典 L266-276。

This case happens when a user can control a URL to an **External** resource and the application makes a request to this URL (e.g. in case of [WebHooks](https://en.wikipedia.org/wiki/Webhook)). Allow lists cannot be used here because the list of IPs/domains is often unknown upfront and is dynamically changing.

In this scenario, *External* refers to any IP that doesn't belong to the internal network, and should be reached by going over the public internet.

Thus, the call from the *Vulnerable Application*:

- **Is NOT** targeting one of the IP/domain *located inside* the company's global network.
- Uses a convention defined between the *VulnerableApplication* and the expected IP/domain in order to *prove* that the call has been legitimately initiated.

##### Challenges in blocking URLs at application layer

原典 L277-285。

Based on the business requirements of the above mentioned applications, the allowlist approach is not a valid solution. Despite knowing that the block-list approach is not an impenetrable wall, it is the best solution in this scenario. It is informing the application what it should **not** do.

Here is why filtering URLs is hard at the Application layer:

- It implies that the application must be able to detect, at the code level, that the provided IP (V4 + V6) is not part of the official [private networks ranges](https://en.wikipedia.org/wiki/Private_network) including also *localhost* and *IPv4/v6 Link-Local* addresses. Not every SDK provides a built-in feature for this kind of verification, and leaves the handling up to the developer to understand all of its pitfalls and possible values, which makes it a demanding task.
- Same remark for domain name: The company must maintain a list of all internal domain names and provide a centralized service to allow an application to verify if a provided domain name is an internal one. For this verification, an internal DNS resolver can be queried by the application but this internal DNS resolver must not resolve external domain names.

##### Available protections

原典 L286-289。

Taking into consideration the same assumption in the following [example](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.md#example) for the following sections.

###### Application layer

原典 L290-312。

Like for the case [n°1](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.md#case-1---application-can-send-request-only-to-identified-and-trusted-applications), it is assumed that the `IP Address` or `domain name` is required to create the request that will be sent to the *TargetApplication*.

The first validation on the input data presented in the case [n°1](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.md#application-layer) on the 3 types of data will be the same for this case **BUT the second validation will differ**. Indeed, here we must use the block-list approach.

> **Regarding the proof of legitimacy of the request**: The *TargetedApplication* that will receive the request must generate a random token (ex: alphanumeric of 20 characters) that is expected to be passed by the caller (in body via a parameter for which the name is also defined by the application itself and only allow characters set `[a-z]{1,10}`) to perform a valid request. The receiving endpoint must only accept HTTP POST requests.

**Validation flow (if one the validation steps fail then the request is rejected):**

1. The application will receive the IP address or domain name of the *TargetedApplication* and it will apply the first validation on the input data using the libraries/regex mentioned in this [section](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.md#application-layer).
2. The second validation will be applied against the IP address or domain name of the *TargetedApplication* using the following block-list approach:
   - For IP address:
     - The application will verify that it is a public one (see the hint provided in the next paragraph with the python code sample).
   - For domain name:
        1. The application will verify that it is a public one by trying to resolve the domain name against the DNS resolver that will only resolve internal domain name. Here, it must return a response indicating that it do not know the provided domain because the expected value received must be a public domain.
        2. Retrieve all the IP addresses behind the domain name (records *A* + *AAAA* for IPv4 + IPv6) and apply the same public-address verification described above. Bind the connection to a validated address as described under [domain name validation](#domain-name); checking DNS answers separately does not prevent DNS rebinding.
3. The application will receive the protocol to use for the request via a dedicated input parameter for which it will verify the value against an allowed list of protocols (`HTTP` or `HTTPS`).
4. The application will receive the parameter name for the token to pass to the *TargetedApplication* via a dedicated input parameter for which it will only allow the characters set `[a-z]{1,10}`.
5. The application will receive the token itself via a dedicated input parameter for which it will only allow the characters set `[a-zA-Z0-9]{20}`.
6. The application will receive and validate (from a security point of view) any business data needed to perform a valid call.
7. The application will build the HTTP POST request **using only validated information** and connect only to an IP address validated in step 2 (*don't forget to disable the support for [redirection](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections) in the web client used*).

###### Network layer

原典 L313-316。

Similar to the following [section](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.md#network-layer).

### IMDSv2 in AWS

原典 L317-324。

In cloud environments SSRF is often used to access and steal credentials and access tokens from metadata services (e.g. AWS Instance Metadata Service, Azure Instance Metadata Service, GCP metadata server).

[IMDSv2](https://aws.amazon.com/blogs/security/defense-in-depth-open-firewalls-reverse-proxies-ssrf-vulnerabilities-ec2-instance-metadata-service/) is an additional defense-in-depth mechanism for AWS that mitigates some of the instances of SSRF.

To leverage this protection migrate to IMDSv2 and disable old IMDSv1. Check out [AWS documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html) for more details.

### Deny-list (Last Resort)

原典 L325-348。

**Deny-lists are bypass-prone. Prefer allow-lists.**

**When unavoidable, block these minimum ranges:**

| Service | Block IPs/Domains |
|---------|-------------------|
| **AWS IMDS** | `169.254.169.254`, `metadata.amazonaws.com` |
| **GCP Metadata** | `metadata.google.internal`, `169.254.169.254` |
| **Azure IMDS** | `169.254.169.254` |
| **Localhost** | `127.0.0.0/8`, `0.0.0.0/8`, `::1/128` |
| **RFC1918 Private** | `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` |
| **IPv6 Unique Local** | `fc00::/7` ([RFC 4193](https://datatracker.ietf.org/doc/html/rfc4193#section-3.1)) |
| **IPv6 Link-Local** | `fe80::/10` ([RFC 4291](https://datatracker.ietf.org/doc/html/rfc4291#section-2.5.6)) |
| **Multicast** | `224.0.0.0/4`, `ff00::/8` |

**Full production example:** [ComputerCraft SSRF deny-list](https://github.com/cc-tweaked/CC-Tweaked/blob/b9ed66983d714bcb5c6bf15b428e01a035106dbf/projects/core/src/main/java/dan200/computercraft/core/apis/http/options/AddressPredicate.java#L112-L157)

**Sources:**

- [IANA IPv4 Special Registry](https://www.iana.org/assignments/iana-ipv4-special-registry/iana-ipv4-special-registry.xhtml)
- [IANA IPv6 Special Registry](https://www.iana.org/assignments/iana-ipv6-special-registry/iana-ipv6-special-registry.xhtml)
