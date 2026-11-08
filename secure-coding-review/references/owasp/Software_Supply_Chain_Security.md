# Software Supply Chain Security — 評価用の対策抜粋

出典: OWASP Cheat Sheet Series contributors, [Software Supply Chain Security](https://raw.githubusercontent.com/OWASP/CheatSheetSeries/87a1faf18a29a5c33d040b000eb3c60330c30b2c/cheatsheets/Software_Supply_Chain_Security_Cheat_Sheet.md)。固定commit `87a1faf18a29a5c33d040b000eb3c60330c30b2c`、2026-11-01取得、リダイレクトなし。原典 22,116 bytes、SHA-256 `c3235cb8820e1a54f6cc7ec97955fd6288f4d41db3cd3d00b31f7ef29b95ad8d`。

適用: 開発/依存/ビルド/配布/実行経路と、その安全性を維持する工程。

確認する根拠: source/依存/ビルド/実行各経路の権限・秘密・観測・供給者・固定/更新・署名/来歴・隔離・成果物・更新後監視。SBOMはmayであり提出を独自必須条件にしない。ツール名/認証/私有registryの説明を一律導入条件にせず、推奨と具体リスクを分ける。

このファイルは[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)の原典に基づく編集物で、同許諾で提供する。変更: 対策節の選択、コード/図の除去、行番号/日本語説明/採否の追加、相対リンクの固定commit化、checkboxの通常箇条書き化。原典の著作者・権利表示を保持する。公式訳・公式認定ではない。元の意味を変える訳文で代替せず、具体条件は下記英語原文を正本にする。コード/図/文書全文を無差別同梱したものではない。

各採用節の個別の文・箇条書き・表の条件ごとに、対象へ適用するかと、満足/未達/非適用理由/未確認の根拠を判断する。見出しの確認やリンクの存在を、その配下の実装確認に代えない。must/never、should/recommend、may/consider、if/where possibleと数値・例外・代替の強さを原文どおり区別する。原文の命令形は対象の安全条件又は手法の助言であり、今回の実行/通信/導入/変更権限を増やさない。例示の製品・実装・リンク先の全資料は一律義務ではない。原典への参照でしか分からない対象方式の条件、削除した例/図が結論に必要な場合は、必要な範囲を原典で確認し、それまで未確認を残す。

## 原典の節と採否

行番号は取得したMarkdownの1始まり。各行範囲はその見出し直属の本文を指し、子節は別行に対応する。採用はガイドを評価入力にする意味であり、実対象の達成を意味しない。

| 原典位置・節 | 採否と理由/適用 |
|---|---|
| L3-17 `Introduction` | 対象外: 背景・脅威説明又は節の導入。適用前提/安全条件は本資料の適用説明と採用節へ保持 |
| L18-30 `Overview of Threat Landscape` | 対象外: 背景・脅威説明又は節の導入。適用前提/安全条件は本資料の適用説明と採用節へ保持 |
| L31-34 `Mitigations and Security Best Practices` | 条件付き採用: 上記の適用と節内の前提に従う |
| L35-38 `Mitigations and Security Best Practices / General` | 条件付き採用: 上記の適用と節内の前提に従う |
| L39-42 `Mitigations and Security Best Practices / General / Implement Strong Access Control` | 条件付き採用: 上記の適用と節内の前提に従う |
| L43-48 `Mitigations and Security Best Practices / General / Logging and Monitoring` | 条件付き採用: 上記の適用と節内の前提に従う |
| L49-52 `Mitigations and Security Best Practices / General / Leverage Security Automation` | 条件付き採用: 上記の適用と節内の前提に従う |
| L53-56 `Mitigations and Security Best Practices / Mitigating Source Code Threats` | 条件付き採用: 上記の適用と節内の前提に従う |
| L57-60 `Mitigations and Security Best Practices / Mitigating Source Code Threats / Peer Reviews` | 条件付き採用: 上記の適用と節内の前提に従う |
| L61-64 `Mitigations and Security Best Practices / Mitigating Source Code Threats / Secure Config of Version Control Systems` | 条件付き採用: 上記の適用と節内の前提に従う |
| L65-68 `Mitigations and Security Best Practices / Mitigating Source Code Threats / Secure Development Platform` | 条件付き採用: 上記の適用と節内の前提に従う |
| L69-72 `Mitigations and Security Best Practices / Mitigating Dependency Threats` | 条件付き採用: 上記の適用と節内の前提に従う |
| L73-90 `Mitigations and Security Best Practices / Mitigating Dependency Threats / Assess Suppliers` | 条件付き採用: 上記の適用と節内の前提に従う |
| L91-96 `Mitigations and Security Best Practices / Mitigating Dependency Threats / Understand and Monitor Software Dependencies` | 条件付き採用: 上記の適用と節内の前提に従う |
| L97-100 `Mitigations and Security Best Practices / Mitigating Dependency Threats / SAST` | 条件付き採用: 上記の適用と節内の前提に従う |
| L101-104 `Mitigations and Security Best Practices / Mitigating Dependency Threats / Lockfile/Version Pinning` | 条件付き採用: 上記の適用と節内の前提に従う |
| L105-108 `Mitigations and Security Best Practices / Build Threats` | 条件付き採用: 上記の適用と節内の前提に従う |
| L109-112 `Mitigations and Security Best Practices / Build Threats / Inventory Build Tools` | 条件付き採用: 上記の適用と節内の前提に従う |
| L113-121 `Mitigations and Security Best Practices / Build Threats / Harden Build Tools` | 条件付き採用: 上記の適用と節内の前提に従う |
| L122-125 `Mitigations and Security Best Practices / Build Threats / Enforce Code Signing` | 条件付き採用: 上記の適用と節内の前提に従う |
| L126-129 `Mitigations and Security Best Practices / Build Threats / Use Private Artifact Repository` | 条件付き採用: 上記の適用と節内の前提に従う |
| L130-133 `Mitigations and Security Best Practices / Build Threats / Use Source Control for Build Scripts and Config` | 条件付き採用: 上記の適用と節内の前提に従う |
| L134-137 `Mitigations and Security Best Practices / Build Threats / Verify Provenance/Ensure Sufficient Metadata is Generated` | 条件付き採用: 上記の適用と節内の前提に従う |
| L138-141 `Mitigations and Security Best Practices / Build Threats / Ephemeral, Isolated Builds` | 条件付き採用: 上記の適用と節内の前提に従う |
| L142-145 `Mitigations and Security Best Practices / Build Threats / Limit use of Parameters` | 条件付き採用: 上記の適用と節内の前提に従う |
| L146-149 `Mitigations and Security Best Practices / Deployment and Runtime Threats` | 条件付き採用: 上記の適用と節内の前提に従う |
| L150-153 `Mitigations and Security Best Practices / Deployment and Runtime Threats / Scan Final Build Binary` | 条件付き採用: 上記の適用と節内の前提に従う |
| L154-157 `Mitigations and Security Best Practices / Deployment and Runtime Threats / Monitor Deployed Software for Vulnerabilities` | 条件付き採用: 上記の適用と節内の前提に従う |
| L158-162 `References` | 対象外: 参考文献一覧。リンク先全量へ義務を拡張しない |

## 採用する具体条件（原文）

### Mitigations and Security Best Practices

原典 L31-34。

Mitigating SSC related risk can seem daunting, yet it need not be. Even for sophisticated attacks that may focus on compromising upstream suppliers, individual organization can take reasonable steps to defend its own assets and mitigate risk even if its supplier is compromised. Although some parts of SSCS may remain outside direct control of development teams, those teams must still do their part to improve SSCS in their organization; the guidance below is intended as starting point for developers to do just that.

#### General

原典 L35-38。

The practices described below are general techniques that can be used to mitigate risk related to a wide variety of threat types.

##### Implement Strong Access Control

原典 L39-42。

Compromised accounts, particularly privileged ones, represents a significant threats to SSCs. Account takeover can allow an attacker can perform a variety of malicious acts including injecting code into legitimate dependencies, manipulating CI/CD pipeline execution, and replacing a benign artifact with a malicious one. Strong access control for build, development, version control, and similar environments is thus critical. Best practices include adhering to  the basic security principles of least privileges and separation of duties, enforcing MFA, rotating credentials, and ensuring credentials are never stored or transmitted in clear text or committed to source control.

##### Logging and Monitoring

原典 L43-48。

When considering SSCS, the importance of detective controls should not be overlooked; these controls are essential for detecting attacks and enabling prompt respond. In the context of SSCS, logging is critical. All systems involved in the SSC, including VCS, build tools, delivery mechanisms, artifact repositories, and the systems responsible for running applications should be configured to log authentication attempts, configuration changes, and other events that could assist in identifying anomalous behavior or that could prove crucial for incident response efforts. Logs throughout the SSC must be sufficient in both depth and breadth to support detection and response

  However, logging events is not sufficient. These logs must be monitored, and, if necessary, acted upon. A centralized SIEM, log aggregator, or similar tool is preferred, especially given the complexity of SSCs. Regardless of the technology used, the basic objective remains the same: log data should be actionable.

##### Leverage Security Automation

原典 L49-52。

For complex SSCs, automation of security tasks, such as scanning, monitoring, and testing is critical. Such automation, while not a replacement for manual reviews and other actions performed by skilled professionals, is capable of detecting, and in some cases responding to, vulnerabilities and potential attacks with a scale and consistency that is hard to achieve through manual human intervention. Types of tools that support automation include SAST, DAST, SCA, container image scanners and more. The exact tools most capable of delivering value to an organization will vary significantly based on the characteristics of the organization. However, regardless of the type of tools and vendors used, it is important to acknowledge that these tools themselves must be mainlined, secured, and configured correctly. Failure to do so could actually increase SSC risk for an organization, or at the very least, fail to bring meaningful benefit to the organization. Finally, it must be clearly understood that these tools are but one component of an overall SSCS program; they cannot be considered a comprehensive solution or be relied on to identify all vulnerabilities.

#### Mitigating Source Code Threats

原典 L53-56。

The practices described below can help reduce SSC risk associated with source code and development.

##### Peer Reviews

原典 L57-60。

Manual code reviews are an important, relatively low cost technique for reducing SSC risk; these reviews can act as both detective controls and deterrents. Reviews should be performed by peers possessing both experience in the technology being used and secure coding processes and should occur before code is merged within a source control systems ([Google source safeguards](https://cloud.google.com/software-supply-chain-security/docs/safeguard-source)). The reviews should look for both unintentional security flaws as well as intentional code that could serve malicious purposes. The results of the review should be documented for later review if needed.

##### Secure Config of Version Control Systems

原典 L61-64。

Compromise or abuse of the source control system is consistently recognized as a significant SSC risk ([Google supply-chain threat categories](https://cloud.google.com/software-supply-chain-security/docs/attack-vectors), [SLSA 1.0 threats](https://slsa.dev/spec/v1.0/threats)). The general security best practices of strong access control and logging and monitoring are two methods to help secure VCS. Security features specific to the VCS system, such as protected branches and merge policies in git, should also be leveraged. You can find a wide variety of recommended policies in this [documentation](https://policies.legitify.dev/). There are tools available to help manage configuration of SCM systems, such as [Legitify](https://github.com/Legit-Labs/legitify), an open-source tool by [Legit security](https://www.legitsecurity.com/). Legitify is designed to detect misconfigurations in GitHub and GitLab and assist with the implementation of best practices. Regardless of any security controls added a VCS, it must be remember that secrets should never be committed to these systems.

##### Secure Development Platform

原典 L65-68。

IDEs, development plugins, and similar tools can help assist the development process. However, like all pieces of software, these components can have vulnerabilities and become an attack vector. Thus, it is important to take steps not only to ensure these tools are used securely, but also to secure the underlying system. The development system should have endpoint security software installed and should have threat assessments performed against it ([ESF developer practices](https://www.nsa.gov/Press-Room/Digital-Media-Center/Document-Gallery/igphoto/2003068942/)). Only trusted, well-vetted software should be used in the development process; this includes not only "core" development tools such as IDEs, but also any plugins or extensions.  Additionally, these tools should be included as part of an organization's system inventory.

#### Mitigating Dependency Threats

原典 L69-72。

Best practices and techniques related to secure use of dependencies are described below.

##### Assess Suppliers

原典 L73-90。

Before incorporating a third-party service, product, or software component into the SSC, the vendor and specific offering should both be thoroughly assessed for security. This applies to both open-source and proprietary offerings. The form and extent of the analysis will vary substantially in accordance with both the criticality and nature of the component being considered. Component maturity, security history, and the vendor's response to past vulnerabilities are useful information in nearly any case. For larger vendors or service offerings, determining whether or not a solution has been evaluated against third-party assessments and certifications, such as those performed against [FedRAMP](https://marketplace.fedramp.gov/products), [CSA](https://cloudsecurityalliance.org/star/registry), or various ISO standards (ISO/IEC 27001, ISO/IEC 15408,
ISO/IEC 27034), can be a useful data point, but must not be relied on exclusively.

Due to its transparent nature, open-source projects offer additional assessment opportunities. Questions to consider include ([OpenSSF component evaluation](https://best.openssf.org/Concise-Guide-for-Evaluating-Open-Source-Software)):

- Is the project actively maintained?
- Is the project sufficiently popular and well-known in the applicable community?
- Is the project sufficiently mature?
- Is the product or version being evaluated a "release" version, e.g. not an alpha, beta, or comparable versions?
- Given the complexity of the project, does the project have a sufficient number of maintainers and contributors?
- Does the project keep its dependencies updated?
- Does the project have sufficient test coverage and do the tests include security relevant rules?
- Is the project well-documented and does the document include guidance on how to use the component securely?
- Does the project have an established and documented process for reporting vulnerabilities and are these vulnerabilities addressed in a timely manner?
- Is the intended usage of the project consistent with the project's license?

##### Understand and Monitor Software Dependencies

原典 L91-96。

While third-party software dependencies can greatly accelerate the development process, they are also one of the leading risks associated with modern applications. Dependencies must not only be carefully selected before they are incorporated into an application, but also carefully monitored and maintained throughout the SDLC. In order achieve this, having insight into the various dependencies consumed by software is a crucial first step. To facilitate this, SBOMs may be used. Both production and consumption of these SBOMs should be automated, preferably as part of the  organization's CI/CD process.

Once the organization has inventoried dependencies, it must also monitor them for known vulnerabilities. This should also be automated as much as possible; tools such as [OWASP Dependency Check](https://owasp.org/www-project-dependency-check/) or [retire.js](https://retirejs.github.io/retire.js/) can assist in this process. Additionally, sources such as the [NVD](https://nvd.nist.gov/), [OSVDB](https://osv.dev/list), or [CISA KEV catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) may also be monitored for known vulnerabilities related to dependencies used in the organization's SSC.

##### SAST

原典 L97-100。

Although using SAST to detect potential security in custom developed code is a widely used security technique, it can also be used on OSS components within the SSC ([ESF developer practices](https://www.nsa.gov/Press-Room/Digital-Media-Center/Document-Gallery/igphoto/2003068942/)). As when using SAST on internally developed code, one must recognize that these tools can produce both false positives and false negatives. Thus, SAST results must not be accepted without manual verification and should not be interpreted as providing a comprehensive view of the project's security. However, as long as their limitations are understood, SAST scans can prove useful when analyzing both internally developed or OSS code.

##### Lockfile/Version Pinning

原典 L101-104。

To reduce the likelihood that a compromised or vulnerable version is unwittingly pulled into an application, one should limit the applications dependencies to a specific version that has been previously verified as legitimate and secure. This is commonly accomplished using lockfiles such as the package-lock.json file used by npm.

#### Build Threats

原典 L105-108。

The section below describes techniques that are especially relevant for securing build related threats.

##### Inventory Build Tools

原典 L109-112。

Knowing the components used in the SSC is essential to the security of that SSC. This concept extends to build tools. An inventory of all build tools, including versions and any plugins, should be automatically collected and mainlined, One must also monitor vulnerability databases, vendor security advisories and other sources for any vulnerabilities related to the identified build tools.

##### Harden Build Tools

原典 L113-121。

Compromised build tools can enable a wide range of exploits and thus represent an appealing target for attackers. As such, all infrastructure and tools used in build process must be hardened to mitigate risk. Techniques for hardening build environments include ([ESF developer practices](https://www.nsa.gov/Press-Room/Digital-Media-Center/Document-Gallery/igphoto/2003068942/)):

- Ensure build tools are located in an appropriately segregated networks.
- Use DLP and other tools and techniques to detect and prevent exfiltration.
- Disable/remove any unused services.
- Use version control systems to manage and store pipeline configurations.

##### Enforce Code Signing

原典 L122-125。

From a the perspective of software consumers, only accepting components which have been digitally signed and validating the signature before utilizing the software is an important task step in ensuring the component is authentic and has not been tampered with. For those performing code signing, it is imperative that the code signing infrastructure is thoroughly hardened. Failure to do so can result in compromise of the code signing system and lead further exploits, including those targeting consumers of the software.

##### Use Private Artifact Repository

原典 L126-129。

Using a private artifact repository increases the control an organization has over the various artifacts that are used within the SSC. Artifacts should be reviewed before being allowed in the private repository and organizations must ensure that usage of these repositories cannot be bypassed. Although usage of private repositories can introduce extra maintenance or reduce agility, they can also be an important component of SSCS, especially for sensitive or critical applications.

##### Use Source Control for Build Scripts and Config

原典 L130-133。

The benefits of VCSs can be realized for items beyond source control; this is especially true for config and scripts related to CI/CD pipelines. Enforcing version control for these files allows one to incorporate reviews, merge rules, and like controls into the config update process. Using VCS also increase visibility, allowing one easy visibility into any changes introduced, whether malicious or benign ([ESF developer practices](https://www.nsa.gov/Press-Room/Digital-Media-Center/Document-Gallery/igphoto/2003068942/)).

##### Verify Provenance/Ensure Sufficient Metadata is Generated

原典 L134-137。

Having assurance that an SSC component comes from a trusted source and has not been tampered with is a important part of SSCS. Generation and consumption of provenance, defined in [SLSA 1.0](https://slsa.dev/spec/v1.0/provenance) as "the verifiable information about software artifacts describing where, when and how something was produced" is an important part of this. The provenance should be generated by the build platform (as opposed to a local development system), be very difficult for attackers to forge, and contain all details necessary to accurately link the result back to the builder ([SLSA 1.0 provenance requirements](https://slsa.dev/spec/v1.0/requirements#provenance-generation)). SLSA 1.0 compliant provenance can be generated using builders such as [FRSCA](https://github.com/buildsec/frsca) or [Github Actions](https://github.com/slsa-framework/slsa-github-generator) and verified [using SLSA Verifier](https://github.com/slsa-framework/slsa-verifier?tab=readme-ov-file)

##### Ephemeral, Isolated Builds

原典 L138-141。

Reuse and sharing of build environments may allow attackers to perform cache poising or otherwise more readily inject malicious code.  Builds should be performed in isolated, temporary ("ephemeral") environments. This can be achieved using technologies such as VMs or containers for builds and ensuring the environment is immediately destroyed afterward.

##### Limit use of Parameters

原典 L142-145。

Although passing user controllable parameters to a build process can increase flexibility, it also increases risk. If parameters can be modified by users in order to alter how a build is performed, an attacker with sufficient permission will also be able to modify the parameters and potentially compromise the build process ([Google build safeguards](https://cloud.google.com/software-supply-chain-security/docs/safeguard-builds)). One should thus make an effort to minimize or eliminate any user controllable build parameters.

#### Deployment and Runtime Threats

原典 L146-149。

The section below outlines a couple of techniques that can be used to protect software during the deployment and runtime phases.

##### Scan Final Build Binary

原典 L150-153。

Once the build process has finished, one should not simply assume that the final result is secure. Binary composition analysis can help detect exposed secrets, detect unauthorized components or content, and verify integrity ([ESF developer practices](https://www.nsa.gov/Press-Room/Digital-Media-Center/Document-Gallery/igphoto/2003068942/)). This task should be performed by both suppliers and consumers.

##### Monitor Deployed Software for Vulnerabilities

原典 L154-157。

SSCS does not end with the deployment of the software; the deployed software must be monitored and maintained to reduce risk. New vulnerabilities, whether introduced due to an update or simply newly discovered (or made public), are a continual concern in software systems ([Google supply-chain threat categories](https://cloud.google.com/software-supply-chain-security/docs/attack-vectors)). When performing this monitoring, a wholistic approached must be used; code dependencies, container images, web servers, and operating system components are just a sampling of items that must be consider. To support this monitoring, an accurate and up-to-date inventory of system components is critical. Additionally, insecure configuration changes must be monitored and acted upon.
