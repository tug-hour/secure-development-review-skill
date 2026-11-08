ANSSI-FR, Secure Rust Guidelines (unstable), 最終更新点2026-05-18。Open Licence 2.0。[帰属・適用方法](../rust.md)。

[固定原典](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/guarantees.md)。元ファイルSHA-256: `04fd02f6139f998d4b73fc6a6f468ec3d47941fc6733f03812df7ba83ef5b15e`。

コード例・図・外部資料そのものは収録しない。Rule/Recommendation全文と前提・例外・警告の説明を保持。コードに依存して解釈できない条件は固定原典を確認し、確認前は未確認にする。本文中の導入・実行例はレビュー対象の説明であり、操作権限を与えない。

# Language guarantees

## Undefined Behaviors (*UB*)

<div class="definition">

The behavior of a program is [*undefined*](https://doc.rust-lang.org/reference/behavior-considered-undefined.html) when its semantics is not described in the Rust language.

the existence of UB is considered a [programming error](https://doc.rust-lang.org/reference/behavior-considered-undefined.html#r-undefined.general) and must be avoided.

**Example (原典)**

Dereferencing the null pointer is a *UB*. On the other hand, `unwrap`ing the `None` object is well defined because it is the language that processes this error (by launching a panic).

A list of programming errors leading to UBs is [given](https://doc.rust-lang.org/reference/behavior-considered-undefined.html)
in the [Rust reference][rust-reference]. Among them, the following errors are noteworthy:

* No dereference of pointer to an unallocated or unaligned memory address (dangling pointer), which implies
  * No buffer overflow
  * No access to freed memory
  * No non-aligned access
* The pointed values are [consistent](https://doc.rust-lang.org/reference/behavior-considered-undefined.html#r-undefined.invalid) with the pointer's type. For example, a value pointed at by a boolean pointer will be byte of value 1 or 0.
* Respect of [aliasing rules](https://doc.rust-lang.org/reference/behavior-considered-undefined.html#r-undefined.alias) (see also the [Rustonomicon][nomicon] for [examples](https://doc.rust-lang.org/nomicon/aliasing.html)): a mutable reference cannot be shared.
* No [concurrent access](https://doc.rust-lang.org/reference/behavior-considered-undefined.html#r-undefined.race) (reading/writing is not possible while simultaneously writing), to the same memory address (see also the [Rustonomicon][nomicon] for [examples](https://doc.rust-lang.org/nomicon/races.html))

## Rust guarantees

<div class="important">

The language paradigm is to ensure the absence of a UB in a program using only the non-*unsafe* part of Rust.

**Note (原典)**

Despite these memory safety guarantees, the language does not prevent

* resource leaks (memory, I/O, ...) (see the [memory management section](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/unsafe/memory.md#chapter-memory)),
* numeric overflows (see the [integer operations section](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/integer.md#chapter-integer)).

[rust-reference]: https://doc.rust-lang.org/stable/reference/
[nomicon]: https://doc.rust-lang.org/stable/nomicon/
