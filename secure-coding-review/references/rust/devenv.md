ANSSI-FR, Secure Rust Guidelines (unstable), 最終更新点2026-05-18。Open Licence 2.0。[帰属・適用方法](../rust.md)。

[固定原典](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/devenv.md)。元ファイルSHA-256: `ea3d76c12bf1add44a56f50b873a5ab03b28d9570031f28c95d03859b0c162dd`。

コード例・図・外部資料そのものは収録しない。Rule/Recommendation全文と前提・例外・警告の説明を保持。コードに依存して解釈できない条件は固定原典を確認し、確認前は未確認にする。本文中の導入・実行例はレビュー対象の説明であり、操作権限を与えない。

# Development environment

## Rustup

[Rustup] is the Rust toolchain installer. Among other things, it enables
switching between different flavors of the toolchain (stable, beta, nightly),
managing the installation of additional components and keeping them up to date.

**Warning (原典)**

From a security perspective, `rustup` does perform all downloads over HTTPS,
but does not yet validate signatures of downloads. Protection against
downgrade attacks, certificate pinning, and validation of signatures are still
works in progress.
In some cases, it may be preferable to opt for an alternative installation
method listed in the *Install* section of the official Rust website.

[rustup]: https://github.com/rust-lang/rustup.rs

### Rust Editions

Several flavors, called *editions*, of the Rust language coexist.

The concept of editions has been introduced to clarify the
implementation of new features and to make them incremental. A new edition will be produced
every two or three years, as stated in the [Rust Manual][rust-edition-guide], but this doesn’t
mean that new features and improvements will only be shipped in a new edition.

Some editions bring new keywords and language constructs. Recommendations for
secure applications development then remain closely linked to features of the
language, that are used in such applications, rather than to Rust editions.
In the rest of this guide, best effort will be made to highlight constructions
and language features that are specific to a particular Rust edition.

**Note (原典)**

No specific edition is recommended, as long as users follow the
recommendations related to the features offered by the edition that has been
chosen.

[edition guide]: https://doc.rust-lang.org/edition-guide/

### Stable, nightly and beta toolchains

Independently from the choice of edition that allows one to select a flavor (a set of features)
of the Rust language, the Rust toolchain is provided in three different
versions, called *release channels*:

- *nightly* releases are created once a day,
- *beta* releases are created every six weeks, from promoted *nightly* releases,
- *stable* releases are created every six weeks, from promoted *beta* releases.

When playing with different toolchains, it is important to check not only what
the default toolchain is, but also if overrides are currently set for some
directories.

**Example (原典)**

This example shows that toolchain `stable-x86_64-unknown-linux-gnu` is the default toolchain,
but is overridden in directory `/tmp/foo` by `nightly-x86_64-unknown-linux-gnu`.

#### DENV-STABLE — Rule: Use a stable compilation toolchain

Development of a secure application MUST be done using a fully stable
toolchain, for limiting potential compiler, runtime or tool bugs.

When using a specific `cargo` subcommand that requires a nightly component,
it is preferable to run it by switching the toolchain only locally, instead
of explicitly switching the complete toolchain. 

**Example (原典)**

For example, to run the (nightly) latest `rustfmt`:

### Rustc Tier guarantees

As Rustc's backend is based on [LLVM], it inherits and classifies its supported targets into tiers to communicate how much stability and testing each backend receives.

[LLVM]: https://llvm.org/

**Example (原典)**

The list of Rust's targets is given by the following command.

Here targets `wasm32-unknown-unknown` and `x86_64-unknown-linux-gnu` are installed.

#### Tier 1 - guaranteed to work

The target is fully vetted by the community. It passes the entire test suite, receives regular regression testing, and is kept up‑to‑date with new releases. In practice, you can rely on consistent code generation, stable ABI[^abi], and predictable performance. Tier 1 targets can be thought of as "guaranteed to work".

