# CERT C — MEM

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## MEM00-C. Allocate and free memory in the same module, at the same level of abstraction

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/13.memory-management-mem/02.mem00-c.md) — SHA-256 `ff7527a6acfde261c27c3656c9990a06d9de1d7b10ee38a2606194de70c50981`。

Dynamic memory management is a common source of programming flaws that can lead to security [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) . Poor memory management can lead to security issues, such as heap-buffer overflows, dangling pointers, and double-free issues \[ [Seacord 2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Seacord2013) \]. From the programmer's perspective, memory management involves allocating memory, reading and writing to memory, and deallocating memory.

Allocating and freeing memory in different modules and levels of abstraction may make it difficult to determine when and if a block of memory has been freed, leading to programming defects, such as memory leaks, double-free [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) , accessing freed memory, or writing to freed or unallocated memory.

To avoid these situations, memory should be allocated and freed at the same level of abstraction and, ideally, in the same code module. This includes the use of the following memory allocation and deallocation functions described in subclause 7.23.3 of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \]:

Failing to follow this recommendation has led to real-world vulnerabilities. For example, freeing memory in different modules resulted in a vulnerability in MIT Kerberos 5 \[ [MIT 2004](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MIT04) \]. The MIT Kerberos 5 code in this case contained error-handling logic, which freed memory allocated by the ASN.1 decoders if pointers to the allocated memory were non-null. However, if a detectable error occurred, the ASN.1 decoders freed the memory that they had allocated. When some library functions received errors from the ASN.1 decoders, they also attempted to free the same memory, resulting in a double-free vulnerability.

### Noncompliant Code Example

This noncompliant code example shows a double-free vulnerability resulting from memory being allocated and freed at differing levels of abstraction. In this example, memory for the `list` array is allocated in the `process_list()` function. The array is then passed to the `verify_size()` function that performs error checking on the size of the list. If the size of the list is below a minimum size, the memory allocated to the list is freed, and the function returns to the caller. The calling function then frees this same memory again, resulting in a double-free and potentially exploitable vulnerability.

The call to free memory in the `verify_size()` function takes place in a subroutine of the `process_list()` function, at a different level of abstraction from the allocation, resulting in a violation of this recommendation. The memory deallocation also occurs in error-handling code, which is frequently not as well tested as "green paths" through the code.

### Compliant Solution

To correct this problem, the error-handling code in `verify_size()` is modified so that it no longer frees `list` . This change ensures that `list` is freed only once, at the same level of abstraction, in the `process_list()` function.

## MEM01-C. Store a new value in pointers immediately after free()

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/13.memory-management-mem/03.mem01-c.md) — SHA-256 `b6e4311f8c998477fbde9be334b95dd0411c96ecfa237e6dd6ce7487d17337fa`。

Dangling pointers can lead to exploitable double-free and access-freed-memory [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) . A simple yet effective way to eliminate dangling pointers and avoid many memory-related vulnerabilities is to set pointers to `NULL` after they are freed or to set them to another valid object.

### Noncompliant Code Example

In this noncompliant code example, the type of a message is used to determine how to process the message itself. It is assumed that `message_type` is an integer and `message` is a pointer to an array of characters that were allocated dynamically. If `message_type` equals `value_1` , the message is processed accordingly. A similar operation occurs when `message_type` equals `value_2` . However, if `   message_type == value_1  ` evaluates to true and `   message_type == value_2  ` also evaluates to true, then `message` is freed twice, resulting in a double-free vulnerability.

### Compliant Solution

Calling `free()` on a null pointer results in no action being taken by `free()` . Setting `message` to `NULL` after it is freed eliminates the possibility that the `message` pointer can be used to free the same memory more than once.

### Exceptions

**MEM01-C-EX1:** If a nonstatic variable goes out of scope immediately following the `free()` , it is not necessary to clear its value because it is no longer accessible.

## MEM02-C. Immediately cast the result of a memory allocation function call into a pointer to the allocated type

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/13.memory-management-mem/04.mem02-c.md) — SHA-256 `cc1e239e5404f2122fed05b82d4fd923fa914c72a99e4b7366396ef76a46aac5`。

An object of type `   void *  ` is a generic data pointer. It can point to any data object. For any incomplete or object type `T` , C permits implicit conversion from `   T *  `to`   void *  ` or from `   void *  `to`   T *  ` .  C Standard memory allocation functions `aligned_alloc()` , `malloc()` , `calloc()` , and `realloc()` use `   void *  ` to declare parameters and return types of functions designed to work for objects of different types.

For example, the C library declares `malloc()` as

Calling `malloc(s)` allocates memory for an object whose size is `s` and returns either a null pointer or a pointer to the allocated memory. A program can implicitly convert the pointer that `malloc()` returns into a different pointer type.

Because objects returned by the C Standard memory allocation functions are implicitly converted into any object type, we recommend casting the results of these functions into a pointer of the allocated type because it increases the chances that the compiler will catch and diagnose a mismatch between the intended type of the object and the actual type of the object.

### Noncompliant Code Example

The argument to `malloc()` can be *any* value of (unsigned) type `size_t` . If the program uses the allocated storage to represent an object (possibly an array) whose size is greater than the requested size, the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . The implicit pointer conversion lets this slip by without complaint from the compiler.

Consider the following example:

An [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) may add padding to a gadget or widget so that `sizeof(gadget)` equals `sizeof(widget)` , but this is highly unlikely. More likely, `sizeof(gadget)` is less than `sizeof(widget)` . In that case,

