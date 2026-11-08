# CERT C — API

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## API00-C. Functions should validate their parameters

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/02.application-programming-interfaces-api/02.api00-c.md) — SHA-256 `1078576d30a820a06759001da3ff8d2a2a1d6454aadda9db7b376d75f0f3cfa2`。

Redundant testing by caller and by callee as a style of *defensive programming* is largely discredited in the C and C++ communities, the main problem being performance. The usual discipline in C and C++ is to require [validation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-validation) on only one side of each interface.

Requiring the caller to validate arguments can result in faster code because the caller may understand certain invariants that prevent invalid values from being passed. Requiring the callee to validate arguments allows the validation code to be encapsulated in one location, reducing the size of the code and making it more likely that these checks are performed in a consistent and correct fashion.

For safety and security reasons, this standard recommends that the called function validate its parameters. Validity checks allow the function to survive at least some forms of improper usage, enabling an application using the function to likewise survive. Validity checks can also simplify the task of determining the condition that caused the invalid parameter.

### Noncompliant Code Example

In this noncompliant code example, `setfile()` and `usefile()` do not validate their parameters. It is possible that an invalid file pointer can be used by the library, corrupting the library's internal state and exposing a [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) .

The vulnerability can be more severe if the internal state references sensitive or system-critical data.

### Compliant Solution

Validating the function parameters and verifying the internal state leads to consistency of program execution and may eliminate potential vulnerabilities. In addition, implementing commit or rollback semantics (leaving program state unchanged on error) is a desirable practice for error safety.

## API01-C. Avoid laying out strings in memory directly before sensitive data

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/02.application-programming-interfaces-api/03.api01-c.md) — SHA-256 `1362e0e84b9998a4cddfd2f936bbc888dec5fbe2dee6130fc448950c12a65527`。

Strings (both character and wide-character) are often subject to buffer overflows, which will overwrite the memory immediately past the string. Many rules warn against buffer overflows, including [STR31-C. Guarantee that storage for strings has sufficient space for character data and the null terminator](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/characters-and-strings-str/str31-c) . Sometimes the danger of buffer overflows can be minimized by ensuring that arranging memory such that data that might be corrupted by a buffer overflow is not sensitive.

### Noncompliant Code Example

This noncompliant code example stores a set of strings using a linked list:

A buffer overflow on `name` would overwrite the `next` pointer, which could then be used to read or write to arbitrary memory.

### Compliant Solution

This compliant solution creates a linked list of strings but stores the `next` pointer before the string:

If buffer overflow occurs on `name` , the `next` pointer remains uncorrupted.

### Compliant Solution

In this compliant solution, the linked list stores pointers to strings that are stored elsewhere. Storing the strings elsewhere protects the `next` pointer from buffer overflows on the strings.

### Exceptions

**API01-C-EX1:** Using a string before sensitive data such as pointers is permitted when it is not practical to segregate the strings from the sensitive data.

Each of the following code examples creates a linked list of strings, but each node is actually stored inside an array. This practice ensures that the string is always in front of a `next` pointer regardless of how they are ordered in the struct.

## API02-C. Functions that read or write to or from an array should take an argument to specify the source or target size

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/02.application-programming-interfaces-api/04.api02-c.md) — SHA-256 `26261c9fc3a87d9b55e8fb166cb62aca4c9286c4c3190d38c2126296cd57f288`。

Functions that have an array as a parameter should also have an additional parameter that indicates the maximum number of elements that can be stored in the array. That parameter is required to ensure that the function does not access memory outside the bounds of the array and adversely influence program execution. It should be present for each array parameter (in other words, the existence of each array parameter implies the existence of a complementary parameter that represents the maximum number of elements in the array).

Note that *array* is used in this recommendation to mean array, string, or any other pointer to a contiguous block of memory in which one or more elements of a particular type are (potentially) stored. These terms are all effectively synonymous and represent the same potential for error.

Also note that this recommendation suggests the parameter accompanying array parameters indicates the maximum number of elements that can be stored in the array, not the maximum size, in bytes, of the array, because

1.  It does not make sense to think of array sizes in bytes in all cases—for example, in the case of an array of integers.
2.  If the size in bytes of the array is required, it can be derived from the number of elements in the array.
3.  It is better not to add to the cognitive load of the function user by requiring the user to calculate the size in bytes of the array.

In most cases, the distinction between the number of elements and number of bytes is moot: there is a clear mapping between the two, and it is easier to think in terms of number of elements anyway. Unfortunately, this issue can become muddled when working with multibyte strings because the logical entity being manipulated differs from that of the type being used to implement it. Here, it is important to remember that the type of the array is a character, not a multibyte character. Accordingly, the number of elements in the array is represented as a number of characters.

### Noncompliant Code Example

It is not necessary to go beyond the standard C library to find examples that violate this recommendation because the C language often prioritizes performance at the expense of [robustness](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-robustness) . The following are two examples from the C Standard, subclause 7.24 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \]:

These functions have two problems. First, there is no indication of the size of the first array, `s1` . As a result, it is not possible to discern within the function how large `s1` is and how many elements may be written into it. Second, it appears that a size is supplied for `s2` , but the `size_t` parameter `n` actually gives the number of elements to copy. Consequently, there is no way for either function to determine the size of the array `s2` .

### Compliant Solution

The C `strncpy()` and `strncat()` functions could be improved by adding element count parameters as follows:

The `n` parameter is used to specify a number of elements to copy that is less than the total number of elements in the source string.

### Exceptions

**API02-C-EX1:** Functions that can guarantee via their runtime-constraint handlers that no out-of-bounds read or write occurs may omit the maximum-elements argument. For instance, the `s2` parameter to `strcat_s()` needs no `max` parameter.

As another example, consider `strcpy_s()` :

This function provides no explicit maximum argument to `s2` . However, it requires that `s1max` be larger than `s2` , thereby preventing an out-of-bounds read.

## API03-C. Create consistent interfaces and capabilities across related functions

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/02.application-programming-interfaces-api/05.api03-c.md) — SHA-256 `9d4c1f04f066624cfb21d86332dc6478e77284a2034c463fed17a558b02e0ad7`。

Related functions, such as those that make up a library, should provide consistent and usable interfaces. Ralph Waldo Emerson said, "A foolish consistency is the hobgoblin of little minds," but inconsistencies in functional interfaces or behavior can lead to erroneous use, so we understand this to be a "wise consistency." One aspect of providing a consistent interface is to provide a consistent and usable error-checking mechanism. For more information, see [API04-C. Provide a consistent and usable error-checking mechanism](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/application-programming-interfaces-api/api04-c) .

### Noncompliant Code Example (Interface)

It is not necessary to go beyond the standard C library to find examples of inconsistent interfaces: the standard library is a fusion of multiple libraries with various styles and levels of rigor. For example, the `fputs()` defined in the C Standard, subclause 7.21.7.4, is closely related to the `fprintf()` defined in subclause 7.21.6.1. However, `fputs()` 's file handle is at the end, and `fprintf()` 's is at the beginning, as shown by their function declarations:

The argument order can be easily rearranged using macros; for example:

However, according to subclause 7.1.3 of the C Standard, the behavior of a program that defines a symbol, including a macro, with the same name as that of a standard library function, type, macro, or other reserved identifier is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

Using inconsistent interfaces makes the code difficult to read, for example, by causing confusion when moving between code that follows this convention and code that does not. In effect, it becomes impossible to modify an interface once that interface has been broadly adopted. Consequently, it is important to get the interface design right the first time.

### Compliant Solution (Interface)

The POSIX threads library \[ [Butenhof 1997](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Butenhof97) \] defines an interface that is both consistent and fits in with established conventions from the rest of the POSIX library. For example, all initialization functions follow the same consistent pattern of the first argument being a pointer to the object to initialize with the subsequent arguments, if any, optionally providing additional attributes for the initialization:

Function arguments that refer to objects that are not modified are declared `const` . Because the object pointed to by the first argument is modified by the function, it is not `const` . For functions that implement a data abstraction, it is reasonable to define the handle for the data abstraction as the initial parameter. (See [DCL12-C. Implement abstract data types using opaque types](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl12-c) .) Finally, initialization functions that accept a pointer to an attribute object allow it to be `NULL` as an indication that a reasonable default should be used.

### Noncompliant Code Example (Behavior)

The shared folder and file copy functions in the VMware virtual infrastructure (VIX) API are inconsistent in the set of characters they allow in folder names. As a result, you can create a shared folder that you subsequently cannot use in a file copy function such as `VixVM_CopyFileFromHostToGuest()` .

### Compliant Solution (Behavior)

Try to be consistent in the behavior of related functions that perform operations on common objects so that an object created or modified by one function can be successfully processed by a downstream invocation of a related function.

## API04-C. Provide a consistent and usable error-checking mechanism

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/02.application-programming-interfaces-api/06.api04-c.md) — SHA-256 `67a2251fa8b57c9003e5df769bb64551365c016cc5c7d4a04b06b08462410add`。

Functions should provide consistent and usable error-checking mechanisms. Complex interfaces are sometimes ignored by programmers, resulting in code that is not error checked. Inconsistent interfaces are frequently misused and difficult to use, resulting in lower-quality code and higher development costs.

### Noncompliant Code Example ( `strlcpy()` )

The `strlcpy()` function copies a null-terminated source string to a destination array. It is designed to be a safer, more consistent, and less error-prone replacement for `strcpy()` .

The `strlcpy()` function returns the total length of the string it tried to create (the length of the source string).

To detect truncation, perhaps while building a path name, code such as the following might be used:

### Compliant Solution ( `strcpy_m()` )

The managed string library \[ [Burch 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Burch06) \] handles errors by consistently returning the status code in the function return value. This approach encourages status checking because the user can insert the function call as the expression in an `if` statement and take appropriate action on failure.

The `strcpy_m()` function is an example of a managed string function that copies a source-managed string into a destination-managed string:

The greatest disadvantage of this approach is that it prevents functions from returning any other value. Consequently, all values (other than the status) returned by a function must be returned as a pass-by-reference parameter, preventing a programmer from nesting function calls. This trade-off is necessary because nesting function calls can conflict with a programmer's willingness to check status codes.

## API05-C. Use conformant array parameters

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/02.application-programming-interfaces-api/07.api05-c.md) — SHA-256 `8a972d8a6aaad257ada9bb4b75eade361681d82516c5558efb8f61db2766e6c1`。

Traditionally, C arrays are declared with an index that is either a fixed constant or empty. An array with a fixed constant index indicates to the compiler how much space to reserve for the array. An array declaration with an empty index is an incomplete type and indicates that the variable references a pointer to an array of indeterminate size.

The term *conformant array parameter* comes from Pascal; it refers to a function argument that is an array whose size is specified in the function declaration. Since C99, C has supported conformant array parameters by permitting array parameter declarations to use extended syntax. Subclause 6.7.6.2, paragraph 1, of C11 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] summarizes the array index syntax extensions:

> The \[ and \] may delimit an expression or \*. If they delimit an expression (which specifies the size of an array), the  expression shall have an integer type. If the expression is a constant expression, it shall  have a value greater than zero.

Consequently, an array declaration that serves as a function argument may have an index that is a variable or an expression. The array argument is demoted to a pointer and is consequently not a variable length array (VLA). Conformant array parameters can be used by developers to indicate the expected bounds of the array. This information may be used by compilers, or it may be ignored. However, such declarations are useful to developers because they serve to document relationships between array sizes and pointers. This information can also be used by [static analysis](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-staticanalysis) tools to diagnose potential defects.

### Standard Examples

Subclause 6.7.6.3 of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] has several examples of conformant array parameters. Example 4 (paragraph 20) illustrates a variably modified parameter:

Example 4 illustrates a set of compatible function prototype declarators

### Noncompliant Code Example

This code example provides a function that wraps a call to the standard `memset()` function and has a similar set of arguments. However, although this function clearly intends that `p` point to an array of at least `n` chars, this invariant is not explicitly documented.

### Noncompliant Code Example

This noncompliant code example attempts to document the relationship between the pointer and the size using conformant array parameters. However, the variable `n` is used as the index of the array declaration before `n` is itself declared. Consequently, this code example is not standards-compliant and will usually fail to compile.

### Compliant Solution (GCC)

This compliant solution uses a GNU extension to forward declare the `size_t` variable `n` before using it in the subsequent array declaration. Consequently, this code allows the existing parameter order to be retained and successfully documents the relationship between the array parameter and the size parameter.

### Compliant Solution (API change)

This compliant solution changes the function's API by moving the `size_t` variable `n` to before the subsequent array declaration. Consequently, this code complies with the C99 standard and successfully documents the relationship between the array parameter and the size parameter, but requires all callers to be updated.

### Exceptions

**API05-C-EX0:** The extended array syntax is not supported by C++, or platforms that do not support C99, such as MSVC. Consequently, C programs that must support such platforms, including Windows, need not use conformant array parameters. One option for portable code that must support such platforms is to use macros:

## API07-C. Enforce type safety

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/02.application-programming-interfaces-api/08.api07-c.md) — SHA-256 `eb16b026b5983af1cf8b783155b631288f96a5042bb5905a85589f2c1a2913ac`。

  
Upon return, functions should guarantee that any object returned by the function, or any modified value referenced by a pointer argument, is a valid object of function return type or argument type. Otherwise, type errors can occur in the program.

A good example is the null-terminated byte string type in C. If a string lacks the terminating null character, the program may be tricked into accessing storage after the string as legitimate data. A program may, as a result, process a string it should not process, which might be a [security flaw](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-securityflaw) in itself. It may also cause the program to abort, which might be a [denial-of-service attack](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-denial-of-serviceattack) .

The emphasis of this recommendation is to avoid producing unterminated strings; it does not address processing of already existing unterminated strings. However, by preventing the creation of unterminated strings, the need to process them is greatly lessened.

### Noncompliant Code Example

The standard `strncpy()` function does not guarantee that the resulting string is null-terminated. If there is no null character in the first `n` characters of the `source` array, the result may not be null-terminated.

### Compliant Solution ( `strncpy_s()` , C11 Annex K)

The C11 Annex K `strncpy_s()` function copies up to `n` characters from the source array to a destination array. If no null character was copied from the source array, the `n` th position in the destination array is set to a null character, guaranteeing that the resulting string is null-terminated.

## API09-C. Compatible values should have the same type

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/02.application-programming-interfaces-api/09.api09-c.md) — SHA-256 `5362e139c8cbe9c89dd3b9bd4f8bc20a299ad0753b322aa61c345d372a85de75`。

Make sure compatible values have the same type. For example, when the return value of one function is used as an argument to another function, make sure they are the same type. Ensuring compatible values have the same type allows the return value to be passed as an argument to the related function without conversion, reducing the potential for conversion errors.

### Noncompliant Code Example

A source of potential errors may be traced to POSIX's tendency to overload return codes, using −1 to indicate an error condition but 0 for success and positive values as a result indicator (see [ERR02-C. Avoid in-band error indicators](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err02-c) ). A good example is the `read()` system call. This leads to a natural mixing of unsigned and signed quantities, potentially leading to conversion errors.

OpenSSH performs most I/O calls through a "retry on interrupt" function, `atomicio()` . The following is a slightly simplified version of `atomicio.c` , v 1.12 2003/07/31. The function `f()` is either `read()` or `vwrite()` :

This function has a large number of flaws. Pertinent to this recommendation, however, are the following:

-   The `atomicio()` function returns an `ssize_t` (which must be a signed type). The `ssize_t` type is a clear indication of poor interface design because a size should never be negative.
-   Both `res` and `pos` are declared as `ssize_t` .
-   The expression `    n - pos   ` results in the conversion of `pos` from a signed to an unsigned type because of the usual arithmetic conversions (see [INT02-C. Understand integer conversion rules](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int02-c) ).

### Compliant Solution

The `atomicio()` function from `atomicio.c` , v 1.25 2007/06/25, was modified to always return an unsigned quantity and to instead report its error via `errno` :

Changes to this version of the `atomicio()` function include the following:

-   The `atomicio()` function now returns a value of type `size_t` .
-   `pos` is now declared as `size_t` .
-   The assignment `    pos += (size_t)res   ` now requires an explicit cast to convert from the signed return value of `f()` to `size_t` .
-   The expression `    n - pos   ` no longer requires an implicit conversion.

Reducing the need to use signed types makes it easier to enable the compiler's signed/unsigned comparison warnings and fix all of the issues it reports (see [MSC00-C. Compile cleanly at high warning levels](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc00-c) ).

## API10-C. APIs should have security options enabled by default

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/02.application-programming-interfaces-api/10.api10-c.md) — SHA-256 `867a9a9c49ea30821a9d6aaec5bbb55883a2a3863b941fa20df73c2d13c525d2`。

APIS should have security options enabled by default– for example, having best practice cipher suites enabled by default (something that changes over time) while disabling out-of-favor cipher suites by default. When interface stability is also a design requirement, an interface can meet both goals by providing off-by-default options that produce stable behavior, such as `TLS_ENABLE_Y2015_BEST_PRACTICE_CIPHERS_ONLY` .

### Noncompliant Code Example

If the caller of this API in this noncompliant example doesn't understand what the options mean, they will pass 0 or `TLS_DEFAULT_OPTIONS` and get a connection vulnerable to man-in-the-middle attacks and using old versions of TLS.

### Compliant Solution

If the caller of this API doesn't understand the options and passes 0 or `TLS_DEFAULT_OPTIONS` they will get certificate validation with only the current version of TLS enabled.
