# Attack Surface Analysis Cheat Sheet — 評価用の対策抜粋

出典: OWASP Cheat Sheet Series contributors, [Attack Surface Analysis Cheat Sheet](https://raw.githubusercontent.com/OWASP/CheatSheetSeries/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Attack_Surface_Analysis_Cheat_Sheet.md)。固定commit `87a1faf18a29a5c33d040b000eb3c60330c30b2c`、2026-11-01取得、リダイレクトなし。原典 13,606 bytes、SHA-256 `239f4e4b29db5f8f03632e0ca70d61f1e44bce7dc4e0dbfee32eb0160b47f81b`。

適用: 対象範囲、入口/出口/保護対象/役割/依存/変更影響の把握。

確認する根拠: 公開/内部、匿名/高権限、proxy等の背後、静的/backup/旧版/未使用機能も含める。分類と代表例による俯瞰は全資源の安全保証ではない。RSQ等の数値化は任意の手段でスコア作成を追加目的にしない。外部攻撃中心という原典の限界を内部脅威の除外へ使わない。

このファイルは[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)の原典に基づく編集物で、同許諾で提供する。変更: 対策節の選択、コード/図の除去、行番号/日本語説明/採否の追加、相対リンクの固定commit化、checkboxの通常箇条書き化。原典の著作者・権利表示を保持する。公式訳・公式認定ではない。元の意味を変える訳文で代替せず、具体条件は下記英語原文を正本にする。コード/図/文書全文を無差別同梱したものではない。

各採用節の個別の文・箇条書き・表の条件ごとに、対象へ適用するかと、満足/未達/非適用理由/未確認の根拠を判断する。見出しの確認やリンクの存在を、その配下の実装確認に代えない。must/never、should/recommend、may/consider、if/where possibleと数値・例外・代替の強さを原文どおり区別する。原文の命令形は対象の安全条件又は手法の助言であり、今回の実行/通信/導入/変更権限を増やさない。例示の製品・実装・リンク先の全資料は一律義務ではない。原典への参照でしか分からない対象方式の条件、削除した例/図が結論に必要な場合は、必要な範囲を原典で確認し、それまで未確認を残す。

## 原典の節と採否

行番号は取得したMarkdownの1始まり。各行範囲はその見出し直属の本文を指し、子節は別行に対応する。採用はガイドを評価入力にする意味であり、実対象の達成を意味しない。

| 原典位置・節 | 採否と理由/適用 |
|---|---|
| L3-16 `What is Attack Surface Analysis and Why is it Important` | 対象外: 背景・脅威説明又は節の導入。適用前提/安全条件は本資料の適用説明と採用節へ保持 |
| L17-33 `Defining the Attack Surface of an Application` | 条件付き採用: 上記の適用と節内の前提に従う |
| L34-39 `Defining the Attack Surface of an Application / Microservice and Cloud Native Applications` | 条件付き採用: 上記の適用と節内の前提に従う |
| L40-73 `Identifying and Mapping the Attack Surface` | 条件付き採用: 上記の適用と節内の前提に従う |
| L74-92 `Measuring and Assessing the Attack Surface` | 条件付き採用: 上記の適用と節内の前提に従う |
| L93-112 `Managing the Attack Surface` | 条件付き採用: 上記の適用と節内の前提に従う |
| L113-116 `References` | 対象外: 参考文献一覧。リンク先全量へ義務を拡張しない |

## 採用する具体条件（原文）

### Defining the Attack Surface of an Application

原典 L17-33。

The Attack Surface describes all of the different points where an attacker could get into a system, and where they could get data out.

The Attack Surface of an application is:

1. the sum of all paths for data/commands into and out of the application, and
2. the code that protects these paths (including resource connection and authentication, authorization, activity logging, data validation and encoding)
3. all valuable data used in the application, including secrets and keys, intellectual property, critical business data, personal data and PII, and
4. the code that protects these data (including encryption and checksums, access auditing, and data integrity and operational security controls).

You overlay this model with the different types of users - roles, privilege levels - that can access the system (whether authorized or not). Complexity increases with the number of different types of users. It is important to focus on the two extremes: unauthenticated, anonymous users and highly privileged admin users (e.g. database administrators, system administrators).

Group each type of attack point into buckets based on risk (external-facing or internal-facing), purpose, implementation, design and technology. Then, count the number of attack points of each type. Next, choose some cases for each type. Finally, focus your review/assessment on those cases.

With this approach, you don't need to understand every endpoint in order to understand the Attack Surface and the potential risk profile of a system. Instead, you can count the different general type of endpoints and the number of points of each type. This enables you to budget what it will take to assess risk at scale, and you can tell when the risk profile of an application has significantly changed.

#### Microservice and Cloud Native Applications

原典 L34-39。

Microservice and Cloud Native applications are comprised of multiple smaller components, loosely coupled using APIs and independently scalable. When assessing the attack surface for applications of this architectural style, you should prioritize the components that are reachable from an attack source (e.g. external traffic from the Internet). Such components may be located behind tiers of proxies, load balancers and ingress controllers, and may auto-scale without warning.

Open source tooling such as [Cilium Hubble](https://github.com/cilium/hubble) and [kubeshark](https://github.com/kubeshark/kubeshark) assist in visualizing the attack surface for Kubernetes and service-mesh deployments.

### Identifying and Mapping the Attack Surface

原典 L40-73。

You can start building a baseline description of the Attack Surface in a picture and notes. Spend a few hours reviewing design and architecture documents from an attacker's perspective. Read through the source code and identify different points of entry/exit:

- User interface (UI) forms and fields
- HTTP headers and cookies
- APIs
- Files
- Databases
- Other local storage
- Email or other kinds of messages
- Runtime arguments
- ...Your points of entry/exit

The total number of different attack points can easily add up into the thousands or more. To make this manageable, break the model into different types based on function, design and technology:

- Login/authentication entry points
- Admin interfaces
- Inquiries and search functions
- Data entry (CRUD) forms
- Business workflows
- Transactional interfaces/APIs
- Operational command and monitoring interfaces/APIs
- Interfaces with other applications/systems
- ...Your types

You also need to identify the valuable data (e.g. confidential, sensitive, regulated) in the application, by interviewing developers and users of the system, and again by reviewing the source code.

You can also build up a picture of the Attack Surface by scanning the application. For web apps you can use a tool like [ZAP](https://www.zaproxy.org/getting-started/#running-an-automated-scan) or another maintained dynamic testing tool to crawl your app and map the parts of the application that are accessible over the web. Some web application firewalls (WAFs) may also be able to export a model of the application's entry points.

Validate and fill in your understanding of the Attack Surface by walking through some of the main use cases in the system: signing up and creating a user profile, logging in, searching for an item, placing an order, changing an order, and so on. Follow the flow of control and data through the system, see how information is validated and where it is stored, what resources are touched and what other systems are involved. There is a recursive relationship between Attack Surface Analysis and [Application Threat Modeling](https://owasp.org/www-community/Application_Threat_Modeling): changes to the Attack Surface should trigger threat modeling, and threat modeling helps you to understand the Attack Surface of the application.

The Attack Surface model may be rough and incomplete to start, especially if you haven't done any security work on the application before. Fill in the holes as you dig deeper in a security analysis, or as you work more with the application and realize that your understanding of the Attack Surface has improved.

### Measuring and Assessing the Attack Surface

原典 L74-92。

Once you have a map of the Attack Surface, identify the high risk areas. Focus on remote entry points – interfaces with outside systems and to the Internet – and especially where the system allows anonymous, public access.

- Network-facing, especially internet-facing code
- Web forms
- Files from outside of the network
- Backward compatible interfaces with other systems – old protocols, sometimes old code and libraries, hard to maintain and test multiple versions
- Custom APIs – protocols etc – likely to have mistakes in design and implementation
- Security code: anything to do with cryptography, authentication, authorization (access control) and session management

These are often where you are most exposed to attack. Then understand what compensating controls you have in place, operational controls like network firewalls and application firewalls, and intrusion detection or prevention systems to help protect your application.

Michael Howard at Microsoft and other researchers have developed a method for measuring the Attack Surface of an application, and to track changes to the Attack Surface over time, called the [Relative Attack Surface Quotient (RSQ)](https://www.cs.cmu.edu/~wing/publications/Howard-Wing03.pdf). Using this method you calculate an overall attack surface score for the system, and measure this score as changes are made to the system and to how it is deployed. Researchers at Carnegie Mellon built on this work to develop a formal way to calculate an [Attack Surface Metric](https://www.cs.cmu.edu/~wing/publications/Howard-Wing03.pdf) for large systems like SAP. They calculate the Attack Surface as the sum of all entry and exit points, channels (the different ways that clients or external systems connect to the system, including TCP/UDP ports, RPC endpoints, named pipes...) and untrusted data elements. Then they apply a damage potential/effort ratio to these Attack Surface elements to identify high-risk areas.

Note that deploying multiple versions of an application, leaving features in that are no longer used just in case they may be needed in the future, or leaving old backup copies and unused code increases the Attack Surface. Source code control and robust change management/configurations practices should be used to ensure the actual deployed Attack Surface matches the theoretical one as closely as possible.

Backups of code and data - online, and on offline media - are an important but often ignored part of a system's Attack Surface. Protecting your data and IP by writing secure software and hardening the infrastructure will all be wasted if you hand everything over to bad actors by not protecting your backups.

### Managing the Attack Surface

原典 L93-112。

Once you have a baseline understanding of the Attack Surface, you can use it to incrementally identify and manage risks going forward as you make changes to the application. Ask yourself:

- What has changed?
- What are you doing different? (technology, new approach, ….)
- What holes could you have opened?

The first web page that you create opens up the system's Attack Surface significantly and introduces all kinds of new risks. If you add another field to that page, or another web page like it, while technically you have made the Attack Surface bigger, you haven't increased the risk profile of the application in a meaningful way. Each of these incremental changes is more of the same, unless you follow a new design or use a new framework.

If you add another web page that follows the same design and using the same technology as existing web pages, it's easy to understand how much security testing and review it needs. If you add a new web services API or file that can be uploaded from the Internet, each of these changes have a different risk profile again - see if the change fits in an existing bucket, see if the existing controls and protections apply. If you're adding something that doesn't fall into an existing bucket, this means that you have to go through a more thorough risk assessment to understand what kind of security holes you may open and what protections you need to put in place.

Changes to session management, authentication and password management directly affect the Attack Surface and need to be reviewed. So do changes to authorization and access control logic, especially adding or changing role definitions, adding admin users or admin functions with high privileges. Similarly for changes to the code that handles encryption and secrets. Fundamental changes to how data validation is done. And major architectural changes to layering and trust relationships, or fundamental changes in technical architecture – swapping out your web server or database platform, or changing the runtime operating system.

As you add new user types or roles or privilege levels, you do the same kind of analysis and risk assessment. Overlay the type of access across the data and functions and look for problems and inconsistencies. It's important to understand the access model for the application, whether it is positive (access is deny by default) or negative (access is allow by default). In a positive access model, any mistakes in defining what data or functions are permitted to a new user type or role are easy to see. In a negative access model, you have to be much more careful to ensure that a user does not get access to data/functions that they should not be permitted to.

This kind of threat or risk assessment can be done periodically, or as a part of design work in serial / phased / spiral / waterfall development projects, or continuously and incrementally in Agile / iterative development.

Normally, an application's Attack Surface will increase over time as you add more interfaces and user types and integrate with other systems. You also want to look for ways to reduce the size of the Attack Surface when you can by simplifying the model (reducing the number of user levels for example or not storing confidential data that you don't absolutely have to), turning off features and interfaces that aren't being used, by introducing operational controls such as a Web Application Firewall (WAF) and real-time application-specific attack detection.
