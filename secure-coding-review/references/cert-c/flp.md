# CERT C — FLP

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## FLP00-C. Understand the limitations of floating-point numbers

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/10.floating-point-flp/2.flp00-c.md) — SHA-256 `0bfba6a5143fb13795f0b34780a5f24b8365d511aaaca54f73273d4c27cc24e6`。

The C programming language provides the ability to use floating-point numbers for calculations. The C Standard specifies requirements on a [conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conforming) [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) for floating-point numbers but makes few guarantees about the specific underlying floating-point representation because of the existence of competing floating-point systems.

By definition, a floating-point number is of finite precision and, regardless of the underlying implementation, is prone to errors associated with rounding. (See [FLP01-C. Take care in rearranging floating-point expressions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/floating-point-flp/flp01-c) and [FLP02-C. Avoid using floating-point numbers when precise computation is needed](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/floating-point-flp/flp02-c) .)

The most common floating-point system is specified by the IEEE 754 standard. An older floating-point system is the IBM floating-point representation (sometimes called IBM/370). Each of these systems has different precisions and ranges of representable values. As a result, they do not represent all of the same values, are not binary compatible, and have different associated error rates.

Because of a lack of guarantees on the specifics of the underlying floating-point system, no assumptions can be made about either precision or range. Even if code is not intended to be portable, the chosen compiler's behavior must be well understood at all compiler optimization levels.

Here is a simple illustration of precision limitations. The following code prints the decimal representation of 1/3 to 50 decimal places. Ideally, it would print 50 numeral 3s:

On 64-bit Linux, with GCC 4.1, it produces

On 64-bit Windows, with Microsoft Visual Studio 2012, it produces

Additionally, compilers may treat floating-point variables differently under different levels of optimization \[ [Gough 2005](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Gough2005) \]:

When compiled on an IA-32 Linux machine with GCC 3.4.4 at optimization level 1 or higher, or on an IA-64 Windows machine with Microsoft Visual Studio 2012 in Debug or Release mode, this code prints

On an IA-32 Linux machine with GCC 3.4.4 with optimization turned off, this code prints

The reason for this behavior is that Linux uses the internal extended precision mode of the x87 floating-point unit (FPU) on IA-32 machines for increased accuracy during computation. When the result is stored into memory by the assignment to `c` , the FPU automatically rounds the result to fit into a `double` . The value read back from memory now compares unequally to the internal representation, which has extended precision. Windows does not use the extended precision mode, so all computation is done with double precision, and there are no differences in precision between values stored in memory and those internal to the FPU. For GCC, compiling at optimization level 1 or higher eliminates the unnecessary store into memory, so all computation happens within the FPU with extended precision \[ [Gough 2005](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Gough2005) \].

The standard constant `__FLT_EPSILON__` can be used to evaluate if two floating-point values are close enough to be considered equivalent given the granularity of floating-point operations for a given implementation. `__FLT_EPSILON__` represents the difference between 1 and the least value greater than 1 that is representable as a float. The granularity of a floating-point operation is determined by multiplying the operand with the larger absolute value by `__FLT_EPSILON__` .

On all tested platforms, this code prints

For `double` precision and `   long double  ` precision floating-point values, use a similar approach using the `__DBL_EPSILON__` and `__LDBL_EPSILON__` constants, respectively.

Consider using numerical analysis to properly understand the numerical properties of the problem.

## FLP01-C. Take care in rearranging floating-point expressions

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/10.floating-point-flp/3.flp01-c.md) — SHA-256 `981c7ce76707d21c1cac599af35976a53de6742fa707dc7cc0f3ee053c5688d5`。

Be careful when rearranging floating-point expressions to ensure the greatest accuracy of the result.

Subclause 5.1.2.3, paragraph 14, of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], states:

> Rearrangement for floating-point expressions is often restricted because of limitations in precision as well as range. The [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) cannot generally apply the mathematical associative rules for addition or multiplication, nor the distributive rule, because of roundoff error, even in the absence of overflow and underflow. Likewise, implementations cannot generally replace decimal constants to rearrange expressions. In the following fragment, rearrangements suggested by mathematical rules for real numbers are often not valid.
>
>

## FLP02-C. Avoid using floating-point numbers when precise computation is needed

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/10.floating-point-flp/4.flp02-c.md) — SHA-256 `ec26b8849f36cfd963d56af61a5a53e9a8e8a47ac830e4789400c3353d4953c6`。

Computers can represent only a finite number of digits. It is therefore impossible to precisely represent repeating binary-representation values such as 1/3 or 1/5 with the most common floating-point representation: binary floating point.

When precise computation is necessary, use alternative representations that can accurately represent the values. For example, if you are performing arithmetic on decimal values and need an exact decimal rounding, represent the values in binary-coded decimal instead of using floating-point values. Another option is decimal floating-point arithmetic, as specified by ANSI/IEEE 754-2007. ISO/IEC WG14 has drafted a proposal to add support for decimal floating-point arithmetic to the C language \[ [ISO/IEC DTR 24732](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECDTR24732) \].

When precise computation is necessary, carefully and methodically estimate the maximum cumulative error of the computations, regardless of whether decimal or binary is used, to ensure that the resulting error is within tolerances. Consider using numerical analysis to properly understand the problem. An introduction can be found in David Goldberg's "What Every Computer Scientist Should Know about Floating-Point Arithmetic" \[ [Goldberg 1991](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Goldberg91) \].

