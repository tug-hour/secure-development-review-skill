# CERT C — MSC

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## MSC00-C. Compile cleanly at high warning levels

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/02.msc00-c.md) — SHA-256 `96c96979ef7271d3018f0002d624062c61b7f4818778c83b2684bb4e4884fa9b`。

Compile code using the highest warning level available for your compiler and eliminate warnings by modifying the code.

According to the C Standard, subclause 5.1.1.3 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \],

> A conforming implementation shall produce at least one diagnostic message (identified in an [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) manner) if a preprocessing translation unit or translation unit contains a violation of any syntax rule or constraint, even if the behavior is also explicitly specified as [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) or implementation-defined. Diagnostic messages need not be produced in other circumstances.

Assuming a [conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conforming) [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) , eliminating diagnostic messages will eliminate any syntactic or constraint violations.

If suitable source-code-checking tools are available, use them regularly.

Compilers can produce diagnostic messages for correct code, as is permitted by C. It is usually preferable to rewrite code to eliminate compiler warnings, but if the code is correct, it is sufficient to provide a comment explaining why the warning message does not apply. Some compilers provide ways to suppress warnings, such as suitably formatted comments or pragmas, which can be used sparingly when the programmer understands the implications of the warning but has good reason to use the flagged construct anyway.

Do not simply quiet warnings by adding type casts or other means. Instead, understand the reason for the warning and consider a better approach, such as using matching types and avoiding type casts whenever possible.

### Noncompliant Code Example (Windows)

Using the `default` warning specifier with `   #pragma warning  ` resets the behavior of a warning to its default value, which may not be the same as its previous behavior.  Programmers commonly, but incorrectly, use the `default` warning specifier to restore previous warning messages after a message is temporarily disabled.

### Compliant Solution (Windows)

Instead of using the `default` warning specifier, the current state of the warnings should be saved and then restored after the unnecessarily flagged code.

The `   pragma   warning(push)  ` stores the current warning state for every warning. The `   pragma   warning(pop)  ` pops the last warning state pushed onto the stack. Any changes made to the warning state between the push and pop are undone.

## MSC01-C. Strive for logical completeness

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/03.msc01-c.md) — SHA-256 `0cd9d875feae29ce0989e8256742ad7741358bba8585fafdd23b8276b2ab6443`。

Software [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) can result when a programmer fails to consider all possible data states.

### Noncompliant Code Example ( `if` Chain)

This noncompliant code example fails to test for conditions where `a` is neither `b` nor `c` . This behavior may be correct in this case, but failure to account for all the values of `a` can result in logic errors if `a` unexpectedly assumes a different value.

### Compliant Solution ( `if` Chain)

This compliant solution explicitly checks for the unexpected condition and handles it appropriately:

### Noncompliant Code Example ( `switch` )

This noncompliant code example fails to consider all possible cases. Failure to account for all valid values of type `Color` will result in a logic error. Because valid values of an enumerated type include all those of its underlying integer type, unless enumeration constants have been provided for all those values, the `default` label is appropriate and necessary.

#### Implementation Details

Microsoft Visual C++ .NET with `/W4` does not warn when assigning an integer value to an `enum` type or when the `switch` statement does not contain all possible values of the enumeration.

### Compliant Solution ( `switch` )

This compliant solution takes care to provide the `default` label to handle all valid values of type `Color` :

Note that adding a default case to a `switch` statement, even when all possible `switch` labels are specified, is an exception ( [MSC07-C-EX1](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc07-c#MSC07C.Detectandremovedeadcode-MSC07-EX1) ) to [MSC07-C. Detect and remove dead code](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc07-c) .

An alternative compliant solution to the noncompliant code example is to provide a `return` statement after the `switch` statement. Note, however, that this solution may not be appropriate in all situations.

#### Historical Discussion

This practice has been a subject of debate for some time, but a clear direction has emerged.

Originally, the consensus among those writing best practices was simply that each `switch` statement should have a `default` label. Eventually, emerging compilers and [static analysis](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-staticanalysis) tools could verify that a `switch` on an `enum` type contained a `case` label for each enumeration value, but only if no `default` label existed. This led to a shift toward purposely leaving out the `default` label to allow static analysis. However, the resulting code was then vulnerable to `enum` variables being assigned `int` values outside the set of `enum` values.

These two practices have now been merged. A `switch` on an `enum` type should now contain a `case` label for each `enum` value but should also contain a `default` label for safety. This practice does not add difficulty to [static analysis](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-staticanalysis) .

Existing implementations are in transition, with some not yet analyzing `switch` statements with `default` labels. Developers must take extra care to check their own `switch` statements until the new practice becomes universal.

### Noncompliant Code Example (Zune 30)

This noncompliant code example shows incomplete logic when converting dates. The code appeared in the Zune 30 media player, causing many players to lock up on December 30, 2008, at midnight PST. This noncompliant code example comes from the `ConvertDays` function in the real-time clock (RTC) routines for the MC13783 PMIC RTC. It takes the number of days since January 1, 1980, and computes the correct year and number of days since January 1 of the correct year.

The flaw in the code occurs when `days` has the value 366 because the loop never terminates. This bug manifested itself on the 366th day of 2008, which was the first leap year in which this code was active.

### Compliant Solution (Zune 30)

The following proposed rewrite is provided at [http://winjade.net/2009/01/lesson-on-infinite-loops](http://winjade.net/2009/01/lesson-on-infinite-loops/) . The loop is guaranteed to exit, because `days` decreases for each iteration of the loop, unless the `while` condition fails and the loop terminates.

This compliant solution is for illustrative purposes and is not necessarily the solution implemented by Microsoft.

## MSC04-C. Use comments consistently and in a readable fashion

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/04.msc04-c.md) — SHA-256 `c84c7d16fd3dfc170a8cbb739dc6bbe8be1e2acb63aed884b509e12157c49340`。

### Noncompliant Code Example

Do not use the character sequence `/*` within a comment:

In this example, the call to the security-critical function is not executed. A reviewer examining this page could incorrectly assume that the code is executed.

If execution failure is the result of an accidental omission, it is useful to use an editor that provides syntax highlighting or formats the code to help identify issues like missing end-comment delimiters.

Because missing end delimiters are error prone and often viewed as a mistake, this approach is not recommended for commenting out code.

### Compliant Solution (Preprocessor)

Instead of using `/*` and `*/` to comment out blocks of code, use conditional compilation (for example, `#if` , `#ifdef` , or `#ifndef` ):

The text inside a block of code commented out using `#if` , `#ifdef` , or `#ifndef` must still consist of *valid preprocessing tokens* . This means that the characters `"` and `'` must each be paired just as in real C code, and the pairs must not cross line boundaries. In particular, an apostrophe within a contracted word looks like the beginning of a character constant. Consequently, natural-language comments and pseudocode should always be written between the comment delimiters `/*` and `*/` or following `//` .

### Compliant Solution (Compiler)

This compliant solution takes advantage of the compiler's ability to remove unreachable (dead) code. The code inside the `if` block must remain acceptable to the compiler. If other parts of the program, such as macros, types, or function prototypes, later change in a way that would cause syntax errors, the unexecuted code must be brought up to date to correct the problem. Then, if it is needed again in the future, the programmer need only remove the surrounding `if` statement and the `NOTREACHED` comment.

The `NOTREACHED` comment tells some compilers and [static analysis](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-staticanalysis) tools not to complain about this unreachable code. It also serves as documentation.

This code is an instance of exception MSC07-C-EX2 to [MSC07-C. Detect and remove dead code](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc07-c) .

### Noncompliant Code Example

Following are some additional examples of comment styles that are confusing and should be avoided:

### Compliant Solution

Use a consistent style of commenting:

## MSC05-C. Do not manipulate time_t typed values directly

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/05.msc05-c.md) — SHA-256 `d14f031f7083d0bc75b066b0674e7569f784f6884b3d3cc7430a2a4e1e7400e4`。

The type `time_t` is specified as an "arithmetic type capable of representing times." However, the way time is encoded within this arithmetic type by the function `time()` is [unspecified](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unspecifiedbehavior) . See [unspecified behavior 48](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/dd-unspecified-behavior#DD.UnspecifiedBehavior-usb_48) in Annex J of the C Standard. Because the encoding is unspecified, there is no safe way to manually perform arithmetic on the type, and as a result, the values should not be modified directly.

Note that POSIX specifies that the `time()` function must return a value of type `time_t` , representing time in seconds since the Epoch. POSIX-conforming applications that are not intended to be portable to other environments therefore may safely perform arithmetic operations on `time_t` objects.

### Noncompliant Code Example

This noncompliant code example attempts to execute `do_work()` multiple times until at least `seconds_to_work` has passed. However, because the encoding is not defined, there is no guarantee that adding `start` to `seconds_to_work` will result in adding `seconds_to_work` seconds.

### Compliant Solution

This compliant solution uses `difftime()` to determine the difference between two `time_t` values. The `difftime()` function returns the number of seconds, from the second parameter until the first parameter and result, as a `double` .

Note that this loop still might not exit because the range of `time_t` might not be able to represent two times `seconds_to_work` apart.

## MSC06-C. Beware of compiler optimizations

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/06.msc06-c.md) — SHA-256 `6ae65b8464eb14eabc251b623c1844d2e50c3e3a4adbc71c6d09ca5618a8a798`。

Subclause 5.1.2.3 of the C Standard \[ [ISO/IEC 9899:2011](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AABibliography-ISOIEC9899-2011) \] states:

> In the abstract machine, all expressions are evaluated as specified by the semantics. An actual [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) need not evaluate part of an expression if it can deduce that its value is not used and that no needed side effects are produced (including any caused by calling a function or accessing a volatile object).

This clause gives compilers the leeway to remove code deemed unused or unneeded when building a program. Although this functionality is usually beneficial, sometimes the compiler removes code that it thinks is not needed but has been added for a specific (often security-related) purpose.

### Noncompliant Code Example ( `memset()` )

An example of unexpected and unwanted compiler optimizations involves  overwriting the memory of a buffer that is used to store sensitive data. As a result, care must always be taken when dealing with sensitive data to ensure that operations on it always execute as intended. Some compiler optimization modes can remove code sections if the optimizer determines that doing so will not alter the behavior of the program. In this noncompliant code example, optimization may remove the call to `memset()` (which the programmer had hoped would clear sensitive memory) because the variable is not accessed following the write. Check compiler documentation for information about this compiler-specific behavior and which optimization levels can cause this behavior to occur.

For all of the compliant solutions provided for this recommendation, it is strongly recommended that the programmer inspect the generated assembly code in the optimized release build to ensure that memory is actually cleared and none of the function calls are optimized out.

### Noncompliant Code Example ( `   Touching Memory  ` )

This noncompliant code example accesses the buffer again after the call to `memset()` . This technique prevents some compilers from optimizing out the call to `memset()` but does not work for all [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) . For example, the MIPSpro compiler and versions 3 and later of GCC cleverly nullify only the first byte and leave the rest intact. Check compiler documentation to guarantee this behavior for a specific platform.

### Noncompliant Code Example (Windows)

This noncompliant code example uses the `ZeroMemory()` function provided by many versions of the Microsoft Visual Studio compiler:

A call to `ZeroMemory()` may be optimized out in a similar manner to a call to `memset()` .

### Compliant Solution (Windows)

This compliant solution uses a `SecureZeroMemory()` function provided by many versions of the Microsoft Visual Studio compiler. The documentation for the `SecureZeroMemory()` function guarantees that the compiler does not optimize out this call when zeroing memory.

### Compliant Solution (Windows)

The `#pragma` directives in this compliant solution instruct the compiler to avoid optimizing the enclosed code. This `#pragma` directive is supported on some versions of Microsoft Visual Studio and could be supported on other compilers. Check compiler documentation to ensure its availability and its optimization guarantees.

### Compliant Solution (C99)

`memset_s()` is one of the C11 Annex K bounds-checking interfaces (K.3.7.4.1). Annex K is optional and still poorly supported, so this compliant solution defines its own function with that name and signature. It uses the `volatile` type qualifier to inform the compiler that the memory should be overwritten and that the call to the `memset_s()` function should not be optimized out. Unfortunately, this compliant solution may not be as efficient as possible because of the nature of the `volatile` type qualifier preventing the compiler from optimizing the code at all. Typically, some compilers are smart enough to replace calls to `memset()` with equivalent assembly instructions that are much more efficient than the `memset()` implementation. Implementing a `memset_s()` function as shown in the example may prevent the compiler from using the optimal assembly instructions and can result in less efficient code. Check compiler documentation and the assembly output from the compiler.

However, note that both calling functions and accessing `volatile` -qualified objects can still be optimized out (while maintaining strict [conformance](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conforming) to the standard), so this compliant solution still might *not* work in some cases.  The `memset_explicit()` function introduced in C23 is the preferred solution (see [MSC41-C. Never hard code sensitive information](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/miscellaneous-msc/msc41-c) for an example).  The `memset_s()` function, introduced in C11's Annex K, serves the same purpose but is not widely implemented.  If neither function is available on your implementation, this compliant solution is the best alternative, and can be discarded once one of them is supported.

### Noncompliant Code Example

In rare cases, use of an empty infinite loop may be unavoidable. For example, an empty loop may be necessary on a platform that does not support `sleep(3)` or an equivalent function. Another example occurs in OS kernels. A task started before normal scheduler functionality is available may not have access to `sleep(3)` or an equivalent function. An empty infinite loop that does nothing within the loop body is a suboptimal solution because it consumes CPU cycles but performs no useful operations. An optimizing compiler can remove such a loop, which can lead to unexpected results. According to the C Standard, subclause 6.8.5, paragraph 6 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \],

> An iteration statement whose controlling expression is not a constant expression, that performs no input/output operations, does not access volatile objects, and performs no synchronization or atomic operations in its body, controlling expression, or (in the case of a `for` statement) its *expression-3* , may be assumed by the implementation to terminate. <sup>157  
> </sup> 157) This is intended to allow compiler transformations, such as removal of empty loops, even when termination cannot be proven.

This noncompliant code example implements an idle task that continuously executes a loop without executing any instructions within the loop. An optimizing compiler could remove the `while` loop in the example.

### Compliant Solution ( `while` )

To avoid the loop being optimized away, this compliant solution uses a constant expression `(1)` as the controlling expression in the `while` loop:

### Compliant Solution ( `for)`

According to the C Standard, subclause 6.8.5.3, paragraph 2, omitting the *expression-2* from a `for` loop will replace that expression with a nonzero constant.

## MSC07-C. Detect and remove dead code

**Recommendation / 対象外（廃止規則）**。旧規則を独立した要求として強制しない。本文の置換先（INT02-C / MSC12-C）で評価する。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/07.msc07-c.md) — SHA-256 `0ed9e8579240f21d540f5a72e1a62b3a5befe360f12a63b911f6e82ee899afa2`。

Deprecated

This rule has been deprecated.  It has been merged with:

-   [MSC12-C. Detect and remove code that has no effect or is never executed](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc12-c)

10/07/2014 -- Version 2.0

  

Code that is never executed is known as *dead code* . Typically, the presence of dead code indicates that a logic error has occurred as a result of changes to a program or the program's environment. Dead code is usually optimized out of a program during compilation. However, to improve readability and ensure that logic errors are resolved, dead code should be identified, understood, and eliminated.

This recommendation is related to [MSC12-C. Detect and remove code that has no effect or is never executed](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc12-c) .

### Noncompliant Code Example

This noncompliant code example demonstrates how dead code can be introduced into a program \[ [Fortify 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Fortify06) \]. The second conditional statement, `   if (s)  ` , will never evaluate true because it requires that `s` not be assigned `NULL` , and the only path where `s` can be assigned a non-null value ends with a `return` statement.

### Compliant Solution

Remediation of dead code requires the programmer to determine why the code is never executed and then to resolve the situation appropriately. To correct the preceding noncompliant code, the `return` is removed from the body of the first conditional statement.

### Noncompliant Code Example

In this example, the `strlen()` function is used to limit the number of times the function `s_loop()` will iterate. The conditional statement inside the loop evaluates to true when the current character in the string is the null terminator. However, because `strlen()` returns the number of characters that precede the null terminator, the conditional statement never evaluates true.

### Compliant Solution

Removing the dead code depends on the intent of the programmer. Assuming the intent is to flag and process the last character before the null terminator, the conditional is adjusted to correctly determine if the `i` refers to the index of the last character before the null terminator.

### Exceptions

<a id="MSC07C.Detectandremovedeadcode-MSC07-EX1"></a> **MSC07-C-EX1:** In some situations, seemingly dead code may make software resilient. An example is the `default` label in a `switch` statement whose controlling expression has an enumerated type and that specifies labels for all enumerations of the type. (See [MSC01-C. Strive for logical completeness](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc01-c) .) Because valid values of an enumerated type include all those of its underlying integer type, unless enumeration constants are provided for all those values, the `default` label is appropriate and necessary.

<a id="MSC07C.Detectandremovedeadcode-MSC07-EX2"></a> **MSC07-EX2:** It is also permissible to temporarily remove code that may be needed later. (See [MSC04-C. Use comments consistently and in a readable fashion](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc04-c) for an illustration.)

## MSC09-C. Character encoding: Use subset of ASCII for safety

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/23.msc09-c.md) — SHA-256 `8032f43cc56b84f95cd0f761cbd08f003c97f2826acb43fcf1fbb8f5b49e8a21`。

According to subclause 5.2.1 of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \],

> Two sets of characters and their associated collating sequences shall be defined: the set in which source files are written (the *source character set* ), and the set interpreted in the execution environment (the *execution character set* ). Each set is further divided into a *basic character set* , whose contents are given by this subclause, and a set of zero or more [locale-specific](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-locale-specificbehavior) members (which are not members of the basic character set) called *extended characters* . The combined set is also called the *extended character set* . The values of the members of the execution character set are [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) .

There are several national variants of ASCII. As a result, the original ASCII is often called US-ASCII. ISO/IEC 646-1991 defines a character set, similar to US-ASCII, but with code positions corresponding to US-ASCII characters `@[]{|` } as *national use positions* \[ [ISO/IEC 646-1991](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC646-1991) \]. It also gives some liberties with particular characters (e.g., ``#$^`~`` ).  In ISO/IEC 646-1991, several national variants of ASCII are defined, assigning different letters and symbols to the national use positions. Consequently, the characters that appear in those positions, including those in US-ASCII, are less portable in international data transfer. Because of the national variants, some characters are less portable than others: they might be transferred or interpreted incorrectly.

In addition to the letters of the English alphabet (A through Z and a through z), the digits (0 through 9), and the space, only the following characters are portable:

<div class="preformatted panel" style="border-width: 1px;">

<div class="preformattedContent panelContent">

    % & + , - . : = _

</div>

</div>

When naming files, variables, and other objects, only these characters should be considered for use. This recommendation is related to [STR02-C. Sanitize data passed to complex subsystems](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/characters-and-strings-str/str02-c) .

### File Names

File names containing particular characters can be troublesome and can cause [unexpected behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unexpectedbehavior) leading to potential [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) . If a program allows the user to specify a file name in the creation or renaming of a file, certain checks should be made to disallow the following characters and patterns:

-   Leading dashes—Leading dashes can cause problems when programs are called with the file name as a parameter because the first character or characters of the file name might be interpreted as an option switch.
-   Control characters, such as newlines, carriage returns, and escape—Control characters in a file name can cause unexpected results from shell scripts and in logging.
-   Spaces—Spaces can cause problems with scripts and when double quotes are not used to surround the file name.
-   Invalid character encodings—Character encodings can be a huge issue. (See [MSC10-C. Character encoding: UTF8-related issues](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc10-c) .)
-   Any characters other than letters, numbers, and punctuation designated here as portable—Other special characters are included in this recommendation because they are commonly used as separators, and having them in a file name can cause unexpected and potentially insecure behavior.

Also, many of the punctuation characters are not unconditionally safe for file names even of they are portably available.

Most of these characters or patterns are primarily a problem to scripts or automated parsing, but because they are not commonly used, it is best to disallow their use to reduce potential problems. Interoperability concerns also exist because different operating systems handle file names of this sort in different ways.

As a result of the influence of MS-DOS, file names of the form `xxxxxxxx.xxx` , where `x` denotes an alphanumeric character, are generally supported by modern systems. On some platforms, file names are case sensitive, and on other platforms, they are case insensitive. VU#439395 is an example of a vulnerability resulting from a failure to deal appropriately with case-sensitivity issues \[ [VU#439395](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-VU439395) \].

### Noncompliant Code Example (File Name 1)

In this noncompliant code example, unsafe characters are used as part of a file name:

An [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) is free to define its own mapping of the "nonsafe" characters. For example, when run on Red Hat Enterprise Linux 7.5, this noncompliant code example resulted in the following file name being revealed by the `ls` command:

### Compliant Solution (File Name 1)

Use a descriptive file name containing only the subset of ASCII previously described:

### Noncompliant Code Example (File Name 2)

This noncompliant code example is derived from [FIO30-C. Exclude user input from format strings](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/input-output-fio/fio30-c) , except that a newline is removed on the assumption that `fgets()` will include it:

No checks are performed on the file name to prevent troublesome characters. If an attacker knew this code was in a program used to create or rename files that would later be used in a script or automated process of some sort, he or she could choose particular characters in the output file name to confuse the later process for malicious purposes.

### Compliant Solution (File Name 2)

In this compliant solution, the program rejects file names that violate the guidelines for selecting safe characters:

Similarly, you must validate all file names originating from untrusted sources to ensure they contain only safe characters.

## MSC10-C. Character encoding: UTF8-related issues

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/08.msc10-c.md) — SHA-256 `4e504946ef9e4d1af021a7f1e18a221bc3dee95b70a9db5e46164f15ddd3f660`。

UTF-8 is a variable-width encoding for Unicode. UTF-8 uses 1 to 4 bytes per character, depending on the Unicode symbol. UTF-8 has the following properties:

-   The classical US-ASCII characters (0 to 0x7f) encode as themselves, so files and strings that are encoded with ASCII values have the same encoding under both ASCII and UTF-8.
-   It is easy to convert between UTF-8 and UCS-2 and UCS-4 fixed-width representations of characters.
-   The lexicographic sorting order of UCS-4 strings is preserved.
-   All possible 2^21 UCS codes can be encoded using UTF-8.

Generally, programs should validate UTF-8 data before performing other checks. The following table lists the well-formed UTF-8 byte sequences.

|                    |                  |                 |                   |                       |                       |                       |                       |
|:------------------:|:----------------:|:---------------:|:-----------------:|:---------------------:|:---------------------:|:---------------------:|:---------------------:|
| Bits of code point | First code point | Last code point | Bytes in sequence |        Byte 1         |        Byte 2         |        Byte 3         |        Byte 4         |
|         7          |      U+0000      |     U+007F      |         1         | `0xxxxxxx` |                       |                       |                       |
|         11         |      U+0080      |     U+07FF      |         2         | `110xxxxx` | `10xxxxxx` |                       |                       |
|         16         |      U+0800      |     U+FFFF      |         3         | `1110xxxx` | `10xxxxxx` | `10xxxxxx` |                       |
|         21         |     U+10000      |    U+1FFFFF     |         4         | `11110xxx` | `10xxxxxx` | `10xxxxxx` | `10xxxxxx` |

Although UTF-8 originated from the Plan 9 developers \[ [Pike 1993](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Pike93) \], Plan 9's own support covers only the low 16-bit range. In general, many "Unicode" systems support only the low 16-bit range, not the full 21-bit ISO 10646 code space \[ [ISO/IEC 10646:2012](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC10646-2012) \].

### Security-Related Issues

According to [RFC 2279: UTF-8, a transformation format of ISO 10646](http://www.faqs.org/rfcs/rfc2279.html) \[ [Yergeau 1998](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Yergeau98) \],

> Implementors of UTF-8 need to consider the security aspects of how they handle invalid UTF-8 sequences. It is conceivable that, in some circumstances, an attacker would be able to exploit an incautious UTF-8 parser by sending it an octet sequence that is not permitted by the UTF-8 syntax.
>
> A particularly subtle form of this attack can be carried out against a parser that performs security-critical validity checks against the UTF-8 encoded form of its input, but interprets certain invalid octet sequences as characters. For example, a parser might prohibit the null character when encoded as the single-octet sequence `00` , but allow the invalid two-octet sequence `    C0 80   ` and interpret it as a null character. Another example might be a parser which prohibits the octet sequence `    2F 2E 2E 2F   `(`    "/../"   ` ), yet permits the invalid octet sequence `    2F C0 AE 2E 2F   ` .

Following are more specific recommendations.

#### Accept Only the Shortest Form

Only the "shortest" form of UTF-8 should be permitted. Naive decoders might accept encodings that are longer than necessary, allowing for potentially dangerous input to have multiple representations. For example,

1.  Process A performs security checks but does not check for nonshortest UTF-8 forms.
2.  Process B accepts the byte sequence from process A and transforms it into UTF-16 while interpreting possible nonshortest forms.
3.  The UTF-16 text may contain characters that should have been filtered out by process A and can potentially be dangerous. These "nonshortest" UTF-8 attacks have been used to bypass security validations in high-profile products, such as Microsoft's IIS Web server.

[Corrigendum #1: UTF-8 Shortest Form](http://www.unicode.org/versions/corrigendum1.html) to the Unicode Standard \[ [Unicode 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Unicode06) \] describes modifications made to version 3.0 of the Unicode Standard to forbid the interpretation of the nonshortest forms.

#### Handling Invalid Inputs

UTF-8 decoders have no uniformly defined behavior upon encountering an invalid input. Following are several ways a UTF-8 decoder might behave in the event of an invalid byte sequence. Note that implementing these behaviors requires careful security considerations.

1.  Substitute for the replacement character "U+FFFD" or the wildcard character such as "?" when U+FFFD is not available.
2.  Ignore the bytes (for example, delete the invalid byte before the validation process; see "Unicode Technical Report #36, 3.5 Deletion of Code Points" for more information).
3.  Interpret the bytes according to a different character encoding (often the ISO-8859-1 character map; other encoding, such as Shift_JIS, is known to trigger self-XSS, and so is potentially dangerous).
4.  Fail to notice but decode as if the bytes were some similar bit of UTF-8.
5.  Stop decoding and report an error.

The following function, inspired by John Viega and Matt Messier's *Secure Programming Cookbook for C and C++* \[ [Viega 2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Viega03) \], detects invalid character sequences in a string but does not reject nonminimal forms. It returns `1` if the string is composed only of legitimate sequences; otherwise, it returns `0` .

#### Broken Surrogates

Encoding of individual or out-of-order surrogate halves should not be permitted. Broken surrogates are invalid in Unicode and introduce ambiguity when they appear in Unicode data. Broken surrogates are often signs of bad data transmission. They can also indicate internal bugs in an application or intentional efforts to find security vulnerabilities.

## MSC11-C. Incorporate diagnostic tests using assertions

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/09.msc11-c.md) — SHA-256 `b0c046efd44f0d39524bbf17c57620318d4d03180dbc2ff77a0106a9fff352fe`。

Incorporate diagnostic tests into your program using, for example, the `assert()` macro.

The `assert` macro expands to a void expression:

When it is executed, if `expression` (which must have a scalar type) is false, the `assert` macro outputs information about the failed assertion (including the text of the argument, the name of the source file, the source line number, and the name of the enclosing function) on the standard error stream, in an [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) format, and calls the `abort()` function.

In the following example, the test for integer wrap was omitted for the unsigned multiplication on the assumption that `   MAX_TABLE_SIZE * sizeof(char *)  ` cannot exceed `SIZE_MAX` . Although we *know* this is true, it cannot do any harm to codify this assumption.

Assertions are primarily intended for use during debugging and are generally turned off before code is deployed by defining the `NDEBUG` macro (typically as a flag passed to the compiler). Consequently, assertions should be used to protect against incorrect programmer assumptions and not for runtime error checking.

Assertions should never be used to verify the absence of runtime (as opposed to logic) errors, such as

-   Invalid user input (including command-line arguments and environment variables)
-   File errors (for example, errors opening, reading or writing files)
-   Network errors (including network protocol errors)
-   Out-of-memory conditions (for example, `malloc()` or similar failures)
-   System resource exhaustion (for example, out-of-file descriptors, processes, threads)
-   System call errors (for example, errors executing files, locking or unlocking mutexes)
-   Invalid permissions (for example, file, memory, user)

Code that protects against a buffer overflow, for example, cannot be implemented as an assertion because this code must be presented in the deployed executable.

In particular, assertions are generally unsuitable for server programs or embedded systems in deployment. A failed assertion can lead to a [denial-of-service attack](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc11-c#MSC11C.Incorporatediagnostictestsusingassertions-BB.Definitions#denial-of-serviceattack) if triggered by a malicious user, such as `size` being derived, in some way, from client input. In such situations, a soft failure mode, such as writing to a log file and rejecting the request, is more appropriate.

<a id="MSC11C.Incorporatediagnostictestsusingassertions-ncce_malloc"></a>

### Noncompliant Code Example ( `malloc()` )

This noncompliant code example uses the `assert()` macro to verify that memory allocation succeeded. Because memory availability depends on the overall state of the system and can become exhausted at any point during a process lifetime, a robust program must be prepared to gracefully handle and recover from its exhaustion. Consequently, using the `assert()` macro to verify that a memory allocation succeeded would be inappropriate because doing so might lead to an abrupt termination of the process, opening the possibility of a denial-of-service attack. See also [MEM11-C. Do not assume infinite heap space](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/memory-management-mem/mem11-c) and [void MEM32-C. Detect and handle memory allocation errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/the-void/void-mem32-c-detect-and-handle-memory-allocation-errors) .

<a id="MSC11C.Incorporatediagnostictestsusingassertions-cs_malloc"></a>

### Compliant Solution ( `malloc()` )

This compliant solution demonstrates how to detect and handle possible memory exhaustion:

## MSC12-C. Detect and remove code that has no effect or is never executed

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/10.msc12-c.md) — SHA-256 `238f8fb3fd487162c8d79ea07c34d7091c5447a5735db511a3d4710e9e2bed7b`。

Code that has no effect or is never executed (that is, dead or unreachable code) is typically the result of a coding error and can cause [unexpected behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unexpectedbehavior) . Such code is usually optimized out of a program during compilation. However, to improve readability and ensure that logic errors are resolved, it should be identified, understood, and eliminated.

Statements or expressions that have no effect should be identified and removed from code. Most modern compilers, in many cases, can warn about code that has no effect or is never executed. (See [MSC00-C. Compile cleanly at high warning levels](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc00-c) .)

### Noncompliant Code Example

This noncompliant code example demonstrates how dead code can be introduced into a program \[ [Fortify 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Fortify06) \]. The second conditional statement, `   if (s)  ` , will never evaluate true because it requires that `s` not be assigned `NULL` , and the only path where `s` can be assigned a non-null value ends with a `return` statement.

### Compliant Solution

Remediation of dead code requires the programmer to determine why the code is never executed and then to resolve the situation appropriately. To correct the preceding noncompliant code, the `return` is removed from the body of the first conditional statement.

### Noncompliant Code Example

In this example, the `strlen()` function is used to limit the number of times the function `s_loop()` will iterate. The conditional statement inside the loop evaluates to true when the current character in the string is the null terminator. However, because `strlen()` returns the number of characters that precede the null terminator, the conditional statement never evaluates true.

### Compliant Solution

Removing the dead code depends on the intent of the programmer. Assuming the intent is to flag and process the last character before the null terminator, the conditional is adjusted to correctly determine if the `i` refers to the index of the last character before the null terminator.

### Noncompliant Code Example (Assignment)

In this noncompliant code example, the comparison of `a` to `b` has no effect:

This code is likely a case of the programmer mistakenly using the equals operator `==` instead of the assignment operator `=` .

### Compliant Solution (Assignment)

The assignment of `b` to `a` is now properly performed:

### Noncompliant Code Example (Dereference)

In this example, a pointer increment and then a dereference occur, but the dereference has no effect:

### Compliant Solution (Dereference)

Correcting this example depends on the intent of the programmer. For example, if dereferencing `p` was a mistake, then `p` should not be dereferenced.

If the intent was to increment the value referred to by `p` , then parentheses can be used to ensure `p` is dereferenced and then incremented. (See [EXP00-C. Use parentheses for precedence of operation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/expressions-exp/exp00-c) .)

Another possibility is that `p` is being used to reference a memory-mapped device. In this case, the variable `p` should be declared as `volatile` .

### Noncompliant Code Example (if/else if)

A chain of if/else if statements is evaluated from top to bottom. At most, only one branch of the chain will be executed: the first one with a condition that evaluates to true. Consequently, duplicating a condition in a sequence of if/else if statements automatically leads to dead code.

Note that duplicating a condition violates this guideline only if the duplicate conditions always behave similarly...see a compliant solution below for a condition that is textually a duplicate but behaves differently.

### Compliant Solution (if/else if)

In this compliant solution, the third conditional expression has been corrected.

### Compliant Solution (Conditional Side-Effects)

This code does not violate this recommendation, because even though the conditions are textually identical, they have different side effects, because the `getc()` function advances the stream marker.

### Noncompliant Code Example (logical operators)

Using the same subexpression on either side of a logical operator is almost always a mistake.  In this noncompliant code example, the rightmost subexpression of the controlling expression of each `if` statement has no effect.

### Compliant Solution (logical operators)

In this compliant solution, the rightmost subexpression of the controlling expression of each `if` statement has been removed.

### Noncompliant Code Example (Unconditional Jump)

Unconditional jump statements typically has no effect.

### Compliant Solution (Unconditional Jump)

The continue statement has been removed from this compliant solution.

### Exceptions

<a id="MSC12C.Detectandremovecodethathasnoeffectorisneverexecuted-MSC12-EX1"></a> **MSC12-EX1:** In some situations, seemingly dead code may make software resilient. An example is the `default` label in a `switch` statement whose controlling expression has an enumerated type and that specifies labels for all enumerations of the type. (See [MSC01-C. Strive for logical completeness](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc01-c) .) Because valid values of an enumerated type include all those of its underlying integer type, unless enumeration constants are provided for all those values, the `default` label is appropriate and necessary.

<a id="MSC12C.Detectandremovecodethathasnoeffectorisneverexecuted-MSC12-EX2"></a> **MSC12-EX2:** It is permissible to temporarily remove code that may be needed later. (See [MSC04-C. Use comments consistently and in a readable fashion](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc04-c) for an illustration.)

<a id="MSC12C.Detectandremovecodethathasnoeffectorisneverexecuted-MSC12-EX3"></a> **MSC12-EX3:** Unused functions and variables that are part of an exported library do not violate this guideline.  Likewise, code that is never executed because it is `#ifdef` ed out does not violate this guideline, on the grounds that it could be subsequently used in another application, or built on a different platform.

## MSC13-C. Detect and remove unused values

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/11.msc13-c.md) — SHA-256 `d27bd0103147dc0438762b9720de4acb8eb3818d52f8f38f6075f6c0a83e6eeb`。

The presence of unused values may indicate significant logic errors. To prevent such errors, unused values should be identified and removed from code.

This recommendation is a specific case of [MSC12-C. Detect and remove code that has no effect or is never executed](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc12-c) .

### Noncompliant Code Example

In this example, `p2` is assigned the value returned by `bar()` , but that value is never used. Note this example assumes that `foo()` and `bar()` return valid pointers (see [DCL30-C. Declare objects with appropriate storage durations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl30-c) ).

### Compliant Solution

This example can be corrected in many different ways, depending on the intent of the programmer. In this compliant solution, `p2` is found to be extraneous. The calls to `bar()` and `baz()` can be removed if they do not produce any side effects.

### Exceptions

<a id="MSC13C.Detectandremoveunusedvalues-MSC13-EX1"></a> **MSC13-EX1:** Initializing a variable with a default value, such as 0, which gets subsequently overwritten may be inefficient, but is less of a problem than reading an uninitialized value, as per [EXP33-C. Do not read uninitialized memory](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp33-c) .

## MSC14-C. Do not introduce unnecessary platform dependencies

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/12.msc14-c.md) — SHA-256 `e5fe028ada53afc0b80a6a8f25d1330b28376933b15b923ae11bffe040ba6eca`。

Platform dependencies may be introduced to improve performance on a particular platform. This can be a dangerous practice, particularly if these dependencies are not appropriately documented during development and addressed during porting. Platform dependencies that have no performance or other benefits should consequently be avoided because they may introduce errors during porting.

The C Standard identifies four different kinds of nonportable behavior. Each section of Annex J of the C Standard enumerates distinct instances of behaviors of each kind.

|                                                                                                                                         |                                                                                                                                                                                                                       |                                                                      |
|-----------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------|
| Nonportable Behavior                                                                                                                    | Definition                                                                                                                                                                                                            | Annex J Section                                                      |
| [Unspecified behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unspecifiedbehavior)                       | Behavior for which the standard provides two or more possibilities and imposes no further requirements on which is chosen in any instance.                                                                            | J.1                                                                  |
| [Undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior)                           | Behavior, upon use of a nonportable or erroneous program construct or of erroneous data, for which the standard imposes no requirements. An example of undefined behavior is the behavior on signed integer overflow. | [J.2](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior) |
| [Implementation-defined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) | Unspecified behavior whereby each implementation documents how the choice is made.                                                                                                                                    | J.3                                                                  |
| [Locale-specific behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-locale-specificbehavior)               | Behavior that depends on local conventions of nationality, culture, and language that each implementation documents.                                                                                                  | J.4                                                                  |

An example of [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) is passing a null `char*` pointer as an argument to the `printf` function corresponding to the `%s` format specification. Although some [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) (such as the [GNU C Library](http://www.gnu.org/software/libc/) ) provide well-defined semantics for this case, others do not, causing programs that rely on this behavior to fail abnormally.

An example of [unspecified behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unspecifiedbehavior) is the order in which the arguments to a function are evaluated.

An example of [implementation-defined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) is the propagation of the high-order bit when a signed integer is shifted right.

Most legitimate platform dependencies can and should be isolated in separate modules that expose portable, platform-agnostic interfaces to platform-specific implementations. Portable applications that cannot avoid relying on platform-specific details should always provide a generic, portable, standards-based solution as a fallback mechanism for the platform-specific alternative. That way, such an application can be more easily ported to new platforms without an excessive risk of [security flaws](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-securityflaw) caused by assumptions that do not hold in the new environment.

### Noncompliant Code Example

This noncompliant code example uses the complement operator in the test for integer overflow. It assumes both numbers are nonnegative:

This code assumes that the implementation uses two's complement representation. This assumption is commonly true but is not guaranteed by the standard.

This code sample also violates [INT14-C. Avoid performing bitwise and arithmetic operations on the same data](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int14-c) .

### Compliant Solution

This compliant solution implements a [strictly conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-strictlyconforming) test for overflow:

If the noncompliant form of this test is truly faster, talk to your compiler vendor because, if these tests are equivalent, optimization should occur. If both forms have the same performance, prefer the portable form.

<a id="MSC14C.Donotintroduceunnecessaryplatformdependencies-ncce_strerror_r"></a>

### Noncompliant Code Example ( `strerror_r` )

The GNU libc implementation of [strerror_r](https://linux.die.net/man/3/strerror_r) declares the function to return `char*` , in conflict with the POSIX specification. This noncompliant code example relies on this return type to pass the return value as an argument to the `%s` formatting directive to `fprintf` . The behavior of the example will be [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) on a platform that declares the return type of `strerror_r()` to be `int` , in accordance with POSIX.

<a id="MSC14C.Donotintroduceunnecessaryplatformdependencies-cs_strerror_r"></a>

### Compliant Solution ( `strerror_r` )

The compliant solution disables the nonconforming declaration of `strerror_r()` by explicitly requesting POSIX conformance before including the `<string.h>` header that declares the function and handles the function's failure by copying the `   "Unknown error"  ` string into the buffer.

Note that the function assigns the result of the call to `strerror_r()` to a variable of type `int` . This assignment is a defense-in-depth strategy guarding against inadvertently invoking `strerror_r()` that returns `char*` : a [conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conforming) compiler is required to issue a diagnostic for the ill-formed conversion from `char*` to `int` .

## MSC15-C. Do not depend on undefined behavior

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/13.msc15-c.md) — SHA-256 `a78a559c79ddbf9d48328fc6f8d52d04ad36ca4cecfade226a44f3cd9fa921b9`。

The C Standard, subclause 3.5.3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], defines *undefined behavior* as

> behavior, upon use of a nonportable or erroneous program construct or of erroneous data, for which this document imposes no requirements

Subclause 4 explains how the standard identifies undefined behavior (see also [undefined behavior 1](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_1) of Annex J).

> If a "shall" or "shall not" requirement that appears outside of a constraint or runtime-constraint is violated, the behavior is undefined. Undefined behavior is otherwise indicated in this document by the words "undefined behavior" or by the omission of any explicit definition of behavior. There is no difference in emphasis among these three; they all describe "behavior that is undefined".

Annex J, subclause J.2, "Undefined behavior," enumerates the circumstances under which the behavior of a program is undefined. This list is duplicated on the [CC. Undefined Behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior) page.

Behavior can be classified as undefined by the C standards committee for the following reasons:

-   To give the implementor license not to catch certain program errors that are difficult to diagnose

-   <div class="O1">

    To avoid defining obscure corner cases that would favor one [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) strategy over another

    </div>

-   To identify areas of possible [conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conforming) language extension: the implementor may augment the language by providing a definition of the officially [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior)

Conforming implementations can deal with undefined behavior in a variety of fashions, such as ignoring the situation completely, with unpredictable results; translating or executing the program in a documented manner characteristic of the environment (with or without the issuance of a diagnostic message); or terminating a translation or execution (with the issuance of a diagnostic message). Because compilers are not obligated to generate code for undefined behavior, these behaviors are candidates for optimization. By assuming that undefined behaviors will not occur, compilers can generate code with better performance characteristics.

Increasingly, compiler writers are taking advantage of undefined behaviors in the C programming languages to improve optimizations. These optimizations frequently interfere with the ability of developers to perform cause-effect analysis on their source code—that is, to analyze the dependence of downstream results on prior results. Consequently, these optimizations are eliminating causality in software and are increasing the probability of software faults, defects, and [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) .

All of this puts the onus on the programmer to develop code that is free from undefined behaviors, with or without the help of the compiler.

### Noncompliant Code Example

An example of [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) in C is the behavior on signed integer overflow (see also [INT32-C. Ensure that operations on signed integers do not result in overflow](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int32-c) ). This noncompliant code example depends on this behavior to catch the overflow:

This code checks for signed integer overflow by testing whether `   a + 100 > a  ` . This test cannot evaluate to false unless an integer overflow occurs. However, because a [conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conforming) [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) is not required to generate code for undefined behavior, and signed integer overflow is undefined behavior, this code may be compiled out. For example, GCC 4.1.1 optimizes out the assertion for all optimization levels, and GCC 4.2.3 optimizes out the assertion for programs compiled with `-O2` -level optimization and higher.

On some platforms, the integer overflow causes the program to terminate (before it has an opportunity to test).

### Compliant Solution

This compliant solution does not depend on undefined behavior:

## MSC17-C. Finish every set of statements associated with a case label with a break statement

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/14.msc17-c.md) — SHA-256 `e6455c24d8555c7f257031e63c5a50e5449975f257e0212f61c6c3f6ab52a479`。

A `switch` statement consists of several case labels, plus a default label. The default label is optional but recommended. (See [MSC01-C. Strive for logical completeness](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc01-c) .) A series of statements following a case label conventionally ends with a `break` statement; if omitted, control flow falls through to the next case in the `switch` statement block. Because the `break` statement is not required, omitting it does not produce compiler diagnostics. If the omission was unintentional, it can result in an unexpected control flow.

### Noncompliant Code Example

In this noncompliant code example, the case where `widget_type` is `WE_W` lacks a `break` statement. Consequently, statements that should be executed only when `widget_type` is `WE_X` are executed even when `widget_type` is `WE_W` .

### Compliant Solution

In this compliant solution, each sequence of statements following a case label ends with a `break` statement:

A `break` statement is not required following the case where `widget_type` is `WE_Y` because there are no statements before the next case label, indicating that both `WE_Y` and `WE_Z` should be handled in the same fashion.

A `break` statement is not required following the default case because it would not affect the control flow.

### Exceptions

**MSC17-C-EX1:** The last label in a `switch` statement requires no final `break` . It will conventionally be the `default` label.

**MSC17-C-EX2:** When control flow is intended to cross statement labels, it is permissible to omit the `break` statement. In these instances, the unusual control flow must be made clear, such as by adding the `[[fallthrough]]` C23 attribute, the `__attribute__((__fallthrough__))` GNU extension, or a simple comment.

## MSC18-C. Be careful while handling sensitive data, such as passwords, in program code

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/15.msc18-c.md) — SHA-256 `310db2d8e54d8f7e2c3f28a2fcbd3b61d103a59afe6ca50e26606d2629512c52`。

Many applications need to handle sensitive data either in memory or on disk. If this sensitive data is not protected properly, it might lead to loss of secrecy or integrity of the data. It is very difficult (or expensive) to completely secure all the sensitive data. Users tend to use the same passwords everywhere. So even if your program is a simple game that stores the user's profile information and requires the user to enter a password, the user might choose the same password he or she uses for an online bank account for your game program. Now the user's bank account is only as secure as your program enables it to be.

There are simple steps you can take to secure sensitive data in your programs.

###### Prefer the system's authentication dialog (or any other mechanism provided by the OS) for authentication to privileged services.

If you are accessing some privileged service already installed on the system, most likely that service will have some mechanism to take a password from the user. Before asking the user for a user name and password from your application, check if the service itself authenticates the user in some way. If so, let the service handle the authentication because doing so would at least not increase the footprint of the sensitive data.

###### Do not hard code sensitive data in programs.

See [MSC41-C. Never hard code sensitive information](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/miscellaneous-msc/msc41-c) for details.

###### Disable memory dumps.

Memory dumps are automatically created when your program crashes. They can contain information stored in any part of program memory. Therefore, memory dumps should be disabled before an application is shipped to users. See [MEM06-C. Ensure that sensitive data is not written out to disk](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/memory-management-mem/mem06-c) for details.

###### Do not store sensitive data beyond its time of use in a program.

Sensitive data that is stored in memory can get written to disk when a page is swapped out of the physical memory. (See next point for details about keeping sensitive data on disk.) You may be able to "lock" your data to keep it from swapping out. Your program will generally need administrative privileges to do so successfully, but it never hurts to try. See [MEM06-C. Ensure that sensitive data is not written out to disk](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/memory-management-mem/mem06-c) for details.

###### Do not store sensitive data in plaintext (either on disk or in memory).

See [MEM06-C. Ensure that sensitive data is not written out to disk](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/memory-management-mem/mem06-c) .

While using a password, consider storing its hash instead of plaintext. Use the hash for comparisons and other purposes. The following code \[ [Viega 2001](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Viega01) \] illustrates:

###### If you must store sensitive data, encrypt it first.

1.  If encrypting or hashing sensitive data, do not implement your own encryption functions (or library). Use proven secure crypto libraries, which have been extensively tested for security.
2.  If using standard crypto libraries, be aware that they have certain requirements (documented with the library) for the key sizes and other properties. Choose keys that satisfy these conditions.
3.  Do not store the encryption keys (you can derive the key from the hash of the user's password or any other cryptographic mechanism, provided the above condition holds). If the key is to be stored, store it securely.

###### Securely erase sensitive data from disk and memory.

1.  Be aware of compiler optimization when erasing memory. (See [MSC06-C. Beware of compiler optimizations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc06-c) .)
2.  Use secure erase methods specified in U.S. Department of Defense Standard 5220 \[ [DOD 5220](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-DOD5220) \] or Peter Gutmann's paper \[ [Gutmann 1996](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Gutmann96) \].

## MSC19-C. For functions that return an array, prefer returning an empty array over a null value

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/16.msc19-c.md) — SHA-256 `560a4abca19a4234514d2631ddbec94a5a3eb4c05df09aab6e6b5f307eeaf69b`。

Many functions have the option of returning a pointer to an object or returning `NULL` if a valid pointer cannot be produced. Some functions return arrays, which appear like a pointer to an object. However, if a function has the option of returning an array or indicating that a valid array is not possible, it should not return `NULL` . Instead, the function should return an empty array. Often, code that calls a function that returns an array intends merely to iterate over the array elements. In this case, the calling code need not change—iterating over the elements works correctly even if the returned array is empty, so the calling code need not check the return value for `NULL` .

This situation is complicated by the fact that C does not keep track of the length of an array. However, two popular methods have emerged to emulate this behavior. The first is to wrap the array in a struct with an integer storing the length. The second is to place a sentinel value at the end of the data in the array. This second approach is most commonly manifested in null-terminated byte strings (NTBSs).

### Noncompliant Code Example (Struct)

In this noncompliant code example, an inventory system keeps track of the total number of different items (denoted `length` ). Each item is given an index in the array, and the value for that index is the stock of that item. Adding a new item increases `length` in the struct. Stocking more of an item increases the value for that item's index. For example, if 5 books and 2 erasers are in stock, the inventory would be `   stockOfItem[0] = 5  `and`   stockOfItem[1] = 2  ` , assuming books are index 0 and erasers are index 1.

The problem arises in this setup when no items are being stocked. `getStock` would recognize that `   length = 0  ` and would return `NULL` . In this noncompliant code example, erroneous behavior results from `getStock` returning `NULL` while `main` neglects to check for such a value. It results in an [abnormal program termination](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) after returning to the `main` function.

### Compliant Solution

This compliant solution eliminates the `NULL` return and returns the `item` array, even if it is zero-length. The main function can effectively handle this situation without exhibiting erroneous behavior. Since the array lives on the stack, it must prevent returning a value in the stack frame (as mandated by [DCL30-C. Declare objects with appropriate storage durations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl30-c) ). So the getStack() function also takes a pointer to `Inventory` , so that it can return a pointer to its contents safely.

### Noncompliant Code Example (Sentinel Value)

This noncompliant code example implements an inventory system similar to the one described previously. However, instead of storing the length of the array in a struct, a sentinel value of `FINAL_ITEM` is used. The value for the index following the last item is set as `FINAL_ITEM` . It is assumed that out-of-stock items (assigned value 0) are removed from the array, and the contents of later items are shifted to lower indexes.

The following code attempts to return an array of the items in stock, sorted by the amount of each item in stock. The `arraySort` function incorrectly returns `NULL` instead of a pointer to an empty array when no items are in stock. The null return is improperly handled by the `main` function, which is attempting to print out the returned array, and an [abnormal program termination](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) results.

### Compliant Solution (Sentinel Value)

This compliant solution correctly returns an empty array in the `sortedArray` function. If the size of the array is 0, then `sortedArray` allocates an array of size 1 and fills it with the sentinel value. It can then successfully return that array to the caller function.

## MSC20-C. Do not use a switch statement to transfer control into a complex block

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/17.msc20-c.md) — SHA-256 `2ba9260321d5f7a3e58cba46b4270d67c83f12b3bc6952e0978fd2690c79da07`。

A `switch` statement can be mixed with a block of code by starting the block in one case label, then having another case label within the block. The block can be pictured as spanning more than one case statement.

Subclause 6.8.4.2, paragraph 2, of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] says,

> If a switch statement has an associated case or default label within the scope of an identifier with a variably modified type, the entire switch statement shall be within the scope of that identifier. <sup>154</sup>

Footnote 154 says:

> That is, the declaration either precedes the switch statement, or it follows the last case or default label associated with the switch that is in the block containing the declaration.

Note that the standard does not disallow jumping via `goto` or `switch` into loops that do not involve variably modified type identifiers. Consequently, loops and other blocks can be freely intermixed with `switch` statements. Unfortunately, such intermixing creates code that is, at best, confusing and unclear in what it does, which can cause undesirable behavior.

The examples here fall under the exception **MSC17-C-EX2** in [MSC17-C. Finish every set of statements associated with a case label with a break statement](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc17-c) .

### Noncompliant Code Example

This example shows the use of the `switch` statement to jump into a `for` loop:

#### Implementation Details

When `   i = 1  ` , the entire `for` loop is executed. When `   i = 2  ` , two increments to `j` are made before the loop starts. When `   i = 3  ` , one increment to `j` is made before the loop starts. The default case is no loop. Consequently, the function has the following behavior:

|                  |                     |
|------------------|---------------------|
| `i` | `f(i)` |
| 1                | 12                  |
| 2                | 12                  |
| 3                | 11                  |
| Other values     | 0                   |

### Compliant Solution

The compliant solution separates the `switch` and `for` blocks:

### Noncompliant Code Example (Duff's Device)

[Duff's device](http://www.lysator.liu.se/c/duffs-device.html) is a curious optimization applied to code intended to perform a serial copy. That is, it copies a series of bytes into one memory output in turn. A simple code to do this would be as follows:

However, this code might be unacceptably slow because the `while` condition is performed `count` times. The classic code for Duff's device unrolls this loop to minimize the number of comparisons performed:

In this code, the first iteration of the loop is subject to the `switch` statement, so it performs `   count % 8  ` assignments. Each subsequent iteration of the loop performs 8 assignments. (Being outside the loop, the `switch` statement is ignored.) Consequently, this code performs `count` assignments, but only `n` comparisons, so it is usually faster.

The code is widely considered to be valid C and C++ and is supported by all compliant compilers. When describing Duff's device, the creator \[ [Duff 1988](http://AA.%20Bibliography#Duff%2088) \] noted,

> Many people . . . have said that the worst feature of C is that switches don't break automatically before each case label. This code forms some sort of argument in that debate, but I'm not sure whether it's for or against.

### Compliant Solution (Duff's Device)

This is an alternative [implementation](http://BB.%20Definitions#implementation) of Duff's device, which separates the `switch` statement and loop:

## MSC21-C. Use robust loop termination conditions

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/18.msc21-c.md) — SHA-256 `3904deb35bbd4b723fb82a7a432ed84949bec18eaed7a4c1c9acebf1b43fb43a`。

C defines `<` , `>` , `<=` , and `>=` to be *relational operators* , and it defines `==` and `!=` to be *equality operators* .

If a `for` or `while` statement uses a loop counter, than it is safer to use a relational operator (such as `<` ) to terminate the loop than to use an equality operator (such as `!=` ).

<a id="MSC21C.Userobustloopterminationconditions-nce_inequality_multistep"></a>

### Noncompliant Code Example (Equality Operators)

This noncompliant code example appears to have five iterations, but in fact, the loop never terminates:

<a id="MSC21C.Userobustloopterminationconditions-cs_relational_multistep"></a>

### Compliant Solution (Relational Operators)

Using the relational operator `<=` instead of an equality operator guarantees loop termination:

<a id="MSC21C.Userobustloopterminationconditions-nce_inequality"></a>

### Noncompliant Code Example (Equality Operators)

It is also important to ensure termination of loops where the start and end values are variables that might not be properly ordered. The following function assumes that `   begin < end  ` ; if this is not the case, the loop will never terminate:

<a id="MSC21C.Userobustloopterminationconditions-cs_relational"></a>

### Compliant Solution (Relational Operators)

Again, using a relational operator instead of equivalence guarantees loop termination. If `   begin >= end  ` , the loop never executes its body.

<a id="MSC21C.Userobustloopterminationconditions-nce_boundary"></a>

### Noncompliant Code Example (Boundary Conditions)

Numerical comparison operators do not always ensure loop termination when comparing against the minimum or maximum representable value of a type, such as `SIZE_MAX` :

<a id="MSC21C.Userobustloopterminationconditions-cs_boundary"></a>

### Compliant Solution (Boundary Conditions)

A compliant solution is to compare against the difference between the maximum representable value of a type and the increment:

<a id="MSC21C.Userobustloopterminationconditions-MSC21-EX1"></a>

### Exceptions

**MSC21-C-EX1:** If the loop counter is incremented by 1 on each iteration, and it is known that the starting value of a loop is less than or equal to the ending value, then an equality operator may be used to terminate the loop. Likewise, if the loop counter is decremented by 1 on each iteration, and it is known that the starting value of the loop is greater than or equal to the ending value, then an equality operator may be used to terminate the loop.

## MSC22-C. Use the setjmp(), longjmp() facility securely

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/19.msc22-c.md) — SHA-256 `8624ba1760398de541c5bc9a4c0345b760f284326776c75b686ee97a0e92ef97`。

The `setjmp()` macro should be invoked from only one of the contexts listed in subclause 7.13.2.1 of the C Standard \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \]. Invoking `setjmp()` outside of one of these contexts results in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See [undefined behavior 125](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_125) .)

After invoking `longjmp()` , non-volatile-qualified local objects should not be accessed if their values could have changed since the invocation of `setjmp()` . Their value in this case is considered [indeterminate](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) , and accessing them is undefined behavior. (See undefined behaviors [127](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_127) and [10](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_10) .)

The `longjmp()` function should never be used to return control to a function that has terminated execution. (See [undefined behavior 126](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_126) .)

Signal masks, floating-point status flags, and the state of open files are *not* saved by the `setjmp()` function. If signal masks need to be saved, the POSIX `sigsetjmp()` function should be used.

This recommendation is related to [SIG30-C. Call only asynchronous-safe functions within signal handlers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/signals-sig/sig30-c) and [ENV32-C. All exit handlers must return normally](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/environment-env/env32-c) .

### Noncompliant Code Example

This noncompliant code example calls `setjmp()` in an assignment statement, resulting in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) :

### Compliant Solution

Placing the call to `setjmp()` in the `if` statement and, optionally, comparing it with a constant integer removes the undefined behavior, as shown in this compliant solution:

### Noncompliant Code Example

Any attempt to invoke the `longjmp()` function to transfer control to a function that has completed execution results in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) :

#### Implementation Details

Compiled at -O0 using GCC 7.5 or Clang 8.0 on Ubuntu 18.04 (Linux for x86-64), the preceding example outputs the following when run:

Because `g()` has finished executing at the time `longjmp()` is called, it is no longer on the stack. When `do_stuff()` is invoked, its stack frame occupies the same memory as the old stack frame of `g()` . In this case, `a` was located in the same location as the return address of function `g()` . The assignment of `b` overwrites the return address, so when `longjmp()` sends control back to function `g()` , the function returns to the wrong address (in this case, to function `bad()` ).

If the array `b` were user specified, the user would be able to set the return address of function `g()` to any location.

### Compliant Solution

The `longjmp()` function should be used only when the function containing the corresponding `setjmp()` is guaranteed not to have completed execution, as in the following example:

There is no risk of overwriting a return address because the stack frame of `main()` (the function that invoked `setjmp()` ) is still on the stack; so when `do_stuff()` is invoked, the two stack frames will not overlap.

### Noncompliant Code Example

In this noncompliant example, non-volatile-qualified objects local to the function that invoked the corresponding `setjmp()` have [indeterminate values](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) after `longjmp()` is executed if their value has been changed since the invocation of `setjmp()` :

#### Implementation Details

Calling `f()` will print `2` if you compile with `-O0` , but will print `0` if you compile with `-O2` .  This involves using GCC 7.5 or Clang 8.0 on Ubuntu 18.04 (Linux x86-64).

### Compliant Solution

If an object local to the function that invoked `setjmp()` needs to be accessed after `longjmp()` returns control to the function, the object should be volatile-qualified:

This will now correctly print `2` regardless of optimization level.

## MSC23-C. Beware of vendor-specific library and language differences

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/20.msc23-c.md) — SHA-256 `039015ea02d586281d4ec3e7bedf6fa9721785f9617657f90a2c072260e5302b`。

When compiling with a specific vendor's [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) of the C language, and related libraries, be aware that, unfortunately, standards conformance can differ from vendor to vendor. Be certain to read your vendor's documentation to reduce the likelihood of accidentally relying on implementation-specific behavior or deviations.

### Implementation-Specific Deviations

Implementation-specific deviations are listed below. The Version column lists the latest version of the compiler or library that exhibits the behavior.

#### Microsoft Visual Studio

| API | Description | Version |
| --- | --- | --- |
| Variable-length arrays | Absent | 2012 |
| `static` and type qualifiers in<br /><br />parameter array declarators | Absent | 2012 |
| `_Static_assert` | Absent; can use `static_assert` , as in C++11 | 2012 |
| `_Noreturn` | Absent; can use `__declspec(noreturn)` | 2012 |
| `inline` | Absent; can use `__inline` | 2012 |
| Hexadecimal floating-point constants | Absent | 2012 |
| Compound literals | Absent | 2012 |
| Designated initializers | Absent | 2012 |
| Implicit function declarations | Support was removed in C11, but it is still supported in MSVC (with diagnostic) | 2012 |
| Mixed declarations and code | Absent | 2012 |
| `_Pragma` | Absent; can use `__pragma` | 2012 |
| `_Bool` | Absent | 2012 |
| `_Complex` | Absent | 2012 |
| `__func__` | Absent; can use `__FUNCTION__` | 2012 |
| idempotent type qualifiers | Diagnosed, but functional | 2012 |
| `_Thread_local` | Absent; can use `__declspec(thread)` |  |
| <complex.h> | Absent; `__STDC_NO_COMPLEX__` is not defined | 2012 |
| `isblank()` | Absent | 2012 |
| <fenv.h> | Absent | 2012 |
| <inttypes.h> | Absent | 2012 |
| `float_t` , `double_t` , `HUGE_VALF, HUGE_VALL, INFINITY, NAN` | Absent | 2012 |
| #pragma `FP_CONTRACT` | Spelled `fp_contract` instead of `FP_CONTRACT` | 2012 |
| `fpclassify()` | Absent, use `_fpclass()` instead | 2012 |
| `isfinite()` , `isinf()` | Absent, use `_finite()` or `_finitef()` instead | 2012 |
| `isnan()` | Absent, use `_isnan()` or `_isnanf()` instead | 2012 |
| `isnormal()` | Absent, use `_fpclass()` instead | 2012 |
| `signbit()` | Absent | 2012 |
| `acosh()` , `asinh()` , `atanh()` | Absent | 2012 |
| `exp2()` , `expm1()` , `ilogb()` , `log1p()` ,<br /><br />`log2()` | Absent | 2012 |
| `scalbn()` | Absent; use `_scalb()` (or `_scalbf()` on x64 targets only) instead | 2012 |
| `cbrt()` | Absent | 2012 |
| `erf()` , `erfc()` , `lgamma()` , `tgamma()` | Absent | 2012 |
| `nearbyint()` , `rint()` , `lrint()` ,<br /><br />`round()` , `lround()` , `trunk()` | Absent | 2012 |
| `remainder()` , `remquo()` | Absent | 2012 |
| `copysign()` | Absent; use `_copysign()` instead | 2012 |
| `nan()` , `nexttoward()` | Absent | 2012 |
| `nextafter()` | Absent; use `_nextafter()` or `_nextafterf()` instead | 2012 |
| `fdim()` , `fmax(),` `fmin()` | Absent | 2012 |
| `fma()` | Absent | 2012 |
| `isgreater()` , `isgreaterequal()` ,<br /><br />`isless()` , `islessequal()` ,<br /><br />`isunordered()` | Absent | 2012 |
| `<stdalign.h>` | Absent |  |
| `va_copy()` | Absent | 2012 |
| `<stdatomic.h>` | Absent; `__STDC_NO_ATOMICS__` is not defined | 2012 |
| `<stdbool.h>` | Absent; `_Bool` is not supported in `/TC` mode, but `bool` is supported in `/TP` mode | 2012 |
| `max_align_t` | Absent | 2012 |
| Printing format specifier [type field](https://msdn.microsoft.com/en-us/library/hf4y5e3w.aspx) | `S` is MSVC-specific; `c` , `s` , and `z` are not conforming [ [ISO/IEC 9899-2011](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AA.Bibliography-ISO/IEC9899-2011) ]; `F` is unsupported | 2012 |
| Printing format specifier [size field](https://msdn.microsoft.com/en-us/library/tcxf1dw6.aspx) | `I` , `I32` and `I64` are MSVC-specific; `h` , `w` and `l` are not conforming [ [ISO/IEC 9899-2011](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AA.Bibliography-ISO/IEC9899-2011) ]; `hh` , `j` , `z` and `t` unsupported | 2012 |
| Scanning format specifier [type field](https://msdn.microsoft.com/en-us/library/6ttkkkhh.aspx) | `S` is MSVC-specific; `c` and `s` are not conforming [ [ISO/IEC 9899-2011](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AA.Bibliography-ISO/IEC9899-2011) ]; `p` is unsupported | 2012 |
| Scanning format specifier [size field](https://msdn.microsoft.com/en-us/library/xdb9w69d.aspx) | `I64` is MSVC-specific; `h` , `l` , and `L` prefixes are not conforming [ [ISO/IEC 9899-2011](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AA.Bibliography-ISO/IEC9899-2011) ]; `hh` , `j` , `z` , and<br /><br />`t` are unsupported | 2012 |
| `snprintf()` | Absent; beware that `_snprintf()` 's semantics are sufficiently different that it is not advisable as a replacement | 2012 |
| `vfscanf()` , `vscanf()` , `vsscanf()` | Absent | 2012 |
| [`fopen()` , `freopen()`](https://msdn.microsoft.com/en-us/library/yeby3zcb(v=vs.110).aspx) | Mode parameter not conforming [ [ISO/IEC 9899-2011](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AA.Bibliography-ISO/IEC9899-2011) ]; `t` , `c` , `n` , `N` , `S` , `R` , `T` , `D` and `css` are MSVC<br /><br />extensions; `x` is unsupported; see remarks | 2012 |
| `atoll()` | Absent; use `_atoi64()` instead | 2012 |
| `strtof()` , `strtold()` | Absent | 2012 |
| `strtoll()` | Absent; use `_strtoi64()` instead | 2012 |
| `strtoull()` | Absent; use `_strtoui64()` instead | 2012 |
| `aligned_alloc()` | Absent; use `_aligned_malloc()` instead (beware, the parameter order is reversed) | 2012 |
| `at_quick_exit()` , `quick_exit()` | Absent | 2012 |
| `_Exit()` | Absent; use `_exit()` instead | 2012 |
| <stdnoreturn.h> | Absent; use `__declspec(noreturn)` instead | 2012 |
| `<tgmath.h>` | Absent | 2012 |
| `<threads.h>` | Absent; `__STDC_NO_THREADS__` is not defined | 2012 |
| `TIME_UTC,` `struct timespec,`<br /><br />`timespec_get()` | Absent | 2012 |
| `strftime(), wcsftime()` | `z` is not conforming [ [ISO/IEC 9899-2011](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AA.Bibliography-ISO/IEC9899-2011) ]; `C` , `D` , `e` , `F` , `g` , `G` , `h` , `n` , `r` , `R` , `t` , `T` , `u` , `V` unsupported | 2012 |
| `<uchar.h>` | Absent | 2012 |
| `vfwscanf()` , `vswscanf()` , `vwscanf()` | Absent | 2012 |
| `fwide()` | [Unsupported](https://msdn.microsoft.com/en-us/library/aa985619.aspx) | 2012 |
| `wcstof()` , `wcstold()` | Absent | 2012 |
| `wcstoll()` | Absent; use `_wcstoi64()` instead | 2012 |
| `wcstoull()` | Absent; use `_wcstoui64()` instead | 2012 |
| `iswblank()` | Absent | 2012 |
| `FLT_EVAL_METHOD` , `*_HAS_SUBNORM` ,<br /><br />`*_DECIMAL_DIG` , `*_TRUE_MIN` | Absent | 2012 |
| `__STDC_LIB_EXT1__, __STDC_WANT_LIB_EXT1__` | Not defined; instead, MSVC uses `__STDC_SECURE_LIB__` and `__STDC_WANT_SECURE_LIB__` | 2012 |
| `fopen_s(), freopen_s()` | Uses the same mode strings as `fopen()` ; `u` prefix is unsupported; see remarks | 2012 |
| `vfscanf_s()` , `vscanf_s()` ,<br /><br />`vsscanf_s()` | Absent | 2012 |
| `constraint_handler_t` ,<br /><br />`set_constraint_handler_s(), abort_handler_s(), ignore_handler_s()` | Absent; use `_invalid_parameter_handler` and `_set_invalid_parameter_handler()` instead.  Beware that the `_invalid_parameter_handler` signature is considerably different than<br /><br />that of `constraint_handler_t` .  No replacement for `abort_handler_s()` or<br /><br />`ignore_handler_s()` | 2012 |
| `bsearch_s()` | Not conforming [ [ISO/IEC 9899-2011](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AA.Bibliography-ISO/IEC9899-2011) ]; beware that the comparison function pointer's signature is<br /><br />different from the standard; namely that the context parameter comes first in MSVC but last in ISO C | 2012 |
| `qsort_s()` | Not conforming [ [ISO/IEC 9899-2011](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AA.Bibliography-ISO/IEC9899-2011) ]; beware that the comparison function's pointer signature is different<br /><br />from the standard; namely that the context parameter comes first in MSVC but last in ISO C | 2012 |
| `strtok_s()` | Not conforming [ [ISO/IEC 9899-2011](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AA.Bibliography-ISO/IEC9899-2011) ]; the function signature is missing the `rsize_t *` parameter | 2012 |
| `memset_s(), strerrorlen_s()` | Absent | 2012 |
| `gmtime_s() , localtime_s()` | Not conforming [ [ISO/IEC 9899-2011](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AA.Bibliography-ISO/IEC9899-2011) ]; the function signature has the parameters reversed and returns<br /><br />`errno_t` instead of `struct tm *` | 2012 |
| `snwprintf_s()` | Absent; use `_snwprintf_s()` instead.  Note that the parameters differ from the ISO C signature | 2012 |
| `vfwscanf_s()` , `vswscanf_s()` ,<br /><br />`vwscanf_s()` | Absent | 2012 |
| `vsnwprintf_s()` | Absent, use `_vsnwprintf_s()` instead.  Note that the parameters differ from the ISO C signature | 2012 |
| `wcstok_s()` | Not conforming [ [ISO/IEC 9899-2011](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AA.Bibliography-ISO/IEC9899-2011) ]; the function signature is missing the `rsize_t *` parameter | 2012 |

##### `fopen()` & `freopen()`

The C standard does not specify what happens with Windows newline characters (CRLF), and so care should be taken when working with text files.  For instance:

If you save the contents of text_file.txt with Windows line endings (CRLF) and run the program on Windows, it will print 30. However, if you compile the application on a platform which does not use CRLF as its line endings, it will print 32. This is because MSVC's text translation mode will translate the CRLF characters into a single LF character on input, and translate a single LF character to CRLF on output. To ensure consistent behavior between platforms, consider opening the file in binary translation mode explicitly.

This program will print 32 with the given text, regardless of platform.

## MSC24-C. Do not use deprecated or obsolescent functions

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/21.msc24-c.md) — SHA-256 `12141655a99bece8eef20c4682af66e15e8e5e752cf8c87c15bdda4c2661bc53`。

Do not use deprecated or obsolescent functions when more secure equivalent functions are available. Deprecated functions are defined by the C Standard. Obsolescent functions are defined by this recommendation.

### Deprecated Functions

The `gets()` function was deprecated by Technical Corrigendum 3 to C99 and eliminated from C11.  The Annex K `gets_s()` function is a recommended alternative to `gets()` .

### Obsolescent Functions

Functions in the first column of the following table are hereby defined to be *obsolescent functions* . To remediate invocations of obsolescent functions, an application might use inline coding that, in all respects, conforms to this guideline, or an alternative library that, in all respects, conforms to this guideline, or alternative *non-obsolescent functions* .

| Obsolescent<br /><br />Function | Recommended<br /><br />Alternative | Rationale |
| --- | --- | --- |
| `asctime()` | `asctime_s ()` | Non-reentrant |
| `atof ()` | `strtod ()` | No error detection |
| `atoi ()` | `strtol ()` | No error detection |
| `atol ()` | `strtol ()` | No error detection |
| `atoll ()` | `strtoll ()` | No error detection |
| `ctime ()` | `ctime_s ()` | Non-reentrant |
| `fopen ()` | `fopen_s ()` | No exclusive access to file |
| `freopen ()` | `freopen_s ()` | No exclusive access to file |
| `rewind ()` | `fseek ()` | No error detection |
| `setbuf ()` | `setvbuf ()` | No error detection |

The `   atof       ()   ` , atoi `()` , atol `()` , and `   atoll       ()   ` functions are obsolescent because the `   strtod       ()   ` , strtof `()` , strtol `()` , strtold `()` , strtoll `()` , strtoul `()` , and `   strtoull       ()   ` functions can emulate their usage and have more robust error handling capabilities. See [INT05-C. Do not use input functions to convert character data if they cannot handle all possible inputs](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int05-c) .

The `   fopen       ()   `and`   freopen       ()   ` functions are obsolescent because the `   fopen_s       ()   `and`   freopen_s       ()   ` functions can emulate their usage and improve security by protecting the file from unauthorized access by setting its file protection and opening the file with exclusive access \[ [ISO/IEC WG14 N1173](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECWG14N1173) \].

The `   setbuf       ()   ` function is obsolescent because `   setbuf       ()   ` does not return a value and can be emulated using `   setvbuf       ()   ` . See [ERR07-C. Prefer functions that support error checking over equivalent functions that don't](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err07-c) .

The `   rewind       ()   ` function is obsolescent because `   rewind       ()   ` does not return a value and can be emulated using `   fseek       ()   ` . See [ERR07-C. Prefer functions that support error checking over equivalent functions that don't](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err07-c) .

The `   asctime       ()   `and`   ctime       ()   ` functions are obsolescent because they use non-reentrant static buffers and can be emulated using `   asctime_s       ()   `and`   ctime_s       ()   ` .

### Unchecked Obsolescent Functions

If you are using platforms that support Annex K, then functions in the first column of the following table are hereby defined to be *obsolescent functions, with functions in the second column being the recommended alternatives from Annex K.*

| Obsolescent<br /><br />Function | Recommended<br /><br />Alternative |
| --- | --- |
| bsearch() | bsearch_s() |
| fprintf() | fprintf_s() |
| fscanf() | fscanf_s() |
| fwprintf() | fwprintf_s() |
| fwscanf() | fwscanf_s() |
| getenv() | getenv_s() |
| gmtime() | gmtime_s() |
| localtime() | localtime_s() |
| mbsrtowcs() | mbsrtowcs_s() |
| mbstowcs() | mbstowcs_s() |
| memcpy() | memcpy_s() |
| memmove() | memmove_s() |
| printf() | printf_s() |
| qsort() | qsort_s() |
| scanf() | scanf_s() |
| snprintf() | snprintf_s() |
| sprintf() | sprintf_s() |
| sscanf() | sscanf_s() |
| strcat() | strcat_s() |
| strcpy() | strcpy_s() |
| strerror() | strerror_s() |
| strlen() | strnlen_s() |
| strncat() | strncat_s() |
| strncpy() | strncpy_s() |
| strtok() | strtok_s() |
| swprintf() | swprintf_s() |
| swscanf() | swscanf_s() |
| vfprintf() | vfprintf_s() |
| vfscanf() | vfscanf_s() |
| vfwprintf() | vfwprintf_s() |
| vfwscanf() | vfwscanf_s() |
| vprintf() | vprintf_s() |
| vscanf() | vscanf_s() |
| vsnprintf() | vsnprintf_s() |
| vsprintf() | vsprintf_s() |
| vsscanf() | vsscanf_s() |
| vswprintf() | vswprintf_s() |
| vswscanf() | vswscanf_s() |
| vwprintf() | vwprintf_s() |
| vwscanf() | vwscanf_s() |
| wcrtomb() | wcrtomb_s() |
| wcscat() | wcscat_s() |
| wcscpy() | wcscpy_s() |
| wcslen() | wcsnlen_s() |
| wcsncat() | wcsncat_s() |
| wcsncpy() | wcsncpy_s() |
| wcsrtombs() | wcsrtombs_s() |
| wcstok() | wcstok_s() |
| wcstombs() | wcstombs_s() |
| wctomb() | wctomb_s() |
| wmemcpy() | wmemcpy_s() |
| wmemmove() | wmemmove_s() |
| wprintf() | wprintf_s() |
| wscanf() | wscanf_s() |

For information on the `tmpfile()` and `tmpfile_s()` functions, see [FIO21-C. Do not create temporary files in shared directories](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio21-c) .  
For information on the `memset()` and `memset_explicit()` functions, see [MSC06-C. Beware of compiler optimizations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc06-c) .

To remediate invocations of obsolescent functions, an application might use any of the following recommended functions from ISO/IEC TR 24731-2, *Extensions to the C Library—Part II: Dynamic Allocation Functions* \[ [ISO/IEC TR 24731-2](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECTR24731-2-2010) \]:

| `asprintf` | `aswprintf` | `fmemopen` | `fscanf` | `fwscanf` | `getdelim` | `getline` |
| --- | --- | --- | --- | --- | --- | --- |
| `getwdelim` | `getwline` | `open_memstream` | `open_wmemstream` | `strdup` | `strndup` |  |

### Noncompliant Code Example

In this noncompliant code example, the obsolescent functions `strcat()` and `strcpy()` are used:

### Compliant Solution

In this compliant solution, `strcat()` and `strcpy()` are replaced by `strcat_s()` and `strcpy_s()` :

## MSC25-C. Do not use insecure or weak cryptographic algorithms

**Recommendation / 未確定**。原典がunder construction / incomplete / stub。記載済み条件を手掛かりとして残すが、完成した規範や適合認定に補作しない。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/15.miscellaneous-msc/22.msc25-c.md) — SHA-256 `1a323c5946da78185025b23101d00c3b7b813d14f10b1363156df35a2ee6fd8b`。

This rule is a stub.

### Noncompliant Code Example

This noncompliant code example shows an example where ...

### Compliant Solution

In this compliant solution, ...

## MSC30-C. Do not use the rand() function for generating pseudorandom numbers

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/15.miscellaneous-msc/2.msc30-c.md) — SHA-256 `4ef6f9e5f6b1dd4e2a4380df2a1bb0ca26c8e2976d8f9dbb1dd42fac50bdfe0a`。

Pseudorandom number generators use mathematical algorithms to produce a sequence of numbers with good statistical properties, but the numbers produced are not genuinely random.

The C Standard `rand()` function makes no guarantees as to the quality of the random sequence produced. The numbers generated by some implementations of `rand()` have a comparatively short cycle and the numbers can be predictable. Applications that have strong pseudorandom number requirements must use a generator that is known to be sufficient for their needs.

### Noncompliant Code Example

The following noncompliant code generates an ID with a numeric part produced by calling the `rand()` function. The IDs produced are predictable and have limited randomness.

### Compliant Solution (POSIX)

This compliant solution replaces the `rand()` function with the POSIX `random()` function:

The POSIX `random()` function is a better pseudorandom number generator. Although on some platforms the low dozen bits generated by `rand()` go through a cyclic pattern, all the bits generated by `random()` are usable. The `rand48` family of functions provides another alternative for pseudorandom numbers.

Although not specified by POSIX, [`arc4random()`](http://www.openbsd.org/cgi-bin/man.cgi?query=arc4random) is another possibility for systems that support it. The `arc4random(3)` manual page \[ [OpenBSD](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-OpenBSD) \] states

> ... provides higher quality of data than those described in rand(3), random(3), and drand48(3).

To achieve the best random numbers possible, an [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) -specific function must be used. When unpredictability is crucial and speed is not an issue, as in the creation of strong cryptographic keys, use a true entropy source, such as `/dev/random` , or a hardware device capable of generating random numbers. The `/dev/random` device can block for a long time if there are not enough events going on to generate sufficient entropy.

### Compliant Solution (Windows)

On Windows platforms, the [`BCryptGenRandom()`](https://docs.microsoft.com/en-us/windows/desktop/api/bcrypt/nf-bcrypt-bcryptgenrandom) function can be used to generate cryptographically strong random numbers. The Microsoft Developer Network `BCryptGenRandom()` reference \[ [MSDN](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MSDN) \] states:

> The default random number provider implements an algorithm for generating random numbers that complies with the NIST SP800-90 standard, specifically the CTR_DRBG portion of that standard.

## MSC32-C. Properly seed pseudorandom number generators

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/15.miscellaneous-msc/3.msc32-c.md) — SHA-256 `afb5959022ab31906bf9c68beb7ad4fabfc5eda99acb230d976a19ac7653bc02`。

A pseudorandom number generator (PRNG) is a deterministic algorithm capable of generating sequences of numbers that approximate the properties of random numbers. Each sequence is completely determined by the initial state of the PRNG and the algorithm for changing the state. Most PRNGs make it possible to set the initial state, also called the *seed state* . Setting the initial state is called *seeding* the PRNG.

Calling a PRNG in the same initial state, either without seeding it explicitly or by seeding it with the same value, results in generating the same sequence of random numbers in different runs of the program. Consider a PRNG function that is seeded with some initial seed value and is consecutively called to produce a sequence of random numbers, `S` . If the PRNG is subsequently seeded with the same initial seed value, then it will generate the same sequence `S` .

As a result, after the first run of an improperly seeded PRNG, an attacker can predict the sequence of random numbers that will be generated in the future runs. Improperly seeding or failing to seed the PRNG can lead to [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) , especially in security protocols.

The solution is to ensure that the PRNG is always properly seeded. A properly seeded PRNG will generate a different sequence of random numbers each time it is run.

Not all random number generators can be seeded. True random number generators that rely on hardware to produce completely unpredictable results do not need to be and cannot be seeded. Some high-quality PRNGs, such as the `/dev/random` device on some UNIX systems, also cannot be seeded. This rule applies only to algorithmic pseudorandom number generators that can be seeded.

### Noncompliant Code Example (POSIX)

This noncompliant code example generates a sequence of 10 pseudorandom numbers using the `random()` function. When `random()` is not seeded, it behaves like `rand()` , producing the same sequence of random numbers each time any program that uses it is run.

The output is as follows:

### Compliant Solution (POSIX)

Call `srandom()` before invoking `random()` to seed the random sequence generated by `random()` . This compliant solution produces different random number sequences each time the function is called, depending on the resolution of the system clock:

The output is as follows:

This may not be sufficiently random for concurrent execution, which may lead to correlated generated series in different threads. Depending on the application and the desired level of security, a programmer may choose alternative ways to seed PRNGs. In general, hardware is more capable than software of generating real random numbers (for example, by sampling the thermal noise of a diode).

### Compliant Solution (Windows)

The [`BCryptGenRandom()`](https://docs.microsoft.com/en-us/windows/desktop/api/bcrypt/nf-bcrypt-bcryptgenrandom) function does not run the risk of not being properly seeded because its arguments serve as seeders:

The output is as follows:

## MSC33-C. Do not pass invalid data to the asctime() function

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/15.miscellaneous-msc/4.msc33-c.md) — SHA-256 `950da1dd659d8962b4e48bcf9143cecb5e4fc5273c3076ba3e479a27fa307227`。

The C Standard, 7.29.3.1 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], provides the following sample implementation of the `asctime()` function:

This function is supposed to output a character string of 26 characters at most, including the terminating null character. If we count the length indicated by the format directives, we arrive at 25. Taking into account the terminating null character, the array size of the string appears sufficient.

However, this [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) assumes that the values of the `   struct tm  ` data are within normal ranges and does nothing to enforce the range limit. If any of the values print more characters than expected, the `snprintf()` function truncates the string. For example, if `tm_year` has the value `12345,` then 27 characters (including the terminating null character) are needed, so only the first 25 characters are stored. The string is still null-terminated, but the trailing new-line character is lost. An out-of-range `tm_wday` or `tm_mon` value also makes the sample read outside the `wday_name` or `mon_name` array. The C Standard, 7.29.3.1, paragraph 4, says that if any member of the broken-down time is outside its normal range, or if the calculated year exceeds four digits or is less than 1000, the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior), so an implementation's `asctime()` is not required to behave like the sample for such values.

The *POSIX <sup>®</sup> Base Specifications* \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \] says the following about the `asctime()` and `asctime_r()` functions:

> These functions are included only for compatibility with older implementations. They have undefined behavior if the resulting string would be too long, so the use of these functions should be discouraged. On implementations that do not detect output string length overflow, it is possible to overflow the output buffers in such a way as to cause applications to fail, or possible system security violations. Also, these functions do not support localized date and time formats. To avoid these problems, applications should use `strftime()` to generate strings from broken-down times.

The `asctime()` function appears in the list of obsolescent functions in [MSC24-C. Do not use deprecated or obsolescent functions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc24-c) .

### Noncompliant Code Example

This noncompliant code example invokes the `asctime()` function with potentially unsanitized data:

### Compliant Solution ( `strftime()` )

The `strftime()` function allows the programmer to specify a more rigorous format and also to specify the maximum size of the resulting time string:

This call has the same effects as `asctime()` but also ensures that no more than `maxsize` characters are printed, preventing buffer overflow. If the result, including the terminating null character, does not fit in `maxsize` characters, `strftime()` returns 0 and the contents of `s` are indeterminate, so the return value must be checked.

## MSC37-C. Ensure that control never reaches the end of a non-void function

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/15.miscellaneous-msc/5.msc37-c.md) — SHA-256 `c754d93fe6ba3f63e56c4fa7b2cc74c4162994ab1799bafbcb1d9d7d9e4fb3e1`。

If control reaches the closing curly brace ( `}` ) of a non- `void` function without evaluating a `return` statement, using the return value of the function call is [undefined behavior.](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) (See [undefined behavior 84](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_84).)

### Noncompliant Code Example

In this noncompliant code example, control reaches the end of the `checkpass()` function when the two strings passed to `strcmp()` are not equal, resulting in undefined behavior. Many compilers will generate code for the `checkpass()` function, returning various values along the execution path where no `return` statement is defined.

This error is frequently diagnosed by compilers. (See [MSC00-C. Compile cleanly at high warning levels](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc00-c) .)

### Compliant Solution

This compliant solution ensures that the `checkpass()` function always returns a value:

### Noncompliant Code Example

In this noncompliant code example, control reaches the end of the `getlen()` function when `input` does not contain the integer `delim` . Because the potentially undefined return value of `getlen()` is later used as an index into an array, a buffer overflow may occur.

#### Implementation Details (GCC)

Violating this rule can have unexpected consequences, as in the following example:

When this program is compiled with `-Wall` on most versions of the GCC compiler, the following warning is generated:

None of the inputs to the function equal the delimiter, so when run with GCC 5.3 on Linux, control reaches the end of the `getlen()` function, which is undefined behavior and in this test returns `3` , causing an out-of-bounds write to the `data` array.

### Compliant Solution

This compliant solution changes the interface of `getlen()` to store the result in a user-provided pointer and returns a status indicator to report success or failure. The best method for handling this type of error is specific to the application and the type of error. (See [ERR00-C. Adopt and implement a consistent and comprehensive error-handling policy](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err00-c) for more on error handling.)

### Exceptions

**MSC37-C-EX1:** According to the C Standard, 5.1.2.3.4, paragraph 1 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], "Reaching the `}` that terminates the main function returns a value of 0." As a result , it is permissible for control to reach the end of the `main()` function without executing a return statement.

**MSC37-C-EX2:** It is permissible for a control path to not return a value if that code path is never taken and a function marked `_Noreturn` is called as part of that code path. For example:

## MSC38-C. Do not treat a predefined identifier as an object if it might only be implemented as a macro

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/15.miscellaneous-msc/6.msc38-c.md) — SHA-256 `252b50555ac9c69ce39b68e00c604da3c30212b932ce564b055062d3e593afeb`。

The C Standard, 7.1.4 paragraph 1, \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2024) \] states

> Any function declared in a header may be additionally implemented as a function-like macro defined in the header, so if a library function is declared explicitly when its header is included, one of the techniques shown later in the next subclause can be used to ensure the declaration is not affected by such a macro. Any macro definition of a function can be suppressed locally by enclosing the name of the function in parentheses, because the name is then not followed by the left parenthesis that indicates expansion of a macro function name. For the same syntactic reason, it is permitted to take the address of a library function even if it is also defined as a macro. 220) The use of #undef to remove any macro definition will also ensure that an actual function is referred to.
>
> 220\) This means that an implementation is required to provide an actual function for each library function, even if it also provides a macro for that function.

However, the C Standard enumerates specific exceptions in which the behavior of accessing an object or function expanded to be a standard library macro definition is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . The macros are `assert` , `errno` , `math_errhandling` , `setjmp` , `va_arg` , `va_copy` , `va_end` , and `va_start` , as well as all of the generic functions defined in `stdatomic.h` and `stdbit.h` . These cases are described by [undefined behaviors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) [138](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_138) , [139](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_139) , [140](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_140) , [141](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_141) , [143](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_143) , and [145](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_145) . Programmers must not suppress these macros to access the underlying object or function.

### Noncompliant Code Example ( `assert` )

In this noncompliant code example, the standard `assert()` macro is suppressed in an attempt to pass it as a function pointer to the `execute_handler()` function. Attempting to suppress the `assert()` macro is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

### Compliant Solution ( `assert` )

In this compliant solution, the `assert()` macro is wrapped in a helper function, removing the [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) :

### Noncompliant Code Example (Redefining `errno` )

Legacy code is apt to include an incorrect declaration, such as the following in this noncompliant code example:

### Compliant Solution (Declaring `errno` )

This compliant solution demonstrates the correct way to declare `errno` by including the header `<errno.h>` :

[C-conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conformingprogram) [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) are required to declare `errno` in `<errno.h>` , although some historic implementations failed to do so.

## MSC39-C. Do not call va_arg() on a va_list that has an indeterminate value

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/15.miscellaneous-msc/7.msc39-c.md) — SHA-256 `464b84b8a80b9a49f47cbf544027dbb61ba24e212bc3e9b9a5b60328f81e5826`。

Variadic functions access their variable arguments by using `va_start()` to initialize an object of type `va_list` , iteratively invoking the `va_arg()` macro, and finally calling `va_end()` . The `va_list` may be passed as an argument to another function, but calling `va_arg()` within that function causes the `va_list` to have an [indeterminate value](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) in the calling function. As a result, attempting to read variable arguments without reinitializing the `va_list` can have [unexpected behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unexpectedbehavior) . According to the C Standard, 7.16, paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \],

> If access to the varying arguments is desired, the called function shall declare an object (generally referred to as ap in this subclause) having type va_list. The object ap may be passed as an argument to another function; if that function invokes the va_arg macro with parameter ap, the representation of ap in the calling function is indeterminate and shall be passed to the va_end macro prior to any further reference to ap.
>
> 295\) A pointer to a va \_ list can be created and passed to another function, in which case the original function can make further use of the original list after the other function returns.

### Noncompliant Code Example

This noncompliant code example attempts to check that none of its variable arguments are zero by passing a `va_list` to helper function `contains_zero()` . After the call to `contains_zero()` , the value of `ap` is [indeterminate](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) .

### Compliant Solution

The compliant solution modifies `contains_zero()` to take a pointer to a `va_list` . It then uses the `va_copy` macro to make a copy of the list, traverses the copy, and cleans it up. Consequently, the `print_reciprocals()` function is free to traverse the original `va_list` .

## MSC40-C. Do not violate constraints

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/15.miscellaneous-msc/8.msc40-c.md) — SHA-256 `6ccc45162fe154ce5bda43811b8a35efe36150aed67a4e71eab274f84cf7304b`。

According to the C Standard, 3.8 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2011) \], a constraint is a "restriction, either syntactic or semantic, by which the exposition of language elements is to be interpreted."  Despite the similarity of the terms, a runtime constraint is not a kind of constraint.

Violating any *shall* statement within a constraint clause in the C Standard requires an [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) to issue a diagnostic message, the C Standard, 5.1.1.3 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2011) \] states

> A conforming implementation shall produce at least one diagnostic message (identified in an implementation-defined manner) if a preprocessing translation unit or translation unit contains a violation of any syntax rule or constraint, even if the behavior is also explicitly specified as undefined or implementation-defined. Diagnostic messages need not be produced in other circumstances.

The C Standard further explains in a footnote

> The intent is that an implementation should identify the nature of, and where possible localize, each violation. Of course, an implementation is free to produce any number of diagnostics as long as a valid program is still correctly translated. It may also successfully translate an invalid program.

Any constraint violation is a violation of this rule because it can result in an invalid program.

### Noncompliant Code Example (Inline, Internal Linkage)

The C Standard, 6.7.4, paragraph 3 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2011) \], states

> An inline definition of a function with external linkage shall not contain a definition of a modifiable object with static or thread storage duration, and shall not contain a reference to an identifier with internal linkage.

The motivation behind this constraint lies in the semantics of inline definitions. Paragraph 7 of subclause 6.7.4 reads, in part:

> An inline definition provides an alternative to an external definition, which a translator may use to implement any call to the function in the same translation unit. It is unspecified whether a call to the function uses the inline definition or the external definition.

That is, if a function has an external and inline definition, implementations are free to choose which definition to invoke (two distinct invocations of the function may call different definitions, one the external definition, the other the inline definition). Therefore, issues can arise when these definitions reference internally linked objects or mutable objects with static or thread storage duration.

This noncompliant code example refers to a static variable with file scope and internal linkage from within an external inline function:

### Compliant Solution (Inline, Internal Linkage)

This compliant solution omits the `static` qualifier;  consequently,  the variable `I` has external linkage by default:

### Noncompliant Code Example (inline, Modifiable Static)

This noncompliant code example defines a modifiable `static` variable within an `   extern inline  ` function.

### Compliant Solution (Inline, Modifiable Static)

This compliant solution removes the `static` keyword from the local variable definition. If the modifications to `I` must be retained between invocations of `func()` , it must be declared at file scope so that it will be defined with external linkage.

### Noncompliant Code Example (Inline, Modifiable `static` )

This noncompliant code example includes two translation units: `file1.c` and `file2.c` . The first file, `file1.c` , defines a pseudorandom number generation function:

The left-shift operation in the last line may wrap, but this is permitted by exception INT30-C-EX3 to rule [INT30-C. Ensure that unsigned integer operations do not wrap](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int30-c) .

The second file, `file2.c` , defines an `inline` version of this function that references mutable `static` objects—namely, objects that maintain the state of the pseudorandom number generator. Separate invocations of the `get_random()` function can call different definitions, each operating on separate static objects, resulting in a faulty pseudorandom number generator.

### Compliant Solution (Inline, Modifiable `static` )

This compliant solution adds the `static` modifier to the `inline` function definition in `file2.c` , giving it internal linkage. All references to `get_random()` in `file.2.c` will now reference the internally linked definition. The first file, which was not changed, is not shown here.

## MSC41-C. Never hard code sensitive information

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/15.miscellaneous-msc/9.msc41-c.md) — SHA-256 `95211827afe839ce9e7fcd12e1b02fce2472748d8979a44714c8fddb191ca0e1`。

Hard coding sensitive information, such as passwords or encryption keys can expose the information to attackers. Anyone who has access to the executable or dynamic library files can examine them for strings or other critical data, revealing the sensitive information. Leaking data protected by International Traffic in Arms Regulations (ITAR) or the Health Insurance Portability and Accountability Act ( HIPAA) can also have legal consequences. Consequently, programs must not hard code sensitive information.

Hard coding sensitive information also increases the need to manage and accommodate changes to the code. For example, changing a hard-coded password in a deployed program may require distribution of a patch \[ [Chess 2007](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Chess07) \].

### Noncompliant Code Example (Hard-Coded Database Password)

This noncompliant code example must authenticate to a remote service with a code, using the `authenticate()` function declared below. It passes the authentication code to this function as a string literal.

The authentication code exists in the program's binary executable and can be easily discovered.

#### Implementation Details (Unix)

Many Unix platforms provide a `strings` utility that prints out all of the ASCII strings in a binary file. Here is the output of running `strings` on this program, on an Ubuntu 16.04 platform:

### Compliant Solution (C23, `memset_explicit()` )

This compliant solution requires the user to supply the authentication code, and securely erases it when done, using `memset_explicit()` .

Alternatively, the program could read the authentication code from a file, letting file system security protect the file and the code from untrusted users.
