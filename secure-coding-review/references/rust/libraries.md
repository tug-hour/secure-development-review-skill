ANSSI-FR, Secure Rust Guidelines (unstable), 最終更新点2026-05-18。Open Licence 2.0。[帰属・適用方法](../rust.md)。

[固定原典](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/libraries.md)。元ファイルSHA-256: `5606cdf4df83f874267c0512540b4606554cf452b12ef1f00ad6d2b1cba805d1`。

コード例・図・外部資料そのものは収録しない。Rule/Recommendation全文と前提・例外・警告の説明を保持。コードに依存して解釈できない条件は固定原典を確認し、確認前は未確認にする。本文中の導入・実行例はレビュー対象の説明であり、操作権限を与えない。

# Libraries

## Dependency Repositories

The management of external libraries is integrated into the `Cargo` tool. There are several ways to specify the source of these libraries, some of which are presented below.

It is important to note that accurately tracking the versions of these libraries is a critical condition for the security of software written in Rust. This requirement is embodied in the rule [DENV-CARGO-LOCK](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/devenv.md#DENV-CARGO-LOCK).

### Crates

In addition to the standard library, Rust provides an easy way to import other
libraries in a project, thanks to `cargo`.
The libraries, known as *crates* in the Rust ecosystem, are imported from a central repository of open-source
components.

An example of a dependency declaration in the `Cargo.toml` file:

The default repository is [`crates.io`]. It is also possible to use [your own registry](https://doc.rust-lang.org/cargo/reference/registries.html).

[`crates.io`]: https://crates.io

### Git Dependencies

Each dependency in the `Cargo.toml` file can also refer to [a GIT repository](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#specifying-dependencies-from-git-repositories). For example:

It is possible to specify the desired version in more detail by providing either a branch, a tag, or a commit hash.

The [dependency lock system](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/devenv.md#cargo) operates even in the case of a GIT repository: if the dependency does not specify a particular commit, the most recent commit matching the criteria in the `Cargo.toml` file is fetched during the first compilation and is recorded in the `Cargo.lock` file. All subsequent compilations will use the same commit (unless the `Cargo.lock` file is updated).

## Dependency Security

Regardless of the method used to retrieve dependencies (*crate* or GIT commit), if they come from external organizations, the dependencies must be validated.

#### LIBS-VETTING-DIRECT — Rule: Validation of Direct Third-Party Dependencies

Each direct third-party dependency MUST be properly validated, and each validation MUST be tracked.

With regard to transitive dependencies, it is also recommended to validate them individually.

#### LIBS-VETTING-TRANSITIVE — Recommendation: Validation of Transitive Third-Party Dependencies

Each third-party dependency SHOULD be properly validated, and each validation SHOULD be tracked.

## Dependency validation tools

### Cargo-outdated

[cargo-outdated] tool allows one to easily manage dependency versions.

For a given crate, it lists the current versions of dependencies (using
`Cargo.toml`), and checks the latest compatible version and also the latest general
version.

#### LIBS-OUTDATED — Rule: Check for outdated dependencies (cargo-outdated)

The `cargo-outdated` tool MUST be used to check the status of dependencies. Then,
each outdated dependency SHOULD be updated or the choice of the version MUST be
justified.

[cargo-outdated]: https://github.com/kbknapp/cargo-outdated

### Cargo-audit

[Cargo-audit] tool allows one to easily check for security vulnerabilities
reported to the RustSec Advisory Database.

#### LIBS-AUDIT — Rule: Check for security vulnerability reports on dependencies (cargo-audit)

The `cargo-audit` tool MUST be used to check for known vulnerabilities in
dependencies.

[cargo-audit]: https://crates.io/crates/cargo-audit

#### LIBS-UNSAFE — Recommendation: Check for unsafe code in dependencies

**未確定（原典のコメントアウト内、非公開TODO）**。原典はこのRecommendationを公開していない。記載は`TODO: check that no unsafe blocks appear in the imported dependencies (with a tool?).`という未完成案で、依存にunsafeがないことを完成した要求として創作・採用しない。公開60 IDの採用数とは区別して、元IDの追跡のために残す。