[^abi]: Application Binary Interface

#### Tier 2 - guaranteed to build

The target’s source code compiles cleanly, but it does not receive the same level of testing or maintenance as Tier 1. It may lack full coverage in the test suite, and certain optimizations or newer [LLVM] features could be missing or unstable. Users can still generate code for these backends, but they should expect occasional breakage or the need for manual patches. Tier 2 targets can be thought of as "guaranteed to build" but not as "guaranteed to work".

#### Tier 3

Tier 3 targets are simply not officially supported.

The tier distinction helps developers choose a target that matches their risk tolerance: Tier 1 for production‑grade workloads, Tier 2 for experimental or niche architectures where full support isn’t yet met.

#### DENV-TIERS — Rule: Exclusive use of tier 1 of `rustc` for safety-critical software

Rustc tier 1 targets and certified toolchains MUST be used for safety-critical systems.

A comprehensive list of supported targets is available in the [Rust Book][rustc-book].

## Cargo

Once Rustup has set up the appropriate Rust toolchain, [Cargo] is available
through the command line program `cargo`. Cargo is the Rust package manager.
It has a fundamental role in most Rust developments:

- It structures the project by providing the project's skeleton (`cargo new`),
- It compiles the project (`cargo build`),
- It generates the project's documentation (`cargo doc`),
- It runs tests (`cargo test`) and benchmarks (`cargo bench`),
- It manages and downloads dependencies,
- It makes packages distributable and publishes them on [`crates.io`],
- It’s also a front-end to run complementary tools such as those that are
  described below, in the form of sub-commands.

Cargo enables automatic dependency resolution before compilation 
by checking their checksums.
The `Cargo.lock` file contains the checksums of all dependencies, which are compared to
the ones downloaded.
If a difference is detected, compilation fails:

When the `Cargo.lock` file is missing, it is generated on first build and records the
dependencies' checksums, adhering to a TOFU (*Trust On First Use*) policy.
This file can also be created manually with `cargo generate-lockfile`. If it already exists,
the file is overwritten with the latest available version of every crate.

#### DENV-CARGO-LOCK — Rule: Track Cargo.lock in version control system

`Cargo.lock` files MUST be tracked by a version control system.

**Warning (原典)**

Ongoing discussions occur on how to best protect
and verify crates *on their first download* (according to the TOFU rule).
For now, the security of the first download relies on the good security of the
website [`crates.io`] and the GitHub hosted repository containing the
registry index. In some cases, it may be preferable to opt for an alternative
installation method for dependencies.

Cargo proposes many different commands and options to adapt the build process to
your project needs, mainly through the manifest file `Cargo.toml`. For a
complete presentation, see the [Cargo Book][cargo-book].

During the development of a secure application, some of the options may require
some attention. 

Cargo provides the notion of *profiles*. Four profiles are builtin: `dev`, `release`, `test` and `bench`.
They are applied accordingly with `cargo` commands.

For instance, `cargo build` uses `dev` profile, and `cargo build --release` or `cargo install`
use `release` profile.

Profiles can be created/altered in `[profile.*]` sections. For example:

- the [`debug-assertions`] variable controls whether debug assertions are enabled,
- the [`overflow-checks`] variable controls whether overflows are checked for
  integer arithmetic.

Overriding the default options may cause bugs not being detected, even when
using the `dev` profile that normally enables runtime checks (for example it does not enable
[integer overflow checks](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/integer.md#chapter-integer)).

#### DENV-CARGO-OPTS — Rule: Keep default values for critical variables in cargo profiles

The variables [`debug-assertions`] and [`overflow-checks`] MUST NOT be overridden
in development profiles' sections (`[profile.dev]` and `[profile.test]`).

Cargo proposes other ways to set up its configuration and change its behavior on
a given system. This can be very useful, but it may also be difficult to know
and remember at a given time all the options that are effectively used, and
in particular passed to the compiler. At the end, this can affect the confidence
and robustness of the build process. It is preferable to centralize compiler
options and flags in the configuration file `Cargo.toml`. For the case of
environment variable `RUSTC_WRAPPER`, for example, that may be used to generate
part of the code or to run external tools before Rust compilation, it is preferable
to use the Cargo build scripts feature.

#### DENV-CARGO-ENVVARS — Rule: Keep default values for compiler environment variables when running cargo

The environment variables `RUSTC`, `RUSTC_WRAPPER` and `RUSTFLAGS` MUST NOT
be overridden when using Cargo to build the project.

[`crates.io`]: https://crates.io
[cargo]: https://doc.rust-lang.org/stable/cargo/
[`debug-assertions`]: https://doc.rust-lang.org/cargo/reference/profiles.html#debug-assertions
[`overflow-checks`]: https://doc.rust-lang.org/cargo/reference/profiles.html#overflow-checks

### Rustfmt

Included with Rust, [Rustfmt] is a tool that formats your code according to style guidelines.

These guidelines can be customized to your needs by creating a `rustfmt.toml` or
`.rustfmt.toml` file at the root of your project. It will be used to override
the default settings, for instance:

For more information about the guidelines that `rustfmt` will check, have a look
at the [Rust Style Guidelines Book][rust-style].

#### DENV-FORMAT — Recommendation: Use Rust formatter (rustfmt)

The tool `rustfmt` SHOULD be used to ensure that the codebase respects style
guidelines.

[rustfmt]: https://github.com/rust-lang/rustfmt

### Cargo fix

The [`cargo fix`] command is a tool dedicated to
fixing compiler warnings as well as easing transitions between editions.

To prepare a Rust 2021 project to transition to Rust 2024, one can run:

Rustfix will either fix the code to be compatible with Rust 2024 or print a
warning that explains the problem. This problem will have to be fixed manually.
By running the command (and possibly fixing manually some issues) until there
is no warning, one can ensure the code is compatible with both Rust 2021 and
Rust 2024.

To switch definitely to Rust 2024, one may run:

Be advised that this tool provides few guarantees on the soundness of the
proposed fixes. In particular, some corrections (such as some of those
provided with the `--edition-idioms`) are known to break the compilation
or change the program semantics in some cases.

[`cargo fix`]: https://doc.rust-lang.org/cargo/commands/cargo-fix.html

### Clippy

[Clippy] is a tool that provides and checks many lints (bugs, styling, performance
issues, etc.). Since version 1.29, `clippy` can be used within the stable
`rustup` environment. It is recommended to install `clippy` as a component
(`rustup component add clippy`) in the stable toolchain instead of installing it
as a project dependency.

The tool comes with some lint categories regarding the kind of issues it aims to
detect. The warnings should be re-checked by the programmer before committing
the fix that is suggested by `clippy`, especially in the case of lints of the
category `clippy::nursery` since those hints are still under development.

`clippy` now has a `fix` tool similar to `cargo fix`.

[clippy]: https://github.com/rust-lang/rust-clippy

#### DENV-LINTER — Rule: Use linter regularly

A linter, such as `clippy`, MUST be used regularly during the development of
a secure application.

#### DENV-AUTOFIX — Rule: Manually check automatic fixes

In a secure Rust development, any automatic fix (for instance, provided by
`rustfix` or `clippy`) MUST be verified by the developer.

### Others

There exist other useful tools or `cargo` subcommands for enforcing program
security whether by searching for specific code patterns or by providing
convenient commands for testing or fuzzing. They are discussed in the following
chapters, according to their goals.

[rust-edition-guide]: https://doc.rust-lang.org/edition-guide/
[cargo-book]: https://doc.rust-lang.org/cargo/index.html
[rust-style]: https://doc.rust-lang.org/style-guide/index.html
[rustc-book]: https://doc.rust-lang.org/stable/rustc/index.html