quietly assigns `p` to point to storage too small for a widget. The subsequent assignments to `p->i` and `p->d` will most likely produce memory overruns.

Casting the result of `malloc()` to the appropriate pointer type enables the compiler to catch subsequent inadvertent pointer conversions. When allocating individual objects, the "appropriate pointer type" is a pointer to the type argument in the `sizeof` expression passed to `malloc()` .

In this code example, `malloc()` allocates space for a `gadget` , and the cast immediately converts the returned pointer to a `   gadget *  ` :

This lets the compiler detect the invalid assignment because it attempts to convert a `   gadget *  ` into a `   widget *  ` .

### Compliant Solution (Hand Coded)

This compliant solution repeats the same type in the `sizeof` expression and the pointer cast:

### Compliant Solution (Macros)

Repeating the same type in the `sizeof` expression and the pointer cast is easy to do but still invites errors. Packaging the repetition in a macro, such as

further reduces the possibility of error.

Here, the entire allocation expression (to the right of the assignment operator) allocates storage for a `widget` and returns a `   widget *  ` . If `p` were not a `   widget *  ` , the compiler would complain about the assignment.

When allocating an array with `N` elements of type `T` , the appropriate type in the cast expression is still `   T *  ` , but the argument to `malloc()` should be of the form `   N * sizeof(T)  ` . Again, packaging this form as a macro, such as

reduces the chance of error in an allocation expression.

A small collection of macros can provide secure implementations for common uses for the standard memory allocation functions. The omission of a `REALLOC()` macro is intentional (see [EXP39-C. Do not access a variable through a pointer of an incompatible type](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp39-c) ).

The following is an example:

If one or more of the operands to the multiplication operations used in many of these macro definitions can be influenced by untrusted data, these operands should be checked for overflow before the macro is invoked (see [INT32-C. Ensure that operations on signed integers do not result in overflow](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int32-c) ).

The use of type-generic function-like macros is an allowed exception (PRE00-C-EX4) to [PRE00-C. Prefer inline or static functions to function-like macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre00-c) .

### Exceptions

**MEM02-C-EX1:** Do not immediately cast the results of `malloc()` for code that will be compiled using a C90-conforming compiler because it is possible for the cast to hide a more critical defect (see [DCL31-C. Declare identifiers before using them](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl31-c) for a code example that uses `malloc()` without first declaring it).

## MEM03-C. Clear sensitive information stored in reusable resources

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/13.memory-management-mem/05.mem03-c.md) — SHA-256 `fc9105a0954dae4f48bacac5e7ccf4f47f77483604bbb9b4a445fe05614a6dce`。

Sensitive data stored in reusable resources may be inadvertently leaked to a less privileged user or attacker if not properly cleared. Examples of reusable resources include

-   Dynamically allocated memory
-   Statically allocated memory
-   Automatically allocated (stack) memory
-   Memory caches
-   Disk
-   Disk caches

The manner in which sensitive information can be properly cleared varies depending on the resource type and platform.

### Noncompliant Code Example ( `free()` )

Dynamic memory managers are not required to clear freed memory and generally do not because of the additional runtime overhead. Furthermore, dynamic memory managers are free to reallocate this same memory. As a result, it is possible to accidentally leak sensitive information if it is not cleared before calling a function that frees dynamic memory. Programmers also cannot rely on memory being cleared during allocation.

To prevent information leakage, sensitive information must be cleared from dynamically allocated buffers before they are freed. Calling `free()` on a block of dynamic memory causes the space to be deallocated; that is, the memory block is made available for future allocation. However, the data stored in the block of memory to be recycled may be preserved. If this memory block contains sensitive information, that information may be unintentionally exposed.

In this noncompliant example, sensitive information stored in the dynamically allocated memory referenced by `secret` is copied to the dynamically allocated buffer, `new_secret` , which is processed and eventually deallocated by a call to `free()` . Because the memory is not cleared, it may be reallocated to another section of the program where the information stored in `new_secret` may be unintentionally leaked.

### Compliant Solution (C23)

To prevent information leakage, dynamic memory containing sensitive information should be [sanitized](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sanitize) before being freed. Sanitization is commonly accomplished by clearing the allocated space (that is, filling the space with `'\0'` characters).

The `calloc()` function ensures that the newly allocated memory has also been cleared. Because `sizeof(char)` is guaranteed to be 1, this solution does not need to check for a numeric overflow as a result of using `calloc()` . (See [MEM07-C. Ensure that the arguments to calloc(), when multiplied, do not wrap](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/memory-management-mem/mem07-c) .)

See [MSC06-C. Beware of compiler optimizations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc06-c) for a discussion of how a compiler can remove a call to `memset()`, and of alternatives such as `memset_explicit()`.

### Noncompliant Code Example ( `realloc()` )

Reallocating memory using `realloc()` can have the same problem as freeing memory. The `realloc()` function deallocates the old object and returns a pointer to a new object. U sing `realloc()` to resize dynamic memory may inadvertently expose sensitive information, or it may allow heap inspection, as described in *Fortify Taxonomy: Software Security Errors* \[ [Fortify 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Fortify06) \] and NIST's *Source Code Analysis Tool Functional Specification* \[ [Black 2007](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Black07) \].

In this example, when `realloc()` is called, it may allocate a new, larger object, copy the contents of `secret` to this new object, `free()` the original object, and assign the newly allocated object to `secret` . However, the contents of the original object may remain in memory.

The `secret_size` is tested to ensure that the integer multiplication ( `   secret_size * 2  ` ) does not result in an integer overflow. (See [INT30-C. Ensure that unsigned integer operations do not wrap](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int30-c) .)

### Compliant Solution (C23)

A compliant program cannot rely on `realloc()` because it is not possible to clear the memory before the call. Instead, a custom function must be used that operates similarly to `realloc()` but sanitizes sensitive information as heap-based buffers are resized. Again, sanitization is done by overwriting the space to be deallocated with `'\0'` characters.

The `calloc()` function ensures that the newly allocated memory is also cleared. Because `sizeof(char)` is guaranteed to be 1, this solution does not need to check for a numeric overflow as a result of using `calloc()` . (See [MEM07-C. Ensure that the arguments to calloc(), when multiplied, do not wrap](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/memory-management-mem/mem07-c) .)

## MEM04-C. Beware of zero-length allocations

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/13.memory-management-mem/06.mem04-c.md) — SHA-256 `05fff350190da5d2a31641562fb77a19b5fb1f21ad847a50064a3f4a9d35f012`。

When the requested size is 0, the behavior of the memory allocation functions `malloc()` , `calloc()` , and `realloc()` is [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) . Subclause 7.22.3 of the C Standard \[ [ISO/IEC 9899:2011](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AABibliography-ISOIEC9899-2011) \] states:

> If the size of the space requested is zero, the behavior is implementation-defined: either a null pointer is returned, or the behavior is as if the size were some nonzero value, except that the returned pointer shall not be used to access an object.

In addition, the amount of storage allocated by a successful call to the allocation function when 0 bytes was requested is [unspecified](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unspecifiedbehavior) . See [unspecified behavior 41](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/dd-unspecified-behavior#DD.UnspecifiedBehavior-usb_40) in subclause J.1 of the C Standard.

In cases where the memory allocation functions return a non-null pointer, reading from or writing to the allocated memory area results in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . Typically, the pointer refers to a zero-length block of memory consisting entirely of control structures. Overwriting these control structures damages the data structures used by the memory manager.

### Noncompliant Code Example ( `malloc()` )

The result of calling `malloc(0)` to allocate 0 bytes is implementation-defined. In this example, a dynamic array of integers is allocated to store `size` elements. However, if `size` is 0, the call to `malloc(size)` may return a reference to a block of memory of size 0 instead of a null pointer. When (nonempty) data is copied to this location, a heap-buffer overflow occurs.

### Compliant Solution ( `malloc()` )

To ensure that 0 is never passed as a size argument to `malloc()` , `size` is checked to confirm it has a positive value:

### Noncompliant Code Example ( `realloc()` )

The `realloc()` function deallocates the old object and returns a pointer to a new object of a specified size. If memory for the new object cannot be allocated, the `realloc()` function does not deallocate the old object, and its value is unchanged. If the `realloc()` function returns `NULL` , failing to free the original memory will result in a memory leak. As a result, the following idiom is often recommended for reallocating memory:

However, this commonly recommended idiom has problems with zero-length allocations. If the value of `nsize` in this example is 0, the standard allows the option of either returning a null pointer or returning a pointer to an invalid (for example, zero-length) object. In cases where the `realloc()` function frees the memory but returns a null pointer, execution of the code results in a double-free vulnerability. If the `realloc()` function returns a non-null value, but the size was 0, the returned memory will be of size 0, and a heap overflow will occur if nonempty data is copied there.

#### Implementation Details

If this noncompliant code is compiled with GCC 3.4.6 and linked with libc 2.3.4, invoking `   realloc(p, 0)  ` returns a non-null pointer to a zero-sized object (the same as `malloc(0)` ). However, if the same code is compiled with either Microsoft Visual Studio or GCC 4.1.0 , `   realloc(p, 0)  ` returns a null pointer, resulting in a double-free vulnerability.

### Compliant Solution ( `realloc()` )

This compliant solution does not pass a size argument of zero to the `realloc()` function:

## MEM05-C. Avoid large stack allocations

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/13.memory-management-mem/07.mem05-c.md) — SHA-256 `158daf14fe72b5c20c32b95efe98a66a4860e76a58a687b13a2a1f497c173b8c`。

Avoid excessive stack allocations, particularly in situations where the growth of the stack can be controlled or influenced by an attacker. See [INT04-C. Enforce limits on integer values originating from tainted sources](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int04-c) for more information on preventing attacker-controlled integers from exhausting memory.

### Noncompliant Code Example

The C Standard includes support for variable length arrays (VLAs). If the array length is derived from an [untrusted data](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-untrusteddata) source, an attacker can cause the process to perform an excessive allocation on the stack.

This noncompliant code example temporarily stores data read from a source file into a buffer. The buffer is allocated on the stack as a VLA of size `bufsize` . If `bufsize` can be controlled by a malicious user, this code can be [exploited](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-exploit) to cause a [denial-of-service attack](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-denial-of-serviceattack) :

The BSD extension function `alloca()` behaves in a similar fashion to VLAs; its use is not recommended \[ [Loosemore 2007](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Loosemore07) \].

### Compliant Solution

This compliant solution replaces the VLA with a call to `malloc()` . If `malloc()` fails, the return value can be checked to prevent the program from terminating abnormally.

### Noncompliant Code Example

Recursion can also lead to large stack allocations. Recursive functions must ensure that they do not exhaust the stack as a result of excessive recursions.

This noncompliant implementation of the Fibonacci function uses recursion:

The amount of stack space needed grows linearly with respect to the parameter `n` . Large values of `n` have been shown to cause [abnormal program termination](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) .

### Compliant Solution

This implementation of the Fibonacci functions eliminates the use of recursion:

Because there is no recursion, the amount of stack space needed does not depend on the parameter `n` , greatly reducing the risk of stack overflow.

## MEM06-C. Ensure that sensitive data is not written out to disk

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/13.memory-management-mem/08.mem06-c.md) — SHA-256 `9f06f5a1f8cf4e816a6f5060eb4efc3c632e892bbf718fe7c2170e356a5e9b8c`。

Developers should take steps to prevent sensitive information such as passwords, cryptographic keys, and other secrets from being inadvertently leaked. Preventive measures include attempting to keep such data from being written to disk.

Two common mechanisms by which data is inadvertently written to disk are *swapping* and *core dumps* .

Many general-purpose operating systems implement a virtual-memory-management technique called *paging* (also called *swapping* ) to transfer pages between main memory and an auxiliary store, such as a disk drive. This feature is typically implemented as a task running in the kernel of the operating system, and its operation is invisible to the running program.

A core dump is the recorded state of process memory written to disk for later examination by a debugger. Core dumps are typically generated when a program has [terminated abnormally](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) , either through an error resulting in a crash or by receiving a signal that causes such a termination.

The POSIX standard system call for controlling resource limits, `setrlimit()` , can be used to disable the creation of core dumps, which prevents an attacker with the ability to halt the program from gaining access to sensitive data that might be contained in the dump.

### Noncompliant Code Example (C23)

In this noncompliant code example, sensitive information is supposedly stored in the dynamically allocated buffer, `secret` , which is processed and eventually cleared by a call to `memset_explicit()` . The memory page containing `secret` can be swapped out to disk. If the program crashes before the call to `memset_explicit()` completes, the information stored in `secret` may be stored in the core dump.

### Compliant Solution (C23, POSIX)

To prevent the information from being written to a core dump, the size of core dumps that the program will generate should be set to 0 using `setrlimit()` :

### Compliant Solution (C23, Privileged Process, POSIX)

The added security from using `mlock()` is limited. (See the sidebar by Nick Stoughton.)

Processes with elevated privileges can disable paging by locking memory in place using the POSIX `mlock()` function \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \]. Disabling paging ensures that memory is never copied to the hard drive, where it may be retained indefinitely in nonvolatile storage.

This compliant solution not only disables the creation of core files but also ensures that the buffer is not swapped to hard disk:

### Compliant Solution (Windows)

Windows processes can disable paging by locking memory in place using `VirtualLock()` \[ [MSDN](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-AA.CReferences-MSDN) \]:

Note that locking pages of memory on Windows may fail because the operating system allows the process to lock only a small number of pages. If an application requires additional locked pages, the [`SetProcessWorkingSetSize()`](https://msdn.microsoft.com/en-us/library/windows/desktop/ms686234(v=vs.85).aspx) API can be used to increase the application's minimum working set size. Locking pages has severe performance consequences and should be used sparingly.

## MEM07-C. Ensure that the arguments to calloc(), when multiplied, do not wrap

**Recommendation / 条件付き採用**。原典のDeprecatedはC23だけを要求するコードへの非適用。C23より前への適合が必要なら本文どおり適用する。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/13.memory-management-mem/09.mem07-c.md) — SHA-256 `11bf3344b2f80fc337c5c51187173a9d67f6d721fca5f491112bb01c0f830365`。

Deprecated

This guideline does not apply to code that need conform only to C23. Code that must conform to older versions of the C standard should still comply with this guideline.

The `calloc()` function takes two arguments: the number of elements to allocate and the storage size of those elements. Typically, `calloc()` [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementations) multiply these arguments to determine how much memory to allocate. Historically, some implementations failed to check whether out-of-bounds results silently wrapped \[ [RUS-CERT Advisory 2002-08:02](https://seclists.org/bugtraq/2002/Aug/91) \]. If the result of multiplying the number of elements to allocate and the storage size wraps, less memory is allocated than was requested. As a result, it is necessary to ensure that these arguments, when multiplied, do not wrap.

Modern implementations of the C standard library should check for wrap. If the `calloc()` function implemented by the libraries used for a particular implementation properly handles [unsigned integer wrapping](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unsignedintegerwrapping) (in conformance with [INT30-C. Ensure that unsigned integer operations do not wrap](https://www.securecoding.cert.org/confluence/display/seccode/INT30-C.+Ensure+that+unsigned+integer+operations+do+not+wrap) ) when multiplying the number of elements to allocate and the storage size, that is sufficient to comply with this recommendation and no further action is required.

### Noncompliant Code Example

In this noncompliant example, the user-defined function `get_size()` (not shown) is used to calculate the size requirements for a dynamic array of `   long int  ` that is assigned to the variable `num_elements` . When `calloc()` is called to allocate the buffer, `num_elements` is multiplied by `sizeof(long)` to compute the overall size requirements. If the number of elements multiplied by the size cannot be represented as a `size_t` , then `calloc()` may allocate a buffer of insufficient size. When data is copied to that buffer, an overflow may occur.

### Compliant Solution

In this compliant solution, the two arguments `num_elements` and `sizeof(long)` are checked before the call to `calloc()` to determine if wrapping will occur:

Note that the maximum amount of allocatable memory is typically limited to a value less than `SIZE_MAX` (the maximum value of `size_t` ). Always check the return value from a call to any memory allocation function in compliance with [ERR33-C. Detect and handle standard library errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err33-c) .

## MEM10-C. Define and use a pointer validation function

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/13.memory-management-mem/10.mem10-c.md) — SHA-256 `c6205d01129e0aa2f60c9dcf2c7465cea23a43875c328b4d94571af4deb9e16b`。

Many functions accept pointers as arguments. If the function dereferences an [invalid pointer](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-invalidpointer) (as in [EXP34-C. Do not dereference null pointers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp34-c) ) or reads or writes to a pointer that does not refer to an object, the results are [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefined) . Typically, the program will [terminate abnormally](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) when an invalid pointer is dereferenced, but it is possible for an invalid pointer to be dereferenced and its memory changed without abnormal termination \[ [Jack 2007](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Jack07) \]. Such programs can be difficult to debug because of the difficulty in determining if a pointer is [valid](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-validpointer) .

One way to eliminate invalid pointers is to define a function that accepts a pointer argument and indicates whether or not the pointer is valid for some definition of valid. For example, the following function declares any pointer to be valid except `NULL` :

Some platforms have platform-specific pointer validation tools.

The following code relies on the `_etext` address, defined by the loader as the first address following the program text on many platforms, including AIX, Linux, QNX, IRIX, and Solaris. It is not POSIX-compliant, nor is it available on Windows.

On a Linux platform, this program produces the following output:

The `valid()` function does not guarantee validity; it only identifies null pointers and pointers to functions as invalid. However, it can be used to catch a substantial number of problems that might otherwise go undetected.

### Noncompliant Code Example

In this noncompliant code example, the `incr()` function increments the value referenced by its argument. It also ensures that its argument is not a null pointer. But the pointer could still be invalid, causing the function to corrupt memory or terminate abnormally.

### Compliant Solution

This `incr()` function can be improved by using the `valid()` function. The resulting implementation is less likely to dereference an invalid pointer or write to memory that is outside the bounds of a valid object.

The `valid()` function can be implementation dependent and perform additional, platform-dependent checks when possible. In the worst case, the `valid()` function may only perform the same null-pointer check as the noncompliant code example. However, on platforms where additional pointer validation is possible, the use of a `valid()` function can provide checks.

## MEM11-C. Do not assume infinite heap space

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/13.memory-management-mem/11.mem11-c.md) — SHA-256 `fd1426528e6bfdff2b41d038fb8a77f9f70c52f277a169421e1530b319dbd467`。

Memory is a limited resource and can be exhausted. Available memory is typically bounded by the sum of the amount of physical memory and the swap space allocated to the operating system by the administrator. For example, a system with 1GB of physical memory configured with 2GB of swap space may be able to allocate, at most, 3GB of heap space total to all running processes (minus the size of the operating system itself and the text and data segments of all running processes). Once all virtual memory has been allocated, requests for more memory will fail. As discussed in [ERR33-C. Detect and handle standard library errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err33-c) , programs that fail to check for and properly handle memory allocation failures will have [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) and are likely to crash when heap space is exhausted. Heap exhaustion can result from

-   A memory leak
-   An infinite loop
-   The program requiring more memory than is present by default in the heap
-   Incorrect implementation of common data structures (for example, hash tables or vectors)
-   Overall system memory being exhausted, possibly because of other processes
-   The maximum size of a process's data segment being exceeded

If `malloc()` is unable to return the requested memory, it returns `NULL` instead.

However, simply checking for and handling memory allocation failures may not be sufficient. Programs such as long-servers that manipulate large data sets need to be designed in a way that permits them to deliver their services when system resources, including the heap, are in short supply. Making use of additional storage devices, such as disk space or databases, is essential in such systems.

### Noncompliant Code Example

This example places no upper bound on the memory space required. As a result, the program can easily exhaust the heap. A heap error will be generated if the heap continues to be populated, even if there is no space available.

### Compliant Solution

If the objects or data structures are large enough to potentially cause heap exhaustion, the programmer should consider using databases instead to ensure that records are written to the disk and that the data structure does not outgrow the heap.

In the previous noncompliant code example, the user can reuse a single `long` variable to store the input and write that value into a simple database containing a table `User` with a field `userID` along with any other required fields. This prevents the heap from being exhausted.

## MEM12-C. Consider using a goto chain when leaving a function on error when using and releasing resources

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/13.memory-management-mem/12.mem12-c.md) — SHA-256 `3dc8428785fb0ef34a3c1536b480b50447ecb74b3a9e590f351d668fe15bc2db`。

Many functions require the allocation of multiple resources. Failing and returning somewhere in the middle of this function without freeing all of the allocated resources could produce a memory leak. It is a common error to forget to free one (or all) of the resources in this manner, so a `goto` chain is the simplest and cleanest way to organize exits while preserving the order of freed resources.

### Noncompliant Code Example (POSIX)

In this noncompliant example, exit code is written for every instance in which the function can terminate prematurely. Notice how failing to close `fin2` produces a resource leak, leaving an open file descriptor.

Please note that these examples assume `errno_t` and `NOERR` to be defined, as recommended in [DCL09-C. Declare functions that return errno with a return type of errno_t](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl09-c) . An equivalent compatible example would define `errno_t` as an `int` and `NOERR` as zero.

These examples also assume that `errno` is set if `fopen()` or `malloc()` fail. These are guaranteed by POSIX but not by C11. See [ERR30-C. Take care when reading errno](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err30-c) for more details.

This is just a small example; in much larger examples, errors like this are even harder to detect.

### Compliant Solution (POSIX, Nested Ifs)

This compliant solution uses nested if statements to properly close files and free memory in the case that any error occurs. When the number of resources to manage is small (3 in this example), nested if statements will be simpler than a goto chain.

### Compliant Solution (POSIX, Goto Chain)

Occasionally, the number of resources to manage in one function will be too large to permit using nested ifs to manage them.

In this revised version, a `goto` chain replaces each individual return segment. If no error occurs, control flow falls through to the `SUCCESS` label, releases all of the resources, and returns `NOERR` . If an error occurs, the return value is set to `errno` , control flow jumps to the proper failure label, and the appropriate resources are released before returning.

This method is beneficial because the code is cleaner, and the programmer does not need to rewrite similar code upon every function error.

Note that this guideline does not advocate more general uses of goto, which is still [considered harmful](https://homepages.cwi.nl/~storm/teaching/reader/Dijkstra68.pdf) . The use of goto in these cases is specifically to transfer control within a single function body.

### Compliant Solution ( `copy_process()` from Linux kernel)

Some effective examples of `goto` chains are quite large. This compliant solution is an excerpt from the Linux kernel. This is the `copy_process` function from `kernel/fork.c` from version 2.6.29 of the kernel.

The function uses 17 `goto` labels (not all displayed here) to perform cleanup code should any internal function yield an error code. If no errors occur, the program returns a pointer to the new process `p` . If any error occurs, the program diverts control to a particular `goto` label, which performs cleanup for sections of the function that have currently been successfully executed but not for sections that have not yet been executed. Consequently, only resources that were successfully opened are actually closed.

All comments in this excerpt were added to indicate additional code in the kernel not displayed here.

## MEM30-C. Do not access freed memory

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/13.memory-management-mem/2.mem30-c.md) — SHA-256 `d903f76c7ac4ce8f7620bb17e3cc39165049aefcf619f80ae6505e21fc0ff198`。

Evaluating a pointer—including dereferencing the pointer, using it as an operand of an arithmetic operation, type casting it, and using it as the right-hand side of an assignment—into memory that has been deallocated by a memory management function is [undefined behavior 183](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_183) . Pointers to memory that has been deallocated are called *dangling pointers* . Accessing a dangling pointer can result in exploitable [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) .

According to the C Standard, using the value of a pointer that refers to space deallocated by a call to the `free()` or `realloc()` function is undefined behavior. (See [undefined behavior 183](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_183) .)

Reading a pointer to deallocated memory is [undefined behavior 183](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_183) because the pointer value is [indeterminate](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) and might be a [non-value representation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-nonvaluerepresentation) . Fetching a non-value representation might perform a hardware trap (but is not required to).

It is at the memory manager's discretion when to reallocate or recycle the freed memory. When memory is freed, all pointers into it become invalid, and its contents might either be returned to the operating system, making the freed space inaccessible, or remain intact and accessible. As a result, the data at the freed location can appear to be valid but change unexpectedly. Consequently, memory must not be written to or read from once it is freed.

### Noncompliant Code Example

This example from Brian Kernighan and Dennis Ritchie \[ [Kernighan 1988](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Kernighan88) \] shows both the incorrect and correct techniques for freeing the memory associated with a linked list. In their (intentionally) incorrect example, `p` is freed before `p->next` is executed, so that `p->next` reads memory that has already been freed.

### Compliant Solution

Kernighan and Ritchie correct this error by storing a reference to `p->next` in `q` before freeing `p` :

### Noncompliant Code Example

In this noncompliant code example, `buf` is written to after it has been freed. Write-after-free vulnerabilities can be [exploited](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-exploit) to run arbitrary code with the permissions of the vulnerable process. Typically, allocations and frees are far removed, making it difficult to recognize and diagnose these problems.

### Compliant Solution

In this compliant solution, the memory is freed after its final use:

### Noncompliant Code Example

In this noncompliant example, `realloc()` may free `c_str1` when it returns a null pointer, resulting in `c_str1` being freed twice.  The C Standards Committee's proposed response to [Defect Report #400](https://www.open-std.org/jtc1/sc22/wg14/www/docs/dr_400.htm) makes it implementation-defined whether or not the old object is deallocated when `size` is zero and memory for the new object is not allocated. The current implementation of `realloc()` in the GNU C Library and Microsoft Visual Studio's Runtime Library will free `c_str1` and return a null pointer for zero byte allocations.  Freeing a pointer twice can result in a potentially exploitable vulnerability commonly referred to as a *double-free vulnerability* \[ [Seacord 2013b](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Seacord2013) \].

### Compliant Solution

This compliant solution does not pass a size argument of zero to the `realloc()` function, eliminating the possibility of `c_str1` being freed twice:

If the intent of calling `f()` is to reduce the size of the object, then doing nothing when the size is zero would be unexpected; instead, this compliant solution frees the object.

### Noncompliant Code Example

In this noncompliant example ( [CVE-2009-1364](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2009-1364) ) from `libwmf` version 0.2.8.4, the return value of `gdRealloc` (a simple wrapper around `realloc()` that reallocates space pointed to by `im->clip->list` ) is set to `more` . However, the value of `im->clip->list` is used directly afterwards in the code, and the C Standard specifies that if `realloc()` moves the area pointed to, then the original block is freed. An attacker can then execute arbitrary code by forcing a reallocation (with a sufficient `im->clip->count` ) and accessing freed memory \[ [xorl 2009](https://xorl.wordpress.com/2009/05/05/cve-2009-1364-libwmf-pointer-use-after-free/) \].

### Compliant Solution

This compliant solution simply reassigns `im->clip->list` to the value of `more` after the call to `realloc()` :

## MEM31-C. Free dynamically allocated memory when no longer needed

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/13.memory-management-mem/3.mem31-c.md) — SHA-256 `e1cd6f4fb65828c515e37391cb3f4fa5030bc940b66ac0c4aa765132413d8bae`。

Before the lifetime of the last pointer that stores the return value of a call to a standard memory allocation function has ended, it must be matched by a call to `free()` with that pointer value.

### Noncompliant Code Example

In this noncompliant example, the object allocated by the call to `malloc()` is not freed before the end of the lifetime of the last pointer `text_buffer` referring to the object:

### Compliant Solution

In this compliant solution, the pointer is deallocated with a call to `free()` :

### Exceptions

**MEM31-C-EX1** : Allocated memory does not need to be freed if it is assigned to a pointer whose lifetime includes program termination. The following code example illustrates a pointer that stores the return value from `malloc()` in a `static` variable:

## MEM33-C. Allocate and copy structures containing a flexible array member dynamically

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/13.memory-management-mem/4.mem33-c.md) — SHA-256 `669348205686256d2c5ac89292c5db950142f7e2753fdf160f6b08948e8cab7c`。

The C Standard, 6.7.3.2, paragraph 20 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], says

> As a special case, the last element of a structure with more than one named member may have an incomplete array type; this is called a *flexible array member* . In most situations, the ﬂexible array member is ignored. In particular, the size of the structure is as if the flexible array member were omitted except that it may have more trailing padding than the omission would imply.

The following is an example of a structure that contains a flexible array member:

This definition means that when computing the size of such a structure, only the first member, `num` , is considered. Unless the appropriate size of the flexible array member has been explicitly added when allocating storage for an object of the `struct` , the result of accessing the member `data` of a variable of nonpointer type `   struct flex_array_struct  ` is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . [DCL38-C. Use the correct syntax when declaring a flexible array member](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl38-c) describes the correct way to declare a `struct` with a flexible array member.

To avoid the potential for [undefined behavior 59](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_59) , structures that contain a flexible array member should always be allocated dynamically. Flexible array structures must

-   Have dynamic storage duration (be allocated via `malloc()` or another dynamic allocation function)
-   Be dynamically copied using `memcpy()` or a similar function and not by assignment
-   When used as an argument to a function, be passed by pointer and not copied by value

### Noncompliant Code Example (Storage Duration)

This noncompliant code example uses automatic storage for a structure containing a flexible array member:

Because the memory for `flex_struct` is reserved on the stack, no space is reserved for the `data` member. Accessing the `data` member is [undefined behavior 59](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_59) .

### Compliant Solution (Storage Duration)

This compliant solution dynamically allocates storage for `flex_array_struct` :

### Noncompliant Code Example (Copying)

This noncompliant code example attempts to copy an instance of a structure containing a flexible array member ( `   struct       flex_array_struct   ` ) by assignment:

When the structure is copied, the size of the flexible array member is not considered, and only the first member of the structure, `num` , is copied, leaving the array contents untouched.

### Compliant Solution (Copying)

This compliant solution uses `memcpy()` to properly copy the content of `struct_a` into `struct_b` :

### Noncompliant Code Example (Function Arguments)

In this noncompliant code example, the flexible array structure is passed by value to a function that prints the array elements:

Because the argument is passed by value, the size of the flexible array member is not considered when the structure is copied, and only the first member of the structure, `num` , is copied.

### Compliant Solution (Function Arguments)

In this compliant solution, the structure is passed by reference and not by value:

## MEM34-C. Only free memory allocated dynamically

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/13.memory-management-mem/5.mem34-c.md) — SHA-256 `3cfe2f9568f5a6aa25621c1be0ecf8e775175f28b37e01e94ac3b51173a0fd4d`。

The C Standard, Annex J (184) \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states that the behavior of a program is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) when

> The pointer argument to the `free` or `realloc` function does not match a pointer earlier returned by a memory management function, or the space has been deallocated by a call to `free` or `realloc` .

See also [undefined behavior 184](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_14) .

Freeing memory that is not allocated dynamically can result in heap corruption and other serious errors. Do not call `free()` on a pointer other than one returned by a standard memory allocation function, such as `malloc()` , `calloc()` , `realloc()` , or `aligned_alloc()` .

A similar situation arises when `realloc()` is supplied a pointer to non-dynamically allocated memory. The `realloc()` function is used to resize a block of dynamic memory. If `realloc()` is supplied a pointer to memory not allocated by a standard memory allocation function, the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . One consequence is that the program may [terminate abnormally](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) .

This rule does not apply to null pointers. The C Standard guarantees that if `free()` is passed a null pointer, no action occurs.

### Noncompliant Code Example

This noncompliant code example sets `c_str` to reference either dynamically allocated memory or a statically allocated string literal depending on the value of `argc` . In either case, `c_str` is passed as an argument to `free()` . If anything other than dynamically allocated memory is referenced by `c_str` , the call to `free(c_str)` is erroneous.

### Compliant Solution

This compliant solution eliminates the possibility of `c_str` referencing memory that is not allocated dynamically when passed to `free()` :

### Noncompliant Code Example ( `realloc()` )

In this noncompliant example, the pointer parameter to `realloc()` , `buf` , does not refer to dynamically allocated memory:

### Compliant Solution ( `realloc()` )

In this compliant solution, `buf` refers to dynamically allocated memory:

Note that `realloc()` will behave properly even if `malloc()` failed, because when given a null pointer, `realloc()` behaves like a call to `malloc()` .

## MEM35-C. Allocate sufficient memory for an object

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/13.memory-management-mem/6.mem35-c.md) — SHA-256 `91216dfe3741929725305279264a28279af345003dbe4cfe5e00dc263b3ac340`。

The types of integer expressions used as size arguments to `malloc()` , `calloc()` , `realloc()` , or `aligned_alloc()` must have sufficient range to represent the size of the objects to be stored. If size arguments are incorrect or can be manipulated by an attacker, then a buffer overflow may occur. Incorrect size arguments, inadequate range checking, integer overflow, or truncation can result in the allocation of an inadequately sized buffer.

Typically, the amount of memory to allocate will be the size of the type of object to allocate. When allocating space for an array, the size of the object will be multiplied by the bounds of the array. When allocating space for a structure containing a flexible array member, the size of the array member must be added to the size of the structure. (See [MEM33-C. Allocate and copy structures containing a flexible array member dynamically](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/memory-management-mem/mem33-c) .) Use the correct type of the object when computing the size of memory to allocate.

[STR31-C. Guarantee that storage for strings has sufficient space for character data and the null terminator](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/characters-and-strings-str/str31-c) is a specific instance of this rule.

### Noncompliant Code Example (Pointer)

In this noncompliant code example, inadequate space is allocated for a `   struct tm  ` object because the size of the pointer is being used to determine the size of the pointed-to object:

### Compliant Solution (Pointer)

In this compliant solution, the correct amount of memory is allocated for the `   struct tm  ` object. When allocating  space for a single object, passing the (dereferenced) pointer type to the `sizeof` operator is a simple way to allocate sufficient memory. Because the `sizeof` operator does not evaluate its operand, dereferencing an uninitialized or null pointer in this context is well-defined behavior.

### Noncompliant Code Example (Integer)

In this noncompliant code example, an array of `long` is allocated and assigned to `p` . The code attempts to check for unsigned integer overflow in compliance with [INT30-C. Ensure that unsigned integer operations do not wrap](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int30-c) and also ensures that `len` is not equal to zero. (See [MEM04-C. Beware of zero-length allocations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/memory-management-mem/mem04-c) .) However, because `sizeof(int)` is used to compute the size, and not `sizeof(long)` , an insufficient amount of memory can be allocated on implementations where `sizeof(long)` is larger than `sizeof(int)` , and filling the array can cause a heap buffer overflow.

### Compliant Solution (Integer)

This compliant solution uses `sizeof(long)` to correctly size the memory allocation:

### Compliant Solution (Integer)

Alternatively, `sizeof(*p)` can be used to properly size the allocation:

## MEM36-C. Do not modify the alignment of objects by calling realloc()

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/13.memory-management-mem/7.mem36-c.md) — SHA-256 `11ec54f5615950ae595eeb711bd3432fa9b7f75cad9b9b4347a2f73373cee1f9`。

Do not invoke `realloc()` to modify the size of allocated objects that have stricter alignment requirements than those guaranteed by `malloc()` . Storage allocated by a call to the standard `aligned_alloc()` function, for example, can have stricter than normal alignment requirements. The C standard requires only that a pointer returned by `realloc()` be suitably aligned so that it may be assigned to a pointer to any type of object with a fundamental alignment requirement.

### Noncompliant Code Example

This noncompliant code example returns a pointer to allocated memory that has been aligned to a 4096-byte boundary.  If the `resize` argument to the `realloc()` function is larger than the object referenced by `ptr` , then `realloc()` will allocate new memory that is suitably aligned so that it may be assigned to a pointer to any type of object with a fundamental alignment requirement but may not preserve the stricter alignment of the original object.

#### Implementation Details

When compiled with GCC 4.1.2 and run on the x86_64 Red Hat Linux platform, the following code produces the following output:

##### **CODE**

##### **OUTPUT**

`ptr1` is no longer aligned to 4096 bytes.

### Compliant Solution

This compliant solution  allocates `resize` bytes of new memory with the same alignment as the old memory, copies the original memory content, and then frees the old memory. This solution has [implementation-defined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) because it depends on whether extended alignments in excess of `   _Alignof (max_align_t)  ` are supported and the contexts in which they are supported. If not supported, the behavior of this compliant solution is undefined.

### Compliant Solution (Windows)

Windows defines the `_aligned_malloc()` function to allocate memory on a specified alignment boundary.  The `_aligned_realloc()` \[ [MSDN](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MSDN) \] can be used to change the size of this memory. This compliant solution demonstrates one such usage:

The `size` and `alignment` arguments for `_aligned_malloc()` are provided in reverse order of the C Standard `aligned_alloc()` function.
