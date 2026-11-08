ANSSI-FR, Secure Rust Guidelines (unstable), 最終更新点2026-05-18。Open Licence 2.0。[帰属・適用方法](../rust.md)。

[固定原典](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/unsafe/generalities.md)。元ファイルSHA-256: `44aefd158dabff1e859913b143f99612013cd54a5dc112859b7240e8fee02ae3`。

コード例・図・外部資料そのものは収録しない。Rule/Recommendation全文と前提・例外・警告の説明を保持。コードに依存して解釈できない条件は固定原典を確認し、確認前は未確認にする。本文中の導入・実行例はレビュー対象の説明であり、操作権限を与えない。

# General information on `unsafe`

## *Unsafe* operations

Language capabilities can be extended using unsafe code. The full list of these features is given in the [Rust reference](https://doc.rust-lang.org/reference/unsafety.html).
These capabilities may be necessary for system programming but they cause the language to lose its [safety properties](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/guarantees.md#language-guarantees) and Undefined Behaviors may happen.

#### UNSAFE-NOUB — Rule: No Undefined Behavior

NO Undefined Behavior is allowed.

## A keyword with two usages

As described in the [Rust Reference][rust-reference], the `unsafe` keyword is used both for marking unsafety in an API and unlocking unsafety in the implementation.

### `unsafe` marking

Marking with `unsafe` is a delegation of responsibility with respect to memory safety from the API author to the API user.
The use of this keyword in an API *warns* the API user about the potential harmful effects of using the API.

* In a function signature (section [`r-unsafe.fn`] of the [Rust Reference][rust-reference]), `unsafe` means that the behavior of the function may lead to UB if the use of the function does not comply with its interface contract (informally described in its documentation).
* In a trait declaration (section [`r-unsafe.trait`] of the [Rust Reference][rust-reference]), `unsafe` means that an erroneous implementation of this trait may lead to UB if the implementation contract (preferably documented) is not respected.

[`r-unsafe.fn`]: <https://doc.rust-lang.org/reference/unsafe-keyword.html#r-unsafe.fn>
[`r-unsafe.trait`]: <https://doc.rust-lang.org/reference/unsafe-keyword.html#r-unsafe.trait>

### `unsafe` unlocking

Unlocking with `unsafe` means taking responsibility for memory safety from the compiler to the developer.

Using an `unsafe` block in a function body or in a constant declaration is imposed by the compiler (section [`r-unsafe.block`] of the [Rust Reference][rust-reference]) to prevent the *inadvertent use* of `unsafe` capabilities like

* using `unsafe` tagged functions
* modifying static variables
* using `extern` functions

Similarly, the implementation of an `unsafe` trait requires `unsafe` (section [`r-unsafe.impl`] of the [Rust Reference][rust-reference]) for the developer to explicitly take into account the memory safety contracts. The keyword `unsafe` *unlocks* the implementation of `unsafe` traits.

Lastly, Since the 2024 edition, `unsafe` is also required to unlock the following:

* `extern` blocks (section [`r-unsafe.extern`] of the [Rust Reference][rust-reference]), which contain declarations of foreign functions and variables, for [FFI](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/unsafe/ffi.md#chapter-ffi),
* some attributes (for instance , no_mangle, cf. [`r-attributes.safety`] of the [Rust Reference][rust-reference]).

[`r-unsafe.impl`]: <https://doc.rust-lang.org/reference/unsafe-keyword.html#r-unsafe.impl>
[`r-unsafe.block`]: <https://doc.rust-lang.org/reference/unsafe-keyword.html#r-unsafe.block>
[`r-unsafe.extern`]: <https://doc.rust-lang.org/reference/unsafe-keyword.html#r-unsafe.extern>
[`r-attributes.safety`]: <https://doc.rust-lang.org/reference/attributes.html#r-attributes.safety>

## Limitations and precautions

Paraphrasing the [Rustonomicon](https://doc.rust-lang.org/nomicon/), the fundamental principle of Rust can be summed up as follows:

<div class="important">

`unsafe`-free code cannot go wrong

The combined use of the type system and the ownership system enforces a high-level memory safety in Rust programs. This way, the language helps prevent memory overflows, null or invalid pointer constructions, and data races.

This promise is valid only if the code does not use `unsafe` features. When `unsafe` features are used, the compiler can no longer guarantee memory safety. The developer must then ensure that the code respects the invariants that guarantee memory safety.

That is why it is crucial to limit the use of `unsafe` features as much as possible:

#### LANG-UNSAFE — Rule: Don't use unsafe blocks

In a secure Rust development, the `unsafe` blocks SHOULD be avoided, or MUST be justified by at least one of the following points:

* The Foreign Function Interface (FFI) of Rust allows for describing functions whose implementations are written in C, using the `extern "C"` prefix. To use such a function, the `unsafe` keyword is required. “Safe” wrapper shall be defined to safely and seamlessly call C code.

* For embedded device programming, registers and various other resources are often accessed through a fixed memory address. In this case, `unsafe` blocks are required to initialize and dereference those particular pointers in Rust. In order to minimize the number of unsafe accesses in the code and to allow easier identification of them by a programmer, a proper abstraction (data
structure or module) shall be provided.

* When hitting a performance wall on a small portion of code (E.G: Zero-copy buffer modified in-place, Allocation overhead, etc.).

With the exception of these cases, `#![forbid(unsafe_code)]` must appear in the crate root (typically `main.rs` or `lib.rs`) to generate compilation errors if `unsafe` is used in the code base.

If the use of `unsafe` is necessary, it is the responsibility of the developer to:

* ensure that the use of `unsafe` unlocking does not lead to *UB*s,
* ensure that any `unsafe` markings are correctly and exhaustively documented so that no *UB* are possible if the usage conditions (invariants) are respected.

Aside from the `unsafe` code itself, it is also crucial to properly encapsulate the use of `unsafe` features in a component (crate or module) so as to restore the usual Rust memory safety guarantees:

#### LANG-UNSAFE-ENCP — Rule: Encapsulation of *unsafe* features

In secure development of a Rust software component (crate or module), all
*unsafe* code MUST be encapsulated in such a way that:

* either it exposes a safe behavior to the user, in which no safe interaction
  can result in UB (undefined behavior);
* or it exposes features marked as unsafe whose usage conditions
  (preconditions, sequencing, etc.) are exhaustively documented.

Thus, a function using `unsafe` operations can be *safe* if the `unsafe`
operations do not present any *UB* (undefined behavior) given the component's
invariants (typically the type invariant for a method). Conversely, a function
without an `unsafe` block must be marked as `unsafe` if it breaks these
invariants. The choice and knowledge of these invariants are therefore crucial
for secure development.

## Example: Preserving a type invariant

The following code comes from the [Rustonomicon](https://doc.rust-lang.org/nomicon/working-with-unsafe.html).
It could be used to implement a custom `Vec` type.

Soundness and safety of this code rely on the fact that bytes from address `self.ptr` to `self.ptr + self.cap * size_of<T>()` are allocated.

This invariant can be broken with *safe* code. For instance

This function may be necessary for internal use, but it should not be exposed in the API, or it should be marked with the `unsafe` keyword, because its use can lead to UB.

[rust-book]: https://doc.rust-lang.org/stable/book/
[rust-reference]: https://doc.rust-lang.org/stable/reference/
