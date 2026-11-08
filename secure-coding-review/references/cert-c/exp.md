# CERT C — EXP

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## EXP00-C. Use parentheses for precedence of operation

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/02.exp00-c.md) — SHA-256 `fa81184b2593be38b3ed4142eda464eccf70357b4c7e3d125c440c74c7fdd135`。

C programmers commonly make errors regarding the precedence rules of C operators because of the unintuitive low-precedence levels of `&` , `|` , `^` , `<<` , and `>>` . Mistakes regarding precedence rules can be avoided by the suitable use of parentheses. Using parentheses defensively reduces errors and, if not taken to excess, makes the code more readable.

Subclause 6.5 of the C Standard defines the precedence of operation by the order of the subclauses.

### Noncompliant Code Example

The intent of the expression in this noncompliant code example is to test the least significant bit of `x` :

Because of operator precedence rules, the expression is parsed as

which evaluates to

and then to `0` .

### Compliant Solution

In this compliant solution, parentheses are used to ensure the expression evaluates as expected:

### Exceptions

**EXP00-C-EX1:** Mathematical expressions that follow algebraic order do not require parentheses. For instance, in the expression

the multiplication is performed before the addition by mathematical convention. Consequently, parentheses to enforce the algebraic order would be redundant:

## EXP02-C. Be aware of the short-circuit behavior of the logical AND and OR operators

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/03.exp02-c.md) — SHA-256 `eb265891dc212f91b07075f443292ab0f22cea159e96df0c2e8ba0501481fd49`。

The logical AND and logical OR operators ( `&&` and `||` , respectively) exhibit "short-circuit" operation. That is, the second operand is not evaluated if the result can be deduced solely by evaluating the first operand.

Programmers should exercise caution if the second operand contains [side effects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sideeffect) because it may not be apparent whether the side effects actually occur.

In the following code, the value of `i` is incremented only when `   i >= 0  `:

Although the behavior is well defined, it is not immediately obvious whether or not `i` gets incremented.

### Noncompliant Code Example

In this noncompliant code example, the second operand of the logical OR operator invokes a function that results in side effects:

Because `malloc()` is called only if `p` is `NULL` when entering the `if` clause, `free()` might be called with a pointer to local data not allocated by `malloc()` . (See [MEM34-C. Only free memory allocated dynamically](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/memory-management-mem/mem34-c) .) This behavior is partially due to the uncertainty of whether or not `malloc()` is actually called.

### Compliant Solution

In this compliant solution, a second pointer, `q` , is used to indicate whether `malloc()` is called; if not, `q` remains set to `NULL` . Passing `NULL` to `free()` is guaranteed to safely do nothing.

## EXP03-C. Do not assume the size of a structure is the sum of the sizes of its members

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/04.exp03-c.md) — SHA-256 `71e6425240005e9d3c1e094c4b03b241b81f7a3547c713ebd54b18399b0b26c5`。

The size of a structure is not always equal to the sum of the sizes of its members. Subclause 6.7.2.1 of the C Standard states, "There may be unnamed padding within a structure object, but not at its beginning" \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \].

This unnamed padding is often called *structure padding* . Structure members are arranged in memory as they are declared in the program text. Padding may be added to the structure to ensure the structure is properly aligned in memory. Structure padding allows for faster member access on many architectures.

Rearranging the fields in a `struct` can change the size of the `struct` . It is possible to minimize padding anomalies if the fields are arranged in such a way that fields of the same size are grouped together.

Padding is also called *`struct` member alignment* . Many compilers provide a flag that controls how the members of a structure are packed into memory. Modifying this flag may cause the size of the structures to vary. Most compilers also include a keyword that removes all padding; the resulting structures are called *packed structures* . Overriding the default behavior is often unwise because it leads to interface compatibility problems (the nominally same `struct` has its layout interpreted differently in different modules).

### Noncompliant Code Example

This noncompliant code example assumes that the size of `   struct buffer  ` is equal to the sum of the size of its individual components, which may not be the case \[ [Dowd 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Dowd06) \]. The size of `   struct buffer  ` may actually be larger because of structure padding.

### Compliant Solution

Accounting for structure padding prevents these types of errors:

## EXP05-C. Do not cast away a const qualification

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/05.exp05-c.md) — SHA-256 `98da5a9209399ddd1b52380492fb34a65686ebd951fdac14e4ba70471c6eee16`。

Do not cast away a `const` qualification on an object of pointer type. Casting away the `const` qualification allows a program to modify the object referred to by the pointer, which may result in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . See [undefined behavior 61](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_61) in Appendix J of the C Standard.

As an illustration, the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] provides a footnote (subclause 6.7.3, paragraph 4):

> The implementation may place a `const` object that is not volatile in a read-only region of storage. Moreover, the implementation need not allocate storage for such an object if its address is never used.

### Noncompliant Code Example

The `remove_spaces()` function in this noncompliant code example accepts a pointer to a string `str` and a string length `slen` and removes the space character from the string by shifting the remaining characters toward the front of the string. The function `remove_spaces()` is passed a `const` `char` pointer as an argument. The `const` qualification is cast away, and then the contents of the string are modified.

### Compliant Solution

In this compliant solution, the function `remove_spaces()` is passed a non- `const` `char` pointer. The calling function must ensure that the null-terminated byte string passed to the function is not `const` by making a copy of the string or by other means.

### Noncompliant Code Example

In this noncompliant code example, the contents of the `const` `int` array `vals` are cleared by the call to `memset()` :

Because the `memset()` function takes a (non- `const` ) pointer to `void` , the compiler must implicitly cast away `const` .

#### Implementation Details

The GCC compiler issues a warning when an implicit cast is performed.

### Compliant Solution

If the intention is to allow the array values to be modified, do not declare the array as `const` :

Otherwise, do not attempt to modify the contents of the array.

### Exceptions

**EXP05-C-EX1:** An exception to this recommendation is allowed when it is necessary to cast away `const` when invoking a legacy API that does not accept a `const` argument, provided the function does not attempt to modify the referenced variable. For example, the following code casts away the `const` qualification of `INVFNAME` in the call to the `audit_log()` function.

**EXP05-C-EX2:** A number of C standard library functions are specified to return non- `const` pointers that refer to their `const` -qualified arguments. When the actual arguments to such functions reference `const` objects, attempting to use the returned non- `const` pointers to modify the `const` objects would be a violation of [EXP40-C. Do not modify constant objects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp40-c) and would lead to [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . These functions are the following:

| `memchr` | `strchr` | `strpbrk` | `strrchr` |
| --- | --- | --- | --- |
| `strstr` | `strtod` | `strtof` | `strtold` |
| `strtol` | `strtoll` | `strtoul` | `strtoull` |
| `wmemchr` | `wcschr` | `wcspbrk` | `wcsrchr` |
| `wcsstr` |  |  |  |

For instance, in following example, the function `strchr` returns an unqualified `char*` that points to the terminating null character of the constant character array `s` (which could be stored in ROM). Even though the pointer is not `const` , attempting to modify the character it points to would lead to undefined behavior.

Similarly, in the following example, the function `strtol` sets the unqualified `char*` pointer referenced by `end` to point just past the last successfully parsed character of the constant character array `s` (which could be stored in ROM). Even though the pointer is not `const` , attempting to modify the character it points to would lead to undefined behavior.

**EXP05-C-EX3:** Because `const` means "read-only," and not "constant," it is sometimes useful to declare `struct` members as (pointer to) `const` objects to obtain diagnostics when the user tries to change them in some way other than via the functions that are specifically designed to maintain that data type. Within those functions, however, it may be necessary to strip off the `const` qualification to update those members.

## EXP07-C. Do not diminish the benefits of constants by assuming their values in expressions

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/06.exp07-c.md) — SHA-256 `d12fcabc059f45a8c6fd19644dd2bf37c3f33553a8ebb1f935ba50a3c63fa97a`。

If a constant value is given for an identifier, do not diminish the maintainability of the code in which it is used by assuming its value in expressions. Simply giving the constant a name is not enough to ensure modifiability; you must be careful to always use the name, and remember that the value can change. This recommendation is related to [DCL06-C. Use meaningful symbolic constants to represent literal values](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl06-c) .

### Noncompliant Code Example

The header `stdio.h` defines the `BUFSIZ` macro, which expands to an integer constant expression that is the size of the buffer used by the `setbuf()` function. This noncompliant code example defeats the purpose of defining `BUFSIZ` as a constant by assuming its value in the following expression:

The programmer's assumption underlying this code is that "everyone knows that `BUFSIZ` equals 512," and right-shifting 9 bits is the same (for positive numbers) as dividing by 512. However, if `BUFSIZ` changes to 1024 on some systems, modifications are difficult and error prone.

### Compliant Solution

This compliant solution uses the identifier assigned to the constant value in the expression:

Most modern C compilers will optimize this code appropriately.

## EXP08-C. Ensure pointer arithmetic is used correctly

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/07.exp08-c.md) — SHA-256 `0b16b016db550fd33c9387e3f1f87f8c57117c59885607ec80f08fb2ed8aac6d`。

When performing pointer arithmetic, the size of the value to add to a pointer is automatically scaled to the size of the type of the pointed-to object. For instance, when adding a value to the byte address of a 4-byte integer, the value is scaled by a factor of 4 and then added to the pointer. Failing to understand how pointer arithmetic works can lead to miscalculations that result in serious errors, such as buffer overflows.

### Noncompliant Code Example

In this noncompliant code example, integer values returned by `parseint(getdata())` are stored into an array of `INTBUFSIZE` elements of type `int` called `buf` \[ [Dowd 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Dowd06) \]. If data is available for insertion into `buf` (which is indicated by `havedata()` ) and `buf_ptr` has not been incremented past `   buf + sizeof(buf)  ` , an integer value is stored at the address referenced by `buf_ptr` . However, the `sizeof` operator returns the total number of bytes in `buf` , which is typically a multiple of the number of elements in `buf` . This value is scaled to the size of an integer and added to `buf` . As a result, the check to make sure integers are not written past the end of `buf` is incorrect, and a buffer overflow is possible.

### Compliant Solution

In this compliant solution, the size of `buf` , `INTBUFSIZE` , is added directly to `buf` and used as an upper bound. The integer literal `INTBUFSIZE` is scaled to the size of an integer, and the upper bound of `buf` is checked correctly.

An arguably better solution is to use the address of the nonexistent element following the end of the array, as follows:

This solution works because the C Standard guarantees the address of `buf[INTBUFSIZE]` even though no such element exists.

### Noncompliant Code Example

This noncompliant code example is based on a flaw in the OpenBSD operating system. An integer, `skip` , is added as an offset to a pointer of type `   struct big  ` . The adjusted pointer is then used as a destination address in a call to `memset()` . However, when `skip` is added to the `   struct big  ` pointer, it is automatically scaled by the size of `   struct big  ` , which is 32 bytes (assuming 4-byte integers, 8-byte `   long long  ` integers, and no structure padding). This scaling results in the call to `memset()` writing to unintended memory.

