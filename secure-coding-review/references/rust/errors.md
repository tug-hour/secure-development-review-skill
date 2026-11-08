ANSSI-FR, Secure Rust Guidelines (unstable), 最終更新点2026-05-18。Open Licence 2.0。[帰属・適用方法](../rust.md)。

[固定原典](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/errors.md)。元ファイルSHA-256: `dbaad11513deb2c10294f18c9d94c82102f9ebaf006f96ea87b2b5447014e91c`。

コード例・図・外部資料そのものは収録しない。Rule/Recommendation全文と前提・例外・警告の説明を保持。コードに依存して解釈できない条件は固定原典を確認し、確認前は未確認にする。本文中の導入・実行例はレビュー対象の説明であり、操作権限を与えない。

# Error handling

The `Result` type is the preferred way of handling functions that can fail.
A `Result` object must be tested, and never ignored.

#### LANG-ERRWRAP — Recommendation: Implement custom `Error` type wrapping all possible errors

A crate may implement its own `Error` type, wrapping all possible errors.
It MUST be careful to make this type exception-safe ([RFC 1236][RFC-1236]), and implement
`Error + Send + Sync + 'static` as well as `Display`.

Third-party crates may be used to facilitate error handling. Most of them
(notably [snafu] or [thiserror]) address the creation of new custom
error types that implement the necessary traits and allow wrapping other
errors.

Another approach (notably proposed in the [anyhow] crate) consists in an automatic
wrapping of errors into a single universal error type. Such wrappers should not
be used in libraries and complex systems because they do not allow developers to
provide context to the wrapped error.

[snafu]: https://crates.io/crates/snafu
[thiserror]: https://crates.io/crates/thiserror
[anyhow]: https://crates.io/crates/anyhow

## Panics

Explicit error handling (`Result`) should generally be preferred instead of calling
`panic`.  The cause of the error should be available, and generic errors should
be avoided.

#### LANG-LIMIT-PANIC — Rule: Limit `panic` use

A Rust function MUST NOT panic UNLESS its usage conditions have been violated.

Crates providing libraries should never use functions or instructions that can
fail and cause the code to panic.

Following patterns can cause panics (by design):

- using `unwrap` or `expect`,
- using `assert!`.

The following rule is deduced from the [previous rule](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/errors.md#LANG-LIMIT-PANIC).

#### LANG-LIMIT-PANIC-SRC — Rule: Limit use of `panic`-ing functions

Uses of `unwrap`, `expect` and `assert!` MUST be restricted to cases explicitly forbidden by the function’s specification.

The following functions are known to panic if their arguments do not meet the usage conditions.

- an unchecked access to an array (see [next](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/errors.md#LANG-ARRINDEXING) rule),
- integer overflow (in debug mode, see chapter on [integers](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/integer.md#chapter-integer)),
- division by zero,
- large allocations,
- string formatting using `format!`.

#### LANG-ARRINDEXING — Rule: Test properly array indexing or use the `get` method

Array indexing must be properly tested, or the `get` method SHOULD be used to
return an `Option`.

**Warning (原典)**

In certain safety‑critical domains, it is mandatory to transition to a safe‑mode state whenever an error occurs that could otherwise lead to undefined behavior.
In these situations, deliberately aborting execution makes sense because it stops the system before corrupted data or safety faults can propagate.

For a plane or other vehicles, this “fail‑fast” behavior can be crucial: the primary control unit must halt immediately on a serious fault, then hand over control to a redundant or backup subsystem that can bring the vehicle to a safe stop or continue operation in a reduced‑capability mode. Restarting on a trusted secondary system ensures that the plane remains controllable, protects occupants, and prevents hazardous outcomes that could arise from continuing execution in an unpredictable state.

For this use case, enabling the `panic = 'abort'` attribute in the `[profile.release]` section of the Cargo.toml file will terminate the program as soon as a panic occurs.

## FFI and panics

The integration of other languages into Rust is achieved through an *FFI* (*Foreign Function Interface*).
This feature is described in more detail in the [dedicated chapter](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/unsafe/ffi.md#ffi-panic).

[RFC-1236]: https://rust-lang.github.io/rfcs/1236-stabilize-catch-panic.html
