# HTTP Security Response Headers Cheat Sheet — 評価用の対策抜粋

出典: OWASP Cheat Sheet Series contributors, [HTTP Security Response Headers Cheat Sheet](https://raw.githubusercontent.com/OWASP/CheatSheetSeries/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/HTTP_Headers_Cheat_Sheet.md)。固定commit `87a1faf18a29a5c33d040b000eb3c60330c30b2c`、2026-11-01取得、リダイレクトなし。原典 21,329 bytes、SHA-256 `df04323a8b10ef16cf3a248dc1431fc3d19edde6d905e70754f36ab0f3a045d6`。

適用: HTTP応答がある場合。各ヘッダが保護するクライアント/応答種別へ個別適用する。

確認する根拠: API JSONへのXFO/CSPの無意味さ、公開APIのCORS *例外、HSTS長期/全subdomain/preloadの運用前提、cache API別制御、非推奨ヘッダ、エラー等すべての応答で実効値と重複を確認する。具体的な値は原典のrecommendation/例であり、適用要件と両立する理由を判断する。

このファイルは[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)の原典に基づく編集物で、同許諾で提供する。変更: 対策節の選択、コード/図の除去、行番号/日本語説明/採否の追加、相対リンクの固定commit化、checkboxの通常箇条書き化。原典の著作者・権利表示を保持する。公式訳・公式認定ではない。元の意味を変える訳文で代替せず、具体条件は下記英語原文を正本にする。コード/図/文書全文を無差別同梱したものではない。

各採用節の個別の文・箇条書き・表の条件ごとに、対象へ適用するかと、満足/未達/非適用理由/未確認の根拠を判断する。見出しの確認やリンクの存在を、その配下の実装確認に代えない。must/never、should/recommend、may/consider、if/where possibleと数値・例外・代替の強さを原文どおり区別する。原文の命令形は対象の安全条件又は手法の助言であり、今回の実行/通信/導入/変更権限を増やさない。例示の製品・実装・リンク先の全資料は一律義務ではない。原典への参照でしか分からない対象方式の条件、削除した例/図が結論に必要な場合は、必要な範囲を原典で確認し、それまで未確認を残す。

外部資料だけに詳細がある条件: Set-Cookie / RecommendationとContent-Security-Policy / Recommendationは、それぞれ別記事の詳細を案内しており、この固定10記事だけから全設定条件を具体化済みとはしない。原典リンクの全量採用はせず、IPA/ASVS等の具体条件で確定できない必要な設定は未確定として、対象へのレビューで必要な範囲の一次資料確認まで未確認を残す。

## 原典の節と採否

行番号は取得したMarkdownの1始まり。各行範囲はその見出し直属の本文を指し、子節は別行に対応する。採用はガイドを評価入力にする意味であり、実対象の達成を意味しない。

| 原典位置・節 | 採否と理由/適用 |
|---|---|
| L3-8 `Introduction` | 対象外: 背景・脅威説明又は節の導入。適用前提/安全条件は本資料の適用説明と採用節へ保持 |
| L9-10 `Security Headers` | 条件付き採用: 上記の適用と節内の前提に従う |
| L11-18 `Security Headers / X-Frame-Options` | 条件付き採用: 上記の適用と節内の前提に従う |
| L19-25 `Security Headers / X-Frame-Options / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L26-31 `Security Headers / X-XSS-Protection` | 条件付き採用: 上記の適用と節内の前提に従う |
| L32-40 `Security Headers / X-XSS-Protection / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L41-46 `Security Headers / X-Content-Type-Options` | 条件付き採用: 上記の適用と節内の前提に従う |
| L47-52 `Security Headers / X-Content-Type-Options / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L53-56 `Security Headers / Referrer-Policy` | 条件付き採用: 上記の適用と節内の前提に従う |
| L57-64 `Security Headers / Referrer-Policy / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L65-70 `Security Headers / Content-Type` | 条件付き採用: 上記の適用と節内の前提に従う |
| L71-77 `Security Headers / Content-Type / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L78-81 `Security Headers / Cache-Control` | 条件付き採用: 上記の適用と節内の前提に従う |
| L82-90 `Security Headers / Cache-Control / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L91-94 `Security Headers / Cache-Control / References` | 対象外: 参考文献一覧。リンク先全量へ義務を拡張しない |
| L95-100 `Security Headers / Set-Cookie` | 条件付き採用: 上記の適用と節内の前提に従う |
| L101-104 `Security Headers / Set-Cookie / Recommendation` | 未確定: 詳細は別記事への参照のみ。上記の外部資料依存として扱う |
| L105-108 `Security Headers / Strict-Transport-Security (HSTS)` | 条件付き採用: 上記の適用と節内の前提に従う |
| L109-118 `Security Headers / Strict-Transport-Security (HSTS) / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L119-122 `Security Headers / Expect-CT ❌` | 条件付き採用: 上記の適用と節内の前提に従う |
| L123-126 `Security Headers / Expect-CT ❌ / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L127-132 `Security Headers / Content-Security-Policy (CSP)` | 条件付き採用: 上記の適用と節内の前提に従う |
| L133-136 `Security Headers / Content-Security-Policy (CSP) / Recommendation` | 未確定: 詳細は別記事への参照のみ。上記の外部資料依存として扱う |
| L137-142 `Security Headers / Access-Control-Allow-Origin` | 条件付き採用: 上記の適用と節内の前提に従う |
| L143-149 `Security Headers / Access-Control-Allow-Origin / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L150-159 `Security Headers / Cross-Origin-Opener-Policy (COOP)` | 条件付き採用: 上記の適用と節内の前提に従う |
| L160-164 `Security Headers / Cross-Origin-Opener-Policy (COOP) / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L165-170 `Security Headers / Cross-Origin-Embedder-Policy (COEP)` | 条件付き採用: 上記の適用と節内の前提に従う |
| L171-178 `Security Headers / Cross-Origin-Embedder-Policy (COEP) / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L179-182 `Security Headers / Cross-Origin-Resource-Policy (CORP)` | 条件付き採用: 上記の適用と節内の前提に従う |
| L183-187 `Security Headers / Cross-Origin-Resource-Policy (CORP) / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L188-193 `Security Headers / Permissions-Policy (formerly Feature-Policy)` | 条件付き採用: 上記の適用と節内の前提に従う |
| L194-200 `Security Headers / Permissions-Policy (formerly Feature-Policy) / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L201-206 `Security Headers / Server` | 条件付き採用: 上記の適用と節内の前提に従う |
| L207-213 `Security Headers / Server / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L214-217 `Security Headers / X-Powered-By` | 条件付き採用: 上記の適用と節内の前提に従う |
| L218-223 `Security Headers / X-Powered-By / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L224-227 `Security Headers / X-AspNet-Version` | 条件付き採用: 上記の適用と節内の前提に従う |
| L228-237 `Security Headers / X-AspNet-Version / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L238-241 `Security Headers / X-AspNetMvc-Version` | 条件付き採用: 上記の適用と節内の前提に従う |
| L242-251 `Security Headers / X-AspNetMvc-Version / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L252-262 `Security Headers / X-Robots-Tag` | 条件付き採用: 上記の適用と節内の前提に従う |
| L263-279 `Security Headers / X-Robots-Tag / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L280-283 `Security Headers / X-DNS-Prefetch-Control` | 条件付き採用: 上記の適用と節内の前提に従う |
| L284-292 `Security Headers / X-DNS-Prefetch-Control / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L293-296 `Security Headers / Public-Key-Pins (HPKP) ❌` | 条件付き採用: 上記の適用と節内の前提に従う |
| L297-300 `Security Headers / Public-Key-Pins (HPKP) ❌ / Recommendation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L301-310 `Security Headers / Secure File Download Headers` | 条件付き採用: 上記の適用と節内の前提に従う |
| L311-312 `Adding HTTP Headers in Different Technologies` | 条件付き採用: 上記の適用と節内の前提に従う |
| L313-320 `Adding HTTP Headers in Different Technologies / PHP` | 条件付き採用: 上記の適用と節内の前提に従う |
| L321-339 `Adding HTTP Headers in Different Technologies / Apache` | 条件付き採用: 上記の適用と節内の前提に従う |
| L340-355 `Adding HTTP Headers in Different Technologies / IIS` | 条件付き採用: 上記の適用と節内の前提に従う |
| L356-363 `Adding HTTP Headers in Different Technologies / HAProxy` | 条件付き採用: 上記の適用と節内の前提に従う |
| L364-371 `Adding HTTP Headers in Different Technologies / Nginx` | 条件付き採用: 上記の適用と節内の前提に従う |
| L372-386 `Adding HTTP Headers in Different Technologies / Express` | 条件付き採用: 上記の適用と節内の前提に従う |
| L387-388 `Testing Proper Implementation of Security Headers` | 対象外: オンライン製品例。実効応答の検証条件は採用本文と共通契約へ保持 |
| L389-392 `Testing Proper Implementation of Security Headers / Mozilla Observatory` | 対象外: オンライン製品例。実効応答の検証条件は採用本文と共通契約へ保持 |
| L393-397 `Testing Proper Implementation of Security Headers / SmartScanner` | 対象外: オンライン製品例。実効応答の検証条件は採用本文と共通契約へ保持 |
| L398-402 `References` | 対象外: 参考文献一覧。リンク先全量へ義務を拡張しない |

## 採用する具体条件（原文）

### Security Headers

原典 L9-10。



#### X-Frame-Options

原典 L11-18。

The `X-Frame-Options` HTTP response header can be used to indicate whether or not a browser should be allowed to render a page in a `<frame>`, `<iframe>`, `<embed>` or `<object>`. Sites can use this to avoid [clickjacking](https://owasp.org/www-community/attacks/Clickjacking) attacks, by ensuring that their content is not embedded into other sites.

Content Security Policy (CSP) frame-ancestors directive obsoletes X-Frame-Options for supporting browsers ([source](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)).

X-Frame-Options header is only useful when the HTTP response where it is included has something to interact with (e.g. links, buttons). If the HTTP response is a redirect or an API returning JSON data, X-Frame-Options does not provide any security.

##### Recommendation

原典 L19-25。

Use Content Security Policy (CSP) frame-ancestors directive if possible.

Do not allow displaying of the page in a frame.
> `X-Frame-Options: DENY`

#### X-XSS-Protection

原典 L26-31。

The HTTP `X-XSS-Protection` response header is a feature of Internet Explorer, Chrome, and Safari that stops pages from loading when they detect reflected cross-site scripting (XSS) attacks.

WARNING: Even though this header can protect users of older web browsers that don't yet support CSP, in some cases, this header can create XSS vulnerabilities in otherwise safe websites [source](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection).

##### Recommendation

原典 L32-40。

Use a Content Security Policy (CSP) that disables the use of inline JavaScript.

Do not set this header or explicitly turn it off.
> `X-XSS-Protection: 0`

Please see [Mozilla X-XSS-Protection](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection) for details.

#### X-Content-Type-Options

原典 L41-46。

The `X-Content-Type-Options` response HTTP header is used by the server to indicate to the browsers that the MIME types advertised in the Content-Type headers should be followed and not guessed.

This header is used to block browsers' [MIME type sniffing](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types#mime_sniffing), which can transform non-executable MIME types into executable MIME types ([MIME Confusion Attacks](https://blog.mozilla.org/security/2016/08/26/mitigating-mime-confusion-attacks-in-firefox/)).

##### Recommendation

原典 L47-52。

Set the Content-Type header correctly throughout the site.

> `X-Content-Type-Options: nosniff`

#### Referrer-Policy

原典 L53-56。

The `Referrer-Policy` HTTP header controls how much referrer information (sent via the Referer header) should be included with requests.

##### Recommendation

原典 L57-64。

Modern browsers default to [`strict-origin-when-cross-origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Referrer-Policy#strict-origin-when-cross-origin): same-origin requests include the origin, path, and query string in the `Referer` header; cross-origin requests include only the origin, except HTTPS-to-HTTP requests, which omit the header. Set the header explicitly rather than relying on browser defaults.

> `Referrer-Policy: strict-origin-when-cross-origin`

- *NOTE:* For more information on configuring this header please see [Mozilla Referrer-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy).

#### Content-Type

原典 L65-70。

The [`Content-Type`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) representation header is used to indicate the original media type of the resource (before any content encoding is applied for sending). If not set correctly, the resource (e.g. an image) may be interpreted as HTML, making XSS vulnerabilities possible.

Although it is recommended to always set the `Content-Type` header correctly, it would constitute a vulnerability only if the content is intended to be rendered by the client and the resource is untrusted (provided or modified by a user).

##### Recommendation

原典 L71-77。

> `Content-Type: text/html; charset=UTF-8`

- *NOTE:* the `charset` attribute is necessary to prevent XSS in **HTML** pages
- *NOTE*: the `Content-Type` can be any of the possible [MIME types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)

#### Cache-Control

原典 L78-81。

The `Cache-Control` header defines how responses are cached by browsers and intermediate caches.

##### Recommendation

原典 L82-90。

- Use [`no-store`](https://www.rfc-editor.org/rfc/rfc9111.html#section-5.2.2.5) for sensitive responses to prohibit storage in compliant HTTP caches.
- Use `private` to allow caching only in non-shared (user-specific) caches and to prevent storage in shared caches (note that private caches may still persist the response).
- Avoid relying on default caching behavior for sensitive or protected content.
- Be aware that `no-cache` does not prevent caching; it allows caches to store responses. It requires revalidation with the origin server before reuse.

`no-store` does not control every storage mechanism: the [Cache API does not honor HTTP caching headers](https://developer.mozilla.org/en-US/docs/Web/API/Cache). Explicitly exclude sensitive responses from application-managed caches and remove existing sensitive entries; see [Offline Applications](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/HTML5_Security_Cheat_Sheet.md#offline-applications).

#### Set-Cookie

原典 L95-100。

The [`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie) HTTP response header is used to send a cookie from the server to the user agent, so the user agent can send it back to the server later. To send multiple cookies, multiple Set-Cookie headers should be sent in the same response.

This is not a security header per se, but its security attributes are crucial.

##### Recommendation

原典 L101-104。

- Please read [Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html#cookies) for a detailed explanation on cookie configuration options.

#### Strict-Transport-Security (HSTS)

原典 L105-108。

The HTTP [`Strict-Transport-Security`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security) response header (often abbreviated as HSTS) instructs browsers to only access the website using HTTPS, even if a user attempts to connect over HTTP.

##### Recommendation

原典 L109-118。

> `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`

- *NOTE*: Read carefully how this header works before using it. If the HSTS header is misconfigured or if there is a problem with the SSL/TLS certificate being used, legitimate users might be unable to access the website. For example, if the HSTS header is set to a very long duration and the SSL/TLS certificate expires or is revoked, legitimate users might be unable to access the website until the HSTS header duration has expired.

See the [HSTS preload requirements](https://hstspreload.org/) before requesting preload inclusion.

Please check out [HTTP Strict Transport Security Cheat Sheet](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.md) for more information.

#### Expect-CT ❌

原典 L119-122。

The `Expect-CT` header lets sites opt-in to reporting of Certificate Transparency (CT) requirements. Given that mainstream clients now require CT qualification, the only remaining value is reporting such occurrences to the nominated report-uri value in the header. The header is now less about enforcement and more about detection/reporting.

##### Recommendation

原典 L123-126。

Do not use it. Mozilla [recommends](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Expect-CT) avoiding it, and removing it from existing code if possible.

#### Content-Security-Policy (CSP)

原典 L127-132。

Content Security Policy (CSP) is a security feature that is used to specify the origin of content that is allowed to be loaded on a website or in a web application. It is an added layer of security that helps to detect and mitigate certain types of attacks, including Cross-Site Scripting (XSS) and data injection attacks. These attacks are used for everything from data theft to site defacement to distribution of malware.

- *NOTE*: This header is relevant to be applied in pages which can load and interpret scripts and code, but might be meaningless in the response of a REST API that returns content that is not going to be rendered.

##### Recommendation

原典 L133-136。

Content Security Policy is complex to configure and maintain. For an explanation on customization options, please read [Content Security Policy Cheat Sheet](https://github.com/OWASP/CheatSheetSeries/blob/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Content_Security_Policy_Cheat_Sheet.md)

#### Access-Control-Allow-Origin

原典 L137-142。

If you don't use this header, your site is protected by default by the Same Origin Policy (SOP). What this header does is relax this control in specified circumstances.

The `Access-Control-Allow-Origin` is a CORS (cross-origin resource sharing) header. This header indicates whether the response it is related to can be shared with requesting code from the given origin. In other words, if siteA requests a resource from siteB, siteB should indicate in its `Access-Control-Allow-Origin` header that siteA is allowed to fetch that resource, if not, the access is blocked due to Same Origin Policy (SOP).

##### Recommendation

原典 L143-149。

If you use it, set specific [origins](https://developer.mozilla.org/en-US/docs/Glossary/Origin) instead of `*`. Check out [Access-Control-Allow-Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) for details.
> `Access-Control-Allow-Origin: https://yoursite.com`

- *NOTE*: The use of '\*' might be necessary depending on your needs. For example, for a public API that should be accessible from any origin, it might be necessary to allow '\*'.

#### Cross-Origin-Opener-Policy (COOP)

原典 L150-159。

The HTTP `Cross-Origin-Opener-Policy` (COOP) response header allows you to ensure a top-level document does not share a browsing context group with cross-origin documents.

This header works together with Cross-Origin-Embedder-Policy (COEP) and Cross-Origin-Resource-Policy (CORP) explained below.

This mechanism protects against attacks like Spectre which can cross the security boundary established by Same Origin Policy (SOP) for resources in the same browsing context group.

As these headers are very related to browsers, it may not make sense to be applied to REST APIs or clients that are not browsers.

##### Recommendation

原典 L160-164。

Isolates the browsing context exclusively to same-origin documents.
> `Cross-Origin-Opener-Policy: same-origin`

#### Cross-Origin-Embedder-Policy (COEP)

原典 L165-170。

The HTTP [`Cross-Origin-Embedder-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy) (COEP) response header prevents a document from loading any cross-origin resources that don't explicitly grant the document permission (using [CORP](#cross-origin-resource-policy-corp) or CORS).

- *NOTE*: Enabling this will block cross-origin resources not configured correctly from loading.

##### Recommendation

原典 L171-178。

A document can only load resources from the same origin, or resources explicitly marked as loadable from another origin.
> `Cross-Origin-Embedder-Policy: require-corp`

- *NOTE*: The `crossorigin` attribute requests the resource in CORS mode. The resource server must [permit the request through CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Cross-Origin-Embedder-Policy#require-corp); the attribute alone does not grant permission:
- `<img src="https://thirdparty.com/img.png" crossorigin>`

#### Cross-Origin-Resource-Policy (CORP)

原典 L179-182。

The [`Cross-Origin-Resource-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Resource-Policy) (CORP) header allows you to control the set of origins that are empowered to include a resource. It is a robust defense against attacks like [Spectre](https://meltdownattack.com/), as it allows browsers to block a given response before it enters an attacker's process.

##### Recommendation

原典 L183-187。

Limit current resource loading to the site and sub-domains only.
> `Cross-Origin-Resource-Policy: same-site`

#### Permissions-Policy (formerly Feature-Policy)

原典 L188-193。

Permissions-Policy allows you to control which origins can use which browser features, both in the top-level page and in embedded frames. For every feature controlled by Feature Policy, the feature is only enabled in the current document or frame if its origin matches the allowed list of origins. This means that you can configure your site to never allow the camera or microphone to be activated. This prevents that an injection, for example an XSS, enables the camera, the microphone, or other browser feature.

More information: [Permissions-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy)

##### Recommendation

原典 L194-200。

Set it and disable all the features that your site does not need or allow them only to the authorized domains:
> `Permissions-Policy: geolocation=(), camera=(), microphone=()`

- *NOTE*: This example is disabling geolocation, camera, and microphone for all domains.

#### Server

原典 L201-206。

The [`Server`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Server) header describes the software used by the origin server that handled the request — that is, the server that generated the response.

This is not a security header, but how it is used is relevant for security.

##### Recommendation

原典 L207-213。

Remove this header or set non-informative values.
> `Server: webserver`

- *NOTE*: Remember that attackers have other means of fingerprinting the server technology.

#### X-Powered-By

原典 L214-217。

The `X-Powered-By` header describes the technologies used by the webserver. This information exposes the server to attackers. Using the information in this header, attackers can find vulnerabilities easier.

##### Recommendation

原典 L218-223。

Remove all `X-Powered-By` headers.

- *NOTE*: Remember that attackers have other means of fingerprinting your tech stack.

#### X-AspNet-Version

原典 L224-227。

Provides information about the .NET version.

##### Recommendation

原典 L228-237。

Disable sending this header. Add the following line in your `web.config` in the `<system.web>` section to remove it.

[実装コード例を省略。方式固有の挙動に依存する判断では原典の当該範囲へ戻る。]

- *NOTE*: Remember that attackers have other means of fingerprinting your tech stack.

#### X-AspNetMvc-Version

原典 L238-241。

Provides information about the .NET version.

##### Recommendation

原典 L242-251。

Disable sending this header. To remove the `X-AspNetMvc-Version` header, add the below line in `Global.asax` file.

[実装コード例を省略。方式固有の挙動に依存する判断では原典の当該範囲へ戻る。]

- *NOTE*: Remember that attackers have other means of fingerprinting your tech stack.

#### X-Robots-Tag

原典 L252-262。

The HTTP `X-Robots-Tag` response header controls how search engines and other automated crawlers index and display resources such as PDFs, images, and other non-HTML content.
It functions similarly to the `<meta name="robots">` tag, but is applied via the HTTP response header, allowing greater flexibility (e.g., for non-HTML files or server-wide rules).

[実装コード例を省略。方式固有の挙動に依存する判断では原典の当該範囲へ戻る。]

- **Note:** Only compliant crawlers respect these directives, and they must still make an HTTP request to read the headers before deciding how to handle the content.

##### Recommendation

原典 L263-279。

Use the `X-Robots-Tag` header to control crawler behavior:

- For **private or sensitive content** you don’t want indexed:

  > `X-Robots-Tag: noindex, nofollow`
  > This prevents compliant search engines from indexing the resource or following links on it.

- For **public content** you want indexed and discoverable (e.g., documentation, datasets):

  > `X-Robots-Tag: index, follow`
  > This allows search engines to index the resource and follow its links.

You can also use other directives such as `noarchive`, `nosnippet`, or `noimageindex` depending on your needs.
Server configuration can apply this header selectively — for example, only on specific file types (like PDFs or images).

#### X-DNS-Prefetch-Control

原典 L280-283。

The `X-DNS-Prefetch-Control` HTTP response header controls DNS prefetching, a feature by which browsers proactively perform domain name resolution on both links that the user may choose to follow as well as URLs for items referenced by the document, including images, CSS, JavaScript, and so forth.

##### Recommendation

原典 L284-292。

The default behavior of browsers is to perform DNS caching which is good for most websites.
If you do not control links on your website, you might want to set `off` as a value to disable DNS prefetch to avoid leaking information to those domains.

> `X-DNS-Prefetch-Control: off`

- *NOTE*: Do not rely on this functionality for anything production sensitive: it is not standard or fully supported and implementation may vary among browsers.

#### Public-Key-Pins (HPKP) ❌

原典 L293-296。

The HTTP `Public-Key-Pins` response header was used to associate a specific cryptographic public key with a web server to mitigate MITM attacks with forged certificates. It was removed from Chromium in 2018 and is unsupported by all modern browsers.

##### Recommendation

原典 L297-300。

Do not use. Remove any `Public-Key-Pins` or `Public-Key-Pins-Report-Only` headers from production. Rely on Certificate Transparency (CT) and CAA DNS records, which provide superior compromise detection without the operational brittleness of pinning.

#### Secure File Download Headers

原典 L301-310。

When serving user-provided files, proper HTTP headers should be used to prevent unintended execution in the browser.

- Use `Content-Disposition: attachment` to force download instead of inline rendering.
- Use `Content-Type: application/octet-stream` for unknown or binary files.
- Ensure `X-Content-Type-Options: nosniff` is set to prevent MIME type sniffing.

These headers help reduce risks such as Cross-Site Scripting (XSS) and unintended file execution.

### Adding HTTP Headers in Different Technologies

原典 L311-312。



#### PHP

原典 L313-320。

The sample code below sets the `X-Frame-Options` header in PHP.

[実装コード例を省略。方式固有の挙動に依存する判断では原典の当該範囲へ戻る。]

#### Apache

原典 L321-339。

Below is an `.htaccess` sample configuration which sets the `X-Frame-Options` header in Apache.

As described in the [Apache documentation](https://httpd.apache.org/docs/2.4/mod/mod_headers.html#header), `Header set` (default `onsuccess`) and `Header always set` operate on separate internal header tables.

In some cases, both header tables may be used, which can result in duplicate headers if the same header is configured in both contexts.

If a header needs to be removed entirely, it should be unset in both contexts (`onsuccess` and `always`).

To avoid duplication and ensure the header is sent on all responses, unset it first and then use `always set`:

[実装コード例を省略。方式固有の挙動に依存する判断では原典の当該範囲へ戻る。]

#### IIS

原典 L340-355。

Add configurations below to your `Web.config` in IIS to send the `X-Frame-Options` header.

[実装コード例を省略。方式固有の挙動に依存する判断では原典の当該範囲へ戻る。]

#### HAProxy

原典 L356-363。

Add the line below to your front-end, listen, or backend configurations to send the `X-Frame-Options` header.

[実装コード例を省略。方式固有の挙動に依存する判断では原典の当該範囲へ戻る。]

#### Nginx

原典 L364-371。

Below is a sample configuration, it sets the `X-Frame-Options` header in Nginx. Note that without the `always` option, the header will only be sent for certain status codes, as described in [the nginx documentation](https://nginx.org/en/docs/http/ngx_http_headers_module.html#add_header).

[実装コード例を省略。方式固有の挙動に依存する判断では原典の当該範囲へ戻る。]

#### Express

原典 L372-386。

You can use [helmet](https://www.npmjs.com/package/helmet) to setup HTTP headers in Express. The code below is a sample for adding the `X-Frame-Options` header.

[実装コード例を省略。方式固有の挙動に依存する判断では原典の当該範囲へ戻る。]