A similar situation occurred in OpenBSD's `make` command \[ [Murenin 2007](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Murenin07) \].

### Compliant Solution

To correct this example, the `   struct big  ` pointer is cast as a `   char *  ` , which causes `skip` to be scaled by a factor of 1:

## EXP09-C. Use sizeof to determine the size of a type or variable

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/08.exp09-c.md) — SHA-256 `33626ed0aa0e11302f6642b5d874f3b3b191edfc2efa1b1cdf0c50715e8e5f3d`。

Do not hard code the size of a type into an application. Because of alignment, padding, and differences in basic types (e.g., 32-bit versus 64-bit pointers), the size of most types can vary between compilers and even versions of the same compiler. Using the `sizeof` operator to determine sizes improves the clarity of what is meant and ensures that changes between compilers or versions will not affect the code.

Type alignment requirements can also affect the size of structures. For example, the size of the following structure is [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementationdefinedbehavior) :

Assuming 32-bit integers and 64-bit doubles, for example, the size can range from 12 to 16 bytes, depending on alignment rules.

### Noncompliant Code Example

This noncompliant code example attempts to declare a two-dimensional array of integers with variable length rows. On a platform with 64-bit integers, the loop will access memory outside the allocated memory section.

### Compliant Solution

This compliant solution replaces the hard-coded value `4` with `   sizeof(int *)  ` :

Also see [MEM02-C. Immediately cast the result of a memory allocation function call into a pointer to the allocated type](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/memory-management-mem/mem02-c) for a discussion on the use of the `sizeof` operator with memory allocation functions.

### Exceptions

**EXP09-C-EX1:** The C Standard explicitly declares `   sizeof(char) == 1  ` , so any sizes based on characters or character arrays may be evaluated without using `sizeof` . This does not apply to `char*` or any other data types.

## EXP10-C. Do not depend on the order of evaluation of subexpressions or the order in which side effects take place

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/09.exp10-c.md) — SHA-256 `fba0325be35954eaba60be392c951b634ff1a706143d42f734dab78a4b575cd3`。

The order of evaluation of subexpressions and the order in which [side effects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sideeffect) take place are frequently defined as [unspecified behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unspecifiedbehavior) by the C Standard. Counterintuitively, [unspecified behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unspecifiedbehavior) in behavior for which the standard provides two or more possibilities and imposes no further requirements on which is chosen in any instance. Consequently, unspecified behavior can be a portability issue because different [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) can make different choices. If dynamic scheduling is used, however, there may not be a fixed-code execution sequence over the life of a process. Operations that can be executed in different sequences may in fact be executed in a different order.

According to the C Standard, subclause 6.5 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \],

> Except as specified later, side effects and value computations of subexpressions are unsequenced.

Following are specific examples of situations in which the order of evaluation of subexpressions or the order in which [side effects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sideeffect) take place is unspecified:

-   The order in which the arguments to a function are evaluated (C Standard, subclause 6.5.2.2, "Function Calls")
-   The order of evaluation of the operands in an assignment statement (C Standard, subclause 6.5.16, "Assignment Operators")
-   The order in which any side effects occur among the initialization list expressions is unspecified. In particular, the evaluation order need not be the same as the order of subobject initialization (C Standard, subclause 6.7.9, "Initialization")

This recommendation is related to [EXP30-C. Do not depend on the order of evaluation for side effects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp30-c) , but it focuses on behavior that is nonportable or potentially confusing.

### Noncompliant Code Example

The order of evaluation of the function designator, the actual arguments, and subexpressions within the actual arguments are unspecified, but there is a sequence point before the actual call. For example, in the function call

the functions `f1()` , `f2()` , `f3()` , and `f4()` may be called in any order. All side effects have to be completed before the function pointed to by `pf[f1()]` is called.

Consequently, the result of this noncompliant code example depends on [unspecified behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unspecifiedbehavior) :

This code may result in `g` being assigned the value `1` , or equally likely, being assigned the value `2` .

### Compliant Solution

This compliant solution is independent of the order of evaluation of the operands and can be interpreted in only one way:

This code always results in `g` being assigned the value `2` .

### Exceptions

**EXP10-C-EX1:** The `&&` and `||` operators guarantee left-to-right evaluation; there is a sequence point after the evaluation of the first operand.

**EXP10-C-EX2:** The first operand of a condition expression is evaluated; there is a sequence point after its evaluation. The second operand is evaluated only if the first compares unequal to 0; the third operand is evaluated only if the first compares equal to 0.

**EXP10-C-EX3:** There is a sequence point before function calls, meaning that the function designator, the actual arguments, and subexpressions within the actual arguments are evaluated before the function is invoked.

**EXP10-C-EX4:** The left operand of a comma operator is evaluated before the right operand is evaluated. There is a sequence point in between.

Note that whereas commas serve to delimit multiple arguments in a function call, these commas are not considered comma operators. Multiple arguments of a function call may be evaluated in any order, with no sequence points between each other.

## EXP11-C. Do not make assumptions regarding the layout of structures with bit-fields

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/10.exp11-c.md) — SHA-256 `25d5ee1e74ae0a9f899e1382e1083400bc36842c48fd951c608fea1b23285b71`。

The internal representations of bit-field structures have several properties (such as internal padding) that are [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-defined) . Additionally, bit-field structures have several implementation-defined constraints:

-   The alignment of bit-fields in the storage unit (for example, the bit-fields may be allocated from the high end or the low end of the storage unit)
-   Whether or not bit-fields can overlap a storage unit boundary

Consequently, it is impossible to write portable safe code that makes assumptions regarding the layout of bit-field structure members.

### Noncompliant Code Example (Bit-Field Alignment)

Bit-fields can be used to allow flags or other integer values with small ranges to be packed together to save storage space. Bit-fields can improve the storage efficiency of structures. Compilers typically allocate consecutive bit-field structure members into the same `int` -sized storage, as long as they fit completely into that storage unit. However, the order of allocation within a storage unit is implementation-defined. Some [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) are *right-to-left* : the first member occupies the low-order position of the storage unit. Others are *left-to-right* : the first member occupies the high-order position of the storage unit. Calculations that depend on the order of bits within a storage unit may produce different results on different implementations.

Consider the following structure made up of four 8-bit bit-field members:

Right-to-left implementations will allocate `   struct bf  ` as one storage unit with this format:

Conversely, left-to-right implementations will allocate `   struct bf  ` as one storage unit with this format:

The following code behaves differently depending on whether the implementation is left-to-right or right-to-left:

### Compliant Solution (Bit-Field Alignment)

This compliant solution is explicit in which fields it modifies:

### Noncompliant Code Example (Bit-Field Overlap)

In this noncompliant code example, assuming 8 bits to a byte, if bit-fields of 6 and 4 bits are declared, is each bit-field contained within a byte, or are the bit-fields split across multiple bytes?

If each bit-field lives within its own byte, then `m2` (or `m1` , depending on alignment) is incremented by 1. If the bit-fields are indeed packed across 8-bit bytes, then `m2` might be incremented by 4.

### Compliant Solution (Bit-Field Overlap)

This compliant solution is explicit in which fields it modifies:

## EXP12-C. Do not ignore values returned by functions

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/11.exp12-c.md) — SHA-256 `ebd4d9989db82be1d64eda76fac898e7d395f872e2e7bee7462251bd8c3923e1`。

Many functions return useful values whether or not the function has side effects. In most cases, this value is used to signify whether the function successfully completed its task or if some error occurred (see [ERR02-C. Avoid in-band error indicators](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err02-c) ). Other times, the value is the result of some computation and is an integral part of the function's API.

Subclause 6.8.3 of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] states:

> The expression in an expression statement is evaluated as a void expression for its side effects.

All expression statements, such as function calls with an ignored value, are implicitly cast to `void` . Because a return value often contains important information about possible errors, it should always be checked; otherwise, the cast should be made explicit to signify programmer intent. If a function returns no meaningful value, it should be declared with return type `void` .

This recommendation encompasses [ERR33-C. Detect and handle standard library errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err33-c) . Unlike this recommendation, that rule is restricted to functions from the Standard C library.

Compliance with this recommendation is required in order to comply with [ERR00-C. Adopt and implement a consistent and comprehensive error-handling policy](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err00-c)

### Noncompliant Code Example

The `asprintf()` function has been provided by the GNU C library. It works like `sprintf()` , but if given a null pointer as the destination string, it will create a buffer sufficient to hold the resulting string. It relies on `malloc()` to allocate the buffer. If `malloc()` fails, then `asprintf()` returns a negative number.

This noncompliant code example calls `asprintf(),` but fails to check whether the string was successfully created.

### Compliant Solution

This compliant solution checks to make sure no error occurred.

### Exceptions

**EXP12-C-EX1:** If the return value is inconsequential or if any errors can be safely ignored, such as for functions called because of their side effects, the function should be explicitly cast to `void` to signify programmer intent. For an example of this exception, see "Compliant Solution (Remove Existing Destination File)" under the section "Portable Behavior" in [FIO10-C. Take care when using the rename() function](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio10-c) , or Exception **ERR33-C-EX1** in [ERR33-C. Detect and handle standard library errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err33-c) .

**EXP12-C-EX2:** If a function cannot fail or if the return value cannot signify an error condition, the return value may be ignored. Such functions should be added to a whitelist when automatic checkers are used.

## EXP13-C. Treat relational and equality operators as if they were nonassociative

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/12.exp13-c.md) — SHA-256 `86bde4b77d648eb39c079cdda44c43a7fedabf3e3bd261b9963833bf42ed132c`。

The relational and equality operators are left-associative in C. Consequently, C, unlike many other languages, allows chaining of relational and equality operators. Subclause 6.5.8, footnote 107, of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], says:

> The expression `a<b<c` is not interpreted as in ordinary mathematics. As the syntax indicates, it means `(a<b)<c` ; in other words, "if `a` is less than `b` , compare 1 to `c` ; otherwise, compare 0 to `c` ."

These operators are *left-associative* , which means the leftmost comparison is performed first, and the result is compared with the rightmost operand. This syntax allows a programmer to write an expression (particularly an expression used as a condition) that can be easily misinterpreted.

### Noncompliant Code Example

Although this noncompliant code example compiles correctly, it is unlikely that it means what the author of the code intended:

The expression `   a < b < c  ` evaluates to true rather than, as its author probably intended, to false, and the expression `   a == b == c  ` evaluates to false rather than, as its author probably intended, to true.

### Compliant Solution

Treat relational and equality operators as if it were invalid to chain them:

## EXP14-C. Beware of integer promotion when performing bitwise operations on integer types smaller than int

**Recommendation / 対象外（廃止規則）**。旧規則を独立した要求として強制しない。本文の置換先（INT02-C / MSC12-C）で評価する。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/13.exp14-c.md) — SHA-256 `d3c6af7786e63b38262dcec08442d94ee20850fc7d6aa22bd0f59de89c309fb4`。

Deprecated

