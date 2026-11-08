# CERT C — STR

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## STR00-C. Represent characters using an appropriate type

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/04.characters-and-strings-str/02.str00-c.md) — SHA-256 `20bfaeeef4285eacfa24d259ed574ed3f78cdab8a71ebe4b9f1bc68f0bc579ca`。

Strings are a fundamental concept in software engineering, but they are not a built-in type in C. Null-terminated byte strings (NTBS) consist of a contiguous sequence of characters terminated by and including the first null character and are supported in C as the format used for string literals. The C programming language supports single-byte character strings, multibyte character strings, and wide-character strings. Single-byte and multibyte character strings are both described as null-terminated byte strings, which are also called *narrow* character strings.

A pointer to a null-terminated byte string points to its initial character. The length of the string is the number of bytes preceding the null character, and the value of the string is the sequence of the values of the contained characters, in order.

A wide string is a contiguous sequence of wide characters (of type `wchar_t` ) terminated by and including the first null wide character. A pointer to a wide string points to its initial (lowest addressed) wide character. The length of a wide string is the number of wide characters preceding the null wide character, and the value of a wide string is the sequence of code values of the contained wide characters, in order.

Null-terminated byte strings are implemented as arrays of characters and are susceptible to the same problems as arrays. As a result, rules and recommendations for [arrays](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/) should also be applied to null-terminated byte strings.

The C Standard uses the following philosophy for choosing character types, though it is not explicitly stated in one place:

##### `   signed char  `and`   unsigned char  `

-   Suitable for small integer values

##### "Plain" `char`

-   The type of each element of a string literal
-   Used for character data from a limited character set (where signedness has little meaning) as opposed to integer data

##### `int`

-   Used for data that can be either `EOF` (a negative value) or character data interpreted as `    unsigned char   ` and then converted to `int` . As a result, it is returned by `fgetc()` , `getc()` , `getchar()` , and `ungetc()` . Also, accepted by the character-handling functions from `<ctype.h>` because they might be passed the result of `fgetc()` , and so on
-   The type of a character constant; its value is that of a plain `char` converted to `int`

Note that the two different ways a character is used as an `int` (as an `   unsigned char  `+`   EOF  ` or as a plain `char` converted to `int` ) can lead to confusion. For example, `isspace('\200')` results in undefined behavior when `char` is signed.

##### `   unsigned char  `

-   Used internally for string comparison functions even though these functions operate on character data; consequently, the result of a string comparison does not depend on whether plain `char` is signed
-   Used when the object being manipulated might be of any type, and it is necessary to access all bits of that object, as with `fwrite()`

Unlike other integer types, `   unsigned char  ` has the unique property that

> values stored in . . . objects of type `    unsigned char   ` shall be represented using a pure binary notation (C Standard, subclause 6.2.6.1 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \])

where *a pure binary notation* is defined as the following:

> A positional representation for integers that uses the binary digits 0 and 1, in which the values represented by successive bits are additive, begin with 1, and are multiplied by successive integral powers of 2, except perhaps the bit with the highest position. A byte contains `CHAR_BIT` bits, and the values of type `    unsigned char   ` range from 0 to 2 <sup>`CHAR_BIT`</sup> − 1. (subclause 6.2.6, footnote 49)

That is, objects of type `   unsigned char  ` may have no *padding bits* and consequently no *non-value representation* . As a result, non-bit-field objects of any type may be copied into an array of `   unsigned char  ` (for example, via `memcpy()` ) and have their representation examined one byte at a time.

##### `wchar_t`

-   Wide characters are used for natural-language character data

## STR01-C. Adopt and implement a consistent plan for managing strings

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/04.characters-and-strings-str/03.str01-c.md) — SHA-256 `2fa4055851ef326876c56a09ebdfec16842e3cbf80f84604c2a2f271cf60eae0`。

There are two basic approaches for managing strings in C programs: the first is to maintain strings in statically allocated arrays; the second is to dynamically allocate memory as required. Each approach has advantages and disadvantages. However, it generally makes sense to select a single approach to managing strings and apply it consistently across a project. Otherwise, the decision is left to individual programmers who are likely to make different, inconsistent choices.

Statically allocated strings assume a fixed-size character array, meaning that it is impossible to add data after the buffer is filled. Because the static approach discards excess data, actual program data can be lost. Consequently, the resulting string must be fully validated.

Dynamically allocated buffers dynamically resize as additional memory is required. Dynamic approaches scale better and do not discard excess data. The major disadvantage is that, if inputs are not limited, they can exhaust memory on a machine and consequently be used in [denial-of-service](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-denial-of-serviceattack) attacks.

Dynamic allocation is often disallowed in safety-critical systems. For example, the MISRA standard requires that "dynamic heap memory allocation shall not be used" \[ [MISRA 2004](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MISRA04) \]. Some safety-critical systems can take advantage of dynamic memory allocation during initialization but not during operations. For example, avionics software may dynamically allocate memory while initializing the aircraft but not during flight.

A number of existing libraries are available for managing string data; the library selected depends on the approach adopted for managing null-terminated byte strings. The functions defined by the C Standard, subclause 7.24, are primarily intended for managing statically allocated strings. However, these functions are problematic because many of them are insufficiently bounded.

ISO/IEC TR 24731 Part II offers one approach to resolve this, supplying functions that allocate enough memory for their results \[ [ISO/IEC TR 24731-2](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECTR24731-2-2010) \]. It provides an API that dynamically allocates the results of string functions as needed. Almost all of the APIs in this technical report are also in a current international standard. For example, TR 24731 Part II includes POSIX functions, such as `strdup()` \[ [ISO/IEC 9945:2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9945-2003) \], as well as functions from the Linux Standard Base Core Specification such as `asprintf()` \[ [ISO/IEC 23360-1:2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC23360-1-2006) \].

## STR02-C. Sanitize data passed to complex subsystems

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/04.characters-and-strings-str/04.str02-c.md) — SHA-256 `b9e4c7fe6575a2ed8363a0fa26c42d44a6e4730e55e0da14333ad66353fc6540`。

String data passed to complex subsystems may contain special characters that can trigger commands or actions, resulting in a software [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) . As a result, it is necessary to [sanitize](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sanitize) all string data passed to complex subsystems so that the resulting string is innocuous in the context in which it will be interpreted.

These are some examples of complex subsystems:

-   Command processor via a call to `system()` or similar function (also addressed in [ENV03-C. Sanitize the environment when invoking external programs](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/environment-env/env03-c) )
-   External programs
-   Relational databases
-   Third-party commercial off-the-shelf components (for example, an enterprise resource planning subsystem)

### Noncompliant Code Example

Data sanitization requires an understanding of the data being passed and the capabilities of the subsystem. John Viega and Matt Messier provide an example of an application that inputs an email address to a buffer and then uses this string as an argument in a call to `system()` \[ [Viega 2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Viega03) \]:

The risk, of course, is that the user enters the following string as an email address:

For more information on the `system()` call, see [ENV03-C. Sanitize the environment when invoking external programs](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/environment-env/env03-c) and [ENV33-C. Do not call system()](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/environment-env/env33-c) .

### Compliant Solution

It is necessary to ensure that all valid data is accepted, while potentially dangerous data is rejected or [sanitized](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sanitize) . Doing so can be difficult when valid characters or sequences of characters also have special meaning to the subsystem and may involve [validating](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-validation) the data against a grammar. In cases where there is no overlap, whitelisting can be used to eliminate dangerous characters from the data.

The whitelisting approach to data sanitization is to define a list of acceptable characters and remove any character that is not acceptable. The list of valid input values is typically a predictable, well-defined set of manageable size. This compliant solution, based on the `tcp_wrappers` package written by Wietse Venema, shows the whitelisting approach:

The benefit of whitelisting is that a programmer can be certain that a string contains only characters that are considered safe by the programmer. Whitelisting is recommended over blacklisting, which traps all unacceptable characters, because the programmer needs only to ensure that acceptable characters are identified. As a result, the programmer can be less concerned about which characters an attacker may try in an attempt to bypass security checks.

### Noncompliant Code Example

This noncompliant code example is taken from \[ [VU#881872](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-VU881872) \], a vulnerability in the Sun Solaris TELNET daemon ( `in.telnetd` ) that allows a remote attacker to log on to the system with elevated privileges.

The vulnerability in `in.telnetd` invokes the `login` program by calling `execl()` . This call passes unsanitized data from an untrusted source (the `USER` environment variable) as an argument to the `login` program:

An attacker, in this case, can gain unauthenticated access to a system by setting the `USER` environment variable to a string, which is interpreted as an additional command-line option by the `login` program. This kind of attack is called *argument injection* .

### Compliant Solution

This compliant solution inserts the `"--"` (double dash) argument before the call to `getenv("USER")` in the call to `execl()` :

Because the `login` program uses the POSIX `getopt()` function to parse command-line arguments, and because the `"--"` option causes `getopt()` to stop interpreting options in the argument list, the `USER` variable cannot be used by an attacker to inject an additional command-line option. This is a valid means of sanitizing the untrusted user data in this context because the behavior of the interpretation of the resulting string is rendered innocuous.

The call to `execl()` is not susceptible to command injection because the shell command interpreter is not invoked. (See [ENV33-C. Do not call system()](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/environment-env/env33-c) .)

## STR03-C. Do not inadvertently truncate a string

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/04.characters-and-strings-str/05.str03-c.md) — SHA-256 `d883e3d0c2bfeee229640aa1307ab42f06f18e97ade2ff454a8097b7320c2f2f`。

Alternative functions that limit the number of bytes copied are often recommended to mitigate buffer overflow [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) . Examples include

-   `strncpy()` instead of `strcpy()`
-   `strncat()` instead of `strcat()`
-   `fgets()` instead of `gets()`
-   `snprintf()` instead of `sprintf()`

These functions truncate strings that exceed the specified limits. Additionally, some functions, such as `strncpy()` , do not guarantee that the resulting character sequence is null-terminated. (See [STR32-C. Do not pass a non-null-terminated character sequence to a library function that expects a string](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/characters-and-strings-str/str32-c) .)

Unintentional truncation results in a loss of data and in some cases leads to software vulnerabilities.

### Noncompliant Code Example

The standard functions `strncpy()` and `strncat()` copy a specified number of characters `n` from a source string to a destination array. In the case of `strncpy()` , if there is no null character in the first `n` characters of the source array, the result will not be null-terminated and any remaining characters are truncated.

### Compliant Solution (Adequate Space)

Either the `strcpy()` or `strncpy()` function can be used to copy a string and a null character to a destination buffer, provided there is enough space. The programmer must be careful to ensure that the destination buffer is large enough to hold the string to be copied and the null byte to prevent errors, such as data truncation and buffer overflow.

This solution requires that `string_data` is null-terminated; that is, a null byte can be found within the bounds of the referenced character array. Otherwise, `strlen()` will stray into other objects before finding a null byte.

### Exceptions

**STR03-C-EX1:** The intent of the programmer is to purposely truncate the string.

## STR04-C. Use plain char for characters in the basic character set

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/04.characters-and-strings-str/06.str04-c.md) — SHA-256 `e564a136d5556459f8edfbaa6a650e103d38eb4b479fbf2bda377832536e3779`。

There are three *character types* : `char` , `   signed char  ` , and `   unsigned char  ` . Compilers have the latitude to define `char` to have the same range, representation, and behavior as either `   signed char  `or`   unsigned char  ` . Irrespective of the choice made, `char` is a separate type from the other two and is not compatible with either.

For characters in the basic character set, it does not matter which data type is used, except for type compatibility. Consequently, it is best to use plain `char` for character data for compatibility with standard string-handling functions.

In most cases, the only portable operators on plain `char` types are assignment and equality operators ( `=` , `==` , `!=` ). An exception is the translation to and from digits. For example, if the `char` `c` is a digit, `   c - '0'  ` is a value between 0 and 9.

### Noncompliant Code Example

This noncompliant code example simply shows the standard string-handling function `strlen()` being called with a plain character string, a signed character string, and an unsigned character string. The `strlen()` function takes a single argument of type `   const char  ` \*:

Compiling at high warning levels in compliance with [MSC00-C. Compile cleanly at high warning levels](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc00-c) causes warnings to be issued when

-   Converting from `    unsigned char[]   `to`    const char *   `when`    char   ` is signed
-   Converting from `    signed char[]   `to`    const char *   `when`    char   ` is defined to be unsigned

Casts are required to eliminate these warnings, but excessive casts can make code difficult to read and hide legitimate warning messages.

If this C code were compiled using a C++ compiler, conversions from `   unsigned char[]  `to`   const char *  ` and from `   signed char[]  `to`   const char *  ` would be flagged as errors requiring casts.

### Compliant Solution

The compliant solution uses plain `char` for character data:

Conversions are not required, and the code compiles cleanly at high warning levels without casts.

## STR05-C. Use pointers to const when referring to string literals

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/04.characters-and-strings-str/07.str05-c.md) — SHA-256 `09513f8536a734062d047f82db7aac75cd2ccad8d64113d119f58c5284d085ec`。

The type of a narrow string literal is an array of `char` , and the type of a wide string literal is an array of `wchar_t` . However, string literals (of both types) are notionally constant and should consequently be protected by `const` qualification. This recommendation is a specialization of [DCL00-C. Const-qualify immutable objects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl00-c) and also supports [STR30-C. Do not attempt to modify string literals](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/characters-and-strings-str/str30-c) .

Adding `const` qualification may propagate through a program; as `const` qualifiers are added, still more become necessary. This phenomenon is sometimes called *const-poisoning* . Const-poisoning can frequently lead to violations of [EXP05-C. Do not cast away a const qualification](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/expressions-exp/exp05-c) . Although `const` qualification is a good idea, the costs may outweigh the value in the remediation of existing code.

### Noncompliant Code Example (Narrow String Literal)

In this noncompliant code example, the `const` keyword has been omitted:

If a statement such as `   c[0] = 'C'  ` were placed following the declaration in the noncompliant code example, the code is likely to compile cleanly, but the result of the assignment would be [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) because string literals are considered constant.

### Compliant Solution (Immutable Strings)

In this compliant solution, the characters referred to by the pointer `c` are `const` -qualified, meaning that any attempt to assign them to different values is an error:

### Compliant Solution (Mutable Strings)

In cases where the string is meant to be modified, use initialization instead of assignment. In this compliant solution, `c` is a modifiable `char` array that has been initialized using the contents of the corresponding string literal:

Consequently, a statement such as `   c[0] = 'C'  ` is valid and behaves as expected.

### Noncompliant Code Example (Wide String Literal)

In this noncompliant code example, the `const` keyword has been omitted:

If a statement such as `   c[0] = L'C'  ` were placed following this declaration, the code is likely to compile cleanly, but the result of the assignment would be [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) because string literals are considered constant.

### Compliant Solution (Immutable Strings)

In this compliant solution, the characters referred to by the pointer `c` are `const` -qualified, meaning that any attempt to assign them to different values is an error:

### Compliant Solution (Mutable Strings)

In cases where the string is meant to be modified, use initialization instead of assignment. In this compliant solution, `c` is a modifiable `wchar_t` array that has been initialized using the contents of the corresponding string literal:

Consequently, a statement such as `   c[0] = L'C'  ` is valid and behaves as expected.

## STR06-C. Do not assume that strtok() leaves the parse string unchanged

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/04.characters-and-strings-str/08.str06-c.md) — SHA-256 `0e9a6cf4382e9af3a721fe0bff011248db2e0705aba1adebc97d86ad1b82e08b`。

The C function `strtok()` is a string tokenization function that takes two arguments: an initial string to be parsed and a `const` -qualified character delimiter. It returns a pointer to the first character of a token or to a null pointer if there is no token.

The first time `strtok()` is called, the string is parsed into tokens and a character delimiter. The `strtok()` function parses the string up to the first instance of the delimiter character, replaces the character in place with a null byte ( `'\0'` ), and returns the address of the first character in the token. Subsequent calls to `strtok()` begin parsing immediately after the most recently placed null character.

Because `strtok()` modifies the initial string to be parsed, the string is subsequently unsafe and cannot be used in its original form. If you need to preserve the original string, copy it into a buffer and pass the address of the buffer to `strtok()` instead of the original string.

### Noncompliant Code Example

In this example, the `strtok()` function is used to parse the first argument into colon-delimited tokens; it outputs each word from the string on a new line. Assume that `PATH` is `"/usr/bin:/usr/sbin:/sbin"` .

After the loop ends, `path` is modified as follows: `"/usr/bin\0/bin\0/usr/sbin\0/sbin\0"` . This is an issue because the local `path` variable becomes `/usr/bin` and because the environment variable `PATH` has been unintentionally changed, which can have unintended consequences. (See [ENV30-C. Do not modify the object referenced by the return value of certain functions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/environment-env/env30-c) .)

### Compliant Solution

In this compliant solution, the string being tokenized is copied into a temporary buffer that is not referenced after the call to `strtok()` :

Another possibility is to provide your own implementation of `strtok()` that does not modify the initial arguments.

## STR09-C. Don't assume numeric values for expressions with type plain character

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/04.characters-and-strings-str/09.str09-c.md) — SHA-256 `70d43a8a031a96041b520f1e5be8904cc4e90c6dc79b3c7285fe6f7bfdc7ba49`。

For portable applications, use only the assignment `=` operator, the equality operators `==` and `!=` , and the unary `&` operator on plain-character-typed or plain-wide-character-typed expressions.

This practice is recommended because the C Standard requires only the digit characters (0–9) to have consecutive numerical values. Consequently, operations that rely on expected values for plain-character- or plain-wide-character-typed expressions can lead to unexpected behavior.

However, because of the requirement for digit characters, other operators can be used for them according to the following restrictions:

-   The binary `+` operator may be used to add integer values 0 through 9 to `'0'` .
-   The binary `-` operator may be used to subtract character 0.
-   Relational operators `<` , `<=` , `>` , and `>=` can be used to check whether a character or wide character is a digit.

Character types should be chosen and used in accordance with [STR04-C. Use plain char for characters in the basic character set](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/characters-and-strings-str/str04-c) .

### Noncompliant Code Example

This noncompliant code example attempts to determine if the value of a character variable is between `'a'` and `'c'` inclusive. However, because the C Standard does not require the letter characters to be in consecutive or alphabetic order, the check might not work as expected.

### Compliant Solution

In this example, the specific check is enforced using compliant operations on character expressions:

### Exceptions

**STR09-C-EX1** : Consecutive values for characters like `a~z` can be assumed on platforms where ASCII or Unicode is used. This recommendation is primarily concerned with platform portability, for example, if code is migrated from ASCII systems to non-ASCII systems.

## STR10-C. Do not concatenate different type of string literals

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/04.characters-and-strings-str/10.str10-c.md) — SHA-256 `4c85a7c074385cf7c48ec319d7c66aa0bbfabe16b887c04ff14b38e086c874ff`。

According to [MISRA 2008](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MISRA08) , concatenation of wide and narrow string literals leads to [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . This was once considered implicitly undefined behavior until C90 \[ [ISO/IEC 9899:1990](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-1990) \]. However, C99 defined this behavior \[ [ISO/IEC 9899:1999](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-1999) \], and C11 further explains in subclause 6.4.5, paragraph 5 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \]:

> In translation phase 6, the multibyte character sequences specified by any sequence of adjacent character and identically-prefixed string literal tokens are concatenated into a single multibyte character sequence. If any of the tokens has an encoding prefix, the resulting multibyte character sequence is treated as having the same prefix; otherwise, it is treated as a character string literal. Whether differently-prefixed wide string literal tokens can be concatenated and, if so, the treatment of the resulting multibyte character sequence are implementation-defined.

Nonetheless, it is recommended that string literals that are concatenated should all be the same type so as not to rely on [implementation-defined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) or [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) if compiled on a platform that supports only C90.

### Noncompliant Code Example (C90)

This noncompliant code example concatenates wide and narrow string literals. Although the behavior is undefined in C90, the programmer probably intended to create a wide string literal.

### Compliant Solution (C90, Wide String Literals)

If the concatenated string needs to be a wide string literal, each element in the concatenation must be a wide string literal, as in this compliant solution:

### Compliant Solution (C90, Narrow String Literals)

If wide string literals are unnecessary, it is better to use narrow string literals, as in this compliant solution:

## STR11-C. Do not specify the bound of a character array initialized with a string literal

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/04.characters-and-strings-str/11.str11-c.md) — SHA-256 `ec059bb1622d9b86b2bb1fe5488048cc91d208ad36cf5a05a973607186feb41d`。

The C Standard allows an array variable to be declared both with a bound index and with an initialization literal. The initialization literal also implies an array size in the number of elements specified. For strings, the size specified by a string literal is the number of characters in the literal plus one for the terminating null character.

It is common for an array variable to be initialized by a string literal and declared with an explicit bound that matches the number of characters in the string literal. Subclause 6.7.9, paragraph 14, of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], says:

> An array of character type may be initialized by a character string literal or UTF −8 string literal, optionally enclosed in braces. Successive bytes of the string literal (including the terminating null character if there is room or if the array is of unknown size) initialize the elements of the array.

However, if the string is intended to be used as a null-terminated byte string, then the array will have one too few characters to hold the string because it does not account for the terminating null character. Such a sequence of characters has limited utility and has the potential to cause [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) if a null-terminated byte string is assumed.

A better approach is to not specify the bound of a string initialized with a string literal because the compiler will automatically allocate sufficient space for the entire string literal, including the terminating null character. This rule is a specific exception to [ARR02-C. Explicitly specify array bounds, even if implicitly defined by an initializer](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/arrays-arr/arr02-c) .

### Noncompliant Code Example

This noncompliant code example initializes an array of characters using a string literal that defines one character more (counting the terminating `'\0'` ) than the array can hold:

The size of the array `s` is 3, although the size of the string literal is 4. Any subsequent use of the array as a null-terminated byte string can result in a vulnerability, because `s` is not properly null-terminated. (See [STR32-C. Do not pass a non-null-terminated character sequence to a library function that expects a string](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/characters-and-strings-str/str32-c) .)

#### Implementation Details

This code compiles with no warning with Visual Studio 2013 and GCC 4.8.1. It produces a three-character array with no terminating null character, as specified by the standard.

### Compliant Solution

This compliant solution does not specify the bound of a character array in the array declaration. If the array bound is omitted, the compiler allocates sufficient storage to store the entire string literal, including the terminating null character.

This approach is preferred because the size of the array can always be derived even if the size of the string literal changes.

### Exceptions

**STR11-C-EX1:** If the intention is to create a character array and *not* a null-terminated byte string, initializing to fit exactly without a null byte is allowed but not recommended. The preferred approach to create an array containing just the three characters `'a'` , `'b'` , and `'c'` , for example, is to declare each character literal as a separate element as follows:

Also, you should make clear in comments or documentation if a character array is, in fact, not a null-terminated byte string.

**STR11-C-EX2:** If the character array must be larger than the string literal it is initialized with, you may explicitly specify an array bounds. This is particularly important if the array's contents might change during program execution.

## STR30-C. Do not attempt to modify string literals

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/04.characters-and-strings-str/2.str30-c.md) — SHA-256 `36c0dc37097fcea70a235567e8d34b3e879a9ede88f79a845e49bad7ee9bdcdd`。

According to the C Standard, 6.4.5, paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \]:

> A character string literal is a sequence of zero or more multibyte characters enclosed in double-quotes, as in "xyz" . A UTF-8 string literal is the same, except prefixed by u8 . A wchar \_ t string literal is the same, except prefixed by L . A UTF-16 string literal is the same, except prefixed by u . A UTF-32 string literal is the same, except prefixed by U . Collectively, wchar \_ t , UTF-16, and UTF-32 string literals are called wide string literals .

At compile time, string literals are used to create an array of static storage duration of sufficient length to contain the character sequence and a terminating null character. String literals are usually referred to by a pointer to (or array of) characters. Ideally, they should be assigned only to pointers to (or arrays of) `   const char  `or`   const wchar_t  ` . It is unspecified whether these arrays of string literals are distinct from each other. The behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) if a program attempts to modify any portion of a string literal. Modifying a string literal frequently results in an access violation because string literals are typically stored in read-only memory. (See [undefined behavior 32](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_32) .)

Avoid assigning a string literal to a pointer to non- `const` or casting a string literal to a pointer to non- `const` . For the purposes of this rule, a pointer to (or array of) `const` characters must be treated as a string literal. Similarly, the returned value of the following library functions must be treated as a string literal if the first argument is a string literal:

-   `    strpbrk(), strchr(), strrchr(), strstr()   `-`    wcspbrk(), wcschr(), wcsrchr    ()    , wcsstr()   `-`    memchr(), wmemchr()   `

This rule is a specific instance of [EXP40-C. Do not modify constant objects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp40-c) .

### Noncompliant Code Example

In this noncompliant code example, the `char` pointer `str` is initialized to the address of a string literal. Attempting to modify the string literal is [undefined behavior 32](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_32) :

### Compliant Solution

As an array initializer, a string literal specifies the initial values of characters in an array as well as the size of the array. (See [STR11-C. Do not specify the bound of a character array initialized with a string literal](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/characters-and-strings-str/str11-c) .) This code creates a copy of the string literal in the space allocated to the character array `str` . The string stored in `str` can be modified safely.

### Noncompliant Code Example (POSIX)

In this noncompliant code example, a string literal is passed to the (pointer to non- `const` ) parameter of the POSIX function `mkstemp()` , which then modifies the characters of the string literal:

The behavior of `mkstemp()` is described in more detail in [FIO21-C. Do not create temporary files in shared directories](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio21-c) .

### Compliant Solution (POSIX)

This compliant solution uses a named array instead of passing a string literal:

### Noncompliant Code Example (Result of `strrchr()` )

In this noncompliant example, the `   char *  ` result of the `strrchr()` function is used to modify the object pointed to by `pathname` . Because the argument to `strrchr()` points to a string literal, the effects of the modification are undefined.

### Compliant Solution (Result of `strrchr()` )

This compliant solution avoids modifying a `const` object, even if it is possible to obtain a non- `const` pointer to such an object by calling a standard C library function, such as `strrchr()` . To reduce the risk to callers of `get_dirname()` , a buffer and length for the directory name are passed into the function. It is insufficient to change `pathname` to require a `   char *  ` instead of a `   const char *  ` because conforming compilers are not required to diagnose passing a string literal to a function accepting a `   char *  ` .

## STR31-C. Guarantee that storage for strings has sufficient space for character data and the null terminator

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/04.characters-and-strings-str/3.str31-c.md) — SHA-256 `4cac1df7593dab8f6314d64d875b8a0922019c9e2fbd8b8f4fc52374200525ea`。

Copying data to a buffer that is not large enough to hold that data results in a buffer overflow. Buffer overflows occur frequently when manipulating strings \[ [Seacord 2013b](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Seacord2013) \]. To prevent such errors, either limit copies through truncation or, preferably, ensure that the destination is of sufficient size to hold the character data to be copied and the null-termination character. (See [STR03-C. Do not inadvertently truncate a string](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/characters-and-strings-str/str03-c) .)

When strings live on the heap, this rule is a specific instance of [MEM35-C. Allocate sufficient memory for an object](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/memory-management-mem/mem35-c) . Because strings are represented as arrays of characters, this rule is related to both [ARR30-C. Do not form or use out-of-bounds pointers or array subscripts](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr30-c) and [ARR38-C. Guarantee that library functions do not form invalid pointers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr38-c) .

### Noncompliant Code Example (Off-by-One Error)

This noncompliant code example demonstrates an *off-by-one* error \[ [Dowd 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Dowd06) \]. The loop copies data from `src` to `dest` . However, because the loop does not account for the null-termination character, it may be incorrectly written 1 byte past the end of `dest` .

### Compliant Solution (Off-by-One Error)

In this compliant solution, the loop termination condition is modified to account for the null-termination character that is appended to `dest` :

### Noncompliant Code Example ( `gets()` )

The `gets()` function, which was deprecated in the C99 Technical Corrigendum 3 and removed from C11 , is inherently unsafe and should never be used because it provides no way to control how much data is read into a buffer from `stdin` . This noncompliant code example assumes that `gets()` will not read more than `   BUFFER_SIZE - 1  ` characters from `stdin` . This is an invalid assumption, and the resulting operation can result in a buffer overflow.

The `gets()` function reads characters from the `stdin` into a destination array until end-of-file is encountered or a newline character is read. Any newline character is discarded, and a null character is written immediately after the last character read into the array.

See also [MSC24-C. Do not use deprecated or obsolescent functions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc24-c) .

### Compliant Solution ( `fgets()` )

The `fgets()` function reads, at most, one less than the specified number of characters from a stream into an array. This solution is compliant because the number of characters copied from `stdin` to `buf` cannot exceed the allocated memory:

The `fgets()` function is not a strict replacement for the `gets()` function because `fgets()` retains the newline character (if read) and may also return a partial line. It is possible to use `fgets()` to safely process input lines too long to store in the destination array, but this is not recommended for performance reasons. Consider using one of the following compliant solutions when replacing `gets()` .

### Compliant Solution ( `getline()` , POSIX)

The `getline()` function is similar to the `fgets()` function but can dynamically allocate memory for the input buffer. If passed a null pointer, `getline()` dynamically allocates a buffer of sufficient size to hold the input. If passed a pointer to dynamically allocated storage that is too small to hold the contents of the string, the `getline()` function resizes the buffer, using `realloc()` , rather than truncating the input. If successful, the `getline()` function returns the number of characters read, which can be used to determine if the input has any null characters before the newline. The `getline()` function works only with dynamically allocated buffers. Allocated memory must be explicitly deallocated by the caller to avoid memory leaks. (See [MEM31-C. Free dynamically allocated memory when no longer needed](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/memory-management-mem/mem31-c) .)

Note that the `getline()` function uses an [in-band error indicator](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-in-banderrorindicator) , in violation of [ERR02-C. Avoid in-band error indicators](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err02-c) .

### Noncompliant Code Example ( `getchar()` )

Reading one character at a time provides more flexibility in controlling behavior, though with additional performance overhead. This noncompliant code example uses the `getchar()` function to read one character at a time from `stdin` instead of reading the entire line at once. The `stdin` stream is read until end-of-file is encountered or a newline character is read. Any newline character is discarded, and a null character is written immediately after the last character read into the array. Similar to the noncompliant code example that invokes `gets()` , there are no guarantees that this code will not result in a buffer overflow.

After the loop ends, if `   ch == EOF  ` , the loop has read through to the end of the stream without encountering a newline character, or a read error occurred before the loop encountered a newline character. To conform to [FIO34-C. Distinguish between characters read from a file and EOF or WEOF](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/input-output-fio/fio34-c) , the error-handling code must verify that an end-of-file or error has occurred by calling `feof()` or `ferror()` .

### Compliant Solution ( `getchar()` )

In this compliant solution, characters are no longer copied to `buf` once `   index == BUFFERSIZE - 1  ` , leaving room to null-terminate the string. The loop continues to read characters until the end of the line, the end of the file, or an error is encountered. When `   truncated == true  ` , the input string has been truncated.

### Noncompliant Code Example ( `fscanf()` )

In this noncompliant example, the call to `fscanf()` can result in a write outside the character array `buf` :

### Compliant Solution ( `fscanf()` )

In this compliant solution, the call to `fscanf()` is constrained not to overflow `buf` :

### Noncompliant Code Example ( `argv` )

In a [hosted environment](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-hostedenvironment) , arguments read from the command line are stored in process memory. The function `main()` , called at program startup, is typically declared as follows when the program accepts command-line arguments:

Command-line arguments are passed to `main()` as pointers to strings in the array members `argv[0]` through `   argv[argc - 1]  ` . If the value of `argc` is greater than 0, the string pointed to by `argv[0]` is, by convention, the program name. If the value of `argc` is greater than 1, the strings referenced by `argv[1]` through `   argv[argc - 1]  ` are the program arguments.

[Vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) can occur when inadequate space is allocated to copy a command-line argument or other program input. In this noncompliant code example, an attacker can manipulate the contents of `argv[0]` to cause a buffer overflow:

### Compliant Solution ( `argv` )

The `strlen()` function can be used to determine the length of the strings referenced by `argv[0]` through `   argv[argc - 1]  ` so that adequate memory can be dynamically allocated.

Remember to add a byte to the destination string size to accommodate the null-termination character.

### Compliant Solution ( `argv` )

The `strcpy_s()` function provides additional safeguards, including accepting the size of the destination buffer as an additional argument.

The `strcpy_s()` function can be used to copy data to or from dynamically allocated memory or a statically allocated array. If insufficient space is available, `strcpy_s()` returns an error.

### Compliant Solution ( `argv` )

If an argument will not be modified or concatenated, there is no reason to make a copy of the string. Not copying a string is the best way to prevent a buffer overflow and is also the most efficient solution. Care must be taken to avoid assuming that `argv[0]` is non-null.

### Noncompliant Code Example ( `getenv()` )

According to the C Standard, 7.22.4.6 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \]

> The `getenv` function searches an environment list, provided by the host environment, for a string that matches the string pointed to by `name` . The set of environment names and the method for altering the environment list are implementation defined.

Environment variables can be arbitrarily large, and copying them into fixed-length arrays without first determining the size and allocating adequate storage can result in a buffer overflow.

### Compliant Solution ( `getenv()` )

Environmental variables are loaded into process memory when the program is loaded. As a result, the length of these strings can be determined by calling the `strlen()` function, and the resulting length can be used to allocate adequate dynamic memory:

### Noncompliant Code Example ( `sprintf()` )

In this noncompliant code example, `name` refers to an external string; it could have originated from user input, the file system, or the network. The program constructs a file name from the string in preparation for opening the file.

Because the `sprintf()` function makes no guarantees regarding the length of the generated string, a sufficiently long string in `name` could generate a buffer overflow.

### Compliant Solution ( `sprintf()` )

The buffer overflow in the preceding noncompliant example can be prevented by adding a precision to the `%s` conversion specification. If the precision is specified, no more than that many bytes are written. The precision `123` in this compliant solution ensures that `filename` can contain the first 123 characters of `name` , the `.txt` extension, and the null terminator.

You can also use `*` to indicate that the precision should be provided as a variadic argument:

### Compliant Solution ( `snprintf()` )

A more general solution is to use the `snprintf()` function, which also truncates `name` if it will not fit in the `filename` .

## STR32-C. Do not pass a non-null-terminated character sequence to a library function that expects a string

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/04.characters-and-strings-str/4.str32-c.md) — SHA-256 `fd2dbfc4ac5e1b5e0b9adea4187e7cddd10521ea0bcc2bdc0f509c947d63673c`。

Many library functions accept a string or wide string argument with the constraint that the string they receive is properly null-terminated. Passing a character sequence or wide character sequence that is not null-terminated to such a function can result in accessing memory that is outside the bounds of the object. Do not pass a character sequence or wide character sequence that is not null-terminated to a library function that expects a string or wide string argument.

### Noncompliant Code Example

This code example is noncompliant because the character sequence `c_str` will not be null-terminated when passed as an argument to `printf().` (See [STR11-C. Do not specify the bound of a character array initialized with a string literal](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/characters-and-strings-str/str11-c) on how to properly initialize character arrays.)

### Compliant Solution

This compliant solution does not specify the bound of the character array in the array declaration. If the array bound is omitted, the compiler allocates sufficient storage to store the entire string literal, including the terminating null character.

### Noncompliant Code Example

This code example is noncompliant because the wide character sequence `cur_msg` will not be null-terminated when passed to `wcslen()` . This will occur if `lessen_memory_usage()` is invoked while `cur_msg_size` still has its initial value of 1024.

### Compliant Solution

In this compliant solution, `cur_msg` will always be null-terminated when passed to `wcslen()` :

### Noncompliant Code Example ( `strncpy()` )

Although the `strncpy()` function takes a string as input, it does not guarantee that the resulting value is still null-terminated. In the following noncompliant code example, if no null character is contained in the first `n` characters of the `source` array, the result will not be null-terminated. Passing a non-null-terminated character sequence to `strlen()` is [undefined behavior 196](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_196) .

### Compliant Solution (Truncation)

This compliant solution is correct if the programmer's intent is to truncate the string:

### Compliant Solution (Copy without Truncation)

If the programmer's intent is to copy without truncation, this compliant solution copies the data and guarantees that the resulting array is null-terminated. If the string cannot be copied, it is handled as an error condition.

Note that this code is not bulletproof. It gracefully handles the case where `source` is NULL, when it is a valid string, and when `source` is not null-terminated, but at least the first 32 bytes are valid. However, in cases where `source` is not NULL, but points to invalid memory, or any of the first 32 bytes are invalid memory, the first call to `strnlen()` will access this invalid memory, and the resulting behavior is undefined. Unfortunately, standard C provides no way to prevent or even detect this condition without some external knowledge about the memory `source` points to.

## STR34-C. Cast characters to unsigned char before converting to larger integer sizes

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/04.characters-and-strings-str/5.str34-c.md) — SHA-256 `2700e9e11d0671a61fff9586a6aa71cd7cdf51d015aea2cb80dbeee77e535fcf`。

Signed character data must be converted to `   unsigned char  ` before being assigned or converted to a larger integer type. This rule applies to both `   signed char  ` and (plain) `char` characters on implementations where `char` is defined to have the same range, representation, and behaviors as `   signed char  ` .

However, this rule is applicable only in cases where the character data may contain values that can be misinterpreted as negative numbers. For example, if the `char` type is represented by a two's complement 8-bit value, any character value greater than +127 is interpreted as a negative value.

This rule is a generalization of [STR37-C. Arguments to character-handling functions must be representable as an unsigned char](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/characters-and-strings-str/str37-c) .

### Noncompliant Code Example

This noncompliant code example is taken from a [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) in bash versions 1.14.6 and earlier that led to the release of CERT Advisory [CA-1996-22](https://www.cert.org/advisories/CA-1996-22.html) . This vulnerability resulted from the sign extension of character data referenced by the `c_str` pointer in the `yy_string_get()` function in the `parse.y` module of the bash source code:

The `c_str` variable is used to traverse the character string containing the command line to be parsed. As characters are retrieved from this pointer, they are stored in a variable of type `int` . For implementations in which the `char` type is defined to have the same range, representation, and behavior as `   signed char  ` , this value is sign-extended when assigned to the `int` variable. For character code 255 decimal (−1 in two's complement form), this sign extension results in the value −1 being assigned to the integer, which is indistinguishable from `EOF` .

### Noncompliant Code Example

This problem can be repaired by explicitly declaring the `c_str` variable as `   unsigned char  ` :

This example, however, violates [STR04-C. Use plain char for characters in the basic character set](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/characters-and-strings-str/str04-c) .

### Compliant Solution

In this compliant solution, the result of the expression `*c_str++` is cast to `   unsigned char  ` before assignment to the `int` variable `c` :

### Noncompliant Code Example

In this noncompliant code example, the cast of `*s` to `   unsigned int  ` can result in a value in excess of `UCHAR_MAX` because of integer promotions, a violation of [ARR30-C. Do not form or use out-of-bounds pointers or array subscripts](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr30-c) :

### Compliant Solution

This compliant solution casts the value of type `char` to `   unsigned char  ` before the implicit promotion to a larger type:

### Exceptions

**STR34-C-EX1:** This rule only applies to characters that are to be treated as unsigned chars for some purpose, such as being passed to the isdigit() function. Characters that hold small integer values for mathematical purposes need not comply with this rule.

## STR37-C. Arguments to character-handling functions must be representable as an unsigned char

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/04.characters-and-strings-str/6.str37-c.md) — SHA-256 `6afc1d2a9915ade7e61bb23706d2aa6f51cf94dd64e397ef8f60cffbf8b4cd53`。

According to the C Standard, 7.4.1 paragraph 1 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \],

> The header `<ctype.h>` declares several functions useful for classifying and mapping characters. In all cases the argument is an `int` , the value of which shall be representable as an `    unsigned char   ` or shall equal the value of the macro `EOF` . If the argument has any other value, the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

See also [undefined behavior 112](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_112) .

This rule is applicable only to code that runs on platforms where the `char` data type is defined to have the same range, representation, and behavior as `   signed char  ` .

Following are the character classification functions that this rule addresses:

|                           |                                         |                                         |                          |
|---------------------------|-----------------------------------------|-----------------------------------------|--------------------------|
| `isalnum()`  | `isalpha()`                | `isascii()` <sup>XSI</sup> | `isblank()` |
| `iscntrl()`  | `isdigit()`                | `isgraph()`                | `islower()` |
| `isprint()`  | `ispunct()`                | `isspace()`                | `isupper()` |
| `isxdigit()` | `toascii()` <sup>XSI</sup> | `toupper()`                | `tolower()` |

<sup>XSI</sup> denotes an X/Open System Interfaces Extension to ISO/IEC 9945—POSIX. These functions are not defined by the C Standard.

This rule is a specific instance of [STR34-C. Cast characters to unsigned char before converting to larger integer sizes](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/characters-and-strings-str/str34-c) .

### Noncompliant Code Example

On implementations where plain `char` is signed, this code example is noncompliant because the parameter to `isspace()` , `*t` , is defined as a `   const char *  ` , and this value might not be representable as an `   unsigned char  ` :

The argument to `isspace()` must be `EOF` or representable as an `   unsigned char  ` ; otherwise, the result is undefined.

### Compliant Solution

This compliant solution casts the character to `   unsigned char  ` before passing it as an argument to the `isspace()` function:

## STR38-C. Do not confuse narrow and wide character strings and functions

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/04.characters-and-strings-str/7.str38-c.md) — SHA-256 `b6e227158423439167fcbc20772aee3e7c0623172a8210c92556a55b99229c82`。

Passing narrow string arguments to wide string functions or wide string arguments to narrow string functions can lead to [unexpected](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unexpectedbehavior) and [undefined behavior 151](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_151) . Scaling problems are likely because of the difference in size between wide and narrow characters. (See [ARR39-C. Do not add or subtract a scaled integer to a pointer.)](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr39-c) Because wide strings are terminated by a null wide character and can contain null bytes, determining the length is also problematic.

Because `wchar_t` and `char` are distinct types, many compilers will produce a warning diagnostic if an inappropriate function is used. (See [MSC00-C. Compile cleanly at high warning levels](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc00-c) .)

### Noncompliant Code Example (Wide Strings with Narrow String Functions)

This noncompliant code example incorrectly uses the `strncpy()` function in an attempt to copy up to 10 wide characters. However, because wide characters can contain null bytes, the copy operation may end earlier than anticipated, resulting in the truncation of the wide string.

### Noncompliant Code Example (Narrow Strings with Wide String Functions)

This noncompliant code example incorrectly invokes the `wcsncpy()` function to copy up to 10 wide characters from `narrow_str1` to `narrow_str2` . Because `narrow_str2` is a narrow string, it has insufficient memory to store the result of the copy and the copy will result in a buffer overflow.

### Compliant Solution

This compliant solution uses the proper-width functions. Using `wcsncpy()` for wide character strings and `strncpy()` for narrow character strings ensures that data is not truncated and buffer overflow does not occur.

### Noncompliant Code Example ( `strlen()` )

In this noncompliant code example, the `strlen()` function is used to determine the size of a wide character string:

The `strlen()` function determines the number of characters that precede the terminating null character. However, wide characters can contain null bytes, particularly when expressing characters from the ASCII character set, as in this example. As a result, the `strlen()` function will return the number of bytes preceding the first null byte in the wide string.

### Compliant Solution

This compliant solution correctly calculates the number of bytes required to contain a copy of the wide string, including the terminating null wide character:
