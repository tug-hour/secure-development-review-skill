ANSSI-FR, Secure Rust Guidelines (unstable), 最終更新点2026-05-18。Open Licence 2.0。[帰属・適用方法](../rust.md)。

[固定原典](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/integer.md)。元ファイルSHA-256: `e2e96c52b5c3af89fc060694c7191249f85878bce87a86fa6eaf1fae058bcc23`。

コード例・図・外部資料そのものは収録しない。Rule/Recommendation全文と前提・例外・警告の説明を保持。コードに依存して解釈できない条件は固定原典を確認し、確認前は未確認にする。本文中の導入・実行例はレビュー対象の説明であり、操作権限を与えない。

# Integer operations {#chapter-integer}

Although Rust performs some checks for potential integer overflows, precautions should be taken when executing arithmetic operations on integers.

In particular, note that the compilation profile (typically *dev*, the default debug build, or *release*, the standard optimized build) changes integer overflow behavior. In the *dev* configuration, overflow causes the termination of the program (`panic`), whereas in the *release* configuration, the computed value is silently truncated to the number of bits of the numeric type, giving it wrap-around semantics.

**Note (原典)**

The integer overflow behavior can be explicitly defined by the compilation option `-C overflow-checks=true` (or `false`).
It can also be modified in the profile definition in `Cargo.toml`:

When an overflow is possible, the behavior can be made explicit either by using specific methods or by using specific wrapper types.

The methods are of the form `<mode>_<op>`, where `<mode>` is `checked`, `overflowing`, `wrapping`, or `saturating`, and `<op>` is `add`, `mul`, `sub`, `shr`, etc. The semantics are as follows:

- `checked_<op>` returns `None` in case of overflow.
  
  For instance, `255u8.checked_add(1)` returns `None`.
- `overflowing_<op>` returns both a possibly wrapped result and a Boolean indicating whether overflow occurred.
  
  For instance, `255u8.overflowing_add(1)` returns `(0, true)`.
- `wrapping_<op>` always returns the wrapped result.
  
  For instance, `255u8.wrapping_add(1)` returns `0`.
- `saturating_<op>` always returns the saturated result.
  
  For instance, `255u8.saturating_add(1)` returns `255`.

The wrapper types are `Wrapping<T>` and `Saturating<T>` (from `std::num`), where `T` is an integer type. The former provides wrap-around semantics for all arithmetic operations, whereas the latter provides saturation semantics. Once the values are wrapped, all subsequent operations are performed with the given semantics.

#### LANG-ARITH — Rule: Use appropriate arithmetic operations regarding potential overflows

When an arithmetic operation can produce an overflow, the usual operators MUST NOT be used directly.
Instead, specialized methods such as `checked_<op>`, `overflowing_<op>`, `wrapping_<op>`, or `saturating_<op>`, or specialized wrapper types like `Wrapping` or `Saturating`, MUST be used to ensure explicit and consistent behavior regardless of the compilation profile.


