# CERT C — ARR

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## ARR00-C. Understand how arrays work

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/03.arrays-arr/2.arr00-c.md) — SHA-256 `bc330119e053bc21357d9f32fb75323ff56e931033840c54ec7ffa7cd04fb83e`。

The incorrect use of arrays has traditionally been a source of exploitable [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) . Elements referenced within an array using the subscript operator `[]` are not checked unless the programmer provides adequate bounds checking. As a result, the expression `   array [pos] = value  ` can be used by an attacker to transfer control to arbitrary code.

An attacker who can control the values of both `pos` and `value` in the expression `   array [pos] = value  ` can perform an arbitrary write (which is when the attacker overwrites other storage locations with different content). The consequences range from changing a variable used to determine what permissions the program grants to executing arbitrary code with the permissions of the vulnerable process. Arrays are also a common source of buffer overflows when iterators exceed the bounds of the array.

An array is a series of objects, all of which are the same size and type. Each object in an array is called an *array element* . The entire array is stored contiguously in memory (that is, there are no gaps between elements). Arrays are commonly used to represent a sequence of elements where random access is important but there is little or no need to insert new elements into the sequence (which can be an expensive operation with arrays).

Arrays containing a constant number of elements can be declared as follows:

These statements allocate storage for an array of 12 integers referenced by `array` . Arrays are indexed from `0..n-1` (where `n` represents an array bound). Arrays can also be declared as follows:

This array is called an *incomplete type* because the size is unknown. If an array of unknown size is initialized, its size is determined by the largest indexed element with an explicit initializer. At the end of its initializer list, the array no longer has incomplete type.

Although these declarations work fine when the size of the array is known at compile time, it is not possible to declare an array in this fashion when the size can be determined only at runtime. The C Standard adds support for variable length arrays or arrays whose size is determined at runtime. Before the introduction of variable length arrays in C99, however, these "arrays" were typically implemented as pointers to their respective element types allocated using `malloc()` , as shown in this example:

Always check that `malloc()` returns a non-null pointer, as per [ERR33-C. Detect and handle standard library errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err33-c) .

It is important to retain any pointer value returned by `malloc()` so that the referenced memory may eventually be deallocated. One possible way to preserve such a value is to use a constant pointer:

Below we consider some techniques for array initialization.  Both `dis` and `dat` arrays can be initialized as follows:

The `dis` array can also be initialized as follows:

This technique, however, will not work for `dat` .  The `dat` identifier cannot be incremented (produces a fatal compilation error), as it was declared with type `   int * const  ` .  This problem can be circumvented by copying `dat` into a separate pointer:

The variable `p` is declared as a pointer to an integer, initialized with the value stored in `dat` , and then incremented in the loop. This technique can be used to initialize both arrays, and is a better style of programming than incrementing the original pointer to the array (e.g., `dis++` , in the above example), as it avoids having to reset the pointer back to the start of the array after the loop completes.

Obviously, there is a relationship between array subscripts `[]` and pointers. The expression `dis[i]` is equivalent to `*(dis+i)` for all integral values of `i` . In other words, if `dis` is an array object (equivalently, a pointer to the initial element of an array object) and `i` is an integer, `dis[i]` designates the `i` <sup>th</sup> element of `dis` . In fact, because `*(dis+i)` can be expressed as `*(i+dis)` , the expression `dis[i]` can be represented as `i[dis]` , although doing so is not encouraged. Because array indices are zero-based, the first element is designated as `dis[0]` , or equivalently as `*(dis+0)` or simply `*dis` .

## ARR01-C. Do not apply the sizeof operator to a pointer when taking the size of an array

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/03.arrays-arr/3.arr01-c.md) — SHA-256 `9d546ad5eaeeba77d43c052bacab4d8f12818ae10af5884a779c5cad2ab2086f`。

The `sizeof` operator yields the size (in bytes) of its operand, which can be an expression or the parenthesized name of a type. However, using the `sizeof` operator to determine the size of arrays is error prone.

The `sizeof` operator is often used in determining how much memory to allocate via `malloc()` . However using an incorrect size is a violation of [MEM35-C. Allocate sufficient memory for an object](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/memory-management-mem/mem35-c) .

### Noncompliant Code Example

In this noncompliant code example, the function `clear()` zeros the elements in an array. The function has one parameter declared as `   int array[]  ` and is passed a static array consisting of 12 `int` as the argument. The function `clear()` uses the idiom `   sizeof(array) / sizeof(array[0])  ` to determine the number of elements in the array. However, `array` has a pointer type because it is a parameter. As a result, `sizeof(array)` is equal to the `   sizeof(int *)  ` . For example, on an architecture (such as IA-32) where the `   sizeof(int) == 4  ` and the `   sizeof(int *) == 4  ` , the expression `   sizeof(array) / sizeof(array[0])  ` evaluates to 1, regardless of the length of the array passed, leaving the rest of the array unaffected.

Footnote 103 in subclause 6.5.3.4 of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] applies to all array parameters:

> When applied to a parameter declared to have array or function type, the `sizeof` operator yields the size of the adjusted (pointer) type.

### Compliant Solution

In this compliant solution, the size of the array is determined inside the block in which it is declared and passed as an argument to the function:

This `   sizeof(array) / sizeof(array[0])  ` idiom will succeed provided the original definition of `array` is visible.

### Noncompliant Code Example

In this noncompliant code example, `sizeof(a)` does not equal `   100 * sizeof(int)  ` , because the `sizeof` operator, when applied to a parameter declared to have array type, yields the size of the adjusted (pointer) type even if the parameter declaration specifies a length:

### Compliant Solution

In this compliant solution, the size is specified using the expression `   len * sizeof(int)  ` :

## ARR02-C. Explicitly specify array bounds, even if implicitly defined by an initializer

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/03.arrays-arr/4.arr02-c.md) — SHA-256 `68ad8389fdace058aca0b497853cbae98d0edebc08d87457bc1a5cd4ffdd623f`。

The C Standard allows an array variable to be declared both with a bound and with an initialization literal. The initialization literal also implies an array bound in the number of elements specified.

The size implied by an initialization literal is usually specified by the number of elements,

but it is also possible to use designators to initialize array elements in a noncontiguous fashion. Subclause 6.7.9, Example 12, of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] states:

> Space can be "allocated" from both ends of an array by using a single designator:
>
> 
>
> In the above, if `MAX` is greater than ten, there will be some zero-valued elements in the middle; if it is less than ten, some of the values provided by the first five initializers will be overridden by the second five.

The C Standard also dictates how array initialization is handled when the number of initialization elements does not equal the explicit array bound. Subclause 6.7.9, paragraphs 21 and 22, state:

> If there are fewer initializers in a brace-enclosed list than there are elements or members of an aggregate, or fewer characters in a string literal used to initialize an array of known size than there are elements in the array, the remainder of the aggregate shall be initialized implicitly the same as objects that have static storage duration.  
> If an array of unknown size is initialized, its size is determined by the largest indexed element with an explicit initializer. The array type is completed at the end of its initializer list.

Although compilers can compute the size of an array on the basis of its initialization list, explicitly specifying the size of the array provides a redundancy check, ensuring that the array's size is correct. It also enables compilers to emit warnings if the array's size is less than the size implied by the initialization.

Note that this recommendation does not apply (in all cases) to character arrays initialized with string literals. See [STR11-C. Do not specify the bound of a character array initialized with a string literal](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/characters-and-strings-str/str11-c) for more information.

### Noncompliant Code Example (Incorrect Size)

This noncompliant code example initializes an array of integers using an initialization with too many elements for the array:

The size of the array `a` is 3, although the size of the initialization is 4. The last element of the initialization ( `4` ) is ignored. Most compilers will diagnose this error.

#### Implementation Details

This noncompliant code example generates a warning in GCC. Microsoft Visual Studio generates a fatal diagnostic: `   error C2078: too many initializers  ` .

### Noncompliant Code Example (Implicit Size)

In this example, the compiler allocates an array of four integer elements and, because an array bound is not explicitly specified by the programmer, sets the array bound to `4` . However, if the initializer changes, the array bound may also change, causing unexpected results.

### Compliant Solution

This compliant solution explicitly specifies the array bound:

Explicitly specifying the array bound, although it is implicitly defined by an initializer, allows a compiler or other static analysis tool to issue a diagnostic if these values do not agree.

### Exceptions

**ARR02-C-EX1:** [STR11-C. Do not specify the bound of a character array initialized with a string literal](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/characters-and-strings-str/str11-c) is a specific exception to this recommendation; it requires that the bound of a character array initialized with a string literal is unspecified.

## ARR30-C. Do not form or use out-of-bounds pointers or array subscripts

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/03.arrays-arr/2.arr30-c.md) — SHA-256 `5501ea62ffe0f108edfcff944ee4c815632b883ce0ea2cba98945d2d421c92e2`。

The C Standard identifies the following distinct situations in which undefined behavior (UB) can arise as a result of invalid pointer operations:

|                                                                                                |                                                                                                                                                                                                                                                |                                                                                                                                                                                                                                               |
|------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| UB                                                                                             | Description                                                                                                                                                                                                                                    | Example Code                                                                                                                                                                                                                                  |
| [43](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_43) | Addition or subtraction of a pointer into, or just beyond, an array object and an integer type produces a result that does not point into, or just beyond, the same array object.                                                              | [Forming Out-of-Bounds Pointer](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr30-c#ARR30C.Donotformoruseoutofboundspointersorarraysubscripts-FormingOut-of-BoundsPointer) , [Null Pointer Arithmetic](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr30-c#ARR30C.Donotformoruseoutofboundspointersorarraysubscripts-ImproperScaling)               |
| [44](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_44) | Addition or subtraction of a pointer into, or just beyond, an array object and an integer type produces a result that points just beyond the array object and is used as the operand of a unary `*` operator that is evaluated. | [Dereferencing Past the End Pointer](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr30-c#ARR30C.Donotformoruseoutofboundspointersorarraysubscripts-DereferencingPasttheEndPointer) , [Using Past the End Index](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr30-c#ARR30C.Donotformoruseoutofboundspointersorarraysubscripts-UsingPasttheEndIndex) |
| [46](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_46) | An array subscript is out of range, even if an object is apparently accessible with the given subscript, for example, in the lvalue expression `a[1][7]` given the declaration `        int a[4][5]       ` ).                  | [Apparently Accessible Out-of-Range Index](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr30-c#ARR30C.Donotformoruseoutofboundspointersorarraysubscripts-ApparentlyAccessibleOut-of-RangeIndex)                                                                                                  |
| [59](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_59) | An attempt is made to access, or generate a pointer to just past, a flexible array member of a structure when the referenced object provides no elements for that array.                                                                       | [Pointer Past Flexible Array Member](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr30-c#ARR30C.Donotformoruseoutofboundspointersorarraysubscripts-PointerPastFlexibleArrayMember)                                                                                                               |

### <a id="ARR30C.Donotformoruseoutofboundspointersorarraysubscripts-FormingOut-of-BoundsPointer"></a> Noncompliant Code Example (Forming Out-of-Bounds Pointer)

In this noncompliant code example, the function `f()` attempts to validate the `index` before using it as an offset to the statically allocated `table` of integers. However, the function fails to reject negative `index` values. When `index` is less than zero, the behavior of the addition expression in the return statement of the function is [undefined behavior 43](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_43) . On some implementations, the addition alone can trigger a hardware trap. On other implementations, the addition may produce a result that when dereferenced triggers a hardware trap. Other implementations still may produce a dereferenceable pointer that points to an object distinct from `table` . Using such a pointer to access the object may lead to information exposure or cause the wrong object to be modified.

### Compliant Solution

One compliant solution is to detect and reject invalid values of `index` if using them in pointer arithmetic would result in an invalid pointer:

### Compliant Solution

Another slightly simpler and potentially more efficient compliant solution is to use an unsigned type to avoid having to check for negative values while still rejecting out-of-bounds positive values of `index` :

### <a id="ARR30C.Donotformoruseoutofboundspointersorarraysubscripts-DereferencingPasttheEndPointer"></a> Noncompliant Code Example (Dereferencing Past-the-End Pointer)

This noncompliant code example shows the flawed logic in the Windows Distributed Component Object Model (DCOM) Remote Procedure Call (RPC) interface that was exploited by the W32.Blaster.Worm. The error is that the `while` loop in the `GetMachineName()` function (used to extract the host name from a longer string) is not sufficiently bounded. When the character array pointed to by `pwszTemp` does not contain the backslash character among the first `   MAX_COMPUTERNAME_LENGTH_FQDN + 1  ` elements, the final valid iteration of the loop will dereference past the end pointer, resulting in exploitable [undefined behavior 44](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_47) . In this case, the actual exploit allowed the attacker to inject executable code into a running program. Economic damage from the Blaster worm has been estimated to be at least $525 million \[ [Pethia 2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Pethia03) \].

For a discussion of this programming error in the Common Weakness Enumeration database, see [CWE-119](https://cwe.mitre.org/data/definitions/119.html) , "Improper Restriction of Operations within the Bounds of a Memory Buffer," and [CWE-121](https://cwe.mitre.org/data/definitions/121.html) , "Stack-based Buffer Overflow" \[ [MITRE 2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MITRE) \].

### Compliant Solution

In this compliant solution, the `while` loop in the `GetMachineName()` function is bounded so that the loop terminates when a backslash character is found, the null-termination character ( `L'\0'` ) is discovered, or the end of the buffer is reached. Or, as coded, the while loop continues as long as each character is neither a backslash nor a null character and is not at the end of the buffer. This code does not result in a buffer overflow even if no backslash character is found in `wszMachineName` .

This compliant solution is for illustrative purposes and is not necessarily the solution implemented by Microsoft. This particular solution may not be correct because there is no guarantee that a backslash is found.

### <a id="ARR30C.Donotformoruseoutofboundspointersorarraysubscripts-UsingPasttheEndIndex"></a> Noncompliant Code Example (Using Past-the-End Index)

Similar to the [dereferencing-past-the-end-pointer](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr30-c#ARR30C.Donotformoruseoutofboundspointersorarraysubscripts-DereferencingPasttheEndPointer) error, the function `insert_in_table()` in this noncompliant code example uses an otherwise valid index to attempt to store a value in an element just past the end of an array.

First, the function incorrectly validates the index `pos` against the size of the buffer. When `pos` is initially equal to `size` , the function attempts to store `value` in a memory location just past the end of the buffer.

Second, when the index is greater than `size` , the function modifies `size` before growing the size of the buffer. If the call to `realloc()` fails to increase the size of the buffer, the next call to the function with a value of `pos` equal to or greater than the original value of `size` will again attempt to store `value` in a memory location just past the end of the buffer or beyond.

Third, the function violates [INT30-C. Ensure that unsigned integer operations do not wrap](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int30-c) , which could lead to wrapping when 1 is added to `pos` or when `size` is multiplied by the size of `int` .

For a discussion of this programming error in the Common Weakness Enumeration database, see [CWE-122](https://cwe.mitre.org/data/definitions/122.html) , "Heap-based Buffer Overflow," and [CWE-129](https://cwe.mitre.org/data/definitions/129.html) , "Improper Validation of Array Index" \[ [MITRE 2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MITRE) \].

### Compliant Solution

This compliant solution correctly validates the index `pos` by using the `<=` relational operator, ensures the multiplication will not overflow, and avoids modifying `size` until it has verified that the call to `realloc()` was successful:

### <a id="ARR30C.Donotformoruseoutofboundspointersorarraysubscripts-ApparentlyAccessibleOut-of-RangeIndex"></a> Noncompliant Code Example (Apparently Accessible Out-of-Range Index)

This noncompliant code example declares `matrix` to consist of 7 rows and 5 columns in row-major order. The function `init_matrix` iterates over all 35 elements in an attempt to initialize each to the value given by the function argument `x` . However, because multidimensional arrays are declared in C in row-major order, the function iterates over the elements in column-major order, and when the value of `j` reaches the value `COLS` during the first iteration of the outer loop, the function attempts to access element `matrix[0][5]` . Because the type of `matrix` is `int[7][5]` , the `j` subscript is out of range, and the access has [undefined behavior 46](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_49) .

### Compliant Solution

This compliant solution avoids using out-of-range indices by initializing `matrix` elements in the same row-major order as multidimensional objects are declared in C:

### <a id="ARR30C.Donotformoruseoutofboundspointersorarraysubscripts-PointerPastFlexibleArrayMember"></a> Noncompliant Code Example (Pointer Past Flexible Array Member)

In this noncompliant code example, the function `find()` attempts to iterate over the elements of the flexible array member `buf` , starting with the second element. However, because function `g()` does not allocate any storage for the member, the expression `first++` in `find()` attempts to form a pointer just past the end of `buf` when there are no elements. This attempt is [undefined behavior 59](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_) . (See [MSC21-C. Use robust loop termination conditions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc21-c) for more information.)

### Compliant Solution

This compliant solution avoids incrementing the pointer unless a value past the pointer's current value is known to exist:

### <a id="ARR30C.Donotformoruseoutofboundspointersorarraysubscripts-NullPointerArithmetic"></a> Noncompliant Code Example (Null Pointer Arithmetic)

  This noncompliant code example is similar to an [Adobe Flash Player vulnerability](https://www.zerodayinitiative.com/advisories/ZDI-08-032/) that was first exploited in 2008. This code allocates a block of memory and initializes it with some data. The data does not belong at the beginning of the block, which is left uninitialized. Instead, it is placed `offset` bytes within the block. The function ensures that the data fits within the allocated block.

This function fails to check if the allocation succeeds, which is a violation of [ERR33-C. Detect and handle standard library errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err33-c) . If the allocation fails, then `malloc()` returns a null pointer. The null pointer is added to `offset` and passed as the destination argument to `memcpy()` . Because a null pointer does not point to a valid object, the result of the pointer arithmetic is [undefined behavior 43](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_43) .

An attacker who can supply the arguments to this function can exploit it to execute arbitrary code. This can be accomplished by providing an overly large value for `block_size` , which causes `malloc()` to fail and return a null pointer. The `offset` argument will then serve as the destination address to the call to `memcpy()` . The attacker can specify the `data` and `data_size` arguments to provide the address and length of the address, respectively, that the attacker wishes to write into the memory referenced by `offset` . The overall result is that the call to `memcpy()` can be exploited by an attacker to overwrite an arbitrary memory location with an attacker-supplied address, typically resulting in arbitrary code execution.

### Compliant Solution  (Null Pointer Arithmetic)

This compliant solution ensures that the call to `malloc()` succeeds:

## ARR32-C. Ensure size arguments for variable length arrays are in a valid range

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/03.arrays-arr/3.arr32-c.md) — SHA-256 `0b983753459436b78ca582101825605732324e87f8e184e0b55c87c79773d888`。

Variable length arrays (VLAs), a conditionally supported language feature, are essentially the same as traditional C arrays except that they are declared with a size that is not a constant integer expression and can be declared only at block scope or function prototype scope and no linkage. When supported, a variable length array can be declared

where the integer expression `size` and the declaration of `vla` are both evaluated at runtime. If the size argument supplied to a variable length array is not a positive integer value, the behavior is undefined. (See [undefined behavior 72](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_72) .)  Additionally, if the magnitude of the argument is excessive, the program may behave in an unexpected way. An attacker may be able to leverage this behavior to overwrite critical program data \[ [Griffiths 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Griffiths06) \]. The programmer must ensure that size arguments to variable length arrays, especially those derived from untrusted data, are in a valid range.

Because variable length arrays are a conditionally supported feature of C11, their use in portable code should be guarded by testing the value of the macro `__STDC_NO_VLA__` . Implementations that do not support variable length arrays indicate it by setting `__STDC_NO_VLA__` to the integer constant 1.  

### Noncompliant Code Example

In this noncompliant code example, a variable length array of size `size` is declared. The `size` is declared as `size_t` in compliance with [INT01-C. Use size_t for all integer values representing the size of an object](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int01-c) .

However, the value of `size` may be zero or excessive, potentially giving rise to a security [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) .

### Compliant Solution

This compliant solution ensures the `size` argument used to allocate `vla` is in a valid range (between 1 and a programmer-defined maximum); otherwise, it uses an algorithm that relies on dynamic memory allocation. The solution also avoids unsigned integer wrapping that, given a sufficiently large value of `size` , would cause `malloc` to allocate insufficient storage for the array.

### Noncompliant Code Example ( `sizeof` )

The following noncompliant code example defines `A` to be a variable length array and then uses the `sizeof` operator to compute its size at runtime. When the function is called with an argument greater than `   SIZE_MAX / (N1 * sizeof (int))  ` , the runtime `sizeof` expression may wrap around, yielding a result that is smaller than the mathematical product `   N1 * n2 * sizeof (int)  ` . The call to `malloc()` , when successful, will then allocate storage for fewer than `n2` elements of the array, causing one or more of the final `memset()` calls in the `for` loop to write past the end of that storage.

Furthermore, this code also violates [ARR39-C. Do not add or subtract a scaled integer to a pointer](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr39-c) , where `array` is a pointer to the two-dimensional array, where it should really be a pointer to the latter dimension instead. This means that the `memset()` call does out-of-bounds writes on all of its invocations except the first.

### Compliant Solution ( `sizeof` )

This compliant solution prevents `sizeof` wrapping by detecting the condition before it occurs and avoiding the subsequent computation when the condition is detected. The code also uses an additional typedef to fix the type of `array` so that `memset()` never writes past the two-dimensional array.

#### Implementation Details

##### Microsoft

Variable length arrays are not supported by Microsoft compilers.

## ARR36-C. Do not subtract or compare two pointers that do not refer to the same array

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/03.arrays-arr/4.arr36-c.md) — SHA-256 `8c0da8e56e3f464dcd973ed9a20de690def42824fe49ee9237dd0912bff435e7`。

When two pointers are subtracted, both must point to elements of the same array object or just one past the last element of the array object (C Standard, 6.5.7 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \]); the result is the difference of the subscripts of the two array elements. Otherwise, the operation is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See [undefined behavior 45](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_45) .)

Similarly, comparing pointers using the relational operators `<` , `<=` , `>=` , and `>` gives the positions of the pointers relative to each other. Subtracting or comparing pointers that do not refer to the same array is undefined behavior. (See [undefined behavior 45](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_45) and [undefined behavior 50](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_50) .)

Comparing pointers using the equality operators `==` and `!=` has well-defined semantics regardless of whether or not either of the pointers is null, points into the same object, or points one past the last element of an array object or function.

### Noncompliant Code Example

In this noncompliant code example, pointer subtraction is used to determine how many free elements are left in the `nums` array:

This program incorrectly assumes that the `nums` array is adjacent to the `end` variable in memory. A compiler is permitted to insert padding bits between these two variables or even reorder them in memory.

### Compliant Solution

In this compliant solution, the number of free elements is computed by subtracting `next_num_ptr` from the address of the pointer past the `nums` array. While this pointer may not be dereferenced, it may be used in pointer arithmetic.

### Exceptions

**ARR36-C-EX1:** Comparing two pointers to distinct members of the same `struct` object is allowed. Pointers to structure members declared later in the structure compare greater-than pointers to members declared earlier in the structure.

## ARR37-C. Do not add or subtract an integer to a pointer to a non-array object

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/03.arrays-arr/5.arr37-c.md) — SHA-256 `50b1f6b77707d88ecafd09f4f5c2b774ab12299913dd40d06d2e35e191f01134`。

Pointer arithmetic must be performed only on pointers that reference elements of array objects.

The C Standard, 6.5.7 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states the following about pointer arithmetic:

> When an expression that has integer type is added to or subtracted from a pointer, the result has the type of the pointer operand. If the pointer operand points to an element of an array object, and the array is large enough, the result points to an element offset from the original element such that the difference of the subscripts of the resulting and original array elements equals the integer expression.

### Noncompliant Code Example

This noncompliant code example attempts to access structure members using pointer arithmetic. This practice is dangerous because structure members are not guaranteed to be contiguous.

### Compliant Solution

It is possible to use the `->` operator to dereference each structure member:

However, this solution results in code that is hard to write and hard to maintain (especially if there are many more structure members), which is exactly what the author of the noncompliant code example was likely trying to avoid.

### Compliant Solution

A better solution is to define the structure to contain an array member to store the numbers in an array rather than a structure, as in this compliant solution:

Array elements are guaranteed to be contiguous in memory, so this solution is completely portable.

### Exceptions

**ARR37-C-EX1:** Any non-array object in memory can be considered an array consisting of one element. Adding one to a pointer for such an object yields a pointer one element past the array, and subtracting one from that pointer yields the original pointer. This allows for code such as the following:

A more general and safer solution to this problem is to use a flexible array member that guarantees the array that follows the structure is properly aligned by inserting padding, if necessary, between it and the member that immediately precedes it.

## ARR38-C. Guarantee that library functions do not form invalid pointers

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/03.arrays-arr/6.arr38-c.md) — SHA-256 `dfdb11fac0afa9525822c697e38e5b438efeb5746aa18f023cccc5a2f36f0af5`。

C library functions that make changes to arrays or objects take at least two arguments: a pointer to the array or object and an integer indicating the number of elements or bytes to be manipulated. For the purposes of this rule, the element count of a pointer is the size of the object to which it points, expressed by the number of elements that are valid to access. Supplying arguments to such a function might cause the function to form a pointer that does not point into or just past the end of the object, resulting in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

Annex J of the C Standard \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2024) \] states that it is undefined behavior if the "pointer passed to a library function array parameter does not have a value such that all address computations and object accesses are valid." (See [undefined behavior 108](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_108) .)

In the following code,

the element count of the pointer `p` is `   sizeof(arr) / sizeof(arr[0])  ` , that is, `5` . The element count of the pointer `p2` is `sizeof(arr)` , that is, `20` , on implementations where `   sizeof(int) == 4  ` . The element count of the pointer `p3` is `12` on implementations where `   sizeof(int) == 4  ` , because `p3` points two elements past the start of the array `arr` .  The element count of `p4` is treated as though it were `   unsigned char *  ` instead of `   void *  ` , so it is the same as `p2` .

### Pointer + Integer

The following standard library functions take a pointer argument and a size argument, with the constraint that the pointer must point to a valid memory object of at least the number of elements indicated by the size argument.

|                             |                              |                              |                             |
| --------------------------- | ---------------------------- | ---------------------------- | --------------------------- |
| `fgets()`                   | `fgetws()`                   | `mbstowcs()` <sup>1</sup>    | `wcstombs()` <sup>1</sup>   |
| `mbrtoc16()` <sup>2</sup>   | `mbrtoc32()` <sup>2</sup>    | `mbsrtowcs()` <sup>1</sup>   | `wcsrtombs()` <sup>1</sup>  |
| `mbtowc()` <sup>2</sup>     | `mbrtowc()` <sup>2</sup>     | `mblen()`                    | `mbrlen()`                  |
| `memchr()`                  | `wmemchr()`                  | `memset()`                   | `wmemset()`                 |
| `strftime()`                | `wcsftime()`                 | `strxfrm()` <sup>1</sup>     | `wcsxfrm()` <sup>1</sup>    |
| `strncat()` <sup>2></sup>   | `wcsncat()` <sup>2></sup>    | `snprintf()`                 | `vsnprintf()`               |
| `swprintf()`                | `vswprintf()`                | `setvbuf()`                  | `tmpnam_s()`                |
| `snprintf_s()`              | `sprintf_s()`                | `vsnprintf_s()`              | `vsprintf_s()`              |
| `gets_s()`                  | `getenv_s()`                 | `wctomb_s()`                 | `mbstowcs_s()` <sup>3</sup> |
| `wcstombs_s()` <sup>3</sup> | `memcpy_s()` <sup>3</sup>    | `memmove_s()` <sup>3</sup>   | `strncpy_s()` <sup>3</sup>  |
| `strncat_s()` <sup>3</sup>  | `strtok_s()` <sup>2></sup>   | `strerror_s()`               | `strnlen_s()`               |
| `asctime_s()`               | `ctime_s()`                  | `snwprintf_s()`              | `swprintf_s()`              |
| `vsnwprintf_s()`            | `vswprintf_s()`              | `wcsncpy_s()` <sup>3</sup>   | `wmemcpy_s()` <sup>3</sup>  |
| `wmemmove_s()` <sup>3</sup> | `wcsncat_s()` <sup>3</sup>   | `wcstok_s()` <sup>2></sup>   | `wcsnlen_s()`               |
| `wcrtomb_s()`               | `mbsrtowcs_s()` <sup>3</sup> | `wcsrtombs_s()` <sup>3</sup> | `memset_explicit()`         |

<sup>1</sup> Takes two pointers and an integer, but the integer specifies the element count only of the output buffer, not of the input buffer.  
<sup>2</sup> Takes two pointers and an integer, but the integer specifies the element count only of the input buffer, not of the output buffer.  
<sup>3</sup> Takes two pointers and two integers; each integer corresponds to the element count of one of the pointers.  

For calls that take a pointer and an integer size, the given size should not be greater than the element count of the pointer.

### Noncompliant Code Example (Element Count)

In this noncompliant code example, the incorrect element count is used in a call to `wmemcpy()` . The `sizeof` operator returns the size expressed in bytes, but `wmemcpy()` uses an element count based on `   wchar_t *  ` .

### Compliant Solution (Element Count)

When using functions that operate on pointed-to regions, programmers must always express the integer size in terms of the element count expected by the function. For example, `memcpy()` expects the element count expressed in terms of `   void *  ` , but `wmemcpy()` expects the element count expressed in terms of `   wchar_t *  ` .  Instead of the `sizeof` operator, functions that return the number of elements in the string are called, which matches the expected element count for the copy functions. In the case of this compliant solution, where the argument is an array `A` of type `T` , the expression `   sizeof(A) / sizeof(T)  ` , or equivalently `   sizeof(A) / sizeof(*A)  ` , can be used to compute the number of elements in the array.

### Noncompliant Code Example (Pointer + Integer)

This noncompliant code example assigns a value greater than the number of bytes of available memory to `n` , which is then passed to `memset()` :

### Compliant Solution (Pointer + Integer)

This compliant solution ensures that the value of `n` is not greater than the number of bytes of the dynamic memory pointed to by the pointer `p` :

### Noncompliant Code Example (Pointer + Integer)

In this noncompliant code example, the element count of the array `a` is `ARR_SIZE` elements. Because `memset()` expects a byte count, the size of the array is scaled incorrectly by `sizeof(int)` instead of `sizeof(long)` , which can form an invalid pointer on architectures where `   sizeof(int) != sizeof(long)  ` .

### Compliant Solution (Pointer + Integer)

In this compliant solution, the element count required by `memset()` is properly calculated without resorting to scaling:

### Two Pointers + One Integer

The following standard library functions take two pointer arguments and a size argument, with the constraint that both pointers must point to valid memory objects of at least the number of elements indicated by the size argument.

|  |  |  |  |
| --- | --- | --- | --- |
| ` memcpy() ` | ` wmemcpy() ` | ` memmove() ` | ` wmemmove() ` |
| ` strncpy() ` | ` wcsncpy() ` | ` memcmp() ` | ` wmemcmp() ` |
| ` strncmp() ` | ` wcsncmp() ` | ` strcpy_s() ` | ` wcscpy_s() ` |
| ` strcat_s() ` | ` wcscat_s() ` |  |  |

For calls that take two pointers and an integer size, the given size should not be greater than the element count of either pointer.

### Noncompliant Code Example (Two Pointers + One Integer)

In this noncompliant code example, the value of `n` is incorrectly computed, allowing a read past the end of the object referenced by `q` :

### Compliant Solution (Two Pointers + One Integer)

This compliant solution ensures that `n` is equal to the size of the character array:

### One Pointer + Two Integers

The following standard library functions take a pointer argument and two size arguments, with the constraint that the pointer must point to a valid memory object containing at least as many bytes as the product of the two size arguments.

|  |  |  |  |
| --- | --- | --- | --- |
| ` bsearch() ` | ` bsearch_s() ` | ` qsort() ` | ` qsort_s() ` |
| ` fread() ` | ` fwrite() ` | ` ` |  |

For calls that take a pointer and two integers, one integer represents the number of bytes required for an individual object, and a second integer represents the number of elements in the array. The resulting product of the two integers should not be greater than the element count of the pointer were it expressed as an `   unsigned char *  ` .

### Noncompliant Code Example (One Pointer + Two Integers)

This noncompliant code example allocates a variable number of objects of type `   struct obj  ` . The function checks that `num_objs` is small enough to prevent wrapping, in compliance with [INT30-C. Ensure that unsigned integer operations do not wrap](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int30-c) . The size of `   struct obj  ` is assumed to be 16 bytes to account for padding to achieve the assumed alignment of `   long long  ` . However, the padding typically depends on the target architecture, so this object size may be incorrect, resulting in an incorrect element count.

### Compliant Solution (One Pointer + Two Integers)

This compliant solution uses the `sizeof` operator to correctly provide the object size and `num_objs` to provide the element count:

### Noncompliant Code Example (One Pointer + Two Integers)

In this noncompliant code example, the function `f()` calls `fread()` to read `nitems` of type `wchar_t` , each `size` bytes in size, into an array of `BUFFER_SIZE` elements, `wbuf` . However, the expression used to compute the value of `nitems` fails to account for the fact that, unlike the size of `char` , the size of `wchar_t` may be greater than 1. Consequently, `fread()` could attempt to form pointers past the end of `wbuf` and use them to assign values to nonexistent elements of the array. Such an attempt is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See [undefined behavior 109](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_109) .) A likely consequence of this undefined behavior is a buffer overflow. For a discussion of this programming error in the Common Weakness Enumeration database, see [CWE-121](https://cwe.mitre.org/data/definitions/121.html) , "Stack-based Buffer Overflow," and [CWE-805](https://cwe.mitre.org/data/definitions/805.html) , "Buffer Access with Incorrect Length Value."

### Compliant Solution (One Pointer + Two Integers)

This compliant solution correctly computes the maximum number of items for `fread()` to read from the file:

### Noncompliant Code Example (Heartbleed)

CERT vulnerability [720951](https://www.kb.cert.org/vuls/id/720951) describes a vulnerability in OpenSSL versions 1.0.1 through 1.0.1f, popularly known as "Heartbleed." This vulnerability allows an attacker to steal information that under normal conditions would be protected by Secure Socket Layer/Transport Layer Security (SSL/TLS) encryption.

Despite the seriousness of the vulnerability, Heartbleed is the result of a common programming error and an apparent lack of awareness of secure coding principles. Following is the vulnerable code:

<div class="syntaxhighlighter nogutter c">

  

This code processes a "heartbeat" packet from a client. As specified in [RFC 6520](https://tools.ietf.org/html/rfc6520) , when the program receives a heartbeat packet, it must echo the packet's data back to the client. In addition to the data, the packet contains a length field that conventionally indicates the number of bytes in the packet data, but there is nothing to prevent a malicious packet from lying about its data length.

</div>

The `p` pointer, along with `payload` and `p1` , contains data from a packet. The code allocates a `buffer` sufficient to contain `payload` bytes, with some overhead, then copies `payload` bytes starting at `p1` into this buffer and sends it to the client. Notably absent from this code are any checks that the payload integer variable extracted from the heartbeat packet corresponds to the size of the packet data. Because the client can specify an arbitrary value of `payload` , an attacker can cause the server to read and return the contents of memory beyond the end of the packet data, which violates [INT04-C. Enforce limits on integer values originating from tainted sources](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int04-c) . The resulting call to `memcpy()` can then copy the contents of memory past the end of the packet data and the packet itself, potentially exposing sensitive data to the attacker. This call to `memcpy()` violates [ARR38-C. Guarantee that library functions do not form invalid pointers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr38-c) . A version of ARR38-C also appears in [ISO/IEC TS 17961:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECTS17961) , "Forming invalid pointers by library functions \[libptr\]." This rule would require a conforming analyzer to diagnose the Heartbleed vulnerability.

  

### Compliant Solution (Heartbleed)

OpenSSL version 1.0.1g contains the following patch, which guarantees that `payload` is within a valid range. The range is limited by the size of the input record.

## ARR39-C. Do not add or subtract a scaled integer to a pointer

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/03.arrays-arr/7.arr39-c.md) — SHA-256 `09e85e2936195e943f42aa5c1fb808401dd21fed2f46823428c016177047eef1`。

Pointer arithmetic is appropriate only when the pointer argument refers to an array (see [ARR37-C. Do not add or subtract an integer to a pointer to a non-array object](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr37-c) ), including an array of bytes. When performing pointer arithmetic, the size of the value to add to or subtract from a pointer is automatically scaled to the size of the type of the referenced array object. Adding or subtracting a scaled integer value to or from a pointer is invalid because it may yield a pointer that does not point to an element within or one past the end of the array. (See [ARR30-C. Do not form or use out-of-bounds pointers or array subscripts](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/arrays-arr/arr30-c) .)

Adding a pointer to an array of a type other than character to the result of the `sizeof` operator or `offsetof` macro, which returns a size and an offset, respectively, violates this rule. However, adding an array pointer to the number of array elements, for example, by using the `arr[sizeof(arr)/sizeof(arr[0])])` idiom, is allowed provided that `arr` refers to an array and not a pointer.

### Noncompliant Code Example

In this noncompliant code example, `sizeof(buf)` is added to the array `buf` . This example is noncompliant because `sizeof(buf)` is scaled by `int` and then scaled again when added to `buf` .

### Compliant Solution

This compliant solution uses an unscaled integer to obtain a pointer to the end of the array:

### Noncompliant Code Example

In this noncompliant code example, `skip` is added to the pointer `s` . However, `skip` represents the byte offset of `ull_b` in `   struct big  ` . When added to `s` , `skip` is scaled by the size of `   struct big  ` .

### Compliant Solution

This compliant solution uses an `   unsigned char *  ` to calculate the offset instead of using a `   struct big *  ` , which would result in scaled arithmetic:

### Noncompliant Code Example

In this noncompliant code example, `   wcslen(error_msg) * sizeof(wchar_t)  ` bytes are scaled by the size of `wchar_t` when added to `error_msg` :

### Compliant Solution

This compliant solution does not scale the length of the string; `wcslen()` returns the number of characters and the addition to `error_msg` is scaled:
