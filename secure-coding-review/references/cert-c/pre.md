# CERT C — PRE

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## PRE00-C. Prefer inline or static functions to function-like macros

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/17.preprocessor-pre/02.pre00-c.md) — SHA-256 `466aa7f2827d3f264bf7d55f331b325bfb292e9217b51bad024a65fd4b3d42d5`。

Macros are dangerous because their use resembles that of real functions, but they have different semantics. The inline function-specifier was introduced to the C programming language in the C99 standard. Inline functions should be preferred over macros when they can be used interchangeably. Making a function an inline function suggests that calls to the function be as fast as possible by using, for example, an alternative to the usual function call mechanism, such as *inline substitution* . (See also [PRE31-C. Avoid side effects in arguments to unsafe macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/preprocessor-pre/pre31-c) , [PRE01-C. Use parentheses within macros around parameter names](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre01-c) , and [PRE02-C. Macro replacement lists should be parenthesized](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre02-c) .)

Inline substitution is not textual substitution, nor does it create a new function. For example, the expansion of a macro used within the body of the function uses the definition it had at the point the function body appeared, not where the function is called; and identifiers refer to the declarations in scope where the body occurs.

Arguably, a decision to inline a function is a low-level optimization detail that the compiler should make without programmer input. The use of inline functions should be evaluated on the basis of (a) how well they are supported by targeted compilers, (b) what (if any) impact they have on the performance characteristics of your system, and (c) portability concerns. Static functions are often as good as inline functions and are supported in C.

### Noncompliant Code Example

In this noncompliant code example, the macro `CUBE()` has [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) when passed an expression that contains side effects:

For this example, the initialization for `a` expands to

which is undefined (see [EXP30-C. Do not depend on the order of evaluation for side effects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp30-c) ).

### Compliant Solution

When the macro definition is replaced by an inline function, the [side effect](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sideeffect) is executed only once before the function is called:

### Noncompliant Code Example

In this noncompliant code example, the programmer has written a macro called `EXEC_BUMP()` to call a specified function and increment a global counter \[ [Dewhurst 2002](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Dewhurst02) \]. When the expansion of a macro is used within the body of a function, as in this example, identifiers refer to the declarations in scope where the body occurs. As a result, when the macro is called in the `aFunc()` function, it inadvertently increments a local counter with the same name as the global variable. Note that this example also violates [DCL01-C. Do not reuse variable names in subscopes](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl01-c) .

The result is that invoking `aFunc()` (incorrectly) prints out the following line five times:

### Compliant Solution

In this compliant solution, the `EXEC_BUMP()` macro is replaced by the inline function `exec_bump()` . Invoking `aFunc()` now (correctly) prints the value of `count` ranging from 0 to 9:

The use of the inline function binds the identifier `count` to the global variable when the function body is compiled. The name cannot be re-bound to a different variable (with the same name) when the function is called.

### Noncompliant Code Example

Unlike functions, the execution of macros can interleave. Consequently, two macros that are harmless in isolation can cause [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) when combined in the same expression. In this example, `F()` and `G()` both increment the global variable `operations` , which causes problems when the two macros are used together:

The variable `operations` is both read and modified twice in the same expression, so it can receive the wrong value if, for example, the following ordering occurs:

<div class="preformatted panel" style="border-width: 1px;">

<div class="preformattedContent panelContent">

    read operations into register 0
    read operations into register 1
    increment register 0
    increment register 1
    store register 0 into operations
    store register 1 into operations

</div>

</div>

This noncompliant code example also violates [EXP30-C. Do not depend on the order of evaluation for side effects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp30-c) .

### Compliant Solution

The execution of functions, including inline functions, cannot be interleaved, so problematic orderings are not possible:

#### Platform-Specific Details

GNU C (and some other compilers) supported inline functions before they were added to the C Standard and, as a result, have significantly different semantics. Richard Kettlewell provides a good explanation of differences between the C99 and GNU C rules \[ [Kettlewell 2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Kettle03) \].

### Exceptions

**PRE00-C-EX1:** Macros can be used to implement *local functions* (repetitive blocks of code that have access to automatic variables from the enclosing scope) that cannot be achieved with inline functions.

**PRE00-C-EX2:** Macros can be used for concatenating tokens or performing stringification. For example,

calculates only one of the two expressions depending on the selector's value. See [PRE05-C. Understand macro replacement when concatenating tokens or performing stringification](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre05-c) for more information.

**PRE00-C-EX3:** Macros can be used to yield a compile-time constant. This is not always possible using inline functions, as shown by the following example:

In this example, the `ADD_M(3,4)` macro invocation yields a constant expression, but the `add_f(3,4)` function invocation does not.

**PRE00-C-EX4:** Macros can be used to implement type-generic functions that cannot be implemented in the C language without the aid of a mechanism such as C++ templates.

An example of the use of [function-like macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-function-likemacro) to create type-generic functions is shown in [MEM02-C. Immediately cast the result of a memory allocation function call into a pointer to the allocated type](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/memory-management-mem/mem02-c) .

Type-generic macros may also be used, for example, to swap two variables of any type, provided they are of the same type.

**PRE00-C-EX5:** Macro parameters exhibit call-by-name semantics, whereas functions are call by value. Macros must be used in cases where call-by-name semantics are required.

## PRE01-C. Use parentheses within macros around parameter names

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/17.preprocessor-pre/03.pre01-c.md) — SHA-256 `5191e32f714b89146756fc0f6e081feb9d0cbc26bec5aafdde5092f3f45cf263`。

Parenthesize all parameter names in macro definitions. See also [PRE00-C. Prefer inline or static functions to function-like macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre00-c) and [PRE02-C. Macro replacement lists should be parenthesized](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre02-c) .

### Noncompliant Code Example

This `CUBE()` macro definition is noncompliant because it fails to parenthesize the parameter names:

As a result, the invocation

expands to

which is clearly not the desired result.

### Compliant Solution

Parenthesizing all parameter names in the `CUBE()` macro allows it to expand correctly (when invoked in this manner):

### Exceptions

**PRE01-C-EX1:** When the parameter names are surrounded by commas in the replacement text, regardless of how complicated the actual arguments are, there is no need for parenthesizing the macro parameters. Because commas have lower precedence than any other operator, there is no chance of the actual arguments being parsed in a surprising way. Comma separators, which separate arguments in a function call, also have lower precedence than other operators, although they are technically different from comma operators.

**PRE01-C-EX2:** Macro parameters cannot be individually parenthesized when concatenating tokens using the `##` operator, converting macro parameters to strings using the `#` operator, or concatenating adjacent string literals. The following `JOIN()` macro concatenates both arguments to form a new token. The `SHOW()` macro converts the single argument into a string literal, which is then passed as a parameter to `printf()` and as a string and as a parameter to the `%d` specifier. For example, if `SHOW()` is invoked as `SHOW(66);` , the macro would be expanded to `   printf("66" " = %d\n", 66);  `.

See [PRE05-C. Understand macro replacement when concatenating tokens or performing stringification](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre05-c) for more information on using the `##` operator to concatenate tokens.

## PRE02-C. Macro replacement lists should be parenthesized

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/17.preprocessor-pre/04.pre02-c.md) — SHA-256 `5a2e26c2c200eba56be30e69d1578616accaa7134ccf68849204dba806547943`。

Macro replacement lists should be parenthesized to protect any lower-precedence operators from the surrounding expression. See also [PRE00-C. Prefer inline or static functions to function-like macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre00-c) and [PRE01-C. Use parentheses within macros around parameter names](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre01-c) .

### Noncompliant Code Example

This `CUBE()` macro definition is noncompliant because it fails to parenthesize the replacement list:

As a result, the invocation

expands to

which evaluates as

which is not the desired behavior.

### Compliant Solution

With its replacement list parenthesized, the `CUBE()` macro expands correctly for this type of invocation.

This compliant solution violates [PRE00-C. Prefer inline or static functions to function-like macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre00-c) . Consequently, this solution would be better implemented as an inline function.

### Noncompliant Code Example

In this noncompliant code example, `END_OF_FILE` is defined as `-1` . The macro replacement list consists of a unary negation operator followed by an integer literal 1:

In this example, the programmer has mistakenly omitted the comparison operator from the conditional statement, which should be `   getchar() != END_OF_FILE  ` . (See [void MSC02-C. Avoid errors of omission](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/the-void/void-msc02-c-avoid-errors-of-omission) .) After macro expansion, the conditional expression is incorrectly evaluated as a binary operation: `getchar()-1` . This statement is syntactically correct, even though it is certainly not what the programmer intended. Note that this example also violates [DCL00-C. Const-qualify immutable objects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl00-c) .

Parenthesizing the `-1` in the declaration of `END_OF_FILE` ensures that the macro expansion is evaluated correctly:

Once this modification is made, the noncompliant code example no longer compiles because the macro expansion results in the conditional expression `   getchar() (-1)  ` , which is no longer syntactically valid. Note that there must be a space after `END_OF_FILE` because, otherwise, it becomes a [function-like macro](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-function-likemacro) (and one that is incorrectly formed because −1 cannot be a formal parameter).

### Compliant Solution

In this compliant solution, the macro definition is replaced with an enumeration constant in compliance with [DCL00-C. Const-qualify immutable objects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl00-c) . In addition, because `EOF` is a reserved macro defined in the `<stdio.h>` header, the compliant solution must also use a different indentifier in order to comply with [DCL37-C. Do not declare or define a reserved identifier](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl37-c) .

### Exceptions

**PRE02-C-EX1:** A macro that expands to a single identifier or function call is not affected by the precedence of any operators in the surrounding expression, so its replacement list need not be parenthesized.

**PRE02-C-EX2:** A macro that expands to an array reference using the array-subscript operator `[]` , or an expression designating a member of a structure or union object using either the member-access `.` or `->` operators is not affected by the precedence of any operators in the surrounding expression, so its replacement list need not be parenthesized.

## PRE04-C. Do not reuse a standard header file name

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/17.preprocessor-pre/05.pre04-c.md) — SHA-256 `753daa75e4972cda1c93aaec3c9483c410dda5672e6e75038fc23d703bb24beb`。

If a file with the same name as a standard header is placed in the search path for included source files, the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

The following table from the C Standard, subclause 7.1.2 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], lists these standard headers:

| `<assert.h>` | `<float.h>` | `<math.h>` | `<stdatomic.h>` | `<stdlib.h>` | `<time.h>` |
| --- | --- | --- | --- | --- | --- |
| `<complex.h>` | `<inttypes.h>` | `<setjmp.h>` | `<stdbool.h>` | `<stdnoreturn.h>` | `<uchar.h>` |
| `<ctype.h>` | `<iso646.h>` | `<signal.h>` | `<stddef.h>` | `<string.h>` | `<wchar.h>` |
| `<errno.h>` | `<limits.h>` | `<stdalign.h>` | `<stdint.h>` | `<tgmath.h>` | `<wctype.h>` |
| `<fenv.h>` | `<locale.h>` | `<stdarg.h>` | `<stdio.h>` | `<threads.h>` |  |

Do not reuse standard header file names, system-specific header file names, or other header file names.

### Noncompliant Code Example

In this noncompliant code example, the programmer chooses to use a local version of the standard library but does not make the change clear:

### Compliant Solution

The solution addresses the problem by giving the local library a unique name (per [PRE08-C. Guarantee that header file names are unique](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre08-c) ), which makes it apparent that the library used is not the original:

## PRE05-C. Understand macro replacement when concatenating tokens or performing stringification

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/17.preprocessor-pre/06.pre05-c.md) — SHA-256 `1201c3fbf01243150a8c113eb8bc573d0a07caba3298184f673ff09609491a45`。

It is necessary to understand how macro replacement works in C, particularly in the context of concatenating tokens using the `##` operator and converting macro parameters to strings using the `#` operator.

## Concatenating Tokens

The `##` preprocessing operator is used to merge two tokens into one while expanding macros, which is called *token pasting* or *token concatenation* . When a macro is expanded, the two tokens on either side of each `##` operator are combined into a single token that replaces the `##` and the two original tokens in the macro expansion \[ [FSF 2005](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-FSF05) \].

Token pasting is most useful when one or both of the tokens come from a macro argument. If either of the tokens next to a `##` is a parameter name, it is replaced by its actual argument before `##` executes. The actual argument is not macro expanded first.

## Stringification

Parameters are not replaced inside string constants, but the `#` preprocessing operator can be used instead. When a macro parameter is used with a leading `#` , the preprocessor replaces it with the literal text of the actual argument converted to a string constant \[ [FSF 2005](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-FSF05) \].

### Noncompliant Code Example

The following definition for `static_assert()` from [DCL03-C. Use a static assertion to test the value of a constant expression](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl03-c) uses the `JOIN()` macro to concatenate the token `assertion_failed_at_line_` with the value of `__LINE__` :

`__LINE__` is a predefined macro name that expands to an integer constant representing the presumed line number of the current source line within the current source file. If the intention is to expand the `__LINE__` macro, which is likely the case here, the following definition for `JOIN()` is noncompliant because the `__LINE__` is not expanded, and the character array is subsequently named `assertion_failed_at_line___LINE__` :

### Compliant Solution

To get the macro to expand, a second level of indirection is required, as shown by this compliant solution:

`   JOIN(x, y)  `calls`   JOIN_AGAIN(x, y)  ` so that if `x` or `y` is a macro, it is expanded before the `##` operator pastes them together.

Note also that macro parameters cannot be individually parenthesized when concatenating tokens using the `##` operator, converting macro parameters to strings using the `#` operator, or concatenating adjacent string literals. This is an exception, **PRE01-C-EX2** , to [PRE01-C. Use parentheses within macros around parameter names](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre01-c) .

### Noncompliant Code Example

This example is noncompliant if the programmer's intent is to expand the macro before stringification:

The macro invocation `str(foo)` expands to `foo` .

### Compliant Solution

To stringify the result of expansion of a macro argument, two levels of macros must be used:

The macro invocation `xstr(foo)` expands to `4` because `s` is stringified when it is used in `str()` , so it is not macro expanded first. However, `s` is an ordinary argument to `xstr()` , so it is completely macro expanded before `xstr()` is expanded. Consequently, by the time `str()` gets to its argument, it has already been macro expanded.

## PRE06-C. Enclose header files in an include guard

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/17.preprocessor-pre/07.pre06-c.md) — SHA-256 `41da5b97243b4aa5441950a7478c9272e113de5436459291c1404645227e7b80`。

Until the early 1980s, large software development projects had a continual problem with the inclusion of headers. One group might have produced a `graphics.h` , for example, which started by including `io.h` . Another group might have produced `keyboard.h` , which also included `io.h` . If `io.h` could not safely be included several times, arguments would break out about which header should include it. Sometimes an agreement was reached that each header should include no other headers, and as a result, some application programs started with dozens of `#include` lines, and sometimes they got the ordering wrong or forgot a required header.

### Compliant Solution

All these complications disappeared with the discovery of a simple technique: each header should `#define` a symbol that means "I have already been included." The entire header is then enclosed in an include guard:

Consequently, the first time `header.h` is `#include` 'd, all of its contents are included. If the header file is subsequently `#include` 'd again, its contents are bypassed.

Because solutions such as this one make it possible to create a header file that can be included more than once, the C Standard guarantees that the standard headers are safe for multiple inclusion.

Note that it is a common mistake to choose a reserved name (such as `_HEADER_H_` or `__HEADER_H__` ) for the name of the macro used in the include guard. See [DCL37-C. Do not declare or define a reserved identifier](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl37-c) for more information.

Until the early 1980s, large software development projects had a continual problem with the inclusion of headers. One group might have produced a `graphics.h` , for example, which started by including `io.h` . Another group might have produced `keyboard.h` , which also included `io.h` . If `io.h` could not safely be included several times, arguments would break out about which header should include it. Sometimes an agreement was reached that each header should include no other headers, and as a result, some application programs started with dozens of `#include` lines, and sometimes they got the ordering wrong or forgot a required header.

### Exceptions

**PRE06-C-EX0** : It is acceptable to include a header file containing an [X-Macro](https://web.archive.org/web/20120928232311/http://www.drdobbs.com/the-new-c-x-macros/184401387) \[ Meyers 2010 \] multiple times, and each time needs no inclusion guard.  Such header files should have an extension of **.def** .

## PRE07-C. Avoid using repeated question marks

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/17.preprocessor-pre/08.pre07-c.md) — SHA-256 `701b7adb19d80f106a0c14e1148f7545be8b17529f42280c57213b2886c00d9a`。

Two consecutive question marks signify the start of a trigraph sequence. According to the C Standard, subclause 5.2.1.1 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \],

> All occurrences in a source file of the following sequences of three characters (that is, *trigraph sequences* ) are replaced with the corresponding single character.
>
> |                      |                    |           |                      |                    |           |                      |                    |
> |----------------------|--------------------|-----------|----------------------|--------------------|-----------|----------------------|--------------------|
> | `??=` | `#` | `       ` | `??)` | `]` | `       ` | `??!` | `|` |
> | `??(` | `[` | `       ` | `??'` | `^` | `       ` | `??>` | `}` |
> | `??/` | `\` | `       ` | `??<` | `{` | `       ` | `??-` | `~` |
>
>   

### Noncompliant Code Example

In this noncompliant code example, `a++` is not executed because the trigraph sequence `??/` is replaced by `\` , logically putting `a++` on the same line as the comment:

### Compliant Solution

This compliant solution eliminates the accidental introduction of the trigraph by separating the question marks:

### Noncompliant Code Example

This noncompliant code example includes the trigraph sequence `??!` , which is replaced by the character `|` :

This example prints `   Over 9000!|  ` if a C-compliant compiler is used.

### Compliant Solution

This compliant solution uses string concatenation to concatenate the two question marks; otherwise, they are interpreted as beginning a trigraph sequence:

This code prints `   Over 9000!??!  ` , as intended.

## PRE08-C. Guarantee that header file names are unique

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/17.preprocessor-pre/09.pre08-c.md) — SHA-256 `14d4556a3e3d2b5249da4ae914128f8b7d3cc823299f368fb9929c53635c6076`。

Make sure that included header file names are unique. According to the C Standard, subclause 6.10.2, paragraph 5 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \],

> The [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) shall provide unique mappings for sequences consisting of one or more nondigits or digits (6.4.2.1) followed by a period (.) and a single nondigit. The first character shall not be a digit. The implementation may ignore distinctions of alphabetical case and restrict the mapping to eight significant characters before the period.

This means that

-   Only the first eight characters in the file name are guaranteed to be significant.
-   The file has only one nondigit character after the period in the file name.
-   The case of the characters in the file name is not guaranteed to be significant.

To guarantee that header file names are unique, all included files should differ (in a case-insensitive manner) in their first eight characters or in their (one-character) file extension.

Note that compliance with this recommendation does not require that short file names are used, only that the file names are unique.

### Noncompliant Code Example

This noncompliant code example contains references to headers that may exist independently in various environments but can be ambiguously interpreted by a C-compliant compiler:

`Library.h` and `library.h` may refer to the same file. Also, because only the first eight characters are guaranteed to be significant, it is unclear whether `utilities_math.h` and `utilities_physics.h` are parsed. Finally, if a file such as `my_libraryOLD.h` exists, it may inadvertently be included instead of `my_library.h` .

### Compliant Solution

This compliant solution avoids the ambiguity by renaming the associated files to be unique under the preceding constraints:

The only solution for mitigating ambiguity of a file, such as `my_libraryOLD.h` , is to rename old files with either a prefix (that would fall within the first eight characters) or add an extension (such as `my_library.h.old` ).

### Exceptions

**PRE08-C-EX1:** Although the C Standard requires only the first eight characters in the file name to be significant, most modern systems have long file names, and compilers on such systems can typically differentiate them. Consequently, long file names in headers may be used, provided that all the implementations to which the code is ported can distinguish between these file names.

## PRE09-C. Do not replace secure functions with deprecated or obsolescent functions

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/17.preprocessor-pre/10.pre09-c.md) — SHA-256 `4c17a12272f5f8642ef896082b5037502fd54b99277d322296eeaae8dfcdff73`。

Macros are frequently used in the remediation of existing code to globally replace one identifier with another, for example, when an existing API changes. Although some risk is always involved, this practice becomes particularly dangerous if a function name is replaced with the function name of a deprecated or obsolescent function. Deprecated functions are defined by the C Standard and Technical Corrigenda. Obsolescent functions are defined by [MSC24-C. Do not use deprecated or obsolescent functions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc24-c) .

Although compliance with rule [MSC24-C. Do not use deprecated or obsolescent functions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc24-c) guarantees compliance with this recommendation, the emphasis of this recommendation is the extremely risky and deceptive practice of replacing functions with less secure alternatives.

### Noncompliant Code Example

The Internet Systems Consortium's (ISC) Dynamic Host Configuration Protocol (DHCP) contained a vulnerability that introduced several potential buffer overflow conditions \[ [VU#654390](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-VU#654390) \]. ISC DHCP makes use of the `vsnprintf()` function for writing various log file strings; `vsnprintf()` is defined in the Portable Operating System Interface (POSIX <sup>®</sup> ), Base Specifications, Issue 7 \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \] as well as in the C Standard. For systems that do not support `vsnprintf()` , a C include file was created that defines the `vsnprintf()` function to `vsprintf()` , as shown in this noncompliant code example:

The `vsprintf()` function does not check bounds. Consequently, `size` is discarded, creating the potential for a buffer overflow when untrusted data is used.

### Compliant Solution

The solution is to include an implementation of the missing function `vsnprintf()` to eliminate the dependency on external library functions when they are not available. This compliant solution assumes that `__USE_ISOC11` is not defined on systems that fail to provide a `vsnprintf()` implementation:

## PRE10-C. Wrap multistatement macros in a do-while loop

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/17.preprocessor-pre/11.pre10-c.md) — SHA-256 `b555a25875c6fd4f0f897c76dc27db7c78dd6cdc49fb208bfaad26cef1b99a3d`。

Macros are often used to execute a sequence of multiple statements as a group.

Inline functions are, in general, more suitable for this task (see [PRE00-C. Prefer inline or static functions to function-like macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre00-c) ). Occasionally, however, they are not feasible (when macros are expected to operate on variables of different types, for example).

When multiple statements are used in a macro, they should be bound together in a `do-while` loop syntactically, so the macro can appear safely inside `if` clauses or other places that expect a single statement or a statement block. Note that this is only effective if none of the multiple statements are `break` or `continue` , as they would be captured by the `do-while` loop. (Alternatively, when an `if` , `for` , or `while` statement uses braces even for a single body statement, then multiple statements in a macro will expand correctly even without a `do-while` loop (see [EXP19-C. Use braces for the body of an if, for, or while statement](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/expressions-exp/exp19-c) ).

### Noncompliant Code Example

This noncompliant code example contains multiple, unbound statements:

This macro expands correctly in a normal sequence of statements but not as the `then` clause in an `if` statement:

It expands to the following, which is certainly not what the programmer intended:

Furthermore, this macro violates [PRE02-C. Macro replacement lists should be parenthesized](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre02-c) .

### Noncompliant Code Example

This noncompliant code example parenthesizes its macro arguments, but inadequately bounds multiple statements:

This macro fails to expand correctly in some case, such as the following example, which is meant to be an `if` statement with two branches:

Following macro expansion, however, this code is interpreted as an `if` statement with only one branch:

The problem is the semicolon ( `;` ) following the block.

### Compliant Solution

Wrapping the macro inside a `do-while` loop mitigates the problem:

The `do-while` loop will always be executed exactly once.

This macro still violates the recommendation [PRE12-C. Do not define unsafe macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre12-c) , because both macro arguments are evaluated twice. It is expected that the arguments are simple lvalues.

## PRE11-C. Do not conclude macro definitions with a semicolon

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/17.preprocessor-pre/12.pre11-c.md) — SHA-256 `8076d2070024a0d0508cb5cd3469a7d2dfc28dd426b78ab65679cb4620b961bf`。

Macros are frequently used to make source code more readable. Macro definitions, regardless of whether they expand to a single or multiple statements, should not conclude with a semicolon. (See [PRE10-C. Wrap multistatement macros in a do-while loop](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre10-c) .) If required, the semicolon should be included following the macro expansion. Inadvertently inserting a semicolon at the end of the macro definition can unexpectedly change the control flow of the program.

Another way to avoid this problem is to prefer inline or static functions over function-like macros. (See also [PRE00-C. Prefer inline or static functions to function-like macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre00-c) .)

In general, the programmer should ensure that there is no semicolon at the end of a macro definition. The responsibility for having a semicolon where needed during the use of such a macro should be delegated to the person invoking the macro.

### Noncompliant Code Example

This noncompliant code example creates a macro definition for a `for` loop in the program. A `for` loop should require braces, even if it contains only a single body statement. (See [EXP19-C. Use braces for the body of an if, for, or while statement](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/expressions-exp/exp19-c) .) This macro takes an integer argument, which is the number of times the loop should run. The programmer has inserted a semicolon at the end of the macro definition by mistake.

The programmer expects to get the following output from the code:

But because of the semicolon at the end of the macro definition, the `for` loop in the program has a null statement, so the statement "Inside for loop" gets printed just once. Essentially, the semicolon at the end of the macro definition changes the program control flow.

Although this example might not actually be used in code, it shows the effect a semicolon in a macro definition can have.

### Compliant Solution

The compliant solution is to write the macro definition without the semicolon at the end, leaving the decision whether or not to have a semicolon up to the person who is using the macro:

### Noncompliant Code Example

In this noncompliant code example, the programmer defines a macro that increments the value of the first argument, `x` , by 1 and modulates it with the value of the second argument, `max` :

In this case, the programmer intends to increment `index` and then use that as a value by adding 2 to it. Unfortunately, the value is equal to the incremented value of `index` because of the semicolon present at the end of the macro. The `   + 2;  ` is treated as a separate statement by the compiler. The user will not get any compilation errors. If the user has not enabled warnings while compiling, the effect of the semicolon in the macro cannot be detected at an early stage.

### Compliant Solution

The compliant solution is to write the macro definition without the semicolon at the end, leaving the decision whether or not to have a semicolon up to the person who is using the macro:

### Compliant Solution

This compliant solution uses an inline function as recommended by [PRE00-C. Prefer inline or static functions to function-like macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre00-c) .

## PRE12-C. Do not define unsafe macros

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/17.preprocessor-pre/13.pre12-c.md) — SHA-256 `7e740f588f5abcce4d03783a04dda5da577cac7df84244a2e6eb4b648beeccd9`。

An [unsafe function-like macro](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unsafefunction-likemacro) is one that, when expanded, evaluates its argument more than once or does not evaluate it at all. Contrasted with function calls, which always evaluate each of their arguments exactly once, unsafe function-like macros often have unexpected and surprising effects and lead to subtle, hard-to-find defects (see [PRE31-C. Avoid side effects in arguments to unsafe macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/preprocessor-pre/pre31-c) ). Consequently, every [function-like macro](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-function-likemacro) should evaluate each of its arguments exactly once. Alternatively and preferably, defining function-like macros should be avoided in favor of inline functions (see [PRE00-C. Prefer inline or static functions to function-like macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre00-c) ).

<a id="PRE12C.Donotdefineunsafemacros-nce_multiple_evaluation"></a>

### Noncompliant Code Example (Multiple Argument Evaluation)

The most severe problem with [unsafe function-like macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unsafefunction-likemacro) is side effects of macro arguments, as shown in this noncompliant code example:

The invocation of the `ABS()` macro in this noncompliant code example expands to the following code. The resulting code has well-defined behavior but causes `n` to be incremented twice rather than once, which may be surprising to those unfamiliar with the [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) of the macro or unaware that they are using a macro in the first place.

<a id="PRE12C.Donotdefineunsafemacros-cs_inline_function"></a>

### Compliant Solution (Inline Function)

A possible and preferable compliant solution is to define an inline function with equivalent but unsurprising semantics:

<a id="PRE12C.Donotdefineunsafemacros-cs_language_extension"></a>

### Compliant Solution (Language Extension)

Some implementations provide language extensions that make it possible to define safe function-like macros, such as the macro `ABS()` , that would otherwise require evaluating their arguments more than once. For example, the GCC extension [Statements and Declarations in Expressions](https://gcc.gnu.org/onlinedocs/gcc/Statement-Exprs.html#Statement-Exprs) makes it possible to implement the macro `ABS()` in a safe way. Note, however, that because relying on implementation-defined extensions introduces undesirable platform dependencies that may make the resulting code nonportable, such solutions should be avoided in favor of portable ones wherever possible (see [MSC14-C. Do not introduce unnecessary platform dependencies](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc14-c) ).

Another GCC extension known as *statement expression* makes it possible for the block statement to appear where an expression is expected. The statement expression extension establishes a scope (note the curly braces) and any declarations in it are distinct from those in enclosing scopes.

## PRE13-C. Use the Standard predefined macros to test for versions and features.

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/17.preprocessor-pre/14.pre13-c.md) — SHA-256 `fc7aaa00547831985ca214b4e6b406ca238d722fa98cb2afbb7c0b1e1c8abba7`。

The C S tandard defines a set of predefined macros (see subclause 6.10.8) to help the user determine if the [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) being used is a [conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conforming) implementation, and if so, to which version of the C Standard it conforms. These macros can also help the user to determine which of the standard features are implemented.

The following tables list these macros and indicate in which version of the C Standard they were introduced. The following macros are required:

| Macro Name | C90 | C99 | C11 |
| --- | --- | --- | --- |
| `__STDC__` | ✓ | ✓ | ✓ |
| `__STDC_HOSTED__` |  | ✓ | ✓ |
| `__STDC_VERSION__ 1` |  | ✓ | ✓ |
| `__DATE__` | ✓ | ✓ | ✓ |
| `__FILE__` | ✓ | ✓ | ✓ |
| `__LINE__` | ✓ | ✓ | ✓ |
| `__TIME__` | ✓ | ✓ | ✓ |

####### 1) \_\_STDC_VERSION\_\_ was introduced by an Amendment to C90, this version of the C Standard is commonly call C94

The following are optional environment macros:

| Macro Name | C90 | C99 | C11 |
| --- | --- | --- | --- |
| `__STDC_ISO_10646__` |  | ✓ | ✓ |
| `__STDC_MB_MIGHT_NEQ_WC__` |  | ✓ | ✓ |
| `__STDC_UTF_16__` |  |  | ✓ |
| `__STDC_UTF_32__` |  |  | ✓ |

The following are optional feature macros:

| Macro Name | C90 | C99 | C11 |
| --- | --- | --- | --- |
| `__STDC_ANALYZABLE__` |  |  | ✓ |
| `__STDC_IEC_559__` |  | ✓ | ✓ |
| `__STDC_IEC_559_COMPLEX__` |  | ✓ | ✓ |
| `__STDC_LIB_EXT1__` |  |  | ✓ |
| `__STDC_NO_ATOMICS__` |  |  | ✓ |
| `__STDC_NO_COMPLEX__` |  |  | ✓ |
| `__STDC_NO_THREADS__` |  |  | ✓ |
| `__STDC_NO_VLA__` |  |  | ✓ |

The following is optional and is defined by the user:

| Macro Name | C90 | C99 | C11 |
| --- | --- | --- | --- |
| `__STDC_WANT_LIB_EXT1__` |  |  | ✓ |

### Noncompliant Code Example (Checking Value of Predefined Macro)

C Standard predefined macros should never be tested for a value before the macro is tested for definition, as shown in this noncompliant code example:

### Compliant Solution (Testing for Definition of Macro)

In this compliant solution, the definition of the predefined macro `__STDC__` is tested before the value of the macro is tested:

### Compliant Solution (Test for Optional Feature)

This compliant solution tests to see if the C11 predefined macro `__STDC_ANALYZABLE__` is defined and what value the implementation has given the macro:

### Compliant Solution (Optional Language Features)

This compliant solution checks for the C11 optional language features in Annex K. If Annex K is supported by the implementation, the functions defined in Annex K are used; if Annex K is not supported, then the standard library functions are used. (See [DCL09-C. Declare functions that return errno with a return type of errno_t](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl09-c) [.](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc14-c) )

### Compliant Solution (Optional Language Features)

The previous compliant solution comes close to violating [PRE09-C. Do not replace secure functions with deprecated or obsolescent functions](https://www.securecoding.cert.org/confluence/display/seccode/PRE09-C.+Do+not+replace+secure+functions+with+deprecated+or+obsolescent+functions) , and would if a function-like macro were defined which called either `strcpy_s()` or `strcpy()` depending on if `USE_EXT1` were defined.  This compliant solution solves the problem by including a custom library that implements the optional language feature, which in this case is the Safe C Library available from [SourceForge](https://safeclib.sourceforge.net/) .

## PRE30-C. Do not create a universal character name through concatenation

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/17.preprocessor-pre/2.pre30-c.md) — SHA-256 `4bbe18431740e977416485f1c326acc612a452fab0a4b0bf685eff4cc99c75e3`。

The C Standard supports universal character names that may be used in identifiers, character constants, and string literals to designate characters that are not in the basic character set. The universal character name `\U` *nnnnnnnn* designates the character whose 8-digit short identifier (as specified by ISO/IEC 10646) is *nnnnnnnn* . Similarly, the universal character name `\u` *nnnn* designates the character whose 4-digit short identifier is *nnnn* (and whose 8-digit short identifier is `0000` *nnnn* ).

The C Standard, 5.1.1.2, paragraph 4 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], says

> If a character sequence that matches the syntax of a universal character name is produced by token concatenation (6.10.5.3), the behavior is undefined.

See also [undefined behavior 3](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_3) .

In general, avoid universal character names in identifiers unless absolutely necessary.

### Noncompliant Code Example

This code example is noncompliant because it produces a universal character name by token concatenation:

#### Implementation Details

This code compiles and runs with Microsoft Visual Studio 2013, assigning 4 to the variable as expected.

GCC 4.8.1 on Linux refuses to compile this code; it emits a diagnostic reading, "stray '\\' in program," referring to the universal character fragment in the invocation of the `assign` macro.

### Compliant Solution

This compliant solution uses a universal character name but does not create it by using token concatenation:

## PRE31-C. Avoid side effects in arguments to unsafe macros

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/17.preprocessor-pre/3.pre31-c.md) — SHA-256 `783df860a66cbfc650e19d7e6a6e7be85fd4248d6a29016066263665fd3e93b0`。

An [unsafe function-like macro](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unsafefunction-likemacro) is one whose expansion results in evaluating one of its parameters more than once or not at all. Never invoke an unsafe macro with arguments containing an assignment, increment, decrement, volatile access, input/output, or other expressions with side effects (including function calls, which may cause side effects).

The documentation for unsafe macros should warn against invoking them with arguments with side effects, but the responsibility is on the programmer using the macro. Because of the risks associated with their use, it is recommended that the creation of unsafe function-like macros be avoided. (See [PRE00-C. Prefer inline or static functions to function-like macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre00-c) .)

This rule is similar to [EXP44-C. Do not rely on side effects in operands to sizeof, \_Alignof, or \_Generic](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp44-c) .

### Noncompliant Code Example

One problem with unsafe macros is [side effects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sideeffect) on macro arguments, as shown by this noncompliant code example:

The invocation of the `ABS()` macro in this example expands to

The resulting code is well defined but causes `n` to be incremented twice rather than once.

### Compliant Solution

In this compliant solution, the increment operation `++n` is performed before the call to the unsafe macro.

Note the comment warning programmers that the macro is unsafe. The macro can also be renamed `ABS_UNSAFE()` to make it clear that the macro is unsafe. This compliant solution, like all the compliant solutions for this rule, has undefined behavior (see [undefined behavior 36](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) ) if the argument to `ABS()` is equal to the minimum (most negative) value for the signed integer type. (See [INT32-C. Ensure that operations on signed integers do not result in overflow](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int32-c) for more information.)

### Compliant Solution

This compliant solution follows the guidance of [PRE00-C. Prefer inline or static functions to function-like macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre00-c) by defining an inline function `iabs()` to replace the `ABS()` macro. Unlike the `ABS()` macro, which operates on operands of any type, the `iabs()` function will truncate arguments of types wider than `int` whose value is not in range of the latter type.

### Compliant Solution

A more flexible compliant solution is to declare the `ABS()` macro using a `_Generic` selection. To support all arithmetic data types, this solution also makes use of inline functions to compute integer absolute values. (See [PRE00-C. Prefer inline or static functions to function-like macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre00-c) and [PRE12-C. Do not define unsafe macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre12-c) .)

According to the C Standard, 6.5.2.1, paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2024) \]:

> The controlling expression of a generic selection is not evaluated. If a generic selection has a generic association with a type name that is compatible with the type of the controlling expression, then the result expression of the generic selection is the expression in that generic association. Otherwise, the result expression of the generic selection is the expression in the `default` generic association. None of the expressions from any other generic association of the generic selection is evaluated.

Because the expression is not evaluated as part of the generic selection, the use of a macro in this solution is guaranteed to evaluate the macro parameter `v` only once.

Generic selections were introduced in C11 and are not available in C99 and earlier editions of the C Standard.

### Compliant Solution (GCC)

GCC's [`__typeof`](https://gcc.gnu.org/onlinedocs/gcc/Typeof.html) extension makes it possible to declare and assign the value of the macro operand to a temporary of the same type and perform the computation on the temporary, consequently guaranteeing that the operand will be evaluated exactly once. Another GCC extension, known as [*statement expression*](https://gcc.gnu.org/onlinedocs/gcc/Statement-Exprs.html) , makes it possible for the block statement to appear where an expression is expected:

Note that relying on such extensions makes code nonportable and violates [MSC14-C. Do not introduce unnecessary platform dependencies](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc14-c) .

### Noncompliant Code Example ( `assert()` )

The `assert()` macro is a convenient mechanism for incorporating diagnostic tests in code. (See [MSC11-C. Incorporate diagnostic tests using assertions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc11-c) .) Expressions used as arguments to the standard `assert()` macro should not have side effects. The behavior of the `assert()` macro depends on the definition of the object-like macro `NDEBUG` . If the macro `NDEBUG` is undefined, the `assert()` macro is defined to evaluate its expression argument and, if the result of the expression compares equal to 0, call the `abort()` function. If `NDEBUG` is defined, `assert` is defined to expand to `((void)0)` . Consequently, the expression in the assertion is not evaluated, and no side effects it may have had otherwise take place in non-debugging executions of the code.

This noncompliant code example includes an `assert()` macro containing an expression ( `index++` ) that has a side effect:

### Compliant Solution ( `assert()` )

This compliant solution avoids the possibility of side effects in assertions by moving the expression containing the side effect outside of the `assert()` macro.

### Exceptions

**PRE31-C-EX1:** An exception can be made for invoking an [unsafe macro](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unsafefunction-likemacro) with a function call argument provided that the function has no [side effects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sideeffect) . However, it is easy to forget about obscure side effects that a function might have, especially library functions for which source code is not available; even changing `errno` is a side effect. Unless the function is user-written and does nothing but perform a computation and return its result without calling any other functions, it is likely that many developers will forget about some side effect. Consequently, this exception must be used with great care.

## PRE32-C. Do not use preprocessor directives in invocations of function-like macros

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/17.preprocessor-pre/4.pre32-c.md) — SHA-256 `7ef3c435eade917c10f4847ced9136ad1ff4f6f5323715f907d5ed5ce4d5086f`。

The arguments to a macro must not include preprocessor directives, such as `#define` , `#ifdef` , and `#include` . Doing so results in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) , according to the C Standard, 6.10.5, paragraph 11 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \]:

> The sequence of preprocessing tokens bounded by the outside-most matching parentheses forms the list of arguments for the function-like macro. The individual arguments within the list are separated by comma preprocessing tokens, but comma preprocessing tokens between matching inner parentheses do not separate arguments. **If there are sequences of preprocessing tokens within the list of arguments that would otherwise act as preprocessing directives, the behavior is undefined.**

See also [undefined behavior 92](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_92) .

This rule also applies to the use of preprocessor directives in arguments to any function where it is unknown whether or not the function is implemented using a macro. This includes all standard library functions, such as `memcpy()` , `printf()` , and `assert()` , because any standard library function may be implemented as a macro. (C24, 7.1.4, paragraph 1).

### Noncompliant Code Example

In this noncompliant code example \[ [GCC Bugs](https://gcc.gnu.org/bugs.html#nonbugs_c) \], the programmer uses preprocessor directives to specify platform-specific arguments to `memcpy()` . However, if `memcpy()` is implemented using a macro, the code results in [undefined behavior 92](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_92) .

### Compliant Solution

In this compliant solution \[ [GCC Bugs](https://gcc.gnu.org/bugs.html#nonbugs_c) \], the appropriate call to `memcpy()` is determined outside the function call:
