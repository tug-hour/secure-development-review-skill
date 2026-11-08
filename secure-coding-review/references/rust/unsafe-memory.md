ANSSI-FR, Secure Rust Guidelines (unstable), 最終更新点2026-05-18。Open Licence 2.0。[帰属・適用方法](../rust.md)。

[固定原典](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/unsafe/memory.md)。元ファイルSHA-256: `5ea8d8d1a8e258565a2c7723e86fd52bff89930f9e06aef5b1b0869592f9789e`。

コード例・図・外部資料そのものは収録しない。Rule/Recommendation全文と前提・例外・警告の説明を保持。コードに依存して解釈できない条件は固定原典を確認し、確認前は未確認にする。本文中の導入・実行例はレビュー対象の説明であり、操作権限を与えない。

# Memory management {#chapter-memory}

In the vast majority of cases, in non-`unsafe` Rust (i.e., code that does not use `unsafe`), the compiler **automatically** determines when it can release the memory occupied by a value in the program.
But, as [noted earlier](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/guarantees.md#rust-guarantees), this is not
a guarantee: code without `unsafe` can still lead to memory leaks. Therefore, some of the
rules presented in this chapter are not strictly related to the `unsafe` keyword. However,

<div class="important">

Even if a function in the following is not `unsafe`,
it should only be used in Rust *unsafe*.

As a general rule, memory leaks must be avoided.

#### MEM-NO-LEAK — Rule: No memory leak

In a secure Rust development, NO memory leak is allowed.

## [`mem::forget`] and memory leaks {#forget-and-memory-leaks}

While the usual way for memory to be reclaimed is for a variable to go out of
scope, Rust provides special functions to manually reclaim memory: [`mem::forget`] and
[`mem::drop`] of the `std::mem` module (or `core::mem`). While [`mem::drop`] simply triggers
an early memory reclamation that calls associated destructors when needed,
[`mem::forget`] skips any call to the destructors.

Both functions are **memory safe** in Rust. However, [`mem::forget`] will make any
resource managed by the value unreachable and unclaimed.

In particular, using [`mem::forget`] may result in not releasing critical resources,
leading to deadlocks or not erasing sensitive data from the memory. This is why
[`mem::forget`] is **unsecure**.

#### MEM-FORGET — Rule: Do not use `mem::forget`

In a secure Rust development, the [`mem::forget`] function of `std::mem`
(`core::mem`) MUST NOT be used.

#### MEM-FORGET-LINT — Recommendation: Use clippy lint to detect use of `mem::forget`

The lint `mem_forget` of Clippy SHOULD be used to automatically detect any use of
[`mem::forget`]. To enforce the absence of [`mem::forget`] in a crate, add the following
line at the top of the root file (usually `src/lib.rs` or `src/main.rs`):

`#![deny(clippy::mem_forget)]`

The standard library includes other way to *forget* dropping values:

- [`Box::leak`] to leak a resource,
- [`Box::into_raw`] to exploit the `Box` as a **pointer** in some unsafe code, notably in FFI,
- [`ManuallyDrop`] (in `std::mem` or `core::mem`) to enforce manual release of some value.

Those alternatives may lead to the same security issue but they have the
additional benefit of making their goal obvious.

#### MEM-LEAK — Rule: Do not use `leak` function

In a secure Rust development, the code MUST NOT leak memory or resource in
particular via [`Box::leak`].

[`ManuallyDrop`] and [`Box::into_raw`] shift the release responsibility from the
compiler to the developer.

#### MEM-MANUALLYDROP — Rule: Do release value wrapped in `ManuallyDrop`

In a secure Rust development, any value wrapped in [`ManuallyDrop`] MUST be
unwrapped to allow for automatic release ([`ManuallyDrop::into_inner`])
or manually released (unsafe [`ManuallyDrop::drop`]).

[`mem::forget`]: https://doc.rust-lang.org/std/mem/fn.forget.html
[`mem::drop`]: https://doc.rust-lang.org/std/mem/fn.drop.html
[`Drop`]: https://doc.rust-lang.org/std/ops/trait.Drop.html
[`Box::leak`]: https://doc.rust-lang.org/std/boxed/struct.Box.html#method.leak
[`Box::into_raw`]: https://doc.rust-lang.org/std/boxed/struct.Box.html#method.into_raw
[`ManuallyDrop`]: https://doc.rust-lang.org/beta/std/mem/struct.ManuallyDrop.html
[`ManuallyDrop::into_inner`]: https://doc.rust-lang.org/beta/std/mem/struct.ManuallyDrop.html#method.into_inner
[`ManuallyDrop::drop`]: https://doc.rust-lang.org/beta/std/mem/struct.ManuallyDrop.html#method.drop

## Raw pointers

These pointers are mainly used for C pointers. They do not have the same protections
as *smart pointers* and often have to be used in `unsafe` context. For instance, freeing 
raw pointers must be done manually without Rust guaranties.

#### MEM-NORAWPOINTER — Rule: Do no convert smart pointer into raw pointer in Rust without `unsafe`

In a secure Rust development without `unsafe`, references and *smart pointers*
SHOULD NOT be converted into *raw pointers*. For instance, functions `into_raw` or `into_non_null`
of smart pointers [`Box`], [`Rc`], [`Arc`], [`rc::Weak`] or [`sync::Weak`] SHOULD NOT be used.

The use of *raw pointers* without `unsafe` code MUST otherwise be documented and justified.

#### MEM-INTOFROMRAWALWAYS — Rule: Always call `from_raw` on `into_raw`ed value

In a secure Rust development, any pointer created with a call to `into_raw`
(or `into_non_null`) from one of the following types:

- [`std::boxed::Box`] (or [`alloc::boxed::Box`]),
- [`std::rc::Rc`] (or [`alloc::rc::Rc`]),
- [`std::rc::Weak`] (or [`alloc::rc::Weak`]),
- [`std::sync::Arc`] (or [`alloc::sync::Arc`]),
- [`std::sync::Weak`] (or [`alloc::sync::Weak`]),
- [`std::ffi::CString`],
- [`std::ffi::OsString`].

MUST eventually be transformed into a value with a call to the respective
`from_raw` to allow for their reclamation.

The converse is also true! That is, `from_raw` should be call **only** on `into_raw`ed value. For instance,
[`Rc`] smart pointers [explicitly request for this condition](https://doc.rust-lang.org/std/rc/struct.Rc.html#method.from_raw)
and, for [`Box`] smart pointers, conversion of C pointers into [`Box`] is [discouraged](https://doc.rust-lang.org/std/boxed/index.html#memory-layout).

#### MEM-INTOFROMRAWONLY — Rule: Call `from_raw` *only* on `into_raw`ed value

In a secure Rust development, `from_raw` MUST ONLY be called on `into_raw`ed values.

**Note (原典)**

In the case of [`Box::into_raw`], manual cleanup is possible but a lot more
complicated than re-boxing the raw pointer and should be avoided:

Because the other types ([`Rc`] and [`Arc`]) are opaque and more complex, manual
cleanup is not possible.

[`Box`]: https://doc.rust-lang.org/std/boxed/struct.Box.html
[`std::boxed::Box`]: https://doc.rust-lang.org/std/boxed/struct.Box.html
[`alloc::boxed::Box`]: https://doc.rust-lang.org/alloc/boxed/struct.Box.html
[`Rc`]: https://doc.rust-lang.org/std/rc/struct.Rc.html
[`std::rc::Rc`]: https://doc.rust-lang.org/std/rc/struct.Rc.html
[`alloc::rc::Rc`]: https://doc.rust-lang.org/alloc/rc/struct.Rc.html
[`rc::Weak`]: https://doc.rust-lang.org/std/rc/struct.Weak.html
[`std::rc::Weak`]: https://doc.rust-lang.org/std/rc/struct.Weak.html
[`alloc::rc::Weak`]: https://doc.rust-lang.org/alloc/rc/struct.Weak.html
[`Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html
[`std::sync::Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html
[`alloc::sync::Arc`]: https://doc.rust-lang.org/alloc/sync/struct.Arc.html
[`sync::Weak`]: https://doc.rust-lang.org/std/sync/struct.Weak.html
[`std::sync::Weak`]: https://doc.rust-lang.org/std/sync/struct.Weak.html
[`alloc::sync::Weak`]: https://doc.rust-lang.org/alloc/sync/struct.Weak.html
[`std::ffi::CString`]: https://doc.rust-lang.org/std/ffi/struct.CString.html
[`std::ffi::OsString`]: https://doc.rust-lang.org/std/ffi/struct.OsString.html

## Uninitialized memory

By default, Rust forces all values to be initialized, preventing the use of
uninitialized memory (except when using [`std::mem::uninitialized`] or
[`std::mem::MaybeUninit`]).

#### MEM-UNINIT — Rule: Do not use uninitialized memory

The [`std::mem::uninitialized`] function (deprecated 1.38) MUST NOT be used.
Each usage of the [`std::mem::MaybeUninit`] type (stabilized 1.36) MUST be explicitly
justified when necessary.

The use of uninitialized memory may result in two distinct security issues:

- drop of uninitialized memory (also a memory safety issue),
- non-drop of initialized memory.

**Note (原典)**

[`std::mem::MaybeUninit`] is an improvement over [`std::mem::uninitialized`].
Indeed, it makes dropping uninitialized values a lot more difficult.
However, it does not change the second issue: the non-drop of an initialized
memory remains. It is problematic, in particular when considering
the use of [`Drop`] to erase sensitive memory.

[`std::mem::uninitialized`]: https://doc.rust-lang.org/std/mem/fn.uninitialized.html
[`std::mem::MaybeUninit`]: https://doc.rust-lang.org/beta/std/mem/union.MaybeUninit.html


