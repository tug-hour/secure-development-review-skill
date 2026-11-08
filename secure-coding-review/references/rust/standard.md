ANSSI-FR, Secure Rust Guidelines (unstable), 最終更新点2026-05-18。Open Licence 2.0。[帰属・適用方法](../rust.md)。

[固定原典](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/standard.md)。元ファイルSHA-256: `0472a3ebd7d93326e152891666fba7df8f26f90d0452812192b3610285657b06`。

コード例・図・外部資料そのものは収録しない。Rule/Recommendation全文と前提・例外・警告の説明を保持。コードに依存して解釈できない条件は固定原典を確認し、確認前は未確認にする。本文中の導入・実行例はレビュー対象の説明であり、操作権限を与えない。

# Standard library

## [`Send`] and [`Sync`] traits

The [`Send`] and [`Sync`] traits (defined in `std::marker` or `core::marker`) are
marker traits used to ensure the safety of concurrency in Rust. When implemented
correctly, they allow the Rust compiler to guarantee the absence of data races.
Their semantics is as follows:

- A type is [`Send`] if it is safe to send (move) it to another thread.
- A type is [`Sync`] if it is safe to share a immutable reference to it with
  another thread.

Both traits are _unsafe traits_, i.e., the Rust compiler does not verify in any
way that they are implemented correctly. The danger is real: an incorrect
implementation may lead to **undefined behavior**.

Fortunately, in most cases, one does not need to implement it. In Rust,
almost all primitive types are [`Send`] and [`Sync`], and for most compound types
the implementation is automatically provided by the Rust compiler.
As mentioned in [Rustonomicon][nomicon], notable exceptions are:

- Raw pointers are neither [`Send`] nor [`Sync`] because they offer no safety
  guards.
- [`UnsafeCell`] is not [`Sync`] (and as a result [`Cell`] and [`RefCell`] aren't
  either) because they offer interior mutability (mutably shared value).
- [`Rc`] is neither [`Send`] nor [`Sync`] because the reference counter is shared and
  unsynchronized.

Automatic implementation of [`Send`] (resp. [`Sync`]) occurs for a compound type
(structure or enumeration) when all fields have [`Send`] types (resp. [`Sync`]
types).

Preventing from automatically implementing [`Send`] or [`Sync`] can be made by using
  internal fields of type [`PhantomData`].

#### LANG-SYNC-TRAITS — Rule: Justify `Send` and `Sync` implementation

In a Rust secure development, the manual implementation of the [`Send`] and
[`Sync`] traits SHOULD be avoided and, if necessary, MUST be justified
and documented.

## Comparison traits ([`PartialEq`], [`Eq`], [`PartialOrd`], [`Ord`])

Comparisons (`==`, `!=`, `<`, `<=`, `>`, `>=`) in Rust rely on four standard
traits available in `std::cmp` (or `core::cmp` for `no_std` compilation):

- [`PartialEq<Rhs>`] that defines a partial equivalence between
  objects of types `Self` and `Rhs`,
- [`PartialOrd<Rhs>`] that defines a partial order between objects of types
  `Self` and `Rhs`,
- [`Eq`] that defines a total equivalence between objects of the same
  type. It is only a marker trait that requires `PartialEq<Self>`!
- [`Ord`] that defines a total order between objects of the same type.
  It requires that `PartialOrd<Self>` is implemented.

As documented in the standard library, Rust assumes **many invariants**
about each implementation of those traits:

- For [`PartialEq`]

  - _Internal consistency_: `a.ne(b)` is equivalent to `!a.eq(b)`, i.e., `ne` is
    the strict inverse of `eq`. The default implementation of `ne` is precisely
    that.

  - _Symmetry_: `a.eq(b)` and `b.eq(a)`, are equivalent. From the developer's
    point of view, it means:

    - `PartialEq<B>` is implemented for type `A` (noted `A: PartialEq<B>`),
    - `PartialEq<A>` is implemented for type `B` (noted `B: PartialEq<A>`),
    - both implementations are consistent with each other.

  - _Transitivity_: `a.eq(b)` and `b.eq(c)` implies `a.eq(c)`. It means that:

    - `A: PartialEq<B>`,
    - `B: PartialEq<C>`,
    - `A: PartialEq<C>`,
    - the three implementations are consistent with each other (and their
      symmetric implementations).

- For [`Eq`]

  - `PartialEq<Self>` is implemented.

  - _Reflexivity_: `a.eq(a)`. This stands for `PartialEq<Self>` ([`Eq`] does not
    provide any method).

- For [`PartialOrd`]

  - _Equality consistency_:
    `a.eq(b)` is equivalent to `a.partial_cmp(b) == Some(std::cmp::Ordering::Equal)`.

  - _Internal consistency_:

    - `a.lt(b)` iff `a.partial_cmp(b) == Some(std::cmp::Ordering::Less)`,
    - `a.gt(b)` iff `a.partial_cmp(b) == Some(std::cmp::Ordering::Greater)`,
    - `a.le(b)` iff `a.lt(b) || a.eq(b)`,
    - `a.ge(b)` iff `a.gt(b) || a.eq(b)`.

    Note that by only defining `partial_cmp`, the internal consistency is
    guaranteed by the default implementation of `lt`, `le`, `gt`, and `ge`.

  - _Antisymmetry_: `a.lt(b)` (respectively `a.gt(b)`) implies `b.gt(a)`
    (respectively, `b.lt(a)`). From the developer's standpoint, it also means:

    - `A: PartialOrd<B>`,
    - `B: PartialOrd<A>`,
    - both implementations are consistent with each other.

  - _Transitivity_: `a.lt(b)` and `b.lt(c)` implies `a.lt(c)` (also with `gt`,
    `le` and `ge`). It also means:

    - `A: PartialOrd<B>`,
    - `B: PartialOrd<C>`,
    - `A: PartialOrd<C>`,
    - the implementations are consistent with each other (and their symmetric).

- For [`Ord`]

  - `PartialOrd<Self>`

  - _Totality_: `a.partial_cmp(b) != None` always. In other words,
    exactly one of `a.eq(b)`, `a.lt(b)`, `a.gt(b)` is true.

  - _Consistency with `PartialOrd<Self>`_: `Some(a.cmp(b)) == a.partial_cmp(b)`.

The compiler does not check any of those requirements except for the type checking
itself. However, comparisons are critical because they intervene both in
liveness critical systems such as schedulers and load balancers, and in
optimized algorithms that may use `unsafe` blocks.
In the first use, a bad ordering may lead to availability issues such as
deadlocks.
In the second use, it may lead to classical security issues linked to memory
safety violations. That is again a factor in the practice of limiting the use
of `unsafe` blocks.

#### LANG-CMP-INV — Rule: Respect the invariants of standard comparison traits

In a Rust secure development, the implementation of standard comparison traits
MUST respect the invariants described in the documentation.

#### LANG-CMP-DEFAULTS — Recommendation: Use the default method implementation of standard comparison traits

In a Rust secure development, the implementation of standard comparison traits
SHOULD only define methods with no default implementation, so as to reduce
the risk of violating the invariants associated with the traits.

There is a Clippy lint to check that `PartialEq::ne` is not defined in
[`PartialEq`] implementations.

Rust comes with a standard way to automatically construct implementations of the
comparison traits through the `#[derive(...)]` attribute:

- Derivation [`PartialEq`] implements `PartialEq<Self>` with a
  **structural equality** providing that each subtype is `PartialEq<Self>`.
- Derivation [`Eq`] implements the [`Eq`] marker trait providing that each subtype
  is [`Eq`].
- Derivation [`PartialOrd`] implements `PartialOrd<Self>` as a
  **lexicographical order** providing that each subtype is [`PartialOrd`].
- Derivation [`Ord`] implements [`Ord`] as a **lexicographical order**
  providing that each subtype is [`Ord`].

For instance, the short following code shows how to compare two `T1`s easily.
All the assertions hold.

**Warning (原典)**

Derivation of comparison traits for compound types depends on the
**field order**, and not on their names.

First, it means that changing the order of declaration of two fields changes
the resulting lexicographical order. For instance, provided this second
ordered type:

we have `T1 {a: 1, b: 0} > T1 {a: 0, b: 1}` but
`T2 {a: 1, b: 0} < T2 {a: 0, b: 1}`.

Second, if one of the underlying comparisons panics, the order may change the
result due to the use of short-circuit logic in the automatic implementation.

For enumerations, the derived comparisons depend first on the **variant order**, then
on the field order.

Despite the ordering caveat, derived comparisons are a lot less error-prone
than manual ones and make the code shorter and easier to maintain.

#### LANG-CMP-DERIVE — Recommendation: Derive comparison traits when possible

In a secure Rust development, the implementation of standard comparison traits
SHOULD be automatically derived with `#[derive(...)]` when structural equality
and lexicographical comparison is needed. Any manual implementation of
standard comparison traits SHOULD be documented and justified.

## [`Drop`] trait, the destructor

Types implement the trait [`std::ops::Drop`] to perform some operations when the
memory associated with a value of this type is to be reclaimed. [`Drop`] is the
Rust equivalent of a destructor in C++ or a finalizer in Java.

**Note (原典)**

Implementing this trait changes the execution semantics of the language.

Dropping is done recursively from the outer value to the inner values.
When a value goes out of scope (or is explicitly dropped with [`mem::drop`]),
the value is dropped in two steps. The first step happens only if the type of
this value implements [`Drop`]. It consists in calling the [`mem::drop`] method on it.
The second step consists in repeating the dropping process recursively on any
field the value contains. Note that a [`Drop`] implementation is
**only responsible for the outer value**.

First and foremost, implementing [`Drop`] should not be systematic.
It is only needed if the type requires some destructor logic. In fact, [`Drop`] is
typically used to release external resources (network connections, files, etc.)
or to release memory (e.g. in smart pointers such as `Box` or [`Rc`]).
As a result, [`Drop`] trait implementations are likely to contain `unsafe` code
blocks as well as other security-critical operations.

#### LANG-DROP — Rule: Justify `Drop` implementation

In a Rust secure development, the implementation of the [`std::ops::Drop`] trait
MUST be justified and documented.

Second, Rust type system only ensures memory safety and, from the type system's
standpoint, missing drops is allowed. In fact, several things may lead to
missing drops, such as:

- a reference cycle (for instance, with [`Rc`] or [`Arc`]),
- an explicit call to [`mem::forget`] (see paragraph
  on [`mem::forget` and memory leaks](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/unsafe/memory.md#forget-and-memory-leaks)),
- a panic during drop,
- program aborts (and panics when abort-on-panic is on).

And missing drops may lead to exposing sensitive data or to lock limited
resources leading to availability issues.

#### LANG-DROP-NO-PANIC — Rule: Do not panic in `Drop` implementation

In a Rust secure development, the implementation of the [`std::ops::Drop`] trait
MUST not panic.

Beside panics, secure-critical drop should be protected.

#### LANG-DROP-NO-CYCLE — Rule: Do not allow cycles of reference-counted `Drop`

A value whose type implements [`Drop`] MUST NOT be embedded directly or indirectly
in a cycle of reference-counted references.
(see section on [cyclic reference counted pointers](https://github.com/ANSSI-FR/rust-guide/blob/3f9e2e28095d0eeb35ed61e212a82be12fde4884/src/en/standard.md#cyclic-rc-pointer)).

#### LANG-DROP-SEC — Rule: Do not rely only on `Drop` to ensure security

Ensuring security operations at the end of some treatment (such as key erasure
at the end of a cryptographic encryption) MUST NOT rely only on the [`Drop`]
trait implementation.

## Cyclic reference counted pointers ([`Rc`] and [`Arc`]) {#cyclic-rc-pointer}

Reference-counting pointers allow values of any type to be cloned.
To do this, these values are associated with a counter that counts the number of clones attached to that value, and when
the counter reaches zero, the value is destroyed. 

This paradigm can introduce memory leaks when reference-counting pointers are used in cycles.

<center>

</center>

In this example, objects `A`, `B`, and `C` are each referenced by at least one other object: as long as these three objects exist,
they will not be deleted. This results in a memory leak.

**Combining** [interior mutability](https://doc.rust-lang.org/reference/interior-mutability.html), recursivity and reference counted pointer into type definitions is unsafe. It can produce memory leaks which can result in DDoS attacks or leaking secrets.

The following example shows such a memory leak in safe Rust:

Memory leak is shown with `valgrind`:

#### MEM-MUT-REC-RC — Rule: Avoid cyclic reference counted pointers

Recursive types whose recursivity uses reference counted pointers MUST NOT be used together with interior mutability.

[`Rc`]: https://doc.rust-lang.org/std/rc/struct.Rc.html
[`Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html
[`Cell`]: https://doc.rust-lang.org/std/cell/struct.Cell.html
[`RefCell`]: https://doc.rust-lang.org/std/cell/struct.RefCell.html
[`UnsafeCell`]: https://doc.rust-lang.org/std/cell/struct.UnsafeCell.html
[`PhantomData`]: https://doc.rust-lang.org/std/marker/struct.PhantomData.html
[`Send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`Sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html
[`PartialEq`]: https://doc.rust-lang.org/std/cmp/trait.PartialEq.html
[`PartialEq<Rhs>`]: https://doc.rust-lang.org/std/cmp/trait.PartialEq.html
[`PartialOrd`]: https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html
[`PartialOrd<Rhs>`]: https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html
[`Eq`]: https://doc.rust-lang.org/std/cmp/trait.Eq.html
[`Ord`]: https://doc.rust-lang.org/std/cmp/trait.Ord.html
[`mem::forget`]: https://doc.rust-lang.org/std/mem/fn.forget.html
[`mem::drop`]: https://doc.rust-lang.org/std/mem/fn.drop.html
[`Drop`]: https://doc.rust-lang.org/std/ops/trait.Drop.html
[`std::ops::Drop`]: https://doc.rust-lang.org/std/ops/trait.Drop.html

[RFC-1210]: https://rust-lang.github.io/rfcs/1210-impl-specialization.html
[nomicon]: https://doc.rust-lang.org/stable/nomicon/