This guideline has been deprecated by

-   [INT02-C. Understand integer conversion rules](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int02-c)

Integer types smaller than `int` are promoted when an operation is performed on them. If all values of the original type can be represented as an `int` , the value of the smaller type is converted to an `int` ; otherwise, it is converted to an `   unsigned int  ` (see [INT02-C. Understand integer conversion rules](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int02-c) ). If the conversion is to a wider type, the original value is zero-extended for unsigned values or sign-extended for signed types. Consequently, bitwise operations on integer types smaller than `int` may have unexpected results.

### Noncompliant Code Example

This noncompliant code example demonstrates how performing bitwise operations on integer types smaller than `int` may have unexpected results.

In this example, a bitwise complement of `port` is first computed and then shifted 4 bits to the right. If both of these operations are performed on an 8-bit unsigned integer, then `result_8` will have the value `0x0a` . However, `port` is first promoted to a `   signed int  ` , with the following results (on a typical architecture where type `int` is 32 bits wide):

| Expression | Type | Value | Notes |
| --- | --- | --- | --- |
| `port` | `uint8_t` | `0x5a` |  |
| `~port` | `int` | `0xffffffa5` |  |
| `~port >> 4` | `int` | `0x0ffffffa` | Whether or not value is negative is implementation-defined. |
| `result_8` | `uint8_t` | `0xfa` |  |

### Compliant Solution

In this compliant solution, the bitwise complement of `port` is converted back to 8 bits. Consequently, `result_8` is assigned the expected value of `0x0aU` .

## EXP15-C. Do not place a semicolon on the same line as an if, for, or while statement

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/14.exp15-c.md) — SHA-256 `4b728a0550cf89a57c4a61bc28631320756aa252026df8148ddd4bee52de0afe`。

Do not use a semicolon on the same line as an `if` , `for` , or `while` statement because it typically indicates programmer error and can result in unexpected behavior.

### Noncompliant Code Example

In this noncompliant code example, a semicolon is used on the same line as an `if` statement:

### Compliant Solution

It is likely, in this example, that the semicolon was accidentally inserted:

## EXP16-C. Do not compare function pointers to constant values

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/15.exp16-c.md) — SHA-256 `63b488d6f76fc6fa273cc78867d13398e1090604f376fe73f6478129bf2aac29`。

Comparing a function pointer to a value that is not a null function pointer of the same type will be diagnosed because it typically indicates programmer error and can result in [unexpected behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unexpectedbehavior) . Implicit comparisons will be diagnosed, as well.

### Noncompliant Code Example

In this noncompliant code example, the addresses of the POSIX functions `getuid` and `geteuid` are compared for equality to 0. Because no function address shall be null, the first subexpression will always evaluate to false (0), and the second subexpression always to true (nonzero). Consequently, the entire expression will always evaluate to true, leading to a potential security vulnerability.

### Noncompliant Code Example

In this noncompliant code example, the function pointers `getuid` and `geteuid` are compared to 0. This example is from an actual [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) ( [VU#837857](https://www.kb.cert.org/vuls/id/837857) ) discovered in some versions of the X Window System server. The vulnerability exists because the programmer neglected to provide the open and close parentheses following the `geteuid()` function identifier. As a result, the `geteuid` token returns the address of the function, which is never equal to 0. Consequently, the `or` condition of this `if` statement is always true, and access is provided to the protected block for all users. Many compilers issue a warning noting such pointless expressions. Therefore, this coding error is normally detected by adherence to [MSC00-C. Compile cleanly at high warning levels](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc00-c) .

### Compliant Solution

The solution is to provide the open and close parentheses following the `geteuid` token so that the function is properly invoked:

### Compliant Solution

A function pointer can be compared to a null function pointer of the same type:

This code should not be diagnosed by an analyzer.

### Noncompliant Code Example

In this noncompliant code example, the function pointer `do_xyz` is implicitly compared unequal to 0:

### Compliant Solution

In this compliant solution, the function `do_xyz()` is invoked and the return value is compared to 0:

## EXP19-C. Use braces for the body of an if, for, or while statement

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/16.exp19-c.md) — SHA-256 `cb5fda2b8d8aef5658add8625e23fdd2e0eb86e499f0c2eb61dc43fb62b31ee2`。

Opening and closing braces for `if` , `for` , and `while` statements should always be used even if the statement's body contains only a single statement.

If an `if` , `while` , or `for` statement is used in a macro, the macro definition should not conclude with a semicolon. (See [PRE11-C. Do not conclude macro definitions with a semicolon](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre11-c) .)

Braces improve the uniformity and readability of code. More important, when inserting an additional statement into a body containing only a single statement, it is easy to forget to add braces because the indentation gives strong (but misleading) guidance to the structure.

Braces also help ensure that macros with multiple statements are properly expanded. Such a macro should be wrapped in a `do-while` loop. (See [PRE10-C. Wrap multistatement macros in a do-while loop](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre10-c) .) However, when the `do-while` loop is not present, braces can still ensure that the macro expands as intended.

### Noncompliant Code Example

This noncompliant code example uses an `if` statement without braces to authenticate a user:

A developer might add a debugging statement to determine when the login is valid but forget to add opening and closing braces:

Because of the indentation of the code, it is difficult to tell that the code will not function as intended by the programmer, potentially leading to a security breach.

### Compliant Solution

In the compliant solution, opening and closing braces are used even when the body is a single statement:

### Noncompliant Code Example

This noncompliant code example has an `if` statement nested in another `if` statement without braces around the `if` and `else` bodies:

The indentation could lead the programmer to believe that a user is given administrator privileges only when the user's login is valid. However, the `else` statement actually attaches to the inner `if` statement:

This is a security loophole: users with invalid logins can still obtain administrator privileges.

### Compliant Solution

In the compliant solution, adding braces removes the ambiguity and ensures that privileges are correctly assigned:

### Noncompliant Code Example (empty block)

This noncompliant code example has a `while` statement with no block:

Note that if `invalid_login()` has no side effects (such as warning the user if their login failed), this code also violates [MSC12-C. Detect and remove code that has no effect or is never executed](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc12-c) .

### Compliant Solution (empty block)

This compliant solution features an explicit empty block, which clarifies the developer's intent:

## EXP20-C. Perform explicit tests to determine success, true and false, and equality

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/09.expressions-exp/17.exp20-c.md) — SHA-256 `c537ab53830f25a3d390ed8bb6d69d8f8eb38bc60712ec2130510a4b791a7f1b`。

Perform explicit tests to determine success, true/false, and equality to improve the readability and maintainability of code and for compatibility with common conventions.

In particular, do not default the test for nonzero. For instance, suppose a `foo()` function returns 0 to indicate failure or a nonzero value to indicate success. Testing for inequality with 0,

is preferable to

despite the convention that 0 indicates failure. Explicitly testing for inequality with 0 benefits maintainability if `foo()` is later modified to return −1 rather than 0 on failure.

This recommendation is derived from and considers the implications of the following common conventions:

1.  Functions return 0 if false and nonzero if true [\[StackOvflw 2009](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-StackOvflw09) \].
2.  Function failures can typically be indicated by −1 or any nonzero number.
3.  Comparison functions (such as the standard library function `strcmp()` , which has a trinary return value) return 0 if the arguments are equal and nonzero otherwise (see \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \] 7.2.6.4.2 "The `strcmp` function).

### Noncompliant Code Example

In this noncompliant code example, `is_banned()` returns 0 if false and nonzero if true:

If a banned user is listed twice, the user is granted access. Although `is_banned()` follows the common convention of returning nonzero for true, `processRequest` checks for equality only with 1.

### Compliant Solution

Because most functions guarantee a return value of nonzero only for true, the preceding code is better written by checking for inequality with 0 (false), as follows:

### Noncompliant Code Example

In noncompliant code, function status can typically be indicated by returning −1 on failure or any nonnegative number on success. This is a common convention in the standard C library, but it is discouraged in [ERR02-C. Avoid in-band error indicators](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err02-c) .

Although failures are frequently indicated by a return value of 0, some common conventions may conflict in the future with code in which the test for nonzero is not explicit. In this case, defaulting the test for nonzero welcomes bugs if and when a developer modifies `validateUser()` to return an error code or −1 rather than 0 to indicate a failure (all of which are also common conventions).

Although the code will work as intended, it is possible that a future modification will result in the following:

In this code, the programmer intended to add error code functionality to indicate the cause of a [validation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-validation) failure. The new code, however, validates any invalid or nonexisting user. Because there is no explicit test in `processRequest()` , the logical error is not obvious and seems correct by certain conventions.

### Compliant Solution

This compliant code is preferable for improved maintenance. By defining what constitutes a failure and explicitly testing for it, the behavior is clearly implied, and future modifications are more likely to preserve it. If a future modification is made, such as in the previous example, it is immediately obvious that the `if` statement in `processRequest()` does not correctly utilize the specification of `validateUser()` .

### Noncompliant Code Example

Comparison functions (such as the standard library `strcmp()` function) return 0 if the arguments are equal and nonzero otherwise.

Because many comparison functions return 0 for equality and nonzero for inequality, they can cause confusion when used to test for equality. If someone were to switch the following `strcmp()` call with a function testing for equality, but the programmer did not follow the same convention as `strcmp()` , the programmer might instinctively just replace the function name. Also, when quickly reviewed, the code could easily appear to test for inequality.

The preceding code works correctly. However, to simplify the login code or to facilitate checking a user's password more than once, a programmer can separate the password-checking code from the login function in the following way:

In an attempt to leave the previous logic intact, the developer just replaces `strcmp()` with a call to the new function. However, doing so produces incorrect behavior. In this case, any user who inputs an incorrect password is granted access. Again, two conventions conflict and produce code that is easily corrupted when modified. To make code maintainable and to avoid these conflicts, such a result should never be defaulted.

### Compliant Solution

This compliant solution, using a comparison function for this purpose, is the preferred approach. By performing an explicit test, any programmer who wishes to modify the equality test can clearly see the implied behavior and convention that is being followed.

## EXP30-C. Do not depend on the order of evaluation for side effects

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/02.exp30-c.md) — SHA-256 `14db88ba4caa03d3c7f8724fbfd62ddb567d75fad5e0a263643c4af44bfe8306`。

Evaluation of an expression may produce [side effects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sideeffect) . At specific points during execution, known as [sequence points](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sequencepoint) , all side effects of previous evaluations are complete, and no side effects of subsequent evaluations have yet taken place. Do not depend on the order of evaluation for side effects unless there is an intervening sequence point.

The C Standard, 6.5, paragraph 2 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> If a side effect on a scalar object is unsequenced relative to either a different side effect on the same scalar object or a value computation using the value of the same scalar object, the behavior is undefined. If there are multiple allowable orderings of the subexpressions of an expression, the behavior is undefined if such an unsequenced side effect occurs in any of the orderings.

This requirement must be met for each allowable ordering of the subexpressions of a full expression; otherwise, the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See [undefined behavior 34](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_34) .)

The following sequence points are defined in the C Standard, Annex C \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \]:

-   Between the evaluations of the function designator and actual arguments in a function call and the actual call
-   Between the evaluations of the first and second operands of the following operators:
    -   Logical AND: `&&`
    -   Logical OR: `||`
    -   Comma: `,`
-   Between the evaluations of the first operand of the conditional `?:` operator and whichever of the second and third operands is evaluated
-   The end of a full declarator
-   Between the evaluation of a full expression and the next full expression to be evaluated; the following are full expressions:
    -   An initializer that is not part of a compound literal
    -   The expression in an expression statement
    -   The controlling expression of a selection statement ( `if` or `switch` )
    -   The controlling expression of a `while` or `do` statement
    -   Each of the (optional) expressions of a `for` statement
    -   The (optional) expression in a `return` statement
-   Immediately before a library function returns
-   After the actions associated with each formatted input/output function conversion specifier
-   Immediately before and immediately after each call to a comparison function, and also between any call to a comparison function and any movement of the objects passed as arguments to that call

Furthermore, Section 6.5.17.1, paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \] says (regarding assignment operations):

> The side effect of updating the stored value of the left operand is sequenced after the value computations of the left and right operands.

This rule means that statements such as

have defined behavior, and statements such as the following do not:

Not all instances of a comma in C code denote a usage of the comma operator. For example, the comma between arguments in a function call is not a sequence point. However, according to the C Standard, 6.5.3.3, paragraph 8 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \]

> Every evaluation in the calling function (including other function calls) that is not otherwise specifically sequenced before or after the execution of the body of the called function is indeterminately sequenced with respect to the execution of the called function.

This rule means that the order of evaluation for function call arguments is unspecified and can happen in any order.

### Noncompliant Code Example

Programs cannot safely rely on the order of evaluation of operands between sequence points. In this noncompliant code example, `i` is evaluated twice without an intervening sequence point, so the behavior of the expression is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) :

### Compliant Solution

These examples are independent of the order of evaluation of the operands and can be interpreted in only one way:

Alternatively:

### Noncompliant Code Example

The call to `func()` in this noncompliant code example has [undefined behavior 34](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_34) because there is no sequence point between the argument expressions:

The first (left) argument expression reads the value of `i` (to determine the value to be stored) and then modifies `i` . The second (right) argument expression reads the value of `i` between the same pair of sequence points as the first argument, but not to determine the value to be stored in `i` . This additional attempt to read the value of `i` has [undefined behavior 34](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_34) .

### Compliant Solution

This compliant solution is appropriate when the programmer intends for both arguments to `func()` to be equivalent:

This compliant solution is appropriate when the programmer intends for the second argument to be 1 greater than the first:

### Noncompliant Code Example

The order of evaluation for function arguments is unspecified. This noncompliant code example exhibits [unspecified behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unspecifiedbehavior) but not [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) :

It is unspecified what order `a()` and `b()` are called in; the only guarantee is that both `a()` and `b()` will be called before `c()` is called. If `a()` or `b()` rely on shared state when calculating their return value, as they do in this example, the resulting arguments passed to `c()` may differ between compilers or architectures.

### Compliant Solution

In this compliant solution, the order of evaluation for `a()` and `b()` is fixed, and so no unspecified behavior occurs:

## EXP32-C. Do not access a volatile object through a nonvolatile reference

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/03.exp32-c.md) — SHA-256 `6f19f35471be6dc5e751e82c84917abc7344af9586882b4d8e7e060a3c0ba805`。

An object that has volatile-qualified type may be modified in ways unknown to the [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) or have other unknown [side effects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sideeffect) . Referencing a volatile object by using a non-volatile lvalue is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . The C Standard, 6.7.4 paragraph 7 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> If an attempt is made to refer to an object defined with a volatile-qualified type through use of an lvalue with non-volatile-qualified type, the behavior is undefined.

See [undefined behavior 62](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_62) .

### Noncompliant Code Example

In this noncompliant code example, a volatile object is accessed through a non-volatile-qualified reference, resulting in [undefined behavior 62](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_62) :

The assignment `   ipp = &ip  ` is not safe because it allows the valid code that follows to reference the value of the volatile object `i` through the non-volatile-qualified reference `ip` . In this example, the compiler may optimize out the entire `if` block because `   *ip != 0  ` must be false if the object to which `ip` points is not volatile.

#### Implementation Details

This example compiles without warning on Microsoft Visual Studio 2013 when compiled in C mode ( `/TC` ) but causes errors when compiled in C++ mode ( `/TP` ).

GCC 4.8.1 generates a warning but compiles successfully.

### Compliant Solution

In this compliant solution, `ip` is declared `volatile` :

## EXP33-C. Do not read uninitialized memory

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/04.exp33-c.md) — SHA-256 `9a4335da79f42f11e71bfc6d17b3f2e3c385efbc08cddf052e3be5fadaea6527`。

Local, automatic variables assume unexpected values if they are read before they are initialized. The C Standard, 6.7.11, paragraph 11, specifies \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \]

> If an object that has automatic storage duration is not initialized explicitly, its representation is [indeterminate](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) .

See [undefined behavior 11](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_11) .

When local, automatic variables are stored on the program stack, for example, their values default to whichever values are currently stored in stack memory.

Additionally, some dynamic memory allocation functions do not initialize the contents of the memory they allocate.

| Function                       | Initialization                                                       |
|--------------------------------|----------------------------------------------------------------------|
| `aligned_alloc()` | Does not perform initialization                                      |
| `calloc()`        | Zero-initializes allocated memory                                    |
| `malloc()`        | Does not perform initialization                                      |
| `realloc()`       | Copies contents from original pointer; may not initialize all memory |

Uninitialized automatic variables or dynamically allocated memory has [indeterminate values](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) , which for objects of some types, can be a [non-value representation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-nonvaluerepresentation) . Reading non-value representations is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) ; it can cause a program to behave in an [unexpected](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unexpectedbehavior) manner and provide an avenue for attack. (See [undefined behavior 10](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_10) and [undefined behavior 12](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_12) .)  In many cases, compilers issue a warning diagnostic message when reading uninitialized variables. (See [MSC00-C. Compile cleanly at high warning levels](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc00-c) for more information.)

### Noncompliant Code Example (Return-by-Reference)

In this noncompliant code example, the `set_flag()` function is intended to set the parameter, `sign_flag` , to the sign of `number` . However, the programmer neglected to account for the case where `number` is equal to `0` . Because the local variable `sign` is uninitialized when calling `set_flag()` and is never written to by `set_flag()` , the comparison operation exhibits [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) when reading `sign` .

Some compilers assume that when the address of an uninitialized variable is passed to a function, the variable is initialized within that function. Because compilers frequently fail to diagnose any resulting failure to initialize the variable, the programmer must apply additional scrutiny to ensure the correctness of the code.

This defect results from a failure to consider all possible data states. (See [MSC01-C. Strive for logical completeness](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc01-c) for more information.)

### Compliant Solution (Return-by-Reference)

This compliant solution trivially repairs the problem by accounting for the possibility that `number` can be equal to 0.

Although compilers and [static analysis](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-staticanalysis) tools often detect uses of uninitialized variables when they have access to the source code, diagnosing the problem is difficult or impossible when either the initialization or the use takes place in object code for which the source code is inaccessible. Unless doing so is prohibitive for performance reasons, an additional defense-in-depth practice worth considering is to initialize local variables immediately after declaration.

### Noncompliant Code Example (Uninitialized Local)

In this noncompliant code example, the programmer mistakenly fails to set the local variable `error_log` to the `msg` argument in the `report_error()` function \[ [Mercy 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-mercy06) \]. Because `error_log` has not been initialized, an [indeterminate value](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) is read. The `sprintf()` call copies data from the arbitrary location pointed to by the indeterminate `error_log` variable until a null byte is reached, which can result in a buffer overflow.

### Noncompliant Code Example (Uninitialized Local)

In this noncompliant code example, the `report_error()` function has been modified so that `error_log` is properly initialized:

This example remains problematic because a buffer overflow will occur if the null-terminated byte string referenced by `msg` is greater than 17 characters, including the null terminator. (See [STR31-C. Guarantee that storage for strings has sufficient space for character data and the null terminator](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/characters-and-strings-str/str31-c) for more information.)

### Compliant Solution (Uninitialized Local)

In this compliant solution, the buffer overflow is eliminated by calling the `snprintf()` function:

### Compliant Solution (Uninitialized Local)

A less error-prone compliant solution is to simply print the error message directly instead of using an intermediate buffer:

### Noncompliant Code Example ( `mbstate_t` )

In this noncompliant code example, the function `mbrlen()` is passed the address of an automatic `mbstate_t` object that has not been properly initialized. This is [undefined behavior 200](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_200) because `mbrlen()` dereferences and reads its third argument.

### Compliant Solution ( `mbstate_t` )

Before being passed to a multibyte conversion function, an `mbstate_t` object must be either initialized to the initial conversion state or set to a value that corresponds to the most recent shift state by a prior call to a multibyte conversion function. This compliant solution sets the `mbstate_t` object to the initial conversion state by setting it to all zeros:

### Noncompliant Code Example (POSIX, Entropy)

In this noncompliant code example described in " [More Randomness or Less](https://kqueue.org/blog/2012/06/25/more-randomness-or-less/) " \[ [Wang 2012](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Wang12) \], the process ID, time of day, and uninitialized memory `junk` is used to seed a random number generator. This behavior is characteristic of some distributions derived from Debian Linux that use uninitialized memory as a source of entropy because the value stored in `junk` is indeterminate. However, because accessing an [indeterminate value](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) is [undefined behavior 11](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_11) , compilers may optimize out the uninitialized variable access completely, leaving only the time and process ID and resulting in a loss of desired entropy.

In security protocols that rely on unpredictability, such as RSA encryption, a loss in entropy results in a less secure system.

### Compliant Solution (POSIX, Entropy)

This compliant solution seeds the random number generator by using the CPU clock and the real-time clock instead of reading uninitialized memory:

### Noncompliant Code Example ( `realloc()` )

The `realloc()` function changes the size of a dynamically allocated memory object. The initial `size` bytes of the returned memory object are unchanged, but any newly added space is uninitialized, and its value is [indeterminate](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) . As in the case of `malloc()` , accessing memory beyond the size of the original object is [undefined behavior 186](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_186) .

It is the programmer's responsibility to ensure that any memory allocated with `malloc()` and `realloc()` is properly initialized before it is used.

In this noncompliant code example, an array is allocated with `malloc()` and properly initialized. At a later point, the array is grown to a larger size but not initialized beyond what the original array contained. Subsequently accessing the uninitialized bytes in the new array is undefined behavior.

### Compliant Solution ( `realloc()` )

In this compliant solution, the `resize_array()` helper function takes a second parameter for the old size of the array so that it can initialize any newly allocated elements:

### Exceptions

**EXP33-C-EX1:** Reading uninitialized memory by an [lvalue](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-lvalue) of type `   unsigned char  ` that could not have been declared with the `register` storage class does not trigger [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . The `   unsigned char  ` type is defined to not have a non-value representation, which allows for moving bytes without knowing if they are initialized. (See the C Standard, 6.2.6.1, paragraph 3.) The requirement that `register` could not have been used (not merely that it was not used) is because on some architectures, such as the Intel Itanium, registers have a bit to indicate whether or not they have been initialized. The C Standard, 6.3.2.1, paragraph 2, allows such [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) to cause a trap for an object that never had its address taken and is stored in a register if such an object is referred to in any way.

## EXP34-C. Do not dereference null pointers

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/05.exp34-c.md) — SHA-256 `076df575a04d765ccee7c357ebbc3ea0a8d97451f3077499a040893d30729aa4`。

Dereferencing a null pointer is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

On many platforms, dereferencing a null pointer results in [abnormal program termination](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) , but this is not required by the standard. See " [Clever Attack Exploits Fully-Patched Linux Kernel](http://www.theregister.co.uk/2009/07/17/linux_kernel_exploit/) " \[ [Goodin 2009](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Goodin2009) \] for an example of a code execution [exploit](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-exploit) that resulted from a null pointer dereference.

### Noncompliant Code Example

This noncompliant code example is derived from a real-world example taken from a vulnerable version of the `libpng` library as deployed on a popular ARM-based cell phone \[ [Jack 2007](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Jack07) \]. The `libpng` library allows applications to read, create, and manipulate PNG (Portable Network Graphics) raster image files. The `libpng` library implements its own wrapper to `malloc()` that returns a null pointer on error or on being passed a 0-byte-length argument.

This code also violates [ERR33-C. Detect and handle standard library errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err33-c) .

If `length` has the value `−1` , the addition yields 0, and `png_malloc()` subsequently returns a null pointer, which is assigned to `chunkdata` . The `chunkdata` pointer is later used as a destination argument in a call to `memcpy()` , resulting in user-defined data overwriting memory starting at address 0. In the case of the ARM and XScale architectures, the `0x0` address is mapped in memory and serves as the exception vector table; consequently, dereferencing `0x0` did not cause an [abnormal program termination](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) .

### Compliant Solution

This compliant solution ensures that the pointer returned by `png_malloc()` is not null. It also uses the unsigned type `size_t` to pass the `length` parameter, ensuring that negative values are not passed to `func()` .

This solution also ensures that the `user_data` pointer is not null. Passing a null pointer to memcpy() would produce undefined behavior, even if the number of bytes to copy were 0.  The `user_data` pointer could be invalid in other ways, such as pointing to freed memory. However there is no portable way to verify that the pointer is valid, other than checking for null.

### Noncompliant Code Example

In this noncompliant code example, `input_str` is copied into dynamically allocated memory referenced by `c_str` . If `malloc()` fails, it returns a null pointer that is assigned to `c_str` . When `c_str` is dereferenced in `memcpy()` , the program exhibits [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .  Additionally, if `input_str` is a null pointer, the call to `strlen()` dereferences a null pointer, also resulting in undefined behavior. This code also violates [ERR33-C. Detect and handle standard library errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err33-c) .

### Compliant Solution

This compliant solution ensures that both `input_str` and the pointer returned by `malloc()` are not null:

### Noncompliant Code Example

This noncompliant code example is from a version of `drivers/net/tun.c` and affects Linux kernel 2.6.30 \[ [Goodin 2009](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Goodin2009) \]:

The `sk` pointer is initialized to `tun->sk` before checking if `tun` is a null pointer. Because null pointer dereferencing is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) , the compiler (GCC in this case) can optimize away the `   if (!tun)  ` check because it is performed after `tun->sk` is accessed, implying that `tun` is non-null. As a result, this noncompliant code example is vulnerable to a null pointer dereference exploit, because null pointer dereferencing can be permitted on several platforms, for example, by using `mmap(2)` with the `MAP_FIXED` flag on Linux and Mac OS X, or by using the `shmat()` POSIX function with the `SHM_RND` flag \[ [Liu 2009](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Liu2009) \].

### Compliant Solution

This compliant solution eliminates the null pointer dereference by initializing `sk` to `tun->sk` following the null pointer check. It also adds assertions to document that certain other pointers must not be null.

### Exceptions

**EXP34-C-EX1:** According to C23, section 6.5.4.2, paragraph 3, expressions of the form `&*x` and `&x[y]` effectively cancel out, being equivalent to `x` and `x + y`, respectively. No dereferencing actually occurs in these cases, so they do not violate this rule even if `x` is a null pointer.

## EXP35-C. Do not modify objects with temporary lifetime

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/06.exp35-c.md) — SHA-256 `59aafd7b54d7dccf79bfd8cc677d9379ea1d5c3605a94ce2f08746363584e518`。

The C11 Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] introduced a new term: *temporary lifetime* . This term still remains in the C23 Standard. Modifying an object with temporary lifetime is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . According to subclause 6.2.4, paragraph 8 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \]

> A non-lvalue expression with structure or union type, where the structure or union contains a member with array type (including, recursively, members of all contained structures and unions) refers to an object with automatic storage duration and *temporary* lifetime . Its lifetime begins when the expression is evaluated and its initial value is the value of the expression. Its lifetime ends when the evaluation of the containing full expression ends. Any attempt to modify an object with temporary lifetime results in undefined behavior.

This definition differs from the C99 Standard (which defines modifying the result of a function call or accessing it after the next sequence point as undefined behavior) because a temporary object's lifetime ends when the evaluation containing the full expression or full declarator ends, so the result of a function call can be accessed. This extension to the lifetime of a temporary also removes a quiet change to C90 and improves compatibility with C++.

C functions may not return arrays; however, functions can return a pointer to an array or a `struct` or `union` that contains arrays. Consequently, in any version of C, if a function call returns by value a `struct` or `union` containing an array, do not modify those arrays within the expression containing the function call. In C99 and older, do not access an array returned by a function after the next sequence point or after the evaluation of the containing full expression or full declarator ends.

### Noncompliant Code Example

This noncompliant code example [conforms](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conformingprogram) to the C11 Standard; however, it fails to conform to C99. If compiled with a C99-conforming implementation, this code has [undefined behavior 34](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_34) because the sequence point preceding the call to `printf()` comes between the call and the access by `printf()` of the string in the returned object.

### Compliant Solution (C11 and newer)

This compliant solution checks `__STDC_VERSION__` to ensure that a pre-C11 compiler will fail to compile the code, rather than invoking undefined behavior.

### Compliant Solution

This compliant solution stores the structures returned by the call to `addressee()` before calling the `printf()` function. Consequently, this program conforms to both C99 and C11.

### Noncompliant Code Example

This noncompliant code example attempts to retrieve an array and increment the array's first value. The array is part of a `struct` that is returned by a function call. Consequently, the array has temporary lifetime, and modifying the array is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) in both C99 and C11.

### Compliant Solution

This compliant solution stores the structure returned by the call to `addressee()` as `my_x` before calling the `printf()` function. When the array is modified, its lifetime is no longer temporary but matches the lifetime of the block in `main()` .

### Noncompliant Code Example

This noncompliant code example attempts to save a pointer to an array that is part of a `struct` that is returned by a function call. Consequently, the array has temporary lifetime, and using the pointer to it outside of the full expression is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) in both C99 and C11.

### Compliant Solution

This compliant solution stores the structure returned by the call to `addressee()` as `my_x` before saving a pointer to its array member. When the pointer is used, its lifetime is no longer temporary but matches the lifetime of the block in `main()` .

## EXP36-C. Do not cast pointers into more strictly aligned pointer types

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/07.exp36-c.md) — SHA-256 `a1f097e34f197324b6d2b443d8c3bb2fcf5d2192fda1083ebb5da45228baf73e`。

Do not convert a pointer value to a pointer type that is more strictly aligned than the referenced type. Different alignments are possible for different types of objects. If the type-checking system is overridden by an explicit cast or the pointer is converted to a void pointer ( `   void *  ` ) and then to a different type, the alignment of an object may be changed.

The C Standard, 6.3.2.3, paragraph 7 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> A pointer to an object type may be converted to a pointer to a different object type. If the resulting pointer is not correctly aligned for the referenced type, the behavior is undefined.

See [undefined behavior 24.](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_24)

If the misaligned pointer is dereferenced, the program may [terminate abnormally](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) . On some architectures, the cast alone may cause a loss of information even if the value is not dereferenced if the types involved have differing alignment requirements.

### Noncompliant Code Example

In this noncompliant example, the `char` pointer `&c` is converted to the more strictly aligned `int` pointer `ip` . On some [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) , `cp` will not match `&c` . As a result, if a pointer to one object type is converted to a pointer to a different object type, the second object type must not require stricter alignment than the first.

### Compliant Solution (Intermediate Object)

In this compliant solution, the `char` value is stored into an object of type `int` so that the pointer's value will be properly aligned:

### Noncompliant Code Example

The C Standard allows any object pointer to be cast to and from `   void *  ` . As a result, it is possible to silently convert from one pointer type to another without the compiler diagnosing the problem by storing or casting a pointer to `   void *  ` and then storing or casting it to the final type. In this noncompliant code example, `loop_function()` is passed the `char` pointer `char_ptr` but returns an object of type `int` pointer:

This example compiles without warning using GCC 4.8 on Ubuntu Linux 14.04. However, `int_pointer` can be more strictly aligned than an object of type `   char *  ` .

### Compliant Solution

Because the input parameter directly influences the return value, and `loop_function()` returns an object of type `   int *  ` , the formal parameter `v_pointer` is redeclared to accept only an object of type `   int *  ` :

### Noncompliant Code Example

Some architectures require that pointers are correctly aligned when accessing objects larger than a byte. However, it is common in system code that unaligned data (for example, the network stacks) must be copied to a properly aligned memory location, such as in this noncompliant code example:

Assigning an unaligned value to a pointer that references a type that needs to be aligned is [undefined behavior 24](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_24) . An [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) may notice, for example, that `tmp` and `header` must be aligned and use an inline `memcpy()` that uses instructions that assume aligned data.

### Compliant Solution

This compliant solution avoids the use of the `foo_header` pointer:

### Exceptions

**EXP36-C-EX1:** Some hardware architectures have relaxed requirements with regard to pointer alignment. Using a pointer that is not properly aligned is correctly handled by the architecture, although there might be a performance penalty. On such an architecture, improper pointer alignment is permitted but remains an efficiency problem.

The x86 32- and 64-bit architectures usually impose only a performance penalty for violations of this rule, but under some circumstances, noncompliant code can still exhibit undefined behavior. Consider the following program:

This code tries to read short ints (which are 16 bits long) from odd pairs in a character array, which violates this rule. On 32- and 64-bit x86 platforms, this program should run to completion without incident. However, the program aborts with a SIGSEGV due to the unaligned reads on a 64-bit platform running Debian Linux, when compiled with GCC 4.9.4 using the flags `-O3` or `   -O2 -ftree-loop-vectorize -fvect-cost-model  ` .

If a developer wishes to violate this rule and use undefined behavior, they must not only ensure that the hardware guarantees the behavior of the object code, but they must also ensure that their compiler, along with its optimizer, also respect these guarantees.

  

**EXP36-C-EX2** : If a pointer is known to be correctly aligned to the target type, then a cast to that type is permitted. There are several cases where a pointer is known to be correctly aligned to the target type. The pointer could point to an object declared with a suitable alignment specifier. It could point to an object returned by `aligned_alloc()` , `calloc()` , `malloc()` , or `realloc()` , as per the C standard, section 7.24.3, paragraph 1 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \].

This compliant solution uses the alignment specifier, which is new to C11, to declare the `char` object `c` with the same alignment as that of an object of type `int` . As a result, the two pointers reference equally aligned pointer types:

## EXP37-C. Call functions with the correct number and type of arguments

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/08.exp37-c.md) — SHA-256 `136d7a5995f603acb486a8574f60de93772e1e98b2d463a1be6d32fc53228a36`。

Do not call a function with the wrong number or type of arguments.

The C Standard identifies two distinct situations in which [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) (UB) may arise as a result of invoking a function using a declaration that is incompatible with its definition or by supplying incorrect types or numbers of arguments:

|                                                                                                |                                                                                                                                                                       |
|------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| UB                                                                                             | Description                                                                                                                                                           |
| [25](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_25) | *A pointer is used to call a function whose type is not compatible with the referenced type (6.3.2.3).*                                                               |
| [37](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_37) | *A function is defined with a type that is not compatible with the type (of the expression) pointed to by the expression that denotes the called function (6.5.2.2).* |

Functions that are appropriately declared (as in [DCL40-C. Do not create incompatible declarations of the same function or object](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl40-c) ) will typically generate a compiler diagnostic message if they are supplied with the wrong number or types of arguments. However, there are cases in which supplying the incorrect arguments to a function will, at best, generate compiler [warnings](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-warning) . Although such warnings should be resolved, they do not prevent program compilation. (See [MSC00-C. Compile cleanly at high warning levels](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc00-c) .)

### Noncompliant Code Example

The header `<tgmath.h>` provides type-generic macros for math functions. Although most functions from the `<math.h>` header have a complex counterpart in `<complex.h>` , several functions do not. Calling any of the following type-generic functions with complex values is [undefined behavior 205](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_205) .

**Functions That Should Not Be Called with Complex Values**

|  |  |  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ` atan2() ` | ` erf () ` | ` fdim () ` | ` fmin () ` | ` ilogb () ` | ` llround () ` | ` logb () ` | ` nextafter () ` | ` rint () ` | ` tgamma () ` |
| ` cbrt () ` | ` erfc () ` | ` floor () ` | ` fmod () ` | ` ldexp () ` | ` log10 () ` | ` lrint () ` | ` nexttoward () ` | ` round () ` | ` trunc () ` |
| ` ceil () ` | ` exp2 () ` | ` fma () ` | ` frexp () ` | ` lgamma () ` | ` log1p () ` | ` lround () ` | ` remainder () ` | ` scalbn () ` |  |
| ` copysign () ` | ` expm1 () ` | ` fmax () ` | ` hypot () ` | ` llrint () ` | ` log2 () ` | ` nearbyint () ` | ` remquo () ` | ` scalbln () ` |  |

  

This noncompliant code example attempts to take the base-2 logarithm of a complex number, resulting in undefined behavior:

### Compliant Solution (Complex Number)

If the `clog2()` function is not available for an implementation as an extension, the programmer can take the base-2 logarithm of a complex number, using `log()` instead of `log2()` , because `log()` can be used on complex arguments, as shown in this compliant solution:

### Compliant Solution (Real Number)

The programmer can use this compliant solution if the intent is to take the base-2 logarithm of the real part of the complex number:

### Noncompliant Code Example

In this noncompliant example, the C standard library function `strchr()` is called through the function pointer `fp` declared with a prototype with incorrectly typed arguments. According to the C Standard, 6.3.2.3, paragraph 8 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \]

> A pointer to a function of one type may be converted to a pointer to a function of another type and back again; the result shall compare equal to the original pointer. If a converted pointer is used to call a function whose type is not compatible with the referenced type, the behavior is undefined.

See [undefined behavior 25.](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_25)

### Compliant Solution

In this compliant solution, the function pointer `fp` , which points to the C standard library function `strchr()` , is declared with the correct parameters and is invoked with the correct number and type of arguments:

### Noncompliant Code Example

In this noncompliant example, the function `f()` is defined to take an argument of type `long` but `f()` is called from another file with an argument of type `int` :

### Compliant Solution

In this compliant solution, the prototype for the function `f()` is included in the source file in the scope of where it is called, and the function `f()` is correctly called with an argument of type `long` :

### Noncompliant Code Example (POSIX)

The POSIX function `open()` \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \] is a variadic function with the following prototype:

The `open()` function accepts a third argument to determine a newly created file's access mode. If `open()` is used to create a new file and the third argument is omitted, the file may be created with unintended access permissions. (See [FIO06-C. Create files with appropriate access permissions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio06-c) .)

In this noncompliant code example from a [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) in the `useradd()` function of the `shadow-utils` package [CVE-2006-1174](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2006-1174) , the third argument to `open()` is accidentally omitted:

Technically, it is incorrect to pass a third argument to open() when not creating a new file (that is, with the O_CREAT flag not set).

### Compliant Solution (POSIX)

In this compliant solution, a third argument is specified in the call to `open()` :

## EXP39-C. Do not access a variable through a pointer of an incompatible type

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/09.exp39-c.md) — SHA-256 `f16493aa6c57b2b257a78371605b9be05c049fcfa7fe544198e40abbabfb73c3`。

Modifying a variable through a pointer of an incompatible type (other than `   unsigned char  ` ) can lead to unpredictable results. Subclause 6.2.7 of the C Standard states that two types may be distinct yet compatible and addresses precisely when two distinct types are compatible.

This problem is often caused by a violation of aliasing rules. The C Standard, 6.5, paragraph 7 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], specifies those circumstances in which an object may or may not be aliased.

> An object shall have its stored value accessed only by an lvalue expression that has one of the following types:
>
> -   a type compatible with the effective type of the object,
> -   a qualified version of a type compatible with the effective type of the object,
> -   the signed or unsigned type compatible with the underlying type of the effective type of the object,
> -   the signed or unsigned type compatible with a qualified version of the underlying type of the effective type of the object,
> -   an aggregate or union type that includes one of the aforementioned types among its members (including, recursively, a member of a subaggregate or contained union), or
> -   a character type.

Accessing an object by means of any other [lvalue](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-lvalue) expression (other than `   unsigned char  ` ) is [undefined behavior 36](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_36) .

### Noncompliant Code Example

In this noncompliant example, an object of type `float` is incremented through an `   int *  ` . The programmer can use the unit in the last place to get the next representable value for a floating-point type.  However, accessing an object through a pointer of an incompatible type is undefined behavior.

### Compliant Solution

In this compliant solution, the standard C function `nextafterf()` is used to round toward the highest representable floating-point value:

### Noncompliant Code Example

In this noncompliant code example, an array of two values of type `short` is treated as an integer and assigned an integer value. The resulting values are indeterminate.

When translating this code, an implementation can assume that no access through an integer pointer can change the array `a` , consisting of shorts. Consequently, `printf()` may be called with the original values of `a[0]` and `a[1]` .

#### Implementation Details

Recent versions of GCC turn on the option `-fstrict-aliasing,` which allows alias-based optimizations, by default with `-O2` . Some architectures then print "1111 1111" as a result. Without optimization, the executable generates the *expected* output "2222 2222."

To disable optimizations based on alias analysis for faulty legacy code, the option `-fno-strict-aliasing` can be used as a workaround. The option `-Wstrict-aliasing,` which is included in `-Wall,` warns about some, but not all, violations of aliasing rules when `-fstrict-aliasing` is active.

When GCC 3.4.6 compiles this code with optimization, the assignment through the aliased pointer is effectively eliminated.

### Compliant Solution

This compliant solution uses a `union` type that includes a type compatible with the effective type of the object:

The C standard states:

> If the member used to read the contents of a union object is not the same as the member last used to store a value in the object, the appropriate part of the object representation of the value is reinterpreted as an object representation in the new type as described in 6.2.6 (a process sometimes called "type punning"). This may be a non-value representation.

The call to `printf()` typically outputs "2222 2222". However, there is no guarantee that this will be true; the object representations of `a` and `i` are unspecified and need not be compatible in this way, despite this operation being commonly accepted as an implementation extension. (See [unspecified behavior 11](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/dd-unspecified-behavior#DD.UnspecifiedBehavior-unspecifiedbehavior11) .)

### Noncompliant Code Example

In this noncompliant code example, a `gadget` object is allocated, then `realloc()` is called to create a `widget` object using the memory from the `gadget` object. Although reusing memory to change types is acceptable, accessing the memory copied from the original object is undefined behavior.

### Compliant Solution

This compliant solution reuses the memory from the `gadget` object but reinitializes the memory to a consistent state before reading from it:

### Noncompliant Code Example

According to the C Standard, 6.7.7.3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], using two or more incompatible arrays in an expression is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See also [undefined behavior 73](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_73) .)

For two array types to be compatible, both should have compatible underlying element types, and both size specifiers should have the same constant value. If either of these properties is violated, the resulting behavior is undefined.

In this noncompliant code example, the two arrays `a` and `b` fail to satisfy the equal size specifier criterion for array compatibility. Because `a` and `b` are not equal, writing to what is believed to be a valid member of `a` might exceed its defined memory boundary, resulting in an arbitrary memory overwrite.

Most compilers will produce a warning diagnostic if the two array types used in an expression are incompatible.

### Compliant Solution

In this compliant solution, `b` is declared to point to an array with the same number of elements as `a` , satisfying the size specifier criterion for array compatibility:

## EXP40-C. Do not modify constant objects

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/10.exp40-c.md) — SHA-256 `bd4aadf890c2acae48b3b288448ce9c38ef7763f2a0dbcb3c4cd5e0ba4683c6a`。

The C Standard, 6.7.4, paragraph 7 \[ [IS](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) [O/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> If an attempt is made to modify an object defined with a `const` -qualified type through use of an [lvalue](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-lvalue) with non- `const` -qualified type, the behavior is undefined.

See also [undefined behavior 61](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_61) .

There are existing compiler [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) that allow `const` -qualified objects to be modified without generating a warning message.

Avoid casting away `const` qualification because doing so makes it possible to modify `const` -qualified objects without issuing diagnostics. (See [EXP05-C. Do not cast away a const qualification](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/expressions-exp/exp05-c) and [STR30-C. Do not attempt to modify string literals](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/characters-and-strings-str/str30-c) for more details.)

### Noncompliant Code Example

This noncompliant code example allows a constant object to be modified:

The first assignment is unsafe because it allows the code that follows it to attempt to change the value of the `const` object `i` .

#### Implementation Details

If `ipp` , `ip` , and `i` are declared as automatic variables, this example compiles without warning with Microsoft Visual Studio 2013 when compiled in C mode ( `/TC` ) and the resulting program changes the value of `i` . GCC 4.8.1 generates a warning but compiles, and the resulting program changes the value of `i` .

If `ipp` , `ip` , and `i` are declared with static storage duration, this program compiles without warning and terminates abnormally with Microsoft Visual Studio 2013, and compiles with warning and terminates abnormally with GCC 4.8.1.

### Compliant Solution

The compliant solution depends on the intent of the programmer. If the intent is that the value of `i` is modifiable, then it should not be declared as a constant, as in this compliant solution:

If the intent is that the value of i is not meant to change, then do not write noncompliant code that attempts to modify it.

## EXP42-C. Do not compare padding data

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/11.exp42-c.md) — SHA-256 `971ea85b965e21e69f136c36e25fab52ab36cfa8480add928d094ae2708c2046`。

The C Standard, 6.7.3.2 paragraph 19 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> There may be unnamed padding within a structure object, but not at its beginning. . . . There may be unnamed padding at the end of a structure or union.

Subclause 6.7.11, paragraph 10, states that

> unnamed members of objects of structure and union type do not participate in initialization. Unnamed members of structure objects have indeterminate representation even after initialization.

The only exception is that padding bits are set to zero when a static or thread-local object is implicitly initialized (paragraph 11):

> If an object that has static or thread storage duration is not initialized explicitly, or any object is initialized with an empty initializer, then it is subject to default initialization , which initializes an object as follows:
>
> —  if it is an aggregate, every member is initialized (recursively) according to these rules, and any padding is initialized to zero bits;
>
> —  if it is a union, the first named member is initialized (recursively) according to these rules, and any padding is initialized to zero bits.
>
>   

Because these padding values are unspecified, attempting a byte-by-byte comparison between structures can lead to incorrect results \[ [Summit 1995](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Summit95) \].

### Noncompliant Code Example

In this noncompliant code example, `memcmp()` is used to compare the contents of two structures, including any padding bytes:

### Compliant Solution

In this compliant solution, all of the fields are compared manually to avoid comparing any padding bytes:

### Exceptions

**EXP42-C-EX1** : A structure can be defined such that the members are aligned properly or the structure is packed using implementation-specific packing instructions. This is true only when the members' data types have no padding bits of their own and when their object representations are the same as their value representations. This frequently is not true for the `_Bool` type or floating-point types and need not be true for pointers. In such cases, the compiler does not insert padding, and use of functions such as `memcmp()` is acceptable.

This compliant example uses the [`    #pragma pack   `](https://msdn.microsoft.com/en-us/library/2e70t5y1.aspx) compiler extension from Microsoft Visual Studio to ensure the structure members are packed as tightly as possible:

## EXP43-C. Avoid undefined behavior when using restrict-qualified pointers

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/12.exp43-c.md) — SHA-256 `3f0e63a1b65ba0ea269ac2dd02fe303c4c7dcfcfa6e074daa48b69b88114e2f5`。

An object that is accessed through a `restrict` -qualified pointer has a special association with that pointer. This association requires that all accesses to that object use, directly or indirectly, the value of that particular pointer. The intended use of the restrict qualifier is to promote optimization, and deleting all instances of the qualifier from a program does not change its meaning (that is, observable behavior). In the absence of this qualifier, other pointers can alias this object. Caching the value in an object designated through a `restrict` -qualified pointer is safe at the beginning of the block in which the pointer is declared because no preexisting aliases may also be used to reference that object. The cached value must be restored to the object by the end of the block, where preexisting aliases again become available. New aliases may be formed within the block, but these must all depend on the value of the `restrict` -qualified pointer so that they can be identified and adjusted to refer to the cached value. For a `restrict` -qualified pointer at file scope, the block is the body of each function in the file \[ [Walls 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography) \]. Developers should be aware that C++ does not support the `restrict` qualifier, but some C++ compiler implementations support an equivalent qualifier as an extension.

The C Standard \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \] identifies the following [undefined behavior 66](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_66) :

> A restrict-qualified pointer is assigned a value based on another restricted pointer whose associated block neither began execution before the block associated with this pointer, nor ended before the assignment (6.7.4.2).

This is an oversimplification, however, and it is important to review the formal definition of *restrict* in subclause 6.7.4.2 of the C Standard to properly understand undefined behaviors associated with the use of `restrict` -qualified pointers.

### Noncompliant Code Example

This code example is noncompliant because an assignment is made between two `restrict` -qualified pointers in the same scope:

Note that [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) occurs only when `a` is assigned to `b` . It is valid for `a` and `b` to point into the same array object, provided the range of elements accessed through one of the pointers does not overlap with the range of elements accessed through the other pointer.

### Compliant Solution

One way to eliminate the [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) is simply to remove the `restrict-` qualification from the affected pointers:

### `restrict` -Qualified Function Parameters

The `restrict` qualifier requires that the pointers do not reference overlapping objects. If the objects referenced by arguments to functions overlap (meaning the objects share some common memory addresses), the behavior is undefined.

When calling functions that have `restrict` -qualified function parameters, it is important that the pointer arguments do not reference overlapping objects if one or more of the pointers are used to modify memory. Consequently, it is important to understand the semantics of the function being called.

### Noncompliant Code Example

In this noncompliant code example, the function `f()` accepts three parameters. The function copies `n` integers from the `int` array referenced by the `restrict` -qualified pointer `p` to the `int` array referenced by the `restrict` -qualified pointer `q` . Because the destination array is modified during each execution of the function (for which `n` is nonzero), if the array is accessed through one of the pointer parameters, it cannot also be accessed through the other. Declaring these function parameters as `restrict` -qualified pointers allows aggressive optimization by the compiler but can also result in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) if these pointers refer to overlapping objects.

The function `g()` declares an array `d` consisting of 100 `int` values and then invokes `f()` to copy memory from one area of the array to another. This call has undefined behavior because each of `d[1]` through `d[49]` is accessed through both `p` and `q` .

### Compliant Solution

In this compliant solution, the function `f()` is unchanged but the programmer has ensured that none of the calls to `f()` result in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . The call to `f()` in `g()` is valid because the storage allocated to `d` is effectively divided into two disjoint objects.

### Noncompliant Code Example

In this noncompliant code example, the function `add()` adds the integer array referenced by the `restrict` -qualified pointers lhs to the integer array referenced by the `restrict` -qualified pointer `rhs` and stores the result in the `restrict` -qualified pointer referenced by `res` . The function `f()` declares an array `a` consisting of 100 `int` values and then invokes `add()` to copy memory from one area of the array to another. The call `   add(100, a, a, a)  ` has [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) because the object modified by `res` is accessed by lhs and `rhs` .

### Compliant Solution

In this compliant solution, an unmodified object is aliased through two restricted pointers. Because `a` and `b` are disjoint arrays, a call of the form `   add(100, a, b, b)  ` has defined behavior, because array `b` is not modified within function `add` .

### Invoking Library Functions with `restrict` -Qualified Pointers

Ensure that `restrict` -qualified source and destination pointers do not reference overlapping objects when invoking library functions. For example, the following table lists C standard library functions that copy memory from a source object referenced by a `restrict` -qualified pointer to a destination object that is also referenced by a `restrict` -qualified pointer:

|                        |
|------------------------|
| Standard C             |
| `strcpy()`  |
| `strncpy()` |
| `strcat()`  |
| `strncat()` |
| `memcpy()`  |

If the objects referenced by arguments to functions overlap (meaning the objects share some common memory addresses), the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See also [undefined behavior 65](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_65) .) The result of the functions is unknown, and data may be corrupted. As a result, these functions must never be passed pointers to overlapping objects. If data must be copied between objects that share common memory addresses, a copy function guaranteed to work on overlapping memory, such as `memmove()` , should be used.

### Noncompliant Code Example

In this noncompliant code example, the values of objects referenced by `ptr1` and `ptr2` become unpredictable after the call to `memcpy()` because their memory areas overlap:

### Compliant Solution

In this compliant solution, the call to `memcpy()` is replaced with a call to `memmove()` . The `memmove()` function performs the same operation as `memcpy()` when the memory regions do not overlap. When the memory regions do overlap, the *n* characters from the object pointed to by the source ( `ptr1` ) are first copied into a temporary array of *n* characters that does not overlap the objects pointed to by the destination ( `ptr2` ) or the source. The *n* characters from the temporary array are then copied into the object pointed to by the destination.

Similar solutions using `memmove()` can replace the string functions as long as care is taken regarding the byte size of the characters and proper null-termination of the copied string.

### Calling Functions with `restrict` -Qualified Pointer to a `const` -Qualified Type

Ensure that functions that accept a `restrict` -qualified pointer to a `const` -qualified type do not modify the object referenced by that pointer. Formatted input and output standard library functions frequently fit this description. The following table lists of some of the common functions for which the format argument is a `restrict` -qualified pointer to a `const` -qualified type.

|                         |
|-------------------------|
| Standard C              |
| `printf()`   |
| `scanf()`    |
| `sprintf()`  |
| `snprintf()` |

For formatted output functions such as `printf()` , it is unlikely that a programmer would modify the format string. However, an attacker may attempt to do so if a program violates [FIO30-C. Exclude user input from format strings](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/input-output-fio/fio30-c) and passes tainted values as part of the format string.

### Noncompliant Code Example

In this noncompliant code example, the programmer is attempting to overwrite the format string with a string value read in from `stdin` such as `   "%d%f 1 3.3"  ` and use the resulting modified string of `"%s%d%f"` to input the subsequent values of `1` and `3.3` :

### Compliant Solution

The intended results are achieved by this compliant solution:

### Outer-to-Inner Assignments between Restricted Pointers

The assignment between `restrict` -qualified pointers declared in an inner nested block from an outer block has defined behavior.

### Noncompliant Code Example

The assignment of `restrict` -qualified pointers to other `restrict` -qualified pointers within the same block has [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) :

### Compliant Solution

The intended results can be achieved using an inner nested block, as shown in this compliant solution:

## EXP44-C. Do not rely on side effects in operands to sizeof, \_Alignof, or \_Generic

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/13.exp44-c.md) — SHA-256 `30a29945be88c391c8c856a58ee2acdd7adf5469bba34b9b5d6566fc03871b12`。

Some operators do not evaluate their operands beyond the type information the operands provide. When using one of these operators, do not pass an operand that would otherwise yield a side effect since the side effect will not be generated.

The `sizeof` operator yields the size (in bytes) of its operand, which may be an expression or the parenthesized name of a type.  In most cases, the operand is not evaluated.  A possible exception is when the type of the operand is a variable length array type (VLA); then the expression is evaluated. When part of the operand of the sizeof operator is a VLA type and when changing the value of the VLA's size expression would not affect the result of the operator, it is [unspecified](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unspecifiedbehavior) whether or not the size expression is evaluated. (See [unspecified behavior 22](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/dd-unspecified-behavior#DD.UnspecifiedBehavior-usb_22) .)

The operand passed to `_Alignof` is never evaluated, despite not being an expression. For instance, if the operand is a VLA type and the VLA's size expression contains a side effect, that side effect is never evaluated.

The operand used in the controlling expression of a `_Generic` selection expression is never evaluated.

Providing an expression that appears to produce side effects may be misleading to programmers who are not aware that these expressions are not evaluated, and in the case of a VLA used in `sizeof` , have unspecified results. As a result, programmers may make invalid assumptions about program state, leading to errors and possible software [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) .

This rule is similar to [PRE31-C. Avoid side effects in arguments to unsafe macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/preprocessor-pre/pre31-c) .

### Noncompliant Code Example ( `sizeof` )

In this noncompliant code example, the expression `a++` is not evaluated:

Consequently, the value of `a` after `b` has been initialized is 14.

### Compliant Solution ( `sizeof` )

In this compliant solution, the variable `a` is incremented outside of the `sizeof` operation:

### Noncompliant Code Example ( `sizeof` , VLA)

In this noncompliant code example, the expression `++n` in the initialization expression of `a` must be evaluated because its value affects the size of the VLA operand of the `sizeof` operator. However, in the initialization expression of `b` , the expression `   ++n % 1  ` evaluates to `0.` This means that the value of `n` does not affect the result of the `sizeof` operator. Consequently, it is unspecified whether or not `n` will be incremented when initializing `b` .

### Compliant Solution ( `sizeof` , VLA)

This compliant solution avoids changing the value of the variable `n` used in each `sizeof` expression and instead increments `n` safely afterwards:

### Noncompliant Code Example ( `_Generic` )

This noncompliant code example attempts to modify a variable's value as part of the `_Generic` selection control expression. The programmer may expect that `a` is incremented, but because `_Generic` does not evaluate its control expression, the value of `a` is not modified.

### Compliant Solution ( `_Generic` )

In this compliant solution, a is incremented outside of the `_Generic` selection expression:

### Noncompliant Code Example ( `_Alignof` )

This noncompliant code example attempts to modify a variable while getting its default alignment value. The user may have expected `val` to be incremented as part of the `_Alignof` expression, but because `_Alignof` does not evaluate its operand, `val` is unchanged.

### Compliant Solution ( `_Alignof` )

This compliant solution moves the expression out of the `_Alignof` operator:

### Exceptions

**EXP44-C-EX1** : Reading a `volatile` -qualified value is a side-effecting operation. However, accessing a value through a `volatile` -qualified type does not guarantee side effects will happen on the read of the value unless the underlying object is also `volatile` -qualified. Idiomatic reads of a `volatile` -qualified object are permissible as an operand to a `sizeof()` , `_Alignof()` , or `_Generic` expression, as in the following example:

## EXP45-C. Do not perform assignments in selection statements

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/14.exp45-c.md) — SHA-256 `087892e48f94481d09d0c60b8fc81bc2d3c06fd9f371c3451678e709fb606de7`。

Do not use the assignment operator in the contexts listed in the following table because doing so typically indicates programmer error and can result in [unexpected behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unexpectedbehavior) .

|                           |                                                                                         |
|---------------------------|-----------------------------------------------------------------------------------------|
| Operator                  | Context                                                                                 |
| `if`           | Controlling expression                                                                  |
| `while`        | Controlling expression                                                                  |
| `      do ... while     ` | Controlling expression                                                                  |
| `for`          | Second operand                                                                          |
| `?:`           | First operand                                                                           |
| `?:`           | Second or third operands, where the ternary expression is used in any of these contexts |
| `&&`           | Either operand                                                                          |
| `||`           | either operand                                                                          |
| `,`            | Second operand, when the comma expression is used in any of these contexts              |

  

Performing assignment statements in other contexts do not violate this rule. However, they may violate other rules, such as [EXP30-C. Do not depend on the order of evaluation for side effects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp30-c) .

  

Noncompliant Code Example

In this noncompliant code example, an assignment expression is the outermost expression in an `if` statement:

Although the intent of the code may be to assign `b` to `a` and test the value of the result for equality to 0, it is frequently a case of the programmer mistakenly using the assignment operator `=` instead of the equals operator `==` . Consequently, many compilers will warn about this condition, making this coding error detectable by adhering to [MSC00-C. Compile cleanly at high warning levels](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc00-c) .

### Compliant Solution (Unintentional Assignment)

When the assignment of `b` to `a` is not intended, the conditional block is now executed when `a` is equal to `b` :

### Compliant Solution (Intentional Assignment)

When the assignment is intended, this compliant solution explicitly uses inequality as the outermost expression while performing the assignment in the inner expression:

It is less desirable in general, depending on what was intended, because it mixes the assignment in the condition, but it is clear that the programmer intended the assignment to occur.

### Noncompliant Code Example

In this noncompliant code example, the expression `   x = y  ` is used as the controlling expression of the `while` statement:

### Compliant Solution (Unintentional Assignment)

When the assignment of y to x is not intended, the conditional block should be executed only when x is equal to y , as in this compliant solution:

### Compliant Solution (Intentional Assignment)

When the assignment is intended, this compliant solution can be used:

### Compliant Solution ( `for` statement)

The same result can be obtained using the `for` statement, which is specifically designed to evaluate an expression on each iteration of the loop, just before performing the test in its controlling expression. Remember that its controlling expression is the second operand, where the assignment occurs in its third operand:

### Noncompliant Code Example

In this noncompliant example, the expression `   p = q  ` is used as the controlling expression of the `while` statement:

### Compliant Solution

In this compliant solution, the expression `   x = y  ` is not used as the controlling expression of the `while` statement:

### Noncompliant Code Example

This noncompliant code example has a typo that results in an assignment rather than a comparison.

Many compilers will warn about this condition. This coding error would typically be eliminated by adherence to [MSC00-C. Compile cleanly at high warning levels](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc00-c) . Although this code compiles, it will cause [unexpected behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unexpectedbehavior) to an unsuspecting programmer. If the intent was to verify a string such as a password, user name, or group user ID, the code may produce significant [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) and require significant debugging.

### Compliant Solution (RHS Variable)

When comparisons are made between a variable and a literal or const-qualified variable, placing the variable on the right of the comparison operation can prevent a spurious assignment.

In this code example, the literals are placed on the left-hand side of each comparison. If the programmer were to inadvertently use an assignment operator, the statement would assign `ch` to `'\t'` , which is invalid and produces a diagnostic message.

Due to the diagnostic, the typo will be easily spotted and fixed.

As a result, any mistaken use of the assignment operator that could otherwise create a [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) for operations such as string verification will result in a compiler diagnostic regardless of compiler, warning level, or [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) .

### Exceptions

**EXP45-C-EX1** : Assignment can be used where the result of the assignment is itself an operand to a comparison expression or relational expression. In this compliant example, the expression `   x = y  ` is itself an operand to a comparison operation:

**EXP45-C-EX2** : Assignment can be used where the expression consists of a single primary expression. The following code is compliant because the expression `   x = y  ` is a single primary expression:

The following controlling expression is noncompliant because `&&` is not a comparison or relational operator and the entire expression is not primary:

When the assignment of `v` to `w` is not intended, the following controlling expression can be used to execute the conditional block when `v` is equal to `w` :

When the assignment is intended, the following controlling expression can be used:

**EXP45-C-EX3** : Assignment can be used in a function argument or array index. In this compliant solution, the expression `   x = y  ` is used in a function argument:

## EXP46-C. Do not use a bitwise operator with a Boolean-like operand

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/15.exp46-c.md) — SHA-256 `86889dad136de0b5c8b4c10723851eb82da1de6a01085621ba473147c2f24504`。

Mixing bitwise and relational operators in the same full expression can be a sign of a logic error in the expression where a logical operator is usually the intended operator. Do not use the bitwise AND ( `&` ), bitwise OR ( `|` ), or bitwise XOR ( `^` ) operators with an operand of type `_Bool` , or the result of a *relational-expression* or *equality-expression* . If the bitwise operator is intended, it should be indicated with use of a parenthesized expression.

### Noncompliant Code Example

In this noncompliant code example, a bitwise `&` operator is used with the results of two *equality-expressions* :

### Compliant Solution

This compliant solution uses the `&&` operator for the logical operation within the conditional expression:

## EXP47-C. Do not call va_arg with an argument of the incorrect type

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/09.expressions-exp/16.exp47-c.md) — SHA-256 `3f22d960f4aae1ad32fd0933c015106286336de90f1f92a7477fc6c9b79ec449`。

The variable arguments passed to a variadic function are accessed by calling the `va_arg()` macro. This macro accepts the `va_list` representing the variable arguments of the function invocation and the type denoting the expected argument type for the argument being retrieved. The macro is typically invoked within a loop, being called once for each expected argument. However, there are no type safety guarantees that the type passed to `va_arg` matches the type passed by the caller, and there are generally no compile-time checks that prevent the macro from being invoked with no argument available to the function call.

The C Standard, 7.16.1.1, states \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], in part:

> If type is not compatible with the type of the actual next argument (as promoted according to the default argument promotions), the behavior is undefined, except for the following cases:
>
> —  both types are pointers to qualified or unqualified versions of compatible types;
>
> —  one type is compatible with a signed integer type, the other type is compatible with the corresponding unsigned integer type, and the value is representable in both types;
>
> —  one type is pointer to qualified or unqualified void and the other is a pointer to a qualified or unqualified character type;
>
> —  or, the type of the next argument is nullptr \_ t and type is a pointer type that has the same representation and alignment requirements as a pointer to a character type.

Ensure that an invocation of the `va_arg()` macro does not attempt to access an argument that was not passed to the variadic function. Further, the type passed to the `va_arg()` macro must match the type passed to the variadic function after default argument promotions have been applied. Either circumstance results in [undefined behavior 141](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_141) .

### Noncompliant Code Example

This noncompliant code example attempts to read a variadic argument of type `   unsigned char  `with`   va_arg()  ` . However, when a value of type `   unsigned char  ` is passed to a variadic function, the value undergoes default argument promotions, resulting in a value of type `int` being passed.

### Compliant Solution

The compliant solution accesses the variadic argument with type `int` , and then casts the resulting value to type `   unsigned char  ` :

### Noncompliant Code Example

This noncompliant code example assumes that at least one variadic argument is passed to the function, and attempts to read it using the `va_arg()` macro. This pattern arises frequently when a variadic function uses a sentinel value to denote the end of the variable argument list. However, the caller passes no variadic arguments to the function, which results in undefined behavior.

### Compliant Solution

Standard C provides no mechanism to enable a variadic function to determine how many variadic arguments are actually provided to the function call. That information must be passed in an out-of-band manner. Oftentimes this results in the information being encoded in the initial parameter, as in this compliant solution:
