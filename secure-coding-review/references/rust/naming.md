ANSSI-FR, Secure Rust Guidelines (unstable), 最終更新点2026-05-18。Open Licence 2.0。[帰属・適用方法](../rust.md)。

[固定原典](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/naming.md)。元ファイルSHA-256: `58ad9c75257ffb5601086a669616c084a73c71bad2702b2c5ce2234657297d60`。

コード例・図・外部資料そのものは収録しない。Rule/Recommendation全文と前提・例外・警告の説明を保持。コードに依存して解釈できない条件は固定原典を確認し、確認前は未確認にする。本文中の導入・実行例はレビュー対象の説明であり、操作権限を与えない。

# Naming

The standard library serves as the de facto standard for naming conventions in Rust.
An effort has been made to formalize the conventions through the [RFC 43][RFC-430] and later in the [Rust API Guidelines][rust-guidelines].

The basic rule [`(C-CASE)`] consists in using:

- `UpperCamelCase` for types, traits, enum variants, and generic type parameters,
- `snake_case` for functions, methods, macros, variables, and modules,
- `SCREAMING_SNAKE_CASE` for statics, constants, and generic constant parameters,
- `'lowercase` for lifetimes.

The [Rust API Guidelines][rust-guidelines] also prescribes more precise naming conventions for
particular constructs:

- [`(C-CONV)`] for conversion methods (`as_`, `to_`, `into_`),
- [`(C-GETTER)`] for getters,
- [`(C-ITER)`] for iterator-producing methods,
- [`(C-ITER-TY)`] for iterator types,
- [`(C-FEATURE)`] for feature naming (conditionally enabled functionalities),
- [`(C-WORD-ORDER)`] for word order consistency.

**Note (原典)**

The basic rule [`(C-CASE)`] is checked by the compiler (with the `nonstandard_style` lint set).

In addition to the compiler, the [`clippy`](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/devenv.md#clippy) tool can help in adopting naming conventions with the `style` lint category.
For example, the [`wrong_self_convention`](https://rust-lang.github.io/rust-clippy/master/index.html#wrong_self_convention) lint checks the consistency between conversion method names and their receiver types (`self`, `&self`, `&mut self`) according to [`(C-CONV)`].

#### LANG-NAMING — Rule: Respect naming conventions

Development of a secure application MUST follow the naming conventions
outlined in the [Rust API Guidelines][rust-guidelines].

[`(C-CASE)`]: https://rust-lang.github.io/api-guidelines/naming.html#casing-conforms-to-rfc-430-c-case
[`(C-CONV)`]: https://rust-lang.github.io/api-guidelines/naming.html#ad-hoc-conversions-follow-as_-to_-into_-conventions-c-conv
[`(C-GETTER)`]: https://rust-lang.github.io/api-guidelines/naming.html#getter-names-follow-rust-convention-c-getter
[`(C-ITER)`]: https://rust-lang.github.io/api-guidelines/naming.html#methods-on-collections-that-produce-iterators-follow-iter-iter_mut-into_iter-c-iter
[`(C-ITER-TY)`]: https://rust-lang.github.io/api-guidelines/naming.html#iterator-type-names-match-the-methods-that-produce-them-c-iter-ty
[`(C-FEATURE)`]: https://rust-lang.github.io/api-guidelines/naming.html#feature-names-are-free-of-placeholder-words-c-feature
[`(C-WORD-ORDER)`]: https://rust-lang.github.io/api-guidelines/naming.html#names-use-a-consistent-word-order-c-word-order

[RFC-430]: https://rust-lang.github.io/rfcs/0430-finalizing-naming-conventions.html
[rust-guidelines]: https://rust-lang.github.io/api-guidelines/
