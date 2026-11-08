# CERT C — INT

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## INT00-C. Understand the data model used by your implementation(s)

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/02.int00-c.md) — SHA-256 `c5cb7d6e708172ae99dac205203430f67c5cb702a24451ae29c0759d9ec48c2b`。

A *data model* defines the sizes assigned to standard data types. It is important to understand the data models used by your [implementation](http://BB.%20Definitions#implementation) . However, if your code depends on any assumptions not guaranteed by the standard, you should provide static assertions to ensure that your assumptions are valid. (See [DCL03-C. Use a static assertion to test the value of a constant expression](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl03-c) .) Assumptions concerning integer sizes may become invalid, for example, when porting from a 32-bit architecture to a 64-bit architecture.

### Common Data Models

| Data Type | iAPX86 | IA-32 | IA-64 | SPARC-64 | ARM-32 | Alpha | 64-bit Linux, FreeBSD,<br /><br />NetBSD, and OpenBSD |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `char` | 8 | 8 | 8 | 8 | 8 | 8 | 8 |
| `short` | 16 | 16 | 16 | 16 | 16 | 16 | 16 |
| `int` | 16 | 32 | 32 | 32 | 32 | 32 | 32 |
| `long` | 32 | 32 | 32 | 64 | 32 | 64 | 64 |
| `long long` | N/A | 64 | 64 | 64 | 64 | 64 | 64 |
| Pointer | 16/32 | 32 | 64 | 64 | 32 | 64 | 64 |

Code frequently embeds assumptions about data models. For example, some code bases require pointer and `long` to have the same size, whereas other large code bases require `int` and `long` to be the same size \[ [van de Voort 2007](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-vandeVoort07) \]. These types of assumptions, while common, make the code difficult to port and make the ports error prone. One solution is to avoid any [implementation-defined behavior](http://BB.%20Definitions#implementation-defined%20behavior) . However, this practice can result in inefficient code. Another solution is to include either static or runtime assertions near any platform-specific assumptions, so they can be easily detected and corrected during porting.

#### `<limits.h>`

Possibly more important than knowing the number of bits for a given type is knowing that `limits.h` defines macros that can be used to determine the integral ranges of the standard integer types for any conforming implementation. For example, `UINT_MAX` is the largest possible value of an `   unsigned int  ` , and `LONG_MIN` is the smallest possible value of a `   long int  ` .

#### `<stdint.h>`

The `stdint.h` header introduces types with specific size restrictions that can be used to avoid dependence on a particular data model. For example, `int_least32_t` is the smallest signed integer type supported by the implementation that contains at least 32 bits. The type `uint_fast16_t` is the fastest unsigned integer type supported by the implementation that contains at least 16 bits. The type `intmax_t` is the largest signed integer, and `uintmax_t` is the largest unsigned type, supported by the implementation. The following types are required to be available on all implementations:

|                |                              |                               |
|----------------|------------------------------|-------------------------------|
| Smallest Types | Signed                       | Unsigned                      |
| 8 bits         | `int_least8_t`  | `uint_least8_t`  |
| 16 bits        | `int_least16_t` | `uint_least16_t` |
| 32 bits        | `int_least32_t` | `uint_least32_t` |
| 64 bits        | `int_least64_t` | `uint_least64_t` |
| Fastest Types  | Signed                       | Unsigned                      |
| 8 bits         | `int_fast8_t`   | `uint_fast8_t`   |
| 16 bits        | `int_fast16_t`  | `uint_fast16_t`  |
| 32 bits        | `int_fast32_t`  | `uint_fast32_t`  |
| 64 bits        | `int_fast64_t`  | `uint_fast64_t`  |
| Largest Types  | Signed                       | Unsigned                      |
| Maximum        | `intmax_t`      | `uintmax_t`      |

Additional types may be supported by an implementation, such as `int8_t` , a type of exactly 8 bits, and `uintptr_t` , a type large enough to hold a converted `   void *  ` if such an integer exists in the implementation.

#### `<inttypes.h>`

The `inttypes.h` header declares functions for manipulating greatest-width integers and converting numeric character strings to greatest-width integers.

### Noncompliant Code Example

This noncompliant example attempts to read a `long` into an `int` . This code works for models in which `   sizeof(int) == sizeof(long)  ` . For others, it causes an unexpected memory write similar to a buffer overflow.

Some compilers can generate warnings if a constant format string does not match the argument types.

### Compliant Solution

This compliant solution uses the correct format for the type being used:

### Noncompliant Code Example

This noncompliant code attempts to guarantee that all bits of a multiplication of two `   unsigned int  ` values are retained by performing arithmetic in the type `   unsigned long  ` . This practice works for some platforms, such as 64-bit Linux, but fails for others, such as 64-bit Microsoft Windows.

### Compliant Solution

This compliant solution uses the largest unsigned integer type available if it is guaranteed to hold the result. If it is not, another solution must be found, as discussed in [INT32-C. Ensure that operations on signed integers do not result in overflow](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int32-c) .

## INT01-C. Use size_t for all integer values representing the size of an object

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/03.int01-c.md) — SHA-256 `fd1766ad80debe9f71ea5899b3196132ea67b9ac49f66c88cf96d5fcefc9b096`。

The `size_t` type is the unsigned integer type of the result of the `sizeof` operator. Variables of type `size_t` are guaranteed to be of sufficient precision to represent the size of an object. The limit of `size_t` is specified by the `SIZE_MAX` macro.

The type `size_t` generally covers the entire address space. Any variable that is used to represent the size of an object, including integer values used as sizes, indices, loop counters, and lengths, should be declared `size_t` .

### Noncompliant Code Example

In this noncompliant code example, the dynamically allocated buffer referenced by `p` overflows for values of `   n > INT_MAX  ` :

Signed integer overflow causes [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . The following are two possible conditions under which this code constitutes a serious [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) :

###### `   sizeof(size_t) == sizeof(int)  `

The unsigned `n` may contain a value greater than `INT_MAX` . Assuming quiet wraparound on signed overflow, the loop executes `n` times because the comparison `   i < n  ` is an unsigned comparison. Once `i` is incremented beyond `INT_MAX` , `i` takes on negative values starting with `(INT_MIN)` . Consequently, the memory locations referenced by `p[i]` precede the memory referenced by `p` , and a write outside array bounds occurs.

###### `   sizeof(size_t) > sizeof(int)  `

For values of `n` where `   0 < n <= INT_MAX  ` , the loop executes `n` times, as expected.

For values of `n` where `   INT_MAX < n <= (size_t)INT_MIN  ` , the loop executes `INT_MAX` + 1 times. Once `i` becomes negative, the loop stops, and `i` remains in the range `0` through `INT_MAX` .

For values of `n` where `   (size_t)INT_MIN < n <= SIZE_MAX  `,`   i  ` wraps and takes the values `INT_MIN` to `   INT_MIN + (n - (size_t)INT_MIN - 1)  ` . Execution of the loop overwrites memory from `p[INT_MIN]` through `   p[INT_MIN + (n - (size_t)INT_MIN - 1)]  ` .

### Compliant Solution

Declaring `i` to be of type `size_t` eliminates the possible integer overflow condition (in this example):

### Noncompliant Code Example

In this noncompliant code example, the value of `length` is read from a network connection and passed as an argument to a wrapper to `malloc()` to allocate the appropriate data block. Provided that the size of an `   unsigned long  ` is equal to the size of an `   unsigned int  ` , and both sizes are equal to or smaller than the size of `size_t` , this code runs as expected. However, if the size of an `   unsigned long  ` is greater than the size of an `   unsigned int  ` , the value stored in `length` may be truncated when passed as an argument to `alloc()` .  Both read functions return zero on success and nonzero on failure.

### Compliant Solution

Declaring both `length` and the `blocksize` argument to `alloc()` as `size_t` eliminates the possibility of truncation. This compliant solution assumes that `read_integer_from_network()` and `read_network_data()` can also be modified to accept a `length` argument of type pointer to `size_t` and `size_t` , respectively. If these functions are part of an external library that cannot be updated, care must be taken when casting `length` into an `   unsigned long  ` to ensure that integer truncation does not occur.

## INT02-C. Understand integer conversion rules

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/04.int02-c.md) — SHA-256 `c8e8a8369c55e86861e3f0832eca127611e5993f36cfeb540ad4a7963ce55d61`。

Conversions can occur explicitly as the result of a cast or implicitly as required by an operation. Although conversions are generally required for the correct execution of a program, they can also lead to lost or misinterpreted data. Conversion of an operand value to a compatible type causes no change to the value or the representation.

The C integer conversion rules define how C compilers handle conversions. These rules include *integer promotions* , *integer conversion rank* , and the *usual arithmetic conversions* . The intent of the rules is to ensure that the conversions result in the same numerical values and that these values minimize surprises in the rest of the computation. Prestandard C usually preferred to preserve signedness of the type.

### Integer Promotions

Integer types smaller than `int` are promoted when an operation is performed on them. If all values of the original type can be represented as an `int` , the value of the smaller type is converted to an `int` ; otherwise, it is converted to an `   unsigned int  ` . Integer promotions are applied as part of the usual arithmetic conversions to certain argument expressions; operands of the unary `+` , `-` , and `~` operators; and operands of the shift operators. The following code fragment shows the application of integer promotions:

Integer promotions require the promotion of each variable ( `c1` and `c2` ) to `int` size. The two `int` values are added, and the sum is truncated to fit into the `char` type. Integer promotions are performed to avoid arithmetic errors resulting from the overflow of intermediate values:

In this example, the value of `c1` is multiplied by `c2` . The product of these values is then divided by the value of `c3` (according to operator precedence rules). Assuming that `   signed char  ` is represented as an 8-bit value, the product of `c1` and `c2` (300) cannot be represented. Because of integer promotions, however, `c1` , `c2` , and `c3` are each converted to `int` , and the overall expression is successfully evaluated. The resulting value is truncated and stored in `cresult` . Because the final result (75) is in the range of the `   signed char  ` type, the conversion from `int` back to `   signed char  ` does not result in lost data.

### Integer Conversion Rank

Every integer type has an integer conversion rank that determines how conversions are performed. The ranking is based on the concept that each integer type contains at least as many bits as the types ranked below it. The following rules for determining integer conversion rank are defined in the C Standard, subclause 6.3.1.1 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \]:

-   No two signed integer types shall have the same rank, even if they have the same representation.
-   The rank of a signed integer type shall be greater than the rank of any signed integer type with less precision.
-   The rank of `    long long int   ` shall be greater than the rank of `    long int   ` , which shall be greater than the rank of `int` , which shall be greater than the rank of `    short int   ` , which shall be greater than the rank of `    signed char   ` .
-   The rank of any unsigned integer type shall equal the rank of the corresponding signed integer type, if any.
-   The rank of any standard integer type shall be greater than the rank of any extended integer type with the same width.
-   The rank of `char` shall equal the rank of `    signed char   `and`    unsigned char   ` .
-   The rank of `_Bool` shall be less than the rank of all other standard integer types.
-   The rank of any enumerated type shall equal the rank of the compatible integer type.
-   The rank of any extended signed integer type relative to another extended signed integer type with the same precision is [implementation-defined](http://BB.%20Definitions#implementation-defined) but still subject to the other rules for determining the integer conversion rank.
-   For all integer types `T1` , `T2` , and `T3` , if `T1` has greater rank than `T2` and `T2` has greater rank than `T3` , then `T1` has greater rank than `T3` .

The integer conversion rank is used in the usual arithmetic conversions to determine what conversions need to take place to support an operation on mixed integer types.

### Usual Arithmetic Conversions

The usual arithmetic conversions are rules that provide a mechanism to yield a common type when both operands of a binary operator are balanced to a common type or the second and third operands of the conditional operator ( `   ? :  ` ) are balanced to a common type. Conversions involve two operands of different types, and one or both operands may be converted. Many operators that accept arithmetic operands perform conversions using the usual arithmetic conversions. After integer promotions are performed on both operands, the following rules are applied to the promoted operands:

1.  If both operands have the same type, no further conversion is needed.
2.  If both operands are of the same integer type (signed or unsigned), the operand with the type of lesser integer conversion rank is converted to the type of the operand with greater rank.
3.  If the operand that has unsigned integer type has rank greater than or equal to the rank of the type of the other operand, the operand with signed integer type is converted to the type of the operand with unsigned integer type.
4.  If the type of the operand with signed integer type can represent all of the values of the type of the operand with unsigned integer type, the operand with unsigned integer type is converted to the type of the operand with signed integer type.
5.  Otherwise, both operands are converted to the unsigned integer type corresponding to the type of the operand with signed integer type.

### Example

In the following example, assume the code is compiled using an implementation with 8-bit `char` , 32-bit `int` , and 64-bit `   long long  `:

Both the `   signed char sc  ` and the `   unsigned char uc  ` are subject to integer promotions in this example. Because all values of the original types can be represented as `int` , both values are automatically converted to `int` as part of the integer promotions. Further conversions are possible if the types of these variables are not equivalent as a result of the usual arithmetic conversions. The actual addition operation, in this case, takes place between the two 32-bit `int` values. This operation is not influenced by the resulting value being stored in a `   signed long long  ` integer. The 32-bit value resulting from the addition is simply sign-extended to 64 bits after the addition operation has concluded.

Assuming that the precision of `   signed char  ` is 7 bits, and the precision of `   unsigned char  ` is 8 bits, this operation is perfectly safe. However, if the compiler represents the `   signed char  `and`   unsigned char  ` types using 31- and 32-bit precision (respectively), the variable `uc` would need to be converted to `   unsigned int  ` instead of `   signed int  ` . As a result of the usual arithmetic conversions, the `   signed int  ` is converted to unsigned, and the addition takes place between the two `   unsigned int  ` values. Also, because `uc` is equal to `UCHAR_MAX` , which is equal to `UINT_MAX` , the addition results in an overflow in this example. The resulting value is then zero-extended to fit into the 64-bit storage allocated by `sll` .

### Noncompliant Code Example (Comparison)

The programmer must be careful when performing operations on mixed types. This noncompliant code example shows an idiosyncrasy of integer promotions:

In this example, the comparison operator operates on a `   signed int  ` and an `   unsigned int  ` . By the conversion rules, `si` is converted to an `   unsigned int  ` . Because −1 cannot be represented as an `   unsigned int  ` value, the −1 is converted to `UINT_MAX` in accordance with the C Standard, subclause 6.3.1.3, paragraph 2 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \]:

> Otherwise, if the new type is unsigned, the value is converted by repeatedly adding or subtracting one more than the maximum value that can be represented in the new type until the value is in the range of the new type.

Consequently, the program prints 0 because `UINT_MAX` is not less than 1.

### Compliant Solution

The noncompliant code example can be modified to produce the intuitive result by forcing the comparison to be performed using `   signed int  ` values:

This program prints 1 as expected. Note that `(int)ui` is correct in this case only because the value of `ui` is known to be representable as an `int` . If it were not known, the compliant solution would need to be written as

### Noncompliant Code Example

This noncompliant code example demonstrates how performing bitwise operations on integer types smaller than `int` may have unexpected results:

In this example, a bitwise complement of `port` is first computed and then shifted 4 bits to the right. If both of these operations are performed on an 8-bit unsigned integer, then `result_8` will have the value `0x0a` . However, `port` is first promoted to a `   signed int  ` , with the following results (on a typical architecture where type `int` is 32 bits wide):

| Expression | Type | Value | Notes |
| --- | --- | --- | --- |
| `port` | `uint8_t` | `0x5a` |  |
| `~port` | `int` | `0xffffffa5` |  |
| `~port >> 4` | `int` | `0x0ffffffa` | Whether or not value is negative is [implementation-defined](http://BB.%20Definitions#implementation-defined) |
| `result_8` | `uint8_t` | `0xfa` |  |

### Compliant Solution

In this compliant solution, the bitwise complement of `port` is converted back to 8 bits. Consequently, `result_8` is assigned the expected value of `0x0aU` .

### Noncompliant Code Example

In this example, a character is iterated from 0 to `CHAR_MAX` . However, on a platform where char is signed (such as 32-bit x86), `max` is set to `0x80` while `i` increments from `0x79` to `0xffffff80` (aka -127):

### Compliant Solution

There are several ways to rectify this example. One way is to treat both chars as unsigned, which prevents wraparound:

### Noncompliant Code Example

This noncompliant code example, adapted from the [Cryptography Services blog](https://cryptoservices.github.io/fde/2018/11/30/undefined-behavior.html) , demonstrates how signed overflow can occur even when it seems that only unsigned types are in use:

On implementations where `short` is 16 bits wide and `int` is 32 bits wide, the program results in undefined behavior due to signed overflow. This is because the `   unsigned short  ` s become signed when they are automatically promoted to integer, and their mathematical product (2250000000) is greater than the largest signed 32-bit integer (2 <sup>31</sup> - 1, which is 2147483647).

### Compliant Solution

In this compliant solution, by manually casting one of the operands to `   unsigned int  ` , the multiplication will be unsigned and so will not result in undefined behavior:

## INT04-C. Enforce limits on integer values originating from tainted sources

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/05.int04-c.md) — SHA-256 `f2100a51bce9206140bf3b84ac35353011b25a68fa201301693133bcccdca8b7`。

All integer values originating from [tainted sources](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-taintedsource) should be evaluated to determine if they have identifiable upper and lower bounds. If so, these limits should be enforced by the interface. Restricting the input of excessively large or small integers helps prevent overflow, truncation, and other type range errors. Furthermore, it is easier to find and correct input problems than it is to trace internal errors back to faulty inputs.

### Noncompliant Code Example

In this noncompliant code example, `length` is the value of a user-defined (and thus potentially untrusted) environment variable whose value is used to determine the size of a dynamically allocated array, `table` . In compliance with [INT30-C. Ensure that unsigned integer operations do not wrap](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int30-c) , the code prevents [unsigned integer wrapping](http://BB.%20Definitions#unsigned%20integer%20wrapping) but does not impose any upper bound on the size of the array, making it possible for the user to cause the program to use an excessive amount of memory.

Because `length` is user controlled, the value can result in a large block of memory being allocated or can cause the call to `malloc()` to fail. Depending on how error handling is implemented, it may result in a [denial-of-service attack](http://BB.%20Definitions#denial-of-service%20attack) or other error.

### Compliant Solution

This compliant solution defines the acceptable range for `length` as `   [1, MAX_TABLE_LENGTH]  ` . The `length` parameter is declared as `size_t` , which is unsigned by definition. Consequently, it is not necessary to check `length` for negative values (see [INT01-C. Use size_t for all integer values representing the size of an object](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int01-c) ).

The test for `   length == 0  ` ensures that a nonzero number of bytes is allocated. (See [MEM04-C. Beware of zero-length allocations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/memory-management-mem/mem04-c) .)

### Noncompliant Code Example

In this noncompliant example, the tainted integer `color_index` is used in pointer arithmetic to index into the array table:

### Compliant Solution

This compliant solution defines the acceptable range for `color_index` as `   [1, MAX_COLOR_INDEX]  ` :

### Noncompliant Code Example (Heartbleed)

CERT vulnerability [720951](https://www.kb.cert.org/vuls/id/720951) describes a vulnerability in OpenSSL versions 1.0.1 through 1.0.1f, popularly known as "Heartbleed." This vulnerability allows an attacker to steal information that under normal conditions would be protected by Secure Socket Layer/Transport Layer Security (SSL/TLS) encryption.

Despite the seriousness of the vulnerability, Heartbleed is the result of a common programming error and an apparent lack of awareness of secure coding principles. Following is the vulnerable code:

  

This code processes a "heartbeat" packet from a client. As specified in [RFC 6520](https://tools.ietf.org/html/rfc6520) , when the program receives a heartbeat packet, it must echo the packet's data back to the client. In addition to the data, the packet contains a length field that conventionally indicates the number of bytes in the packet data, but there is nothing to prevent a malicious packet from lying about its data length.

The `p` pointer, along with `payload` and `p1` , contain data from a packet. The code allocates a `buffer` sufficient to contain `payload` bytes, with some overhead, then copies `payload` bytes starting at `p1` into this buffer and sends it to the client. Notably absent from this code are any checks that the payload integer variable extracted from the heartbeat packet corresponds to the size of the packet data. Because the client can specify an arbitrary value of `payload` , an attacker can cause the server to read and return the contents of memory beyond the end of the packet data, which violates [INT04-C. Enforce limits on integer values originating from tainted sources](https://www.securecoding.cert.org/confluence/display/seccode/INT04-C.+Enforce+limits+on+integer+values+originating+from+tainted+sources) . The resulting call to `memcpy()` can then copy the contents of memory past the end of the packet data and the packet itself, potentially exposing sensitive data to the attacker. This call to `memcpy()` violates [ARR38-C. Guarantee that library functions do not form invalid pointers](https://www.securecoding.cert.org/confluence/display/seccode/ARR38-C.+Guarantee+that+library+functions+do+not+form+invalid+pointers) . A version of ARR38-C also appears in [ISO/IEC TS 17961:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECTS17961) , "Forming invalid pointers by library functions \[libptr\]." This rule would require a conforming analyzer to diagnose the Heartbleed vulnerability.

  

  

### Compliant Solution (Heartbleed)

OpenSSL version 1.0.1g contains the following patch, which guarantees that `payload` is within a valid range. The range is limited by the size of the input record.

## INT05-C. Do not use input functions to convert character data if they cannot handle all possible inputs

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/06.int05-c.md) — SHA-256 `8e8b2d3168b47ed0101c9099aeab352f7d32fb2baae48ae3fb1dba04739dfcf2`。

Do not use functions that input characters and convert them to integers if the functions cannot handle all possible inputs. For example, formatted input functions such as `scanf()` , `fscanf()` , `vscanf()` , and `vfscanf()` can be used to read string data from `stdin` or (in the cases of `fscanf()` and `vfscanf()` ) other input streams. These functions work fine for valid integer values but lack robust error handling for invalid values.

Alternatively, input character data as a null-terminated byte string and convert to an integer value using `strtol()` or a related function. (See [ERR34-C. Detect errors when converting a string to a number](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err34-c) .)

### Noncompliant Code Example

This noncompliant code example uses the `scanf()` function to read a string from `stdin` and convert it to a `long` . The `scanf()` and `fscanf()` functions have [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) if the value of the result of this operation cannot be represented as an integer.

In general, do not use `scanf()` to parse integers or floating-point numbers from input strings because the input could contain numbers not representable by the argument type.

### Compliant Solution (Linux)

This compliant example uses the Linux `scanf()` implementation's built-in error handling to validate input. On Linux platforms, `scanf()` sets `errno` to `ERANGE` if the result of integer conversion cannot be represented within the size specified by the format string \[ [Linux 2008](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Linux08) \]. Note that this solution is platform dependent, so it should be used only where portability is not a concern.

### Compliant Solution

This compliant example uses `fgets()` to input a string and `strtol()` to convert the string to an integer. Error checking is provided to make sure the value is a valid integer in the range of `long` .

Note that this solution treats any trailing characters, including whitespace characters, as an error condition.

## INT07-C. Use only explicitly signed or unsigned char type for numeric values

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/07.int07-c.md) — SHA-256 `afd080c2c1be6cd40006dcfef978e4128dd4562c8f97321a0f8f2e3b7cf127c2`。

The three types `char` , `   signed char  ` , and `   unsigned char  ` are collectively called the *character types* . Compilers have the latitude to define `char` to have the same range, representation, and behavior as *either* `   signed char  `or`   unsigned char  ` . Irrespective of the choice made, `char` is a separate type from the other two and is *not* compatible with either.

Use only `   signed char  `and`   unsigned char  ` types for the storage and use of numeric values because it is the only portable way to guarantee the signedness of the character types (see [STR00-C. Represent characters using an appropriate type](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/characters-and-strings-str/str00-c) for more information on representing characters).

### Noncompliant Code Example

In this noncompliant code example, the `char` -type variable `c` may be signed or unsigned. Assuming 8-bit, two's complement character types, this code may print out either `   i/c = 5  ` (unsigned) or `   i/c = -17  ` (signed). It is much more difficult to reason about the correctness of a program without knowing if these integers are signed or unsigned.

### Compliant Solution

In this compliant solution, the variable `c` is declared as `   unsigned char  ` . The subsequent division operation is now independent of the signedness of `char` and consequently has a predictable result.

### Exceptions

**INT07-C-EX1:** [void FIO34-C. Use int to capture the return value of character IO functions that might be used to check for end of file](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/the-void/void-fio34-c-use-int-to-capture-the-return-value-of-character-io-functions-that-might-be-used-to-check-for-end-of-file) mentions that certain character IO functions return a value of type `int` . Despite being returned in an arithmetic type, the value is not actually numeric in nature, so it is acceptable to later store the result into a variable of type `char` .

## INT08-C. Verify that all integer values are in range

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/08.int08-c.md) — SHA-256 `11e8931569cf3788227493d2d810c1ba88b298dd50eaf1e9d8ec76058d0940c8`。

Integer operations must result in an integer value within the range of the integer type (that is, the resulting value is the same as the result produced by unlimited-range integers). Frequently, the range is more restrictive depending on the use of the integer value, for example, as an index. Integer values can be verified by code review or by [static analysis](http://BB.%20Definitions#static%20analysis) .

Integer overflow is [undefined behavior](https://www.securecoding.cert.org/confluence/display/seccode/BB.+Definitions) , so a compiled program can do anything, including go off to play the Game of Life. Furthermore, a compiler may perform optimizations that assume an overflow will never occur, which can easily yield unexpected results. Compilers can optimize away `if` statements that check whether an overflow occurred. See [MSC15-C. Do not depend on undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc15-c) for an example.

Verifiably in-range operations are often preferable to treating out-of-range values as an error condition because the handling of these errors has been repeatedly shown to cause [denial-of-service](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-denial-of-serviceattack) problems in actual applications. The quintessential example is the failure of the Ariane 5 launcher, which occurred because of an improperly handled conversion error that resulted in the processor being shut down \[ [ESA 1996](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ESA96) \].

A program that detects an integer overflow to be imminent may do one of two things: (1) signal some sort of error condition or (2) produce an integer result that is within the range of representable integers on that system. Some situations can be handled by an error condition, where an overflow causes a change in control flow (such as the system complaining about bad input and requesting alternative input from the user). Others are better handled by the latter option because it allows the computation to proceed and generate an integer result, thereby avoiding a denial-of-service attack. However, when continuing to produce an integer result in the face of overflow, the question of what integer result to return to the user must be considered.

The saturation and modwrap algorithms and the technique of restricted range usage, defined in the following subsections, produce integer results that are always within a defined range. This range is between the integer values `MIN` and `MAX` (inclusive), where `MIN` and `MAX` are two representable integers with `   MIN < MAX  ` .

### Saturation Semantics

For saturation semantics, assume that the mathematical result of the computation is `result` . The value actually returned to the user is set out in the following table:

|                                     |                       |
|-------------------------------------|-----------------------|
| Range of Mathematical Result        | Result Returned       |
| `       MAX < result      `|`       MAX      `    |
| `       MIN <= result <= MAX      `|`       result      ` |
| `       result < MIN      `|`       MIN      `    |

### Modwrap Semantics

In modwrap semantics (also called *modulo* arithmetic), integer values "wrap round." That is, adding 1 to `MAX` produces `MIN` . This is the defined behavior for unsigned integers in the C Standard, subclause 6.2.5, paragraph 9. It is frequently the behavior of signed integers, as well. However, it is more sensible in many applications to use saturation semantics instead of modwrap semantics. For example, in the computation of a size (using unsigned integers), it is often better for the size to stay at the maximum value in the event of overflow rather than to suddenly become a very small value.

### Restricted Range Usage

Another technique for avoiding integer overflow is to use only half the range of signed integers. For example, when using an `int` , use only the range \[ `INT_MIN/2` , `INT_MAX/2` \]. This practice has been a trick of the trade in Fortran for some time, and now that optimizing C compilers are more sophisticated, it can be valuable in C.

Consider subtraction. If the user types the expression `   a - b  ` , where both `a` and `b` are in the range `   [INT_MIN/2, INT_MAX/2]  ` , the result will be in the range `   (INT_MIN, INT_MAX]  ` for a typical two's complement machine.

Now, if the user types `   a < b  ` , an implicit subtraction often occurs. On a machine without condition codes, the compiler may simply issue a subtract instruction and check whether the result is negative. This behavior is allowed because the compiler is allowed to assume there is no overflow. If all explicitly user-generated values are kept in the range `   [INT_MIN/2, INT_MAX/2]  ` , then comparisons will always work even if the compiler performs this optimization on such hardware.

### Noncompliant Code Example

In this noncompliant example, `   i + 1  ` will overflow on a 16-bit machine. The C Standard allows signed integers to overflow and produce incorrect results. Compilers can take advantage of this to produce faster code by assuming an overflow will not occur. As a result, the `if` statement that is intended to catch an overflow might be optimized away.

### Compliant Solution

Using a `long` instead of an `int` is guaranteed to accommodate the computed value:

## INT09-C. Ensure enumeration constants map to unique values

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/09.int09-c.md) — SHA-256 `6d8f8b57926b1fedca6a8e5233026379fadf62737cbfd792ff5cde2534d07eb1`。

A C enumeration defines a type with a finite set of values represented by identifiers known as *enumeration constants* , or enumerators. An enumerator is a constant integer expression whose value is representable as an `int` . Although the language allows multiple enumerators of the same type to have the same value, it is a common expectation that all enumerators of the same type have distinct values. However, defining two or more enumerators of the same type to have the same value can lead to some nonobvious errors.

### Noncompliant Code Example

In this noncompliant code example, two enumerators of type `Color` are assigned explicit values. It may not be obvious to the programmer that `yellow` and `indigo` have been declared to be identical values (6), as are `green` and `violet` (7). Probably the least dangerous error that can result from such a definition is attempting to use the enumerators as labels of a `switch` statement. Because all labels in a `switch` statement are required to be unique, the following code violates this semantic constraint and is required to be diagnosed by a [conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conforming) compiler:

### Compliant Solution

To prevent the error discussed of the noncompliant code example, enumeration type declarations must take one of the following forms:

-   Provide no explicit integer assignments, as in this example:

-   Assign a value to the first member only (the rest are then sequential), as in this example:

-   Assign a value to all members so any equivalence is explicit, as in this example:

It is also advisable to provide a comment explaining why multiple enumeration type members are being assigned the same value so that future maintainers do not mistakenly identify this form as an error.

Of these three options, providing no explicit integer assignments is the simplest and consequently the preferred approach unless the first enumerator must have a nonzero value.

### Exceptions

<a id="INT09C.Ensureenumerationconstantsmaptouniquevalues-INT09-EX1"></a>  
**INT09-C-EX1:** In cases where defining an enumeration with two or more enumerators with the same value is intended, the constant expression used to define the value of the duplicate enumerator should reference the enumerator rather than the original enumerator's value. This practice makes the intent clear to both human readers of the code and automated code analysis tools that detect violations of this guideline and would diagnose them otherwise. Note, however, that it does not make it possible to use such enumerators in contexts where unique values are required (such as in a `switch` statement, as discussed earlier).

## INT10-C. Do not assume a positive remainder when using the % operator

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/10.int10-c.md) — SHA-256 `6f065eda7add7880d87e888e7145737dcd4202022e6d164f8cf3be37925d2f83`。

In C89 (and historical K&R [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) ), the meaning of the remainder operator for negative operands was [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementationdefinedbehavior) . This behavior was changed in C99, and the change remains in C11.

Because not all C compilers are strictly C-conforming, programmers cannot rely on the behavior of the `%` operator if they need to run on a wide range of platforms with many different compilers.

The C Standard, subclause 6.5.5 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], states:

> The result of the `/` operator is the quotient from the division of the first operand by the second; the result of the `%` operator is the remainder. In both operations, if the value of the second operand is zero, the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

and

> When integers are divided, the result of the `/` operator is the algebraic quotient with any fractional part discarded. If the quotient `a/b` is representable, the expression `    (a/b)*b + a%b   ` shall equal `a` .

Discarding the fractional part of the remainder is often called *truncation toward zero* .

The C definition of the `%` operator implies the following behavior:

The result has the same sign as the dividend (the first operand in the expression).

### Noncompliant Code Example

In this noncompliant code example, the `insert()` function adds values to a buffer in a modulo fashion, that is, by inserting values at the beginning of the buffer once the end is reached. However, both `size` and `index` are declared as `int` and consequently are not guaranteed to be positive. Depending on the [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) and on the sign of `size` and `index` , the result of `   (index + 1) % size  ` may be negative, resulting in a write outside the bounds of the `list` array.

This code also violates [ERR02-C. Avoid in-band error indicators](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err02-c) .

### Noncompliant Code Example

Taking the absolute value of the modulo operation returns a positive value:

However, this noncompliant code example violates [INT01-C. Use size_t for all integer values representing the size of an object](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int01-c) . There is also a possibility that `   (index + 1)  ` could result in a signed integer overflow in violation of [INT32-C. Ensure that operations on signed integers do not result in overflow](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int32-c) .

### Compliant Solution (Unsigned Types)

The most appropriate solution in this case is to use unsigned types to eliminate any possible [implementation-defined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) , as in this compliant solution. For compliance with [ERR02-C. Avoid in-band error indicators](https://www.securecoding.cert.org/confluence/display/seccode/ERR02-C.+Avoid+in-band+error+indicators) , this solution fills a result argument with the mathematical result and returns nonzero only if the operation succeeds.

## INT12-C. Do not make assumptions about the type of a plain int bit-field when used in an expression

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/11.int12-c.md) — SHA-256 `cb57b3c37ae9825af363ef936304d32e78d4fec751f15f29ee28f4ae5da6147b`。

Bit-fields can be used to allow flags or other integer values with small ranges to be packed together to save storage space.

It is [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementationdefinedbehavior) whether the specifier `int` designates the same type as `   signed int  ` or the same type as `   unsigned int  ` for bit-fields. According to the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], C integer promotions also require that "if an `int` can represent all values of the original type (as restricted by the width, for a bit-field), the value is converted to an `int` ; otherwise, it is converted to an `   unsigned int  ` ."

This issue is similar to the signedness of plain `char` , discussed in [INT07-C. Use only explicitly signed or unsigned char type for numeric values](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int07-c) . A plain `int` bit-field that is treated as unsigned will promote to `int` as long as its field width is less than that of `int` because `int` can hold all values of the original type. This behavior is the same as that of a plain `char` treated as unsigned. However, a plain `int` bit-field treated as unsigned will promote to `   unsigned int  ` if its field width is the same as that of `int` . This difference makes a plain `int` bit-field even trickier than a plain `char` .

Bit-field types other than `_Bool` , `int` , `   signed int  ` , and `   unsigned int  ` are implementation-defined. They still obey the integer promotions quoted previously when the specified width is at least as narrow as `CHAR_BIT*sizeof(int)` , but wider bit-fields are not portable.

### Noncompliant Code Example

This noncompliant code depends on [implementation-defined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) . It prints either `-1` or `255` , depending on whether a plain `int` bit-field is signed or unsigned.

### Compliant Solution

This compliant solution uses an `   unsigned int  ` bit-field and does not depend on implementation-defined behavior:

## INT13-C. Use bitwise operators only on unsigned operands

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/12.int13-c.md) — SHA-256 `57c8f3b5f96d5322ed96a7c5b1ae45f035344df66066c8b7f5de0b9b5ac3dd4e`。

Bitwise operators include the complement operator `~` , bitwise shift operators `>>` and `<<` , bitwise AND operator `&` , bitwise exclusive OR operator `^` , bitwise inclusive OR operator `|` and compound assignment operators \>\>=, \<\<=, &=, ^= and \|=. Bitwise operators should be used only with unsigned integer operands, as the results of bitwise operations on signed integers are [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) .

The C11 standard, section 6.5, paragraph 4 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] , states:

> Some operators (the unary operator \~ , and the binary operators \<\<, \>\>, &, ^, and \|, collectively described as bitwise operators) shall have operands that have integral type. These operators return values that depend on the internal representations of integers, and thus have implementation-defined and undefined aspects for signed types.

Furthermore, the bitwise shift operators \<\< and `>>` are undefined under many circumstances, and are implementation-defined for signed integers for more circumstances; see rule [INT34-C. Do not shift an expression by a negative number of bits or by greater than or equal to the number of bits that exist in the operand](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int34-c) for more information.

#### Implementation details

The Microsoft C compiler documentation says that:

> Bitwise operations on signed integers work the same as bitwise operations on unsigned integers.

On-line GCC documentation about the implementation of bitwise operations on signed integers says:

> Bitwise operators act on the representation of the value including both the sign and value bits, where the sign bit is considered immediately above the highest-value value bit.

### Noncompliant Code Example (Right Shift)

The right-shift operation may be implemented as either an arithmetic (signed) shift or a logical (unsigned) shift. If `E1` in the expression `   E1 >> E2  ` has a signed type and a negative value, the resulting value is implementation-defined. Also, a bitwise shift can result in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See [INT34-C. Do not shift an expression by a negative number of bits or by greater than or equal to the number of bits that exist in the operand](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int34-c) .)

This noncompliant code example can result in an error condition on [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) in which an arithmetic shift is performed, and the sign bit is propagated as the number is shifted \[ [Dowd 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Dowd06) \]:

In this example, `   stringify >> 24  ` evaluates to `0xFFFFFF80` , or 4,294,967,168. When converted to a string, the resulting value `"4294967168"` is too large to store in `buf` and is truncated by `snprintf()` .

If this code had been implemented using `sprintf()` instead of `snprintf()` , this noncompliant code example would have resulted in a buffer overflow.

### Compliant Solution (Right Shift)

In this compliant solution, `stringify` is declared as an `unsigned` integer. The value of the result of the right-shift operation is the integral part of the quotient of `stringify` `   / 2 ^ 24  ` :

### Exceptions

**INT13-C-EX1:** When used as bit flags, it is acceptable to use preprocessor macros or enumeration constants as arguments to the `&` and `|` operators even if the value is not explicitly declared as unsigned.

**INT13-C-EX2:** If the right-side operand to a shift operator is known at compile time, it is acceptable for the value to be represented with a signed type provided it is positive.

## INT14-C. Avoid performing bitwise and arithmetic operations on the same data

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/13.int14-c.md) — SHA-256 `808eca08d61a716bf14e9d3ef5867fa4f5fc8d24ea907d88bfe5277235b0c9e8`。

Avoid performing bitwise and arithmetic operations on the same data. In particular, bitwise operations are frequently performed on arithmetic values as a form of premature optimization. Bitwise operators include the unary operator `~` and the binary operators `<<` , `>>` , `&` , `^` , and `|` . Although such operations are valid and will compile, they can reduce code readability. Declaring a variable as containing a numeric value or a bitmap makes the programmer's intentions clearer and the code more maintainable.

Bitmapped types may be defined to further separate bit collections from numeric types. Doing so may make it easier to verify that bitwise operations are performed only on variables that represent bitmaps.

The `typedef` name `uintN_t` designates an unsigned integer type with width `N` . Consequently, `uint32_t` denotes an unsigned integer type with a width of exactly 32 bits. Bitmaps should be declared as unsigned. See [INT13-C. Use bitwise operators only on unsigned operands](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int13-c) .

Left- and right-shift operators are often employed to multiply or divide a number by a power of 2. However, using shift operators to represent multiplication or division is an optimization that renders the code less portable and less readable. Furthermore, most compilers routinely optimize multiplications and divisions by constant powers of 2 with bit-shift operations, and they are more familiar with the implementation details of the current platform.

### Noncompliant Code Example (Left Shift)

In this noncompliant code example, both bit manipulation and arithmetic manipulation are performed on the integer `x` . The result is a (prematurely) optimized statement that assigns `   5x + 1  `to`   x  ` for [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) where integers are represented as two's complement values.

Although this is a valid manipulation, the result of the shift depends on the underlying representation of the integer type and is consequently [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementationdefinedbehavior) . Additionally, the readability of the code is reduced.

### Compliant Solution (Left Shift)

In this compliant solution, the assignment statement is modified to reflect the arithmetic nature of `x` , resulting in a clearer indication of the programmer's intentions:

A reviewer may now recognize that the operation should also be checked for wrapping. This might not have been apparent in the original, noncompliant code example.

### Noncompliant Code Example (Right Shift)

In this noncompliant code example, the programmer prematurely optimizes code by replacing a division with a right shift:

Although this code is likely to perform the division correctly, it is not guaranteed to. If `x` has a signed type and a negative value, the operation is implementation-defined and can be implemented as either an arithmetic shift or a logical shift. In the event of a logical shift, if the integer is represented in either one's complement or two's complement form, the most significant bit (which controls the sign for both representations) will be set to 0, causing a once negative number to become a possibly very large, positive number. For more details, see [INT13-C. Use bitwise operators only on unsigned operands](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int13-c) .

For example, if the internal representation of `x` is `   0xFFFF FFCE  ` (two's complement), an arithmetic shift results in `   0xFFFF FFF3  ` (−13 in two's complement), whereas a logical shift results in `   0x3FFF FFF3  ` (1,073,741,811 in two's complement).

### Compliant Solution (Right Shift)

In this compliant solution, the right shift is replaced by division:

The resulting value is now more likely to be consistent with the programmer's expectations.

### Exceptions

**INT14-C-EX0** : Routines may treat integers as bit vectors for I/O purposes. That is, they may treat an integer as a series of bits in order to write it to a file or socket. They may also read a series of bits from a file or socket and create an integer from the bits. Bitwise operations are also permitted when reading or writing the data from a tightly packed data structure of bytes.

**NUM01-J-EX1:** Bitwise operations may be used to construct constant expressions.

Nevertheless, as a matter of style, it is preferable to replace such constant expressions with the equivalent hexadecimal constants.

## INT15-C. Use intmax_t or uintmax_t for formatted IO on programmer-defined integer types

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/14.int15-c.md) — SHA-256 `a14c3a73077c5359b8095af4b93f886869e130009155586676d23f0fd4f212fa`。

Few programmers consider the issues around formatted I/O and type definitions. A programmer-defined integer type might be any type supported by the [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) , even a type larger than `   unsigned long long  ` . For example, given an implementation that supports 128-bit unsigned integers and provides a `uint_fast128_t` type, a programmer may define the following type:

Furthermore, the definition of programmer-defined types may change, which creates a problem when these types are used with formatted output functions, such as `printf()` , and formatted input functions, such as `scanf()` . (See [FIO47-C. Use valid format strings](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/input-output-fio/fio47-c) .)

The C `intmax_t` and `uintmax_t` types can represent any value representable by any other integer types of the same signedness. (See [INT00-C. Understand the data model used by your implementation(s)](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int00-c) .) This capability allows conversion between programmer-defined integer types (of the same signedness) and `intmax_t` and `uintmax_t` :

Formatted I/O functions can be used to input and output greatest-width integer typed values. The `j` length modifier in a format string indicates that the following `d` , `i` , `o` , `u` , `x` , `X` , or `n` conversion specifier will apply to an argument with type `intmax_t` or `uintmax_t` . C also specifies the `z` length modifier for use with arguments of type `size_t` and the `t` length modifier for arguments of type `ptrdiff_t` .

In addition to programmer-defined types, there is no requirement that an implementation provide format-length modifiers for [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-defined) integer types. For example, a machine with an implementation-defined 48-bit integer type may not provide format-length modifiers for the type. Such a machine still must have a 64-bit `   long long  ` , with `intmax_t` being at least that large.

### Noncompliant Code Example ( `printf()` )

This noncompliant code example prints the value of `x` as an `   unsigned long long  ` value even though the value is of a programmer-defined integer type:

There is no guarantee that this code prints the correct value of `x` , as `x` may be too large to represent as an `   unsigned long long  ` .

### Compliant Solution ( `printf()` )

The C `intmax_t` and `uintmax_t` can be safely used to perform formatted I/O with programmer-defined integer types by converting signed programmer-defined integer types to `intmax_t` and unsigned programmer-defined integer types to `uintmax_t` , then outputting these values using the `j` length modifier. Similarly, programmer-defined integer types can be input to variables of `intmax_t` or `uintmax_t` (whichever matches the signedness of the programmer-defined integer type) and then converted to programmer-defined integer types using appropriate range checks.

This compliant solution guarantees that the correct value of `x` is printed, regardless of its length, provided that `mytypedef_t` is an unsigned type:

### Compliant Solution (Microsoft `printf()` )

Visual Studio 2012 and earlier versions do not support the standard `j` length modifier and do not have a nonstandard analog. Consequently, the programmer must hard code the knowledge that `intmax_t` is `int64_t` and `uintmax_t` is `uint64_t` for Microsoft Visual Studio versions.

A feature request has been submitted to Microsoft to add support for the `j` length modifier to a future release of Microsoft Visual Studio.

### Noncompliant Code Example ( `scanf()` )

This noncompliant code example reads an `   unsigned long long  ` value from standard input and stores the result in `x` , which is of a programmer-defined integer type:

This noncompliant code example can result in a buffer overflow if the size of `mytypedef_t` is smaller than `   unsigned long long  ` , or it might result in an incorrect value if the size of `mytypedef_t` is larger than `   unsigned long long  ` .  Moreover, `scanf()` lacks the error checking capabilities of alternative conversion routines, such as `strtol()` . For more information, see [INT06-C. Use strtol() or a related function to convert a string token to an integer](https://www.securecoding.cert.org/confluence/display/seccode/INT06-C.+Use+strtol%28%29+or+a+related+function+to+convert+a+string+token+to+an+integer) .

### Compliant Solution ( `strtoumax()` )

This compliant solution guarantees that a correct value in the range of `mytypedef_t` is read, or an error condition is detected, assuming the value of `MYTYPEDEF_MAX` is correct as the largest value representable by `mytypedef_t` :  The `strtoumax()` function is used instead of `scanf()` as it provides enhanced error checking functionality.  The `fgets()` function is used to read input from `stdin` .

## INT16-C. Do not make assumptions about representation of signed integers

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/15.int16-c.md) — SHA-256 `75903aa91d08c9145a816a95e3cc7b94b347f85ae68c0d18fc11efc4a13f1fa1`。

Although many common implementations use a two's complement representation of signed integers, the C Standard declares such use as [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) and allows all of the following representations:

-   Sign and magnitude
-   Two's complement
-   One's complement

This is a specific example of [MSC14-C. Do not introduce unnecessary platform dependencies](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc14-c) .

### Noncompliant Code Example

One way to check whether a number is even or odd is to examine the least significant bit, but the results will be inconsistent. Specifically, this example gives unexpected behavior on all one's complement implementations:

### Compliant Solution

The same thing can be achieved compliantly using the modulo operator:

### Compliant Solution

Using bitwise operators is safe on unsigned integers:

## INT17-C. Define integer constants in an implementation-independent manner

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/16.int17-c.md) — SHA-256 `109954aae96d0234ad272ea300e41944dc3f9b0fc3193c38f50e43c2b05b485a`。

Integer constants are often used as masks or specific bit values. Frequently, these constants are expressed in hexadecimal form to indicate to the programmer how the data might be represented in the machine. However, hexadecimal integer constants are frequently used in a nonportable manner.

### Noncompliant Code Example

In this pedagogical noncompliant code example, the `flipbits()` function complements the value stored in `x` by performing a bitwise exclusive OR against a mask with all bits set to 1. For [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) where `   unsigned long  ` is represented by a 32-bit value, each bit of `x` is correctly complemented.

However, on implementations where values of type `   unsigned long  ` are represented by greater than 32 bits, `mask` will have leading 0s. For example, on implementations where values of type `   unsigned long  ` are 64 bits long, `mask` is assigned the value `0x00000000FFFFFFFF` . Consequently, only the lower-order bits of `x` are complemented.

### Compliant Solution (−1)

In this compliant solution, the integer constant `-1` is used to set all bits in `mask` to 1. The integer constant `-1` is of type `   signed int  ` . Because `-1` cannot be represented by a variable of type `   unsigned long  ` , it is converted to a representable number according to the rule in subclause 6.3.1.3, paragraph 2, of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \]:

> \[If the value can't be represented by the new type and\] if the new type is unsigned, the value is converted by repeatedly adding or subtracting one more than the maximum value that can be represented in the new type until the value is in the range of the new type.

"One more than the maximum value that can be represented in the new type," `   ULONG_MAX + 1  ` , is added to `-1` , resulting in a right-side value of `ULONG_MAX` . The representation of `ULONG_MAX` is guaranteed to have all bits set to 1 by subclause 6.2.6.2, paragraph 1:

> For unsigned integer types other than `    unsigned char   ` , the bits of the object representation shall be divided into two groups: value bits and padding bits (there need not be any of the latter). If there are *N* valuebits, each bit shall represent a different power of 2 between 1 and 2 <sup>*N* − 1</sup> , so that objects of that type shall be capable of representing values from 0 to 2 <sup>*N*</sup> − 1 using a **pure binary representation** ; this shall be known as the value representation. The values of any padding bits are unspecified.

By the same reasoning, `-1` is suitable for setting all bits to one of any unsigned integer variable. Subclause 6.2.6.1, paragraph 3, guarantees the same results for `   unsigned char  ` :

> Values stored in unsigned bit-fields and objects of type `    unsigned char   ` shall be represented using a **pure binary notation** .

### Noncompliant Code Example

In this noncompliant code example, a programmer attempts to set the most significant bit:

This code has the desired effect for implementations where `   unsigned long  ` has a precision of 32 bits but not for implementations where `   unsigned long  ` has a precision of 64 bits.

### Compliant Solution

A portable (and safer) way of setting the high-order bit is to use a shift expression, as in this compliant solution:

## INT18-C. Evaluate integer expressions in a larger size before comparing or assigning to that size

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/12.integers-int/17.int18-c.md) — SHA-256 `061040c0caa5644bb5c2fdfc7243b8a6484f5dca61b7a2392e40d05ef783b316`。

If an integer expression involving an operation is compared to or assigned to a larger integer size, that integer expression should be evaluated in that larger size by explicitly casting one of the operands.

### Noncompliant Code Example

This code example is noncompliant on systems where `size_t` is an unsigned 32-bit value and `   long long  ` is a 64-bit value. In this example, the programmer tests for wrapping by comparing `SIZE_MAX` to `   length + BLOCK_HEADER_SIZE  ` . Because `length` is declared as `size_t` , the addition is performed as a 32-bit operation and can result in wrapping. The comparison with `SIZE_MAX` will always test false. If any wrapping occurs, `malloc()` will allocate insufficient space for `mBlock` , which can lead to a subsequent buffer overflow.

Some compilers will diagnose this condition.

### Compliant Solution (Upcast)

In this compliant solution, the `length` operand is upcast to `   unsigned long long  ` , ensuring that the addition takes place in this size:

This test for wrapping is effective only when the `   sizeof(unsigned long long) > sizeof(size_t)  ` . If both `size_t` and `   unsigned long long  ` types are represented as 64-bit unsigned values, the result of the addition operation may not be representable as an `   unsigned long long  ` value.

### Compliant Solution (Rearrange Expression)

In this compliant solution, `length` is subtracted from `SIZE_MAX` , ensuring that wrapping cannot occur. See [INT30-C. Ensure that unsigned integer operations do not wrap](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int30-c) .

### Noncompliant Code Example

In this noncompliant code example, the programmer attempts to prevent wrapping by allocating an `   unsigned long long  ` integer called `alloc` and assigning it the result from `   cBlocks * 16  ` :

Two problems occur in this noncompliant code example. The first problem is that this code assumes an [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) where `   unsigned long long  ` has at least 4 more bits than `size_t` . The second problem, assuming an implementation where `size_t` is a 32-bit value and `   unsigned long long  ` is represented by a 64-bit value, is that to be compliant with C, multiplying two 32-bit numbers in this context must yield a 32-bit result. Any wrapping resulting from this multiplication will remain undetected by this code, and the expression `   alloc < UINT_MAX  ` will always be true.

### Compliant Solution

In this compliant solution, the `cBlocks` operand is upcast to `   unsigned long long  ` , ensuring that the multiplication takes place in this size:

Note that this code does not prevent wrapping unless the `   unsigned long long  ` type is at least 4 bits larger than `size_t` .

### Noncompliant Code Example ( `size_t` )

The `mbstowcs()` function converts a multibyte string to a wide character string, returning the number of characters converted. If an invalid multibyte character is encountered, `mbstowcs()` returns `(size_t)(-1)` . Depending on how `size_t` is implemented, comparing the return value of `mbstowcs()` to signed integer literal `-1` may not evaluate as expected.

### Compliant Solution ( `size_t` )

To ensure the comparison is properly performed, the return value of `mbstowcs()` should be compared against `-1` cast to type `size_t` :

## INT30-C. Ensure that unsigned integer operations do not wrap

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/12.integers-int/2.int30-c.md) — SHA-256 `3b93bde334618f5fcba1b716a963c8987b85c7f263e8ed6f80035c73e008e428`。

The C Standard, 6.2.5, paragraph 11 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> A computation involving unsigned operands can never produce an overflow, because arithmetic for the unsigned type is performed modulo 2^ N .

This behavior is more informally called [unsigned integer wrapping](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unsignedintegerwrapping) . Unsigned integer operations can wrap if the resulting value cannot be represented by the underlying representation of the integer. The following table indicates which operators can result in wrapping:

|                                  |      |                                   |      |                                  |      |                   |      |
|----------------------------------|------|-----------------------------------|------|----------------------------------|------|-------------------|------|
| Operator                         | Wrap | Operator                          | Wrap | Operator                         | Wrap | Operator          | Wrap |
| `+`  | Yes  | `-=`  | Yes  | `<<` | Yes  | `<`  | No   |
| `-`  | Yes  | `*=`  | Yes  | `>>`                | No   | `>`  | No   |
| `*`  | Yes  | `/=`                 | No   | `&`                 | No   | `>=` | No   |
| `/`                 | No   | `%=`                 | No   | `|`                 | No   | `<=` | No   |
| `%`                 | No   | `<<=` | Yes  | `^`                 | No   | `==` | No   |
| `++`                | Yes  | `>>=`                | No   | `~`                 | No   | `!=` | No   |
| `--`                | Yes  | `&=`                 | No   | `!`                 | No   | `&&` | No   |
| `=`                 | No   | `|=`                 | No   | `       un +      `              | No   | `||` | No   |
| `+=` | Yes  | `^=`                 | No   | `       un -      `              | Yes  | `?:` | No   |

  

The following sections examine specific operations that are susceptible to unsigned integer wrap. When operating on integer types with less precision than `int` , integer promotions are applied. The usual arithmetic conversions may also be applied to (implicitly) convert operands to equivalent types before arithmetic operations are performed. Programmers should understand integer conversion rules before trying to implement secure arithmetic operations. (See [INT02-C. Understand integer conversion rules](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int02-c) .)

Integer values must not be allowed to wrap, especially if they are used in any of the following ways:

-   Integer operands of any pointer arithmetic, including array indexing
-   The assignment expression for the declaration of a variable length array
-   The postfix expression preceding square brackets `[]` or the expression in square brackets `[]` of a subscripted designation of an element of an array object
-   Function arguments of type `size_t` or `rsize_t` (for example, an argument to a memory allocation function)
-   In security-critical code

The C Standard defines arithmetic on atomic integer types as read-modify-write operations with the same representation as regular integer types. As a result, wrapping of atomic unsigned integers is identical to regular unsigned integers and should also be prevented or detected.

<a id="INT30C.Ensurethatunsignedintegeroperationsdonotwrap-Addition"></a>

### Addition

Addition is between two operands of arithmetic type or between a pointer to an object type and an integer type. This rule applies only to addition between two operands of arithmetic type. (See [ARR37-C. Do not add or subtract an integer to a pointer to a non-array object](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr37-c) and [ARR30-C. Do not form or use out-of-bounds pointers or array subscripts](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr30-c) .)

Incrementing is equivalent to adding 1.

### Noncompliant Code Example

This noncompliant code example can result in an unsigned integer wrap during the addition of the unsigned operands `ui_a` and `ui_b` . If this behavior is [unexpected](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unexpectedbehavior) , the resulting value may be used to allocate insufficient memory for a subsequent operation or in some other manner that can lead to an exploitable [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) .

### Compliant Solution (Precondition Test)

This compliant solution performs a precondition test of the operands of the addition to guarantee there is no possibility of unsigned wrap:

### Compliant Solution (Postcondition Test)

This compliant solution performs a postcondition test to ensure that the result of the unsigned addition operation `usum` is not less than the first operand:

### Compliant Solution (C23, Checked Integers)

This compliant solution uses the new-to-C23 checked integer arithmetic to safely perform integer addition:

<a id="INT30C.Ensurethatunsignedintegeroperationsdonotwrap-Subtraction"></a>

### Subtraction

Subtraction is between two operands of arithmetic type, two pointers to qualified or unqualified versions of compatible object types, or a pointer to an object type and an integer type. This rule applies only to subtraction between two operands of arithmetic type. (See [ARR36-C. Do not subtract or compare two pointers that do not refer to the same array](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr36-c) , [ARR37-C. Do not add or subtract an integer to a pointer to a non-array object](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr37-c) , and [ARR30-C. Do not form or use out-of-bounds pointers or array subscripts](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr30-c) for information about pointer subtraction.)

Decrementing is equivalent to subtracting 1.

### Noncompliant Code Example

This noncompliant code example can result in an unsigned integer wrap during the subtraction of the unsigned operands `ui_a` and `ui_b` . If this behavior is unanticipated, it may lead to an exploitable [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) .

### Compliant Solution (Precondition Test)

This compliant solution performs a precondition test of the unsigned operands of the subtraction operation to guarantee there is no possibility of unsigned wrap:

### Compliant Solution (Postcondition Test)

This compliant solution performs a postcondition test that the result of the unsigned subtraction operation `udiff` is not greater than the minuend:

### Compliant Solution (C23, Checked Integers)

This compliant solution uses the new-to-C23 checked integer arithmetic to safely perform integer subtraction:

<a id="INT30C.Ensurethatunsignedintegeroperationsdonotwrap-Multiplication"></a>

### Multiplication

Multiplication is between two operands of arithmetic type.

### Noncompliant Code Example

The Mozilla Foundation Security Advisory 2007-01 describes a heap buffer overflow vulnerability in the Mozilla Scalable Vector Graphics (SVG) viewer resulting from an unsigned integer wrap during the multiplication of the `   signed int  `value`   pen->num_vertices  ` and the `size_t` value `sizeof(cairo_pen_vertex_t)` \[ [VU#551436](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-VU551436) \]. The `   signed int  ` operand is converted to `size_t` prior to the multiplication operation so that the multiplication takes place between two `size_t` integers, which are unsigned. (See [INT02-C. Understand integer conversion rules](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int02-c) .)

The unsigned integer wrap can result in allocating memory of insufficient size.

### Compliant Solution

This compliant solution tests the operands of the multiplication to guarantee that there is no unsigned integer wrap:

### Compliant Solution (C23, Checked Integers)

This compliant solution uses the new-to-C23 checked integer arithmetic to safely perform integer multiplication:

### Exceptions

**INT30-C-EX1:** Unsigned integers can exhibit modulo behavior (wrapping) when necessary for the proper execution of the program. It is recommended that the variable declaration be clearly commented as supporting modulo behavior and that each operation on that integer also be clearly commented as supporting modulo behavior.

**INT30-C-EX2:** Checks for wraparound can be omitted when it can be determined at compile time that wraparound will not occur. As such, the following operations on unsigned integers require no validation:

-   Operations on two compile-time constants
-   Operations on a variable and 0 (except division or remainder by 0)
-   Subtracting any variable from its type's maximum; for example, any `    unsigned int   ` may safely be subtracted from `UINT_MAX`
-   Multiplying any variable by 1
-   Division or remainder, as long as the divisor is nonzero. This also excludes the case where the divisor is -1 and the dividend is the minimum integer for its type, such as {{INT_MIN}}.
-   Right-shifting any type maximum by any number less than the type precision; for example, `    UINT_MAX >> x   ` is valid as long as `    0 <=  x < 32   ` (assuming that the precision of `    unsigned int   ` is 32 bits)

<a id="INT30C.Ensurethatunsignedintegeroperationsdonotwrap-LeftShiftOperator"></a>

**INT30-C-EX3.** The left-shift operator takes two operands of integer type. Unsigned left shift `<<` can exhibit modulo behavior (wrapping).  This exception is provided because of common usage, because this behavior is usually expected by the programmer, and because the behavior is well defined. For examples of usage of the left-shift operator, see [INT34-C. Do not shift an expression by a negative number of bits or by greater than or equal to the number of bits that exist in the operand](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int34-c) .

## INT31-C. Ensure that integer conversions do not result in lost or misinterpreted data

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/12.integers-int/3.int31-c.md) — SHA-256 `a8ee6c613f04a6a43d49b92ed323254901b5b18bad6f74a9dd8dd8259bc2bb21`。

Integer conversions, both implicit and explicit (using a cast), must be guaranteed not to result in lost or misinterpreted data. This rule is particularly true for integer values that originate from untrusted sources and are used in any of the following ways:

-   Integer operands of any pointer arithmetic, including array indexing
-   The assignment expression for the declaration of a variable length array
-   The postfix expression preceding square brackets `[]` or the expression in square brackets `[]` of a subscripted designation of an element of an array object
-   Function arguments of type `size_t` or `rsize_t` (for example, an argument to a memory allocation function)

This rule also applies to arguments passed to the following library functions that are converted to `   unsigned char  ` :

-   `memset()`
-   `memset_explicit()`
-   `fprintf()` and related functions (For the length modifier `c` , if no `l` length modifier is present, the `int` argument is converted to an `    unsigned char   ` , and the resulting character is written.)
-   `fputc()`
-   `ungetc()`
-   `memchr()`

and to arguments to the following library functions that are converted to `char` :

-   `strchr()`
-   `strrchr()`
-   All of the functions listed in `<ctype.h>`

The only integer type conversions that are guaranteed to be safe for all data values and all possible conforming implementations are conversions of an integral value to a wider type of the same signedness.

The C Standard, subclause 6.3.1.3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECTR24731-2-2024) \], says

> When a value with integer type is converted to another integer type other than `_Bool` , if the value can be represented by the new type, it is unchanged.
>
> Otherwise, if the new type is unsigned, the value is converted by repeatedly adding or subtracting one more than the maximum value that can be represented in the new type until the value is in the range of the new type.
>
> Otherwise, the new type is signed and the value cannot be represented in it; either the result is [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) or an implementation-defined signal is raised.

Typically, converting an integer to a smaller type results in truncation of the high-order bits.

### Noncompliant Code Example (Unsigned to Signed)

Type range errors, including loss of data (truncation) and loss of sign (sign errors), can occur when converting from a value of an unsigned integer type to a value of a signed integer type. This noncompliant code example results in a truncation error on most [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) :

### Compliant Solution (Unsigned to Signed)

Validate ranges when converting from an unsigned type to a signed type. This compliant solution can be used to convert a value of `   unsigned long int  ` type to a value of `   signed char  ` type:

### Noncompliant Code Example (Signed to Unsigned)

Type range errors, including loss of data (truncation) and loss of sign (sign errors), can occur when converting from a value of a signed type to a value of an unsigned type. This noncompliant code example results in a negative number being misinterpreted as a large positive number.

### Compliant Solution (Signed to Unsigned)

Validate ranges when converting from a signed type to an unsigned type. This compliant solution converts a value of a `   signed int  ` type to a value of an `   unsigned int  ` type:

Subclause 6.2.5, paragraph 11, of the C Standard \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECTR24731-2-2024) \] provides the necessary guarantees to ensure this solution works on a [conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conformingprogram) [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) :

> The range of nonnegative values of a signed integer type is a subrange of the corresponding unsigned integer type, and the representation of the same value in each type is the same.

### Noncompliant Code Example (Signed, Loss of Precision)

A loss of data (truncation) can occur when converting from a value of a signed integer type to a value of a signed type with less precision. This noncompliant code example results in a truncation error on most [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) :

### Compliant Solution (Signed, Loss of Precision)

Validate ranges when converting from a signed type to a signed type with less precision. This compliant solution converts a value of a `   signed long int  ` type to a value of a `   signed char  ` type:

Conversions from a value of a signed integer type to a value of a signed integer type with less precision requires that both the upper and lower bounds are checked.

### Noncompliant Code Example (Unsigned, Loss of Precision)

A loss of data (truncation) can occur when converting from a value of an unsigned integer type to a value of an unsigned type with less precision. This noncompliant code example results in a truncation error on most [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) :

### Compliant Solution (Unsigned, Loss of Precision)

Validate ranges when converting a value of an unsigned integer type to a value of an unsigned integer type with less precision. This compliant solution converts a value of an `   unsigned long int  ` type to a value of an `   unsigned char  ` type:

Conversions from unsigned types with greater precision to unsigned types with less precision require only the upper bounds to be checked.

### Noncompliant Code Example ( `time_t` Return Value)

The `time()` function returns the value `(time_t)(-1)` to indicate that the calendar time is not available. The C Standard requires that the `time_t` type is only a *real type* capable of representing time. (The integer and real floating types are collectively called real types.) It is left to the implementor to decide the best real type to use to represent time. If `time_t` is implemented as an unsigned integer type with less precision than a signed `int` , the return value of `time()` will never compare equal to the integer literal `-1` .

### Compliant Solution ( `time_t` Return Value)

To ensure the comparison is properly performed, the return value of `time()` should be compared against `-1` cast to type `time_t` :

This solution is in accordance with [INT18-C. Evaluate integer expressions in a larger size before comparing or assigning to that size](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int18-c) . Note that `(time_+t)-1` also complies with **INT31-C-EX3** .

### Noncompliant Code Example ( `memset()` )

For historical reasons, certain C Standard functions accept an argument of type `int` and convert it to either `   unsigned char  ` or plain `char` . This conversion can result in unexpected behavior if the value cannot be represented in the smaller type. The second argument to `memset()` is an example; it indicates what byte to store in the range of memory indicated by the first and third arguments.  If the second argument is outside the range of a `   signed char  ` or plain `char` , then its higher order bits will typically be truncated. Consequently, this noncompliant solution unexpectedly sets all elements in the array to 0, rather than 4096:

### Compliant Solution ( memset() )

In general, the `memset()` function should not be used to initialize an integer array unless it is to set or clear all the bits, as in this compliant solution:

### Exceptions

**INT31-C-EX1:** The C Standard defines minimum ranges for standard integer types. For example, the minimum range for an object of type `   unsigned short int  ` is 0 to 65,535, whereas the minimum range for `int` is −32,767 to +32,767. Consequently, it is not always possible to represent all possible values of an `   unsigned short int  ` as an `int` . However, on the IA-32 architecture, for example, the actual integer range is from −2,147,483,648 to +2,147,483,647, meaning that it is quite possible to represent all the values of an `   unsigned short int  ` as an `int` for this architecture. As a result, it is not necessary to provide a test for this conversion on IA-32. It is not possible to make assumptions about conversions without knowing the precision of the underlying types. If these tests are not provided, assumptions concerning precision must be clearly documented, as the resulting code cannot be safely ported to a system where these assumptions are invalid. A good way to document these assumptions is to use static assertions. (See [DCL03-C. Use a static assertion to test the value of a constant expression](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl03-c) .)

**INT31-C-EX2:** Conversion from any integer type with a value between `SCHAR_MIN` and `UCHAR_MAX` to a character type is permitted provided the value represents a character and not an integer.

Conversions to unsigned character types are well defined by C to have modular behavior. A character's value is not misinterpreted by the loss of sign or conversion to a negative number. For example, the Euro symbol `€` is sometimes represented by bit pattern `0x80` which can have the numerical value 128 or −127 depending on the signedness of the type.

Conversions to signed character types are more problematic.

The C Standard, subclause 6.3.1.3, paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECTR24731-2-2010) \], says, regarding conversions

> Otherwise, the new type is signed and the value cannot be represented in it; either the result is implementation-defined or an implementation-defined signal is raised.

Furthermore, subclause 6.2.6.2, paragraph 2, says, regarding integer modifications

> Each bit that is a value bit shall have the same value as the same bit in the object representation of the corresponding unsigned type. If the sign bit is zero, it shall not affect the resulting value. If the sign bit is one, it has value −(2N−1). There may or may not be any padding bits signed char shall not have any padding bits.

  

Consequently, the standard allows for this code to trap:

However, platforms where this code traps or produces an unexpected value are rare. According to *[The New C Standard: An Economic and Cultural Commentary](http://www.knosof.co.uk/cbook/cbook.html)* by Derek Jones \[ [Jones 2008](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Jones08) \],

> Implementations with such trap representations are thought to have existed in the past. Your author was unable to locate any documents describing such processors.

**INT31-C-EX3:** The C Standard, subclause 7.29.2.5, paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECTR24731-2-2010) \] says:

> The time function returns the implementation’s best approximation to the current calendar time. The value ( time \_ t )(-1) is returned if the calendar time is not available.

If `time_t` is an unsigned type, then the expression `   ((time_t) (-1))  ` is guaranteed to yield a large positive value.

Therefore, conversion of a negative compile-time constant to an unsigned value with the same or larger width is permitted by this rule. This exception does not apply to conversion of unsigned to signed values, nor does it apply if the resulting value would undergo truncation.

## INT32-C. Ensure that operations on signed integers do not result in overflow

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/12.integers-int/4.int32-c.md) — SHA-256 `3cbb6542e031ce18973d2f49fde7c5bdf5eea2b0c1537914ed3b0e3308839ba8`。

Signed integer overflow is [undefined behavior 35](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . Consequently, [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) have considerable latitude in how they deal with signed integer overflow. (See [MSC15-C. Do not depend on undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc15-c) .) An implementation that defines signed integer types as being modulo, for example, need not detect integer overflow. Implementations may also trap on signed arithmetic overflows, or simply assume that overflows will never happen and generate object code accordingly.  It is also possible for the same conforming implementation to emit code that exhibits different behavior in different contexts. For example, an implementation may determine that a signed integer loop control variable declared in a local scope cannot overflow and may emit efficient code on the basis of that determination, while the same implementation may determine that a global variable used in a similar context will wrap.

For these reasons, it is important to ensure that operations on signed integers do not result in overflow. Of particular importance are operations on signed integer values that originate from a [tainted source](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-taintedsource) and are used as

-   Integer operands of any pointer arithmetic, including array indexing
-   The assignment expression for the declaration of a variable length array
-   The postfix expression preceding square brackets `[]` or the expression in square brackets `[]` of a subscripted designation of an element of an array object
-   Function arguments of type `size_t` or `rsize_t` (for example, an argument to a memory allocation function)

Integer operations will overflow if the resulting value cannot be represented by the underlying representation of the integer. The following table indicates which operations can result in overflow.

|                   |          |                    |          |                        |          |                   |          |
|-------------------|----------|--------------------|----------|------------------------|----------|-------------------|----------|
| Operator          | Overflow | Operator           | Overflow | Operator               | Overflow | Operator          | Overflow |
| `+`  | Yes      | `-=`  | Yes      | `<<`      | Yes      | `<`  | No       |
| `-`  | Yes      | `*=`  | Yes      | `>>`      | No       | `>`  | No       |
| `*`  | Yes      | `/=`  | Yes      | `&`       | No       | `>=` | No       |
| `/`  | Yes      | `%=`  | Yes      | `|`       | No       | `<=` | No       |
| `%`  | Yes      | `<<=` | Yes      | `^`       | No       | `==` | No       |
| `++` | Yes      | `>>=` | No       | `~`       | No       | `!=` | No       |
| `--` | Yes      | `&=`  | No       | `!`       | No       | `&&` | No       |
| `=`  | No       | `|=`  | No       | `       unary +      ` | No       | `||` | No       |
| `+=` | Yes      | `^=`  | No       | `       unary -      ` | Yes      | `?:` | No       |

The following sections examine specific operations that are susceptible to integer overflow. When operating on integer types with less precision than `int` , integer promotions are applied. The usual arithmetic conversions may also be applied to (implicitly) convert operands to equivalent types before arithmetic operations are performed. Programmers should understand integer conversion rules before trying to implement secure arithmetic operations. (See [INT02-C. Understand integer conversion rules](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int02-c) .)

### Implementation Details

GNU GCC invoked with the `-fwrapv` command-line option defines the same modulo arithmetic for both unsigned and signed integers.

GNU GCC invoked with the `-ftrapv` command-line option causes a trap to be generated when a signed integer overflows, which will most likely abnormally exit. On a UNIX system, the result of such an event may be a signal sent to the process.

GNU GCC invoked without either the `-fwrapv` or the `-ftrapv` option may simply assume that signed integers never overflow and may generate object code accordingly.

### Atomic Integers

The C Standard defines the behavior of arithmetic on atomic signed integer types to use two's complement representation with silent wraparound on overflow; there are no undefined results. Although defined, these results may be unexpected and therefore carry similar risks to [unsigned integer wrapping](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unsignedintegerwrapping) . (See [INT30-C. Ensure that unsigned integer operations do not wrap](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int30-c) .) Consequently, signed integer overflow of atomic integer types should also be prevented or detected.

### Addition

Addition is between two operands of arithmetic type or between a pointer to an object type and an integer type. This rule applies only to addition between two operands of arithmetic type. (See [ARR37-C. Do not add or subtract an integer to a pointer to a non-array object](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr37-c) and [ARR30-C. Do not form or use out-of-bounds pointers or array subscripts](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr30-c) .)

Incrementing is equivalent to adding 1.

### Noncompliant Code Example

This noncompliant code example can result in a signed integer overflow during the addition of the signed operands `si_a` and `si_b` :

### Compliant Solution

This compliant solution ensures that the addition operation cannot overflow, regardless of representation:

### Compliant Solution (C23, Checked Integers)

This compliant solution uses the new-to-C23 checked integer arithmetic to safely perform integer addition:

### Subtraction

Subtraction is between two operands of arithmetic type, two pointers to qualified or unqualified versions of compatible object types, or a pointer to an object type and an integer type. This rule applies only to subtraction between two operands of arithmetic type. (See [ARR36-C. Do not subtract or compare two pointers that do not refer to the same array](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr36-c) , [ARR37-C. Do not add or subtract an integer to a pointer to a non-array object](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr37-c) , and [ARR30-C. Do not form or use out-of-bounds pointers or array subscripts](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr30-c) for information about pointer subtraction.)

Decrementing is equivalent to subtracting 1.

### Noncompliant Code Example

This noncompliant code example can result in a signed integer overflow during the subtraction of the signed operands `si_a` and `si_b` :

### Compliant Solution

This compliant solution tests the operands of the subtraction to guarantee there is no possibility of signed overflow, regardless of representation:

### Compliant Solution (C23, Checked Integers)

This compliant solution uses the new-to-C23 checked integer arithmetic to safely perform integer subtraction:

### Multiplication

Multiplication is between two operands of arithmetic type.

### Noncompliant Code Example

This noncompliant code example can result in a signed integer overflow during the multiplication of the signed operands `si_a` and `si_b` :

### Compliant Solution

The product of two operands can always be represented using twice the number of bits than exist in the precision of the larger of the two operands. This compliant solution eliminates signed overflow on systems where `   long long  ` is at least twice the precision of `int` :

The assertion fails if `   long long  ` has less than twice the precision of `int` . The `PRECISION()` macro and `popcount()` function provide the correct precision for any integer type. (See [INT35-C. Use correct integer precisions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int35-c) .)  

### Compliant Solution

The following portable compliant solution can be used with any conforming implementation, including those that do not have an integer type that is at least twice the precision of `int` :

### Compliant Solution (C23, Checked Integers)

This compliant solution uses the new-to-C23 checked integer arithmetic to safely perform integer subtraction:

### Division

Division is between two operands of arithmetic type. Overflow can occur during two's complement signed integer division when the dividend is equal to the minimum (negative) value for the signed integer type and the divisor is equal to `−1` . Division operations are also susceptible to divide-by-zero errors. (See [INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int33-c) .)

### Noncompliant Code Example

This noncompliant code example prevents divide-by-zero errors in compliance with [INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int33-c) but does not prevent a signed integer overflow error in two's-complement.

#### Implementation Details

On the x86-32 architecture, overflow results in a fault, which can be exploited as a [denial-of-service attack](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-denial-of-service) .

### Compliant Solution

This compliant solution eliminates the possibility of divide-by-zero errors or signed overflow:

### Remainder

The remainder operator provides the remainder when two operands of integer type are divided. Because many platforms implement remainder and division in the same instruction, the remainder operator is also susceptible to arithmetic overflow and division by zero. (See [INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int33-c) .)

### Noncompliant Code Example

Many hardware architectures implement remainder as part of the division operator, which can overflow. Overflow can occur during a remainder operation when the dividend is equal to the minimum (negative) value for the signed integer type and the divisor is equal to −1. It occurs even though the result of such a remainder operation is mathematically 0. This noncompliant code example prevents divide-by-zero errors in compliance with [INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int33-c) but does not prevent integer overflow:

#### Implementation Details

On x86-32 platforms, the remainder operator for signed integers is implemented by the `idiv` instruction code, along with the divide operator. Because `   LONG_MIN / −1  ` overflows, it results in a software exception with `   LONG_MIN % −1  ` as well.

### Compliant Solution

This compliant solution also tests the remainder operands to guarantee there is no possibility of an overflow:

### Left-Shift Operator

The left-shift operator takes two integer operands. The result of `   E1 << E2  `is`   E1  `left-shifted`   E2  ` bit positions; vacated bits are filled with zeros.

The C Standard, 6.5.8, paragraph 4 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> If `E1` has a signed type and nonnegative value, and `    E1 × 2         E2       ` is representable in the result type, then that is the resulting value; otherwise, the behavior is undefined.

In almost every case, an attempt to shift by a negative number of bits or by more bits than exist in the operand indicates a logic error. These issues are covered by [INT34-C. Do not shift an expression by a negative number of bits or by greater than or equal to the number of bits that exist in the operand](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int34-c) .

### Noncompliant Code Example

This noncompliant code example performs a left shift, after verifying that the number being shifted is not negative, and the number of bits to shift is valid.  The `PRECISION()` macro and `popcount()` function provide the correct precision for any integer type. (See [INT35-C. Use correct integer precisions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int35-c) .) However, because this code does no overflow check, it can result in an unrepresentable value.

### Compliant Solution

This compliant solution eliminates the possibility of overflow resulting from a left-shift operation:

### Unary Negation

The unary negation operator takes an operand of arithmetic type. Overflow can occur during two's complement unary negation when the operand is equal to the minimum (negative) value for the signed integer type.

### Noncompliant Code Example

This noncompliant code example can result in a signed integer overflow during the unary negation of the signed operand `s_a` :

### Compliant Solution

This compliant solution tests the negation operation to guarantee there is no possibility of signed overflow:

## INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/12.integers-int/5.int33-c.md) — SHA-256 `33d5abaf2361642bdd23264de491327a6158602a62b9c017b36656f387f4269f`。

The C Standard identifies the following condition under which division and remainder operations result in [undefined behavior (UB)](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) :

|                                                                                                |                                                                                                       |
|------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------|
| UB                                                                                             | Description                                                                                           |
| [41](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_41) | The value of the second operand of the `/` or `%` operator is zero (6.5.5). |

Ensure that division and remainder operations do not result in divide-by-zero errors.

### Division

The result of the `/` operator is the quotient from the division of the first arithmetic operand by the second arithmetic operand. Division operations are susceptible to divide-by-zero errors. Overflow can also occur during two's complement signed integer division when the dividend is equal to the minimum (most negative) value for the signed integer type and the divisor is equal to `−1.` (See [INT32-C. Ensure that operations on signed integers do not result in overflow](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int32-c) .)

### Noncompliant Code Example

This noncompliant code example prevents signed integer overflow in compliance with [INT32-C. Ensure that operations on signed integers do not result in overflow](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int32-c) but fails to prevent a divide-by-zero error during the division of the signed operands `s_a` and `s_b` : `     `

### Compliant Solution

This compliant solution tests the division operation to guarantee there is no possibility of divide-by-zero errors or signed overflow:

### Remainder

The remainder operator provides the remainder when two operands of integer type are divided.

### Noncompliant Code Example

This noncompliant code example prevents signed integer overflow in compliance with [INT32-C. Ensure that operations on signed integers do not result in overflow](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int32-c) but fails to prevent a divide-by-zero error during the remainder operation on the signed operands `s_a` and `s_b` :

### Compliant Solution

This compliant solution tests the remainder operand to guarantee there is no possibility of a divide-by-zero error or an overflow error:

## INT34-C. Do not shift an expression by a negative number of bits or by greater than or equal to the number of bits that exist in the operand

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/12.integers-int/6.int34-c.md) — SHA-256 `a8aedc0ecc0781c97f555396861d36d79192134db4f94656649d965e7b2330ff`。

Bitwise shifts include left-shift operations of the form *shift-expression* `<<` *additive-expression* and right-shift operations of the form *shift-expression* `>>` *additive-expression* . The standard integer promotions are first performed on the operands, each of which has an integer type. The type of the result is that of the promoted left operand. If the value of the right operand is negative or is greater than or equal to the width of the promoted left operand, the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See [undefined behavior 48](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_48) .)

Do not shift an expression by a negative number of bits or by a number greater than or equal to the *precision* of the promoted left operand. The precision of an integer type is the number of bits it uses to represent values, excluding any sign and padding bits. For unsigned integer types, the width and the precision are the same; whereas for signed integer types, the width is one greater than the precision. This rule uses precision instead of width because, in almost every case, an attempt to shift by a number of bits greater than or equal to the precision of the operand indicates a bug (logic error). A logic error is different from overflow, in which there is simply a representational deficiency.  In general, shifts should be performed only on unsigned operands. (See [INT13-C. Use bitwise operators only on unsigned operands](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int13-c) .)

### Noncompliant Code Example (Left Shift, Unsigned Type)

The result of `   E1 << E2  `is`   E1  `left-shifted`   E2  ` bit positions; vacated bits are filled with zeros. The following diagram illustrates the left-shift operation.

According to the C Standard, if `E1` has an unsigned type, the value of the result is `E1` \* `2` `E2` , reduced modulo 1 more than the maximum value representable in the result type.

This noncompliant code example fails to ensure that the right operand is less than the precision of the promoted left operand:

### Compliant Solution (Left Shift, Unsigned Type)

This compliant solution eliminates the possibility of shifting by greater than or equal to the number of bits that exist in the precision of the left operand:

The `PRECISION()` macro and `popcount()` function provide the correct precision for any integer type. (See [INT35-C. Use correct integer precisions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int35-c) .)

Modulo behavior resulting from left-shifting an unsigned integer type is permitted by exception INT30-EX3 to [INT30-C. Ensure that unsigned integer operations do not wrap](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int30-c) .

### Noncompliant Code Example (Left Shift, Signed Type)

The result of `   E1 << E2  `is`   E1  `left-shifted`   E2  ` bit positions; vacated bits are filled with zeros. If `E1` has a signed type and nonnegative value, and `E1` \* `2` `E2` is representable in the result type, then that is the resulting value; otherwise, the behavior is undefined (See [undefined behavior 48](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_48).)

This noncompliant code example fails to ensure that left and right operands have nonnegative values and that the right operand is less than the precision of the promoted left operand. This example does check for signed integer overflow in compliance with [INT32-C. Ensure that operations on signed integers do not result in overflow](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int32-c).

Shift operators and other bitwise operators should be used only with unsigned integer operands in accordance with [INT13-C. Use bitwise operators only on unsigned operands](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int13-c) .

### Compliant Solution (Left Shift, Signed Type)

In addition to the check for overflow, this compliant solution ensures that both the left and right operands have nonnegative values and that the right operand is less than the precision of the promoted left operand:

### Noncompliant Code Example (Right Shift)

The result of `   E1 >> E2  `is`   E1  `right-shifted`   E2  ` bit positions. If `E1` has an unsigned type or if `E1` has a signed type and a nonnegative value, the value of the result is the integral part of the quotient of `E1` / `2` `E2` . If `E1` has a signed type and a negative value, the resulting value is [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) and can be either an arithmetic (signed) shift

or a logical (unsigned) shift

This noncompliant code example fails to test whether the right operand is greater than or equal to the precision of the promoted left operand, allowing undefined behavior (See [undefined behavior 48](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_48)):

When working with signed operands, making assumptions about whether a right shift is implemented as an arithmetic (signed) shift or a logical (unsigned) shift can also lead to [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability). (See [INT13-C. Use bitwise operators only on unsigned operands](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int13-c).)

### Compliant Solution (Right Shift)

This compliant solution eliminates the possibility of shifting by greater than or equal to the number of bits that exist in the precision of the left operand:

#### Implementation Details

GCC has no options to handle shifts by negative amounts or by amounts outside the width of the type predictably or to trap on them; they are always treated as undefined. Processors may reduce the shift amount modulo the width of the type. For example, 32-bit right shifts are implemented using the following instruction on x86-32:

The `sarl` instruction takes a bit mask of the least significant 5 bits from `%cl` to produce a value in the range \[0, 31\] and then shift `%eax` that many bits:

where `%eax` stores the least significant bits in the doubleword to be shifted, and `%edx` stores the most significant bits.

## INT35-C. Use correct integer precisions

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/12.integers-int/7.int35-c.md) — SHA-256 `da7faf5c216c7735777e509dba3ab51169b3342bcbdcc9696b25629450182f67`。

Integer types in C have both a *size* and a *precision* . The size indicates the number of bytes used by an object and can be retrieved for any object or type using the `sizeof` operator.  The precision of an integer type is the number of bits it uses to represent values, excluding any sign and padding bits.

Padding bits contribute to the integer's size, but not to its precision. Consequently, inferring the precision of an integer type from its size may result in too large a value, which can then lead to incorrect assumptions about the numeric range of these types.  Programmers should use correct integer precisions in their code, and in particular, should not use the `sizeof` operator to compute the precision of an integer type on architectures that use padding bits or in strictly conforming (that is, portable) programs.

### Noncompliant Code Example

This noncompliant code example illustrates a function that produces 2 raised to the power of the function argument. To prevent undefined behavior (See [undefined behavior 48](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_48) .) in compliance with [INT34-C. Do not shift an expression by a negative number of bits or by greater than or equal to the number of bits that exist in the operand](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int34-c) , the function ensures that the argument is less than the number of bits used to store a value of type `unsigned int`.

However, if this code runs on a platform where `unsigned int` has one or more padding bits, it can still result in values for `exp` that are too large. For example, on a platform that stores `unsigned int` in 64 bits, but uses only 48 bits to represent the value, a left shift of 56 bits would result in undefined behavior (See [undefined behavior 48](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_48).)

### Compliant Solution ( `popcount()` )

This compliant solution uses a `popcount()` function, which counts the number of bits set on any unsigned integer, allowing this code to determine the precision of any integer type, signed or unsigned.

Implementations can replace the `PRECISION()` macro with a type-generic macro that returns an integer constant expression that is the precision of the specified type for that implementation. This return value can then be used anywhere an integer constant expression can be used, such as in a static assertion. (See [DCL03-C. Use a static assertion to test the value of a constant expression](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl03-c) .) The following type generic macro, for example, might be used for a specific implementation targeting the IA-32 architecture:

The revised version of the `pow2()` function uses the `PRECISION()` macro to determine the precision of the unsigned type:

#### Implementation Details

Some platforms, such as the Cray Linux Environment (CLE; supported on Cray XT CNL compute nodes), provide `   a _popcnt  ` instruction that can substitute for the `popcount()` function.

### Compliant Solution (C23)

The C23 standard provides various `*_WIDTH` macros that define the number of width bits for each integer type. This is effectively the size of the type (multiplied by 8) less the number of padding bits. The following compliant solution uses the `UINT_WIDTH` type to obtain the width of an unsigned int.

## INT36-C. Converting a pointer to integer or integer to pointer

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/12.integers-int/8.int36-c.md) — SHA-256 `eca2980da232cf777152e7287157625a38330bc16c13592dcdd4ad7685e2b715`。

Although programmers often use integers and pointers interchangeably in C, pointer-to-integer and integer-to-pointer conversions are [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) .

Conversions between integers and pointers can have undesired consequences depending on the [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) .

According to the C Standard, subclause 6.3.2.3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \],

> An integer may be converted to any pointer type. Except as previously specified, the result is implementation-defined, may not be correctly aligned, may not point to an entity of the referenced type, and can produce an indeterminate representation when stored into an object.

> Any pointer type may be converted to an integer type. Except as previously specified, the result is implementation-defined. If the result cannot be represented in the integer type, the behavior is undefined. The result is not required to be in the range of values of any integer type.

Do not convert an integer type to a pointer type if the resulting pointer is incorrectly aligned, does not point to an entity of the referenced type, or is a [non-value representation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-nonvaluerepresentation) .

Do not convert a pointer type to an integer type if the result cannot be represented in the integer type. (See [undefined behavior 23](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_23) .)

The mapping between pointers and integers must be consistent with the addressing structure of the execution environment. Issues may arise, for example, on architectures that have a segmented memory model.

### Noncompliant Code Example

The size of a pointer can be greater than the size of an integer, such as in an implementation where pointers are 64 bits and unsigned integers are 32 bits. This code example is noncompliant on such implementations because the result of converting the 64-bit `ptr` cannot be represented in the 32-bit integer type:

### Compliant Solution

Any valid pointer to `void` can be converted to `intptr_t` or `uintptr_t` and back with no change in value. (See **INT36-EX2** .) The C Standard guarantees that a pointer to `void` may be converted to or from a pointer to any object type and back again and that the result must compare equal to the original pointer. Consequently, converting directly from a `   char *  ` pointer to a `uintptr_t` , as in this compliant solution, is allowed on implementations that support the `uintptr_t` type.

### Noncompliant Code Example

In this noncompliant code example, the pointer `ptr` is converted to an integer value. The high-order 9 bits of the number are used to hold a flag value, and the result is converted back into a pointer. This example is noncompliant on an implementation where pointers are 64 bits and unsigned integers are 32 bits because the result of converting the 64-bit `ptr` cannot be represented in the 32-bit integer type.

A similar scheme was used in early versions of Emacs, limiting its portability and preventing the ability to edit files larger than 8MB.

### Compliant Solution

This compliant solution uses a `struct` to provide storage for both the pointer and the flag value. This solution is portable to machines of different word sizes, both smaller and larger than 32 bits, working even when pointers cannot be represented in any integer type.

### Noncompliant Code Example

It is sometimes necessary to access memory at a specific location, requiring a literal integer to pointer conversion. For example, in this noncompliant code a pointer is set directly to an integer constant:

Use of integer literals as memory addresses in ISO C, like this example, is [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior). This makes the code nonportable and on some platforms the result might not be correctly aligned, might not point to an entity of the referenced type, and might be a [non-value representation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-nonvaluerepresentation). This makes every integer literal use as memory address a security concern.

### Compliant Solution

Unfortunately, use of an integer literal to access a specific memory location cannot be made portable and safe while strictly conforming to ISO C.

A particular platform (that is, hardware, operating system, compiler, and Standard C library) might guarantee that a memory address is correctly aligned for the pointer type and actually contains a value for that type. When using an integer literal as a memory address, due care must be taken to ensure proper use for every platform that the code will execute on. For example, a common practice is to ensure that integer literals access memory addresses that are known to point to hardware that provides valid values for the specific targeted platforms. Another consideration is to annotate that the code is for specific targeted platforms and is nonportable.

### Exceptions

**INT36-C-EX1:** The integer value 0 can be converted to a pointer; it becomes the null pointer.

**INT36-C-EX2:** Any valid pointer to `void` can be converted to `intptr_t` or `uintptr_t` or their underlying types and back again with no change in value. Use of underlying types instead of `intptr_t` or `uintptr_t` is discouraged, however, because it limits portability.

**INT36-C-EX3:** An integer may be converted to a `void*` and back as long as the pointer is not dereferenced, and the integer is in range (that is, the appropriate range for an `intptr_t` or `uintptr_t` ).

The following POSIX code passes an integer, cast as a `void*` to a thread, and the thread prints the integer.