### Noncompliant Code Example

This noncompliant code example takes the mean of 10 identical numbers and checks to see if the mean matches this number. It should match because the 10 numbers are all `10.1` . Yet, because of the imprecision of floating-point arithmetic, the computed mean does not match this number.

On a 64-bit Linux machine using GCC 4.1, this program yields the following output:

### Compliant Solution

The noncompliant code can be fixed by replacing the floating-point numbers with integers for the internal additions. Floats are used only when printing results and when doing the division to compute the mean.

On a 64-bit Linux machine using GCC 4.1, this program yields the following expected output:

## FLP03-C. Detect and handle floating-point errors

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/10.floating-point-flp/5.flp03-c.md) — SHA-256 `34e9a322e6c58b12a20aa487b281e1b3b49b1f3eb1108abeabff2b9e4e235dd4`。

Errors during floating-point operations are often neglected by programmers who instead focus on validating operands before an operation. Errors that occur during floating-point operations are admittedly difficult to determine and diagnose, but the benefits of doing so often outweigh the costs. This recommendation suggests ways to capture errors during floating-point operations.

The following code exhibits undefined behavior:

On most implementations, integer division by zero is a terminal error, commonly printing a diagnostic message and aborting the program:

Floating-point division by zero also results in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) , although most [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) do not treat it as a terminal error. If additional precautions are not taken, it results in a silent error.

The most portable way to determine if a floating-point exceptional condition has occurred is to use the floating-point exception facilities provided by C in `fenv.h` .

However, the C floating-point exception functions are not without problems. The following caveats exist regarding the interaction between floating-point exceptions and conversions:

-   Conversion from floating-point to integer may cause an "invalid" floating-point exception. If this occurs, the value of that integer is undefined and should not be used.
-   Most implementations fail to raise "invalid" for conversions from any negative or "large" positive floating-point values to unsigned integer types or to `    signed char   ` . (See [tflt2int.c](http://www.tybor.com/tflt2int.c) .)
-   When a noninteger floating-point value is converted to an integer, the "inexact" floating-point exception is raised.

For information regarding floating-point number conversions, see [FLP34-C. Ensure that floating-point conversions are within range of the new type](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/floating-point-flp/flp34-c) .

The C Standard does not require all implementations to support floating-point exceptions. Each exception macro in `fenv.h` is defined if, and only if, the corresponding exception is supported. Only implementations that use IEC 60559 (formerly IEEE-754) floating-point arithmetic are required to support all five exceptions defined by C (see the C Standard, subclause 7.6.2 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \]). Nevertheless, these functions are the most portable solution for handling floating-point exceptions.

A less portable but potentially more secure solution is to use the capabilities provided by the underlying implementation. If this approach is taken, the caveats of that system must be well understood. The following table provides a starting point for some common operating systems:

| Operating System | How to Handle Floating-Point Errors |
| --- | --- |
| Linux<br /><br />Solaris 10<br /><br />AIX 5.3<br /><br />HP-UX 11.31<br /><br />Mac OS X 10.5 | Use the C floating-point exception functions |
| Windows | Use either the C floating-point exception functions or structured exception handling through `_fpieee_flt` [ [MSDN](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MSDN) ] |

### Noncompliant Code Example

In this noncompliant code example, floating-point operations are performed without checking for errors. Note that range checking has been intentionally omitted because the intent is to detect errors following the floating-point operation.

However, exceptional conditions (as indicated by the comments) occur that may lead to unexpected arithmetic results.

### Compliant Solution (C)

This compliant solution uses C Standard functions to handle floating-point errors:

### Compliant Solution (Windows)

Microsoft Visual Studio 2008 and earlier versions do not support C functions to handle floating-point errors. Windows provides an alternative method using `_statusfp()` , `_statusfp2()` , and `_clearfp()` .

### Compliant Solution (Windows SEH)

Microsoft Visual Studio 2008 also uses structured exception handling (SEH) to handle floating-point operations. SEH provides more information about the error and allows the programmer to change the results of the floating-point operation that caused the error condition.

## FLP04-C. Check floating-point inputs for exceptional values

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/10.floating-point-flp/6.flp04-c.md) — SHA-256 `3fed5aefa4c9ce2b766c7e4cf71962a149d03059b867db24a5085a907678820c`。

Floating-point numbers can take on two classes of exceptional values; infinity and NaN (not-a-number). These values are returned as the result of exceptional or otherwise unresolvable floating-point operations. (See also [FLP32-C. Prevent or detect domain and range errors in math functions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/floating-point-flp/flp32-c) .) Additionally, they can be directly input by a user by `scanf` or similar functions. Failure to detect and handle such values can result in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

NaN values are particularly problematic because the expression NaN == NaN (for every possible value of NaN) returns false. Any comparisons made with NaN as one of the arguments returns false, and all arithmetic functions on NaNs simply propagate them through the code. Hence, a NaN entered in one location in the code and not properly handled could potentially cause problems in other, more distant sections.

Formatted-input functions such as `scanf` will accept the values `INF` , `INFINITY` , or `NAN` (case insensitive) as valid inputs for the `%f` format specification, allowing malicious users to feed them directly to a program. Programs should therefore check to ensure that all input floating-point values (especially those controlled by the user) have neither of these values if doing so would be inappropriate. The `<math.h>` library provides two macros for this purpose: `isinf` and `isnan` .

### `isinf` and `isnan`

The `isinf` macro tests an input floating-point value for infinity. `isinf(val)` is nonzero if `val` is an infinity (positive or negative), and 0 otherwise.

`isnan` tests if an input is NaN. `isnan(val)` is nonzero if `val` is a NaN, and 0 otherwise.

If infinity or NaN values are not acceptable inputs in a program, these macros should be used to ensure they are not passed to vulnerable functions.

### Noncompliant Code Example

This noncompliant code example accepts user data without first validating it:

This can be a problem if an invalid value is entered for `val` and subsequently used for calculations or as control values. The user could, for example, input the strings `"INF"` , `"INFINITY"` , or `"NAN"` (case insensitive) on the command line, which would be parsed by `scanf` into the floating-point representations of infinity and NaN. All subsequent calculations using these values would be invalid, possibly crashing the program and enabling a [denial-of-service attack](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-denial-of-serviceattack) .

Here, for example, entering `"nan"` for `val` would force `currentBalance` to also equal `"nan"` , corrupting its value. If this value is used elsewhere for calculations, every resulting value would also be a NaN, possibly destroying important data.

#### Implementation Details

The following code was run on 32-bit GNU Linux using the GCC 3.4.6 compiler. On this platform, `FLT_MAX` has the value 340282346638528859811704183484516925440.000000.

The following table shows the value of `currentBalance` returned for various arguments:

|                          |                               |
|--------------------------|-------------------------------|
| Input                    | `currentBalance` |
| `25`        | `25.00000`       |
| `infinity`  | `inf`            |
| `inf`       | `inf`            |
| `-infinity` | `-inf`           |
| `NaN`       | `nan`            |
| `nan`       | `nan`            |
| `1e9999`    | `inf`            |
| `-1e9999`   | `-inf`           |

As this example demonstrates, the user can enter the exceptional values `infinity` and `NaN` , as well as force a float's value to be infinite, by entering out-of-range floats. These entries subsequently corrupt the value of `currentBalance` . So by entering exceptional floats, an attacker can corrupt the program data, possibly leading to a crash.

### Compliant Solution

This compliant solution first validates the input float before using it. The value is tested to ensure that it is neither an infinity nor a NaN.

### Exceptions

Occasionally, NaN or infinity may be acceptable or expected inputs to a program. If this is the case, then explicit checks may not be necessary. Such programs must, however, be prepared to handle these inputs gracefully and not blindly use them in mathematical expressions where they are not appropriate.

## FLP05-C. Do not use denormalized numbers

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/10.floating-point-flp/7.flp05-c.md) — SHA-256 `dd3620521f469f3fba0894dce746a337bdd0809daae631eede4cab7fdece4586`。

Most implementations of C use the IEEE 754 standard for floating-point representation. In this representation, floats are encoded using 1 sign bit, 8 exponent bits, and 23 mantissa bits. Doubles are encoded and used exactly the same way, except they use 1 sign bit, 11 exponent bits, and 52 mantissa bits. These bits encode the values of *s* , the sign; *M* , the significand; and *E* , the exponent. Floating-point numbers are then calculated as (−1) *<sup>s</sup>* \* *M* \* 2 *<sup>E</sup>* .

Ordinarily, all of the mantissa bits are used to express significant figures, in addition to a leading 1, which is implied and therefore left out. Consequently, floats ordinarily have 24 significant bits of precision, and doubles ordinarily have 53 significant bits of precision. Such numbers are called *normalized numbers* . All floating-point numbers are limited in the sense that they have fixed precision. See [FLP00-C. Understand the limitations of floating-point numbers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/floating-point-flp/flp00-c) .

Mantissa bits are used to express extremely small numbers that are too small to encode normally because of the lack of available exponent bits. Using mantissa bits extends the possible range of exponents. Because these bits no longer function as significant bits of precision, the total precision of extremely small numbers is less than usual. Such numbers are called *denormalized,* and they are more limited than normalized numbers. However, even using normalized numbers where precision is required can pose a risk. See [FLP02-C. Avoid using floating-point numbers when precise computation is needed](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/floating-point-flp/flp02-c) for more information.

Denormalized numbers can severely impair the precision of floating-point numbers and should not be used.

### Noncompliant Code Example

This code attempts to reduce a floating-point number to a denormalized value and then restore the value. This operation is imprecise.

This code produces the following output on [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) that use IEEE 754 floats:

### Compliant Solution

Do not allow code to produce denormalized numbers. If floats are producing denormalized numbers, use doubles instead.

If using doubles also produces denormalized numbers, using long doubles may or may not help. (On some implementations, long double has the same exponent range as double.) If using long doubles produces denormalized numbers, some other solution must be found.

#### Printing Denormalized Numbers

Denormalized numbers can also be troublesome because some functions have [implementation-defined](https://www.securecoding.cert.org/confluence/display/seccode/BB.+Definitions) behavior when used with denormalized values. For example, using the `%a` or `$%A` conversion specifier in a format string can produce implementation-defined results when applied to denormalized numbers.

Subclause 7.21.6.1, paragraph 8, of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], states:

> A double argument representing a floating-point number is converted in the style `    [-]0         x        h.hhhh p         ±        d   ` , where there is one hexadecimal digit (which is nonzero if the argument is a normalized floating-point number and is otherwise unspecified) before the decimal-point character.

Relying on the `%a` and `%A` specifiers to produce values without a leading zero is error prone.

#### Implementation Details

On a 32-bit Linux machine using GCC 4.3.2, this code produces the following output:

## FLP06-C. Convert integers to floating point for floating-point operations

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/10.floating-point-flp/8.flp06-c.md) — SHA-256 `889780b8bce3e4814dd4cfc9fdb9220ec60ee7cece685b6e0cf54c40cced7e06`。

Using integer arithmetic to calculate a value for assignment to a floating-point variable may lead to loss of information. This problem can be avoided by converting one of the integers in the expression to a floating type.

When converting integers to floating-point values, and vice versa, it is important to carry out proper range checks to avoid undefined behavior (see [FLP34-C. Ensure that floating-point conversions are within range of the new type](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/floating-point-flp/flp34-c) ).

### Noncompliant Code Example

In this noncompliant code example, the division and multiplication operations take place on integers and are then converted to floating point. Consequently, floating-point variables `d` , `e` , and `f` are not initialized correctly because the operations take place before the values are converted to floating-point values. The results are truncated to the nearest integer or may overflow.

### Compliant Solution (Floating-Point Literal)

In this compliant solution, the decimal error in initialization is eliminated by ensuring that at least one of the operands to the division operation is floating point:

### Compliant Solution (Conversion)

In this compliant solution, the decimal error in initialization is eliminated by first storing the integer in the floating-point variable and then performing the arithmetic operation. This practice ensures that at least one of the operands is a floating-point number and that the subsequent arithmetic operation is performed on floating-point operands.

### Exceptions

**FLP06-C-EX0:** It may be desirable to have the operation take place as integers before the conversion (obviating the need for a call to `trunc()` , for example). If this is the programmer's intention, it should be clearly documented to help future maintainers understand that this behavior is intentional.

## FLP07-C. Cast the return value of a function that returns a floating-point type

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/10.floating-point-flp/9.flp07-c.md) — SHA-256 `22bcaaad2fd588f4461a93df1d1952e46ea004e998dedf829819591a7cadc6bc`。

Cast the return value of a function that returns a floating point type to ensure predictable program execution.

Subclause 6.8.6.4, paragraph 3, of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] states:

> If a return statement with an expression is executed, the value of the expression is returned to the caller as the value of the function call expression. If the expression has a type different from the return type of the function in which it appears, the value is converted as if by assignment to an object having the return type of the function.

This paragraph is annotated (footnote 160) as follows:

> The return statement is not an assignment. The overlap restriction of subclause 6.5.16.1 does not apply to the case of function return. The representation of floating-point values may have wider range or precision than implied by the type; a cast may be used to remove this extra range and precision.

Conversion as if by assignment to the type of the function is required if the return expression has a different type than the function but not if the return expression has a wider value only because of wide evaluation. This allows seemingly inconsistent and confusing behavior. Consider the following code as an example:

Function `f()` is allowed to return a value wider than `float` , but function `g()` (which uses the wider constant) is not.

Although the standard does not require narrowing return expressions of the same type as the function, it does not clearly state what is allowed. Is it allowed to narrow the result? Is it allowed to narrow the result sometimes but not always? Is it allowed to partially narrow the result (for example, if the application binary interface \[ABI\] returns floats in double format, but a float function has a float return expression evaluated to wider than double)? An aggressive implementation could argue yes€? to all these questions, though the resulting behavior would complicate debugging and error analysis.

Footnote 160 in the C Standard says a cast may be used to remove extra range and precision from the return expression. This means that a predictable program must have casts on all function calls that return floating-point values (except where the function directly feeds an operator-like assignment that implies the conversion). With type-generic math `(tgmath.h)` , the programmer has to reason through the `tgmath.h` resolution rules to determine which casts to apply. These are significant obstacles to writing predictable code.

The C Standard, subclause F.6 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], states:

> If the return expression is evaluated in a floating-point format different from the return type, the expression is converted as if by assignment <sup>362</sup> to the return type of the function and the resulting value is returned to the caller.
>
> 362\) Assignment removes any extra range and precision.

This applies only to implementations that conforms to the optional Annex F, "IEC 60559 Floating-Point Arithmetic."  The macro `__STDC_IEC_559__` can be used to determine whether an implementation conforms to Annex F.

### Noncompliant Code Example

This noncompliant code example fails to cast the result of the expression in the return statement and thereby guarantee that the range or precision is no wider than expected. The uncertainty in this example is introduced by the constant `0.1f` . This constant may be stored with a range or precision that is greater than that of `float` . Consequently, the result of `   x * 0.1f  ` may also have a range or precision greater than that of `float` . As described previously, this range or precision may not be reduced to that of a `float` , so the caller of `calcPercentage()` may receive a value that is more precise than expected. This may lead to inconsistent program execution across different platforms.

### Compliant Solution (within the Function)

This compliant solution casts the value of the expression in the return statement. It forces the return value to have the expected range and precision, as described in subclause 5.2.4.2.2, paragraph 9, of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \].

Forcing the range and precision inside the `calcPercentage()` function is a good way to fix the problem once without having to apply fixes in multiple locations (every time `calcPercentage()` is called).

### Compliant Solution (Outside the Function)

Source code to the called function may not always be available. This compliant solution casts the return value of the `calcPercentage()` function to `float` to force the correct range and precision when the source of the called function cannot be modified.

## FLP30-C. Do not use floating-point variables as loop counters

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/10.floating-point-flp/2.flp30-c.md) — SHA-256 `d58ae7b70c7aba7b2de828f68bc1f1c517ee480b0cbd967ec0d1e81a15f4c9c5`。

Because floating-point numbers represent real numbers, it is often mistakenly assumed that they can represent any simple fraction exactly. Floating-point numbers are subject to representational limitations just as integers are, and binary floating-point numbers cannot represent all real numbers exactly, even if they can be represented in a small number of decimal digits.

In addition, because floating-point numbers can represent large values, it is often mistakenly assumed that they can represent all significant digits of those values. To gain a large dynamic range, floating-point numbers maintain a fixed number of precision bits (also called the significand) and an exponent, which limit the number of significant digits they can represent.

Different implementations have different precision limitations, and to keep code portable, floating-point variables must not be used as the loop induction variable. See Goldberg's work for an introduction to this topic \[ [Goldberg 1991](https://cmu-sei.github.io/secure-coding-standards/sei-cert-oracle-coding-standard-for-java/back-matter/rule-aa-references#RuleAA.References-Goldberg91) \].

For the purpose of this rule, a *loop counter* is an induction variable that is used as an operand of a comparison expression that is used as the controlling expression of a `do` , `while` , or `for` loop. An *induction variable* is a variable that gets increased or decreased by a fixed amount on every iteration of a loop \[ [Aho 1986](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Aho1986) \]. Furthermore, the change to the variable must occur directly in the loop body (rather than inside a function executed within the loop).

### Noncompliant Code Example

In this noncompliant code example, a floating-point variable is used as a loop counter. The decimal number `0.1` is a repeating fraction in binary and cannot be exactly represented as a binary floating-point number. Depending on the implementation, the loop may iterate 9 or 10 times.

For example, when compiled with GCC or Microsoft Visual Studio 2013 and executed on an x86 processor, the loop is evaluated only nine times.

### Compliant Solution

In this compliant solution, the loop counter is an integer from which the floating-point value is derived:

### Noncompliant Code Example

In this noncompliant code example, a floating-point loop counter is incremented by an amount that is too small to change its value given its precision:

On many implementations, this produces an infinite loop.

### Compliant Solution

In this compliant solution, the loop counter is an integer from which the floating-point value is derived. The variable `x` is assigned a computed value to reduce compounded rounding errors that are present in the noncompliant code example.

## FLP32-C. Prevent or detect domain and range errors in math functions

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/10.floating-point-flp/3.flp32-c.md) — SHA-256 `fcb9b97b76f394a7f662c11062c73f1b3e8c91ab31906c0d407560797d843cc3`。

The C Standard, 7.12.1 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], defines three types of errors that relate specifically to math functions in `<math.h>` .  Paragraph 2 states

> *A domain error* occurs if an input argument is outside the domain over which the mathematical function is defined.

Paragraph 3 states

> A *pole error* (also known as a singularity or infinitary) occurs if the mathematical function has an exact infinite result as the finite input argument(s) are approached in the limit.

Paragraph 4 states

> a *range error* occurs if and only if the result overflows or underflows

An example of a domain error is the square root of a negative number, such as `sqrt(-1.0)` , which has no meaning in real arithmetic. Contrastingly, 10 raised to the 1-millionth power, `   pow(10., 1e6)  ` , cannot be represented in many floating-point [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) because of the limited range of the type `double` and consequently constitutes a range error. In both cases, the function will return some value, but the value returned is not the correct result of the computation. An example of a pole error is `log(0.0)` , which results in negative infinity.

Programmers can prevent domain and pole errors by carefully bounds-checking the arguments before calling mathematical functions and taking alternative action if the bounds are violated.

Range errors usually cannot be prevented because they are dependent on the implementation of floating-point numbers as well as on the function being applied. Instead of preventing range errors, programmers should attempt to detect them and take alternative action if a range error occurs.

The following table lists the `double` forms of standard mathematical functions, along with checks that should be performed to ensure a proper input domain, and indicates whether they can also result in range or pole errors, as reported by the C Standard. Both `float` and `   long double  ` forms of these functions also exist but are omitted from the table for brevity. If a function has a specific domain over which it is defined, the programmer must check its input values. The programmer must also check for range errors where they might occur. The standard math functions not listed in this table, such as `fabs()` , have no domain restrictions and cannot result in range or pole errors.

| Function | Domain | Range | Pole |
| --- | --- | --- | --- |
| ` acos(x) ` | ` -1 <= x && x <= 1 ` | No | No |
| ` asin(x) ` | ` -1 <= x && x <= 1 ` | Yes | No |
| ` atan(x) ` | None | Yes | No |
| ` atan2(y, x) ` | ` None ` | No | No |
| ` acosh(x) ` | ` x >= 1 ` | Yes | No |
| ` asinh(x) ` | None | Yes | No |
| ` atanh(x) ` | ` -1 < x && x < 1 ` | Yes | Yes |
| ` cosh(x) ` , ` sinh(x) ` | None | Yes | No |
| ` exp(x) ` , ` exp2(x) ` , ` expm1(x) ` | None | Yes | No |
| ` ldexp(x, exp) ` | None | Yes | No |
| ` log(x) ` , ` log10(x) ` , ` log2(x) ` | ` x >= 0 ` | No | Yes |
| ` log1p(x) ` | ` x >= -1 ` | No | Yes |
| ` ilogb(x) ` | ` x != 0 && !isinf(x) && !isnan(x) ` | Yes | No |
| ` logb(x) ` | ` x != 0 ` | Yes | Yes |
| ` scalbn(x, n) ` , ` scalbln(x, n) ` | None | Yes | No |
| ` hypot(x, y) ` | None | Yes | No |
| ` pow(x,y) ` | ` x > 0 \|\| (x == 0 && y > 0) \|\| ` <br> ( ` x < 0 && y ` is an integer) | Yes | Yes |
| ` sqrt(x) ` | ` x >= 0 ` | No | No |
| ` erf(x) ` | None | Yes | No |
| ` erfc(x) ` | None | Yes | No |
| ` lgamma(x) ` , ` tgamma(x) ` | ` x != 0 && ` ` ! ` ( ` x < 0 && x ` is an integer) | Yes | Yes |
| ` lrint(x) ` , ` lround(x) ` | None | Yes | No |
| ` fmod(x, y) ` , ` remainder(x, y) ` , <br> ` remquo(x, y, quo) ` | ` y != 0 ` | Yes | No |
| ` nextafter(x, y) ` , <br> ` nexttoward(x, y) ` | None | Yes | No |
| ` fdim(x,y) ` | None | Yes | No |
| ` fma(x,y,z) ` | None | Yes | No |

### Domain and Pole Checking

The most reliable way to handle domain and pole errors is to prevent them by checking arguments beforehand, as in the following exemplar:

### Range Checking

Programmers usually cannot prevent range errors, so the most reliable way to handle them is to detect when they have occurred and act accordingly.

The exact treatment of error conditions from math functions is tedious. The C Standard, 7.12.1 paragraph 5 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], defines the following behavior for floating-point overflow:

> A floating result overflows if a finite result value with ordinary accuracy would have magnitude (absolute value) too large for the representation with full precision in the specified type. A result that is exactly an infinity does not overflow. If a floating result overflows and default rounding is in effect, then the function returns the value of the macro HUGE \_ VAL , HUGE \_ VALF , or HUGE \_ VALL according to the return type, with the same sign as the correct value of the function; however, for the types with reduced-precision representations of numbers beyond the overflow threshold, the function may return a representation of the result with less than full precision for the type. If a floating result overflows and default rounding is in effect and the integer expression math \_ errhandling & MATH \_ ERRNO is nonzero, then the integer expression errno acquires the value ERANGE . If a floating result overflows, and the integer expression math \_ errhandling & MATH \_ ERREXCEPT is nonzero, the "overflow" floating-point exception is raised (regardless of whether default rounding is in effect).

It is preferable not to check for errors by comparing the returned value against `HUGE_VAL` or `0` for several reasons:

-   These are, in general, valid (albeit unlikely) data values.
-   Making such tests requires detailed knowledge of the various error returns for each math function.
-   Multiple results aside from `HUGE_VAL` and `0` are possible, and programmers must know which are possible in each case.
-   Different versions of the library have varied in their error-return behavior.

It can be unreliable to check for math errors using `errno` because an [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) might not set `errno` . For real functions, the programmer determines if the implementation sets `errno` by checking whether `   math_errhandling & MATH_ERRNO  ` is nonzero.

The C Standard, 7.3.2, paragraph 1 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \],  states:

> an implementation may set `errno` but is not required to.

The obsolete *System V Interface Definition* (SVID3) \[ [UNIX 1992](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-UNIX92) \] provides more control over the treatment of errors in the math library. The programmer can define a function named `matherr()` that is invoked if errors occur in a math function. This function can print diagnostics, terminate the execution, or specify the desired return value. The `matherr()` function has not been adopted by C or POSIX, so it is not generally portable.

The following error-handing template uses C Standard functions for floating-point errors when the C macro `math_errhandling` is defined and indicates that they should be used; otherwise, it examines `errno` :

See [FLP03-C. Detect and handle floating-point errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/floating-point-flp/flp03-c) for more details on how to detect floating-point errors.

### Subnormal Numbers

A subnormal number is a nonzero number that does not use all of its precision bits \[ [IEEE 754 2019](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEE7542019) \]. These numbers can be used to represent values that are closer to 0 than the smallest normal number (one that uses all of its precision bits). However, the `asin()` , `asinh()` , `atan()` , `atanh()` , and `erf()` functions, along with their `float` analogues, may produce range errors, specifically when passed a subnormal number. When, evaluated with a subnormal number, these functions can produce an inexact, subnormal value, which is an underflow error.

The C Standard, 7.12.1, paragraph 6 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], defines the following behavior for floating-point underflow:

> The result underflows if a nonzero result value with ordinary accuracy would have magnitude (absolute value) less than the minimum normalized number in the type; however a zero result that is specified to be an exact zero does not underflow. Also, a result with ordinary accuracy and the magnitude of the minimum normalized number may underflow. 269) If the result underflows, the function returns an implementation-defined value whose magnitude is no greater than the smallest normalized positive number in the specified type; if the integer expression math \_ errhandling & MATH \_ ERRNO is nonzero, whether errno acquires the value ERANGE is implementation-defined; if the integer expression math \_ errhandling & MATH \_ ERREXCEPT s nonzero, whether the"underflow" floating-point exception is raised is implementation-defined.

Implementations that support floating-point arithmetic but do not support subnormal numbers, such as IBM S/360 hex floating-point or nonconforming IEEE-754 implementations that skip subnormals (or support them by flushing them to zero), can return a range error when calling one of the following families of functions with the following arguments:

-   `    fmod         ((min+subnorm), min)    `-`    remainder         ((min+                  subnorm      ` ), min)
-   `    remquo         ((min+                  subnorm      ` ), min, quo)

where `min` is the minimum value for the corresponding floating point type and `subnorm` is a subnormal value.

If Annex F is supported and subnormal results are supported, the returned value is exact and a range error cannot occur. The C Standard, F.10.7.1 paragraph 2 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], specifies the following for the `fmod()` , `remainder()` , and `remquo()` functions:

> When subnormal results are supported, the returned value is exact and is independent of the current rounding direction mode.

Annex F, subclause F.10.7.2, paragraph 2, and subclause F.10.7.3, paragraph 2, of the C Standard identify when subnormal results are supported.

### Noncompliant Code Example ( `sqrt()` )

This noncompliant code example determines the square root of `x` :

However, this code may produce a domain error if `x` is negative.

### Compliant Solution ( `sqrt()` )

Because this function has domain errors but no range errors, bounds checking can be used to prevent domain errors:

### Noncompliant Code Example ( `sinh()` , Range Errors)

This noncompliant code example determines the hyperbolic sine of `x` :

This code may produce a range error if `x` has a very large magnitude.

### Compliant Solution ( `sinh()` , Range Errors)

Because this function has no domain errors but may have range errors, the programmer must detect a range error and act accordingly:

### Noncompliant Code Example ( `pow()` )

This noncompliant code example raises `x` to the power of `y` :

This code may produce a domain error if `x` is negative and `y` is not an integer value or if `x` is 0 and `y` is 0. A domain error or pole error may occur if `x` is 0 and `y` is negative, and a range error may occur if the result cannot be represented as a `double` .

### Compliant Solution ( `pow()` )

Because the `pow()` function can produce domain errors, pole errors, and range errors, the programmer must first check that `x` and `y` lie within the proper domain and do not generate a pole error and then detect whether a range error occurs and act accordingly:

### Noncompliant Code Example ( `asin()` , Subnormal Number)

This noncompliant code example determines the inverse sine of `x` :

### Compliant Solution ( `asin()` , Subnormal Number)

Because this function permits domain errors or range errors, the programmer must detect either and act accordingly. As noted earlier, a subnormal number may produce a range error, which must be detected:

## FLP34-C. Ensure that floating-point conversions are within range of the new type

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/10.floating-point-flp/4.flp34-c.md) — SHA-256 `75236568e0b9113d1375566de9b7a0cc2718a0a1764b701a12e383d734eb444c`。

If a floating-point value is to be converted to a floating-point value of a smaller range and precision or to an integer type, or if an integer type is to be converted to a floating-point type, the value must be representable in the destination type.

The C Standard, 6.3.1.4, paragraph 1 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], says,

> When a finite value of standard floating type is converted to an integer type other than bool, the fractional part is discarded (i.e. the value is truncated toward zero). If the value of the integral part cannot be represented by the integer type, the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

Paragraph 3 of the same subclause says,

> When a value of integer type is converted to a standard floating type, if the value being converted can be represented exactly in the new type, it is unchanged. If the value being converted is in the range of values that can be represented but cannot be represented exactly, the result is either the nearest higher or nearest lower representable value, chosen in an [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) manner. If the value being converted is outside the range of values that can be represented, the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

And subclause 6.3.1.5, paragraph 1+2, says,

> When a value of real floating type is converted to a real floating type, if the value being converted can be represented exactly in the new type, it is unchanged.
>
> When a value of real floating type is converted to a standard floating type, if the value being converted is in the range of values that can be represented but cannot be represented exactly, the result is either the nearest higher or nearest lower representable value, chosen in an [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) manner. If the value being converted is outside the range of values that can be represented, the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

See [undefined behaviors 16](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_16) and [17](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_17) .

This rule does not apply to demotions of floating-point types on [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) that support signed infinity, such as IEEE 754, as all values are within range.

### Noncompliant Code Example ( `float` to `int` )

This noncompliant code example leads to [undefined behavior 16](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_16) if the integral part of `f_a` cannot be represented as an integer:

### Compliant Solution ( `float` to `int` )

This compliant solution tests to ensure that the `float` value will fit within the `int` variable before performing the assignment.

### Noncompliant Code Example (Narrowing Conversion)

This noncompliant code example attempts to perform conversions that may result in truncating values outside the range of the destination types:

As a result of these conversions, it is possible that `d_a` is outside the range of values that can be represented by a float or that `big_d` is outside the range of values that can be represented as either a `float` or a `double` . If this is the case, the result is [undefined behavior 17](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_17) on implementations that do not support Annex F, "IEC 60559 Floating-Point Arithmetic."

### Compliant Solution (Narrowing Conversion)

This compliant solution checks whether the values to be stored can be represented in the new type:

## FLP36-C. Preserve precision when converting integral values to floating-point type

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/10.floating-point-flp/5.flp36-c.md) — SHA-256 `d0081edc0da70d1c7bfc51343b1e2f94014e0444e18c840aea81b90aaff3ff2b`。

Narrower arithmetic types can be cast to wider types without any effect on the magnitude of numeric values. However, whereas integer types represent exact values, floating-point types have limited precision. The C Standard, 6.3.1.4 paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> When a value of integer type is converted to a standard floating type, if the value being converted can be represented exactly in the new type, it is unchanged. If the value being converted is in the range of values that can be represented but cannot be represented exactly, the result is either the nearest higher or nearest lower representable value, chosen in an [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) manner. If the value being converted is outside the range of values that can be represented, the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . Results of some implicit conversions may be represented in greater range and precision than that required by the new type (see 6.3.1.8 and 6.8.7.5).

Conversion from integral types to floating-point types without sufficient precision can lead to loss of precision (loss of least significant bits). No runtime exception occurs despite the loss.

### Noncompliant Code Example

In this noncompliant example, a large value of type `   long int  ` is converted to a value of type `float` without ensuring it is representable in the type:

For most floating-point hardware, the value closest to `1234567890` that is representable in type `float` is `1234567844` ; consequently, this program prints the value `-46` .

### Compliant Solution

This compliant solution replaces the type `float` with a `double` . Furthermore, it uses an assertion to guarantee that the `double` type can represent any `   long int  ` without loss of precision. (See [INT35-C. Use correct integer precisions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int35-c) and [MSC11-C. Incorporate diagnostic tests using assertions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc11-c) .)

On the same implementation, this program prints `0` , implying that the integer value `1234567890` is representable in type `double` without change.

### Exceptions

**FLP36-C-EX1** : Loss of precision can be acceptable when necessary for the proper execution of the program. It is recommended that the conversion be clearly commented as permitting loss of precision.

## FLP37-C. Do not use object representations to compare floating-point values

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/10.floating-point-flp/6.flp37-c.md) — SHA-256 `842d1014a9ff2c109d1a659f6d4bc48de47bc1e048dc65149ff002053ad76850`。

The object representation for floating-point values is implementation defined. However, an implementation that defines the `__STDC_IEC_559__` macro shall conform to the IEC 60559 floating-point standard and uses what is frequently referred to as IEEE 754 floating-point arithmetic \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2024) \] . The floating-point object representation used by IEC 60559 is one of the most common floating-point object representations in use today.  

All floating-point object representations use specific bit patterns to encode the value of the floating-point number being represented. However, equivalence of floating-point values is not encoded solely by the bit pattern used to represent the value. For instance, if the floating-point format supports negative zero values (as IEC 60559 does), the values `-0.0` and `0.0` are equivalent and will compare as equal, but the bit patterns used in the object representation are not identical. Similarly, if two floating-point values are both (the same) NaN, they will not compare as equal, despite the bit patterns being identical, because they are not equivalent.

Do not compare floating-point object representations directly, such as by calling `memcmp()` or its moral equivalents. Instead, the equality operators ( `==` and `!=` ) should be used to determine if two floating-point values are equivalent.

### Noncompliant Code Example

In this noncompliant code example, `memcmp()` is used to compare two structures for equality. However, since the structure contains a floating-point object, this code may not behave as the programmer intended.

### Compliant Solution

In this compliant solution, the structure members are compared individually:

## FLP38-C. Avoid undefined behavior while using type-generic macro functions

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/10.floating-point-flp/7.flp38-c.md) — SHA-256 `7a9203105fcc96d6ae2a4219fb0c0416c60117024474d71da731d2308116d92d`。

Section 7.27 of [ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2024) \] provides the headers `<tgmath.h>`, `<math.h>` and `<complex.h>` and provides type-generic macros. Many of these are designed to work on complex numbers or various types of floating-point numbers. However, there are many ways to misuse these functions, which can lead to undefined behaviors [203](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_203), [204](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_204), [206](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_206), and [207](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_207).

### Noncompliant Code Example

This noncompliant code example attempts to compute the remainder between a double and a `_Decimal64`, which violates undefined behavior 203.

#### Implementation Details

While this program fails to compile on modern versions of Clang and MSVC, it compiles correctly on GCC 16.1 and the (deprecated) Intel x86-64 icc 2021.10.0 compiler, and produces the expected result of -1.0.

### Compliant Solution

This compliant solution casts the `_Decimal64` to a double for doing the remainder.

### Noncompliant Code Example

This noncompliant code example attempts to compute the next number beyond 2.0, but passes a `_Float32` and a long double to the `nexttoward` function. This violates undefined behavior 204.

#### Implementation Details

While this program fails to compile on modern versions of Clang, and MSVC, it compiles correctly on GCC 16.1 and the Intel x86-64 icx 2024.0 compiler, and produces the expected result of 2.000000.

### Compliant Solution

This compliant solution casts the `_Float32` to a double for the computation, producing the correct answer.
