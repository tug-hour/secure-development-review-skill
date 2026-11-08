# CERT C — ERR

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## ERR00-C. Adopt and implement a consistent and comprehensive error-handling policy

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/08.error-handling-err/2.err00-c.md) — SHA-256 `3ffd8dc769ae899d60ebe59e9beeba610aa3b1e8586b2e76648c7dab7e461707`。

A secure system is invariably subject to stresses, such as those caused by attack, erroneous or malicious inputs, hardware or software faults, unanticipated user behavior, and unexpected environmental changes that are outside the bounds of "normal operation." Yet the system must continue to deliver essential services in a timely manner, safely and securely. To accomplish this, the system must exhibit qualities such as [robustness](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-robustness) , [reliability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-reliability) , [error tolerance](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-errortolerance) , [fault tolerance](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-faulttolerance) , performance, and security. All of these system-quality attributes depend on consistent and comprehensive error handling that supports the goals of the overall system.

ISO/IEC TR 24772, section 6.39.1 \[ [ISO/IEC TR 24772](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECTR24772-2013) \], says:

> Expectations that a system will be dependable are based on the confidence that the system will operate as expected and not fail in normal use. The dependability of a system and its fault tolerance can be measured through the component part's reliability, availability, safety and security. Reliability is the ability of a system or component to perform its required functions under stated conditions for a specified period of time \[ [IEEE 1990 glossary](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd610.121990) \]. Availability is how timely and reliable the system is to its intended users. Both of these factors matter highly in systems used for safety and security. In spite of the best intentions, systems may encounter a failure, either from internally poorly written software or external forces such as power outages/variations, floods, or other natural disasters. The reaction to a fault can affect the performance of a system and in particular, the safety and security of the system and its users.

Effective error handling (which includes error reporting, report aggregation, analysis, response, and recovery) is a central aspect of the design, implementation, maintenance, and operation of systems that exhibit survivability under stress. Survivability is the capability of a system to fulfill its mission, in a timely manner, despite an attack, accident, or other stress that is outside the bounds of normal operation \[ [Lipson 2000](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Lipson00) \]. If full services cannot be maintained under a given stress, survivable systems degrade gracefully, continue to deliver essential services, and recover full services as conditions permit.

Error reporting and error handling play a central role in the engineering and operation of survivable systems. Survivability is an emergent property of a system as a whole \[ [Fisher 1999](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Fisher99) \] and depends on the behavior of all of the system's components and the interactions among them. From the viewpoint of error handling, every system component, down to the smallest routine, can be considered to be a sensor capable of reporting on some aspect of the health of the system. Any error or anomaly, ignored or improperly handled, can threaten delivery of essential system services and, as a result, put at risk the organizational or business mission that the system supports.

The key characteristics of survivability include the 3 Rs: resistance, recognition, and recovery. Resistance refers to measures that *harden* a system against particular stresses, recognition refers to situational awareness with respect to instances of stress and their impact on the system, and recovery is the ability of a system to restore services after (and possibly during) an attack, accident, or other event that has disrupted those services.

Recognition of the full nature of adverse events and the determination of appropriate measures for recovery and response are often not possible in the context of the component or routine in which a related error first manifests. Aggregation of multiple error reports and the interpretation of those reports in a higher context may be required both to understand what is happening and to decide on the appropriate action to take. Of course, the domain-specific context in which the system operates plays a huge role in determining proper recovery strategies and tactics. For safety-critical systems, simply halting the system (or even just terminating an offending process) in response to an error is rarely the best course of action and may lead to disaster. From a system perspective, error-handling strategies should map directly into survivability strategies, which may include recovery by activating fully redundant backup services or by providing alternative sets of roughly equivalent services that fulfill the mission with sufficient diversity to greatly improve the odds of survival against common mode failures.

An error-handling policy must specify a comprehensive approach to error reporting and response. Components and routines should always generate status indicators, and all called routines should have their error returns checked. All input should be checked for compliance with the formal requirements for such input rather than be blindly trusted. Moreover, never assume, on the basis of specific knowledge about the system or its domain, that the success of a called routine is guaranteed. The failure to report or properly respond to errors or other anomalies from a system perspective can threaten the survivability of the system as a whole.

ISO/IEC TR 24772:2013, section 6.39.5 \[ [ISO/IEC TR 24772:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECTR24772-2013) \], describes the following mitigation strategies:

> Software developers can avoid the vulnerability or mitigate its ill effects in the following ways:
>
> -   A strategy for fault handling should be decided. Consistency in fault handling should be the same with respect to critically similar parts.
> -   A multi-tiered approach of fault prevention, fault detection, and fault reaction should be used.
> -   System-defined components that assist in uniformity of fault handling should be used when available. For one example, designing a "runtime constraint handler" permits the application to intercept various erroneous situations and perform one consistent response, such as flushing a previous transaction and restarting at the next one.
> -   When there are multiple tasks, a fault-handling policy should be specified whereby a task may
>     -   halt, and keep its resources available for other tasks (perhaps permitting restarting of the faulting task)
>     -   halt, and remove its resources (perhaps to allow other tasks to use the resources so freed, or to allow a recreation of the task)
>     -   halt, and signal the rest of the program to likewise halt

## ERR01-C. Use ferror() rather than errno to check for FILE stream errors

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/08.error-handling-err/3.err01-c.md) — SHA-256 `aaf28ef62e1ce8a471b5bf877ac433100f55fa96e453630db1517c331db50efb`。

Use `ferror()` rather than `errno` to check whether an error has occurred on a file stream (for example, after a long chain of `stdio` calls). The `ferror()` function tests the error indicator for a specified stream and returns nonzero if and only if the error indicator is set for the stream.

### Noncompliant Code Example

Many [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) of the `stdio` package adjust their behavior slightly if `stdout` is a terminal. To make the determination, these implementations perform some operation that fails (with `ENOTTY` ) if `stdout` is not a terminal. Although the output operation goes on to complete successfully, `errno` still contains `ENOTTY` . This behavior can be mildly confusing, but it is not strictly incorrect because it is meaningful for a program to inspect the contents of `errno` only after an error has been reported. More precisely, `errno` is meaningful only after a library function that sets `errno` on error has returned an error code.

### Compliant Solution

This compliant solution uses `ferror()` to detect an error. In addition, if an early call to `printf()` fails, later calls may modify `errno` , whether they fail or not, so the program cannot rely on being able to detect the root cause of the original failure if it waits until after a sequence of library calls to check.

## ERR02-C. Avoid in-band error indicators

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/08.error-handling-err/4.err02-c.md) — SHA-256 `7ffac9b8964433d8b895a478432e3fcb69ee270852c5f0a27ec2c3bf168469ba`。

Avoid [in-band error indicators](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-in-banderrorindicator) while designing interfaces. This practice is commonly used by C library functions but is not recommended. One example from the C Standard of a troublesome in-band error indicator is `EOF` (see [FIO34-C. Distinguish between characters read from a file and EOF or WEOF](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/input-output-fio/fio34-c) ). Another problematic use of in-band error indicators from the C Standard involving the `size_t` and `time_t` types is described by

-   [INT31-C. Ensure that integer conversions do not result in lost or misinterpreted data](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int31-c)
-   [FLP07-C. Cast the return value of a function that returns a floating-point type](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/floating-point-flp/flp07-c)
-   [INT18-C. Evaluate integer expressions in a larger size before comparing or assigning to that size](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/integers-int/int18-c)

### Noncompliant Code Example ( `sprintf()` )

This noncompliant code example is from the Linux Kernel Mailing List archive site, although similar examples are common:

The `sprintf()` function returns the number of characters written in the array, not counting the terminating null character. This number is frequently added to an existing counter to keep track of the location of the index into the array. However, the call to `sprintf()` can (and will) return −1 on error conditions, such as an encoding error. If this error happens on the first call (which is likely), the `count` variable, already at 0, is decremented. If this index is subsequently used, it will result in an out-of-bounds read or write.

### Compliant Solution ( `sprintf_m()` )

This compliant solution shows the redesigned API for `sprintf()` from the CERT managed string library \[ [Burch 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Burch06) \]:

The `sprintf_m()` API separates the return status of the function from information about the number of characters written. In this case, `*count` is set to the number of characters written in `buf` , and the return value indicates the return status. Returning the status as the return value of the function increases the likelihood that a programmer will check the return status of the function.

The preceding code example can be amended as follows:

### Noncompliant Code Example (POSIX `ssize_t` )

The `ssize_t` data type is designed as a "signed representation of `size_t` ." Consequently, it is often used as a return type for functions that can return an unsigned value upon success and a negative value upon error. For instance, the POSIX `read()` function has the following signature:

`read()` returns −1 if an error occurs; if no errors occur, it returns the number of bytes actually read.

As with all in-band error indicators, this type is not recommended because developers are tempted to ignore the possibility that a `ssize_t` value is negative.

### Compliant Solution (POSIX `size_t` )

An alternative hypothetical signature for the `read()` function is

where `rbytes` is a pointer to a `size_t` . If no error occurs, and `rbytes` is not `NULL` , its value is set to the total number of bytes read, and `read()` returns 0. If an error occurs, `read()` returns a nonzero value indicating the error.

### Exceptions

**ERR02-EX1:** Null pointers are another example of an in-band error indicator. Use of null pointers is allowed because it is supported by the language. According to the C Standard, subclause 6.3.2.3 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \]:

> If a null pointer constant is converted to a pointer type, the resulting pointer, called a null pointer, is guaranteed to compare unequal to a pointer to any object or function.

**ERR02-EX2:** You may design a function that returns in-band error indicators if it is possible to securely guarantee that the program will not try to continue processing should an error occur in the function. For example, after calling the glibc function `       feenableexcept      (FE_OVERFLOW)  ` , you can call the `exp` function without needing to check for overflow on the result yourself, because if it occurs, then a `SIGFPE` will cause the program to terminate in a similar manner as `abort()` .

## ERR04-C. Choose an appropriate termination strategy

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/08.error-handling-err/5.err04-c.md) — SHA-256 `608ec224097012c2070292004cda46e3cb4dea1c58dc2ab179d051f343f0bf6a`。

Some errors, such as out-of-range values, might be the result of erroneous user input. Interactive programs typically handle such errors by rejecting the input and prompting the user for an acceptable value. Servers reject invalid user input by indicating an error to the client while at the same time continuing to service other clients' valid requests. All [robust](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-robustness) programs must be prepared to gracefully handle resource exhaustion, such as low memory or disk space conditions, at a minimum by preventing the loss of user data kept in volatile storage. Interactive programs may give the user the option to save data on an alternative medium, whereas network servers may respond by reducing throughput or otherwise degrading the quality of service. However, when certain kinds of errors are detected, such as irrecoverable logic errors, rather than risk data corruption by continuing to execute in an indeterminate state, the appropriate strategy may be for the system to quickly shut down, allowing the operator to start it afresh in a determinate state.

ISO/IEC TR 24772:2013, Section 6.39, "Termination Strategy \[REU\]," \[ [ISO/IEC TR 24772:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECTR24772-2013) \], says:

> When a fault is detected, there are many ways in which a system can react. The quickest and most noticeable way is to fail hard, also known as fail fast or fail stop. The reaction to a detected fault is to immediately halt the system. Alternatively, the reaction to a detected fault could be to fail soft. The system would keep working with the faults present, but the performance of the system would be degraded. Systems used in a high availability environment such as telephone switching centers, e-commerce, or other "always available" applications would likely use a fail soft approach. What is actually done in a fail soft approach can vary depending on whether the system is used for safety-critical or security critical purposes. For fail-safe systems, such as flight controllers, traffic signals, or medical monitoring systems, there would be no effort to meet normal operational requirements, but rather to limit the damage or danger caused by the fault. A system that fails securely, such as cryptologic systems, would maintain maximum security when a fault is detected, possibly through a denial of service.

And

> The reaction to a fault in a system can depend on the criticality of the part in which the fault originates. When a program consists of several tasks, each task may be critical, or not. If a task is critical, it may or may not be restartable by the rest of the program. Ideally, a task that detects a fault within itself should be able to halt leaving its resources available for use by the rest of the program, halt clearing away its resources, or halt the entire program. The latency of task termination and whether tasks can ignore termination signals should be clearly specified. Having inconsistent reactions to a fault can potentially be a vulnerability.

C provides several options for program termination, including `exit()` , returning from `main()` , `_Exit()` , and `abort()` .

### `exit()`

Calling `exit()` causes [normal program termination](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-normaltermination) to occur. Other than returning from `main()` , calling `exit()` is the typical way to end a program. The function takes one argument of type `int` , which should be either `EXIT_SUCCESS` or `EXIT_FAILURE` , indicating successful or unsuccessful termination respectively. The value of `EXIT_SUCCESS` is guaranteed to be 0. The C Standard, subclause 7.22.4.4 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], says, "If the value of status is zero or `EXIT_SUCCESS` , an [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-defined) form of the status successful termination is returned." The `exit()` function never returns.

Calling `exit()`

-   Flushes unwritten buffered data.
-   Closes all open files.
-   Removes temporary files.
-   Returns an integer exit status to the operating system.

The C Standard `atexit()` function can be used to customize `exit()` to perform additional actions at program termination.

For example, calling

*registers* the `turn_gizmo_off()` function so that a subsequent call to `exit()` will invoke `turn_gizmo_off()` as it terminates the program. C requires that `atexit()` can register at least 32 functions.

Functions registered by the `atexit()` function are called by `exit()` or upon normal completion of `main()` .

Note that the behavior of a program that calls `exit()` from an `atexit` handler is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See undefined behavior [187](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_187) in Annex J of the C Standard. See also [ENV32-C. All exit handlers must return normally](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/environment-env/env32-c) .)

### `return` from `main()`

Returning from `main()` causes [normal program termination](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-normaltermination) to occur, which is the preferred way to terminate a program. Evaluating the `return` statement has the same effect as calling `exit()` with the same argument.

The C Standard, subclause 5.1.2.2.3 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], has this to say about returning from `main()` :

> If the return type of the `main` function is a type compatible with `int` , a return from the initial call to the `main` function is equivalent to calling the `exit` function with the value returned by the `main` function as its argument; reaching the `}` that terminates the `main` function returns a value of 0. If the return type is not compatible with `int` , the termination status returned to the host environment is unspecified.

Consequently, returning from `main()` is equivalent to calling `exit()` . Many compilers implement this behavior with something analogous to

However, exiting from `main` is conditional on correctly handling all errors in a way that does not force premature termination. (See [ERR00-C. Adopt and implement a consistent and comprehensive error-handling policy](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err00-c) and [ERR05-C. Application-independent code should provide error detection without dictating error handling](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err05-c) .)

### `_Exit()`

Calling `_Exit()` causes [normal program termination](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-normaltermination) to occur. Like the `exit()` function, `_Exit()` takes one argument of type `int` and never returns. However, unlike `exit()` , whether `_Exit()` closes open streams, flushes stream buffers, <sup>\[</sup> <sup>[1](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err04-c#ERR04C.Chooseanappropriateterminationstrategy-1) \]</sup> or deletes temporary files is [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) . Functions registered by `atexit()` are not executed.

<a id="ERR04C.Chooseanappropriateterminationstrategy-1"></a> \[1\] Note that POSIX strengthens the specification for `_Exit()` by prohibiting the function from flushing stream buffers. See the documentation of the function in [The Open Group Base Specifications Issue 7,](http://www.opengroup.org/onlinepubs/9699919799) IEEE Std 1003.1, 2013 Edition \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \].

The `_exit()` function is an alias for `_Exit()` .

### `abort()`

Calling `abort()` causes [abnormal program termination](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) to occur unless the `SIGABRT` signal is caught and the signal handler calls `exit()` or `_Exit()` :

As with `_Exit()` , whether open streams with unwritten buffered data are flushed, <sup>\[</sup> <sup>[2](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err04-c#ERR04C.Chooseanappropriateterminationstrategy-2) \]</sup> open streams are closed, or temporary files are removed is [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-defined) . Functions registered by `atexit()` are not executed. (See [ERR06-C. Understand the termination behavior of assert() and abort()](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err06-c) .)

<a id="ERR04C.Chooseanappropriateterminationstrategy-2"></a> \[2\] Unlike in the case of `_Exit()` , POSIX explicitly permits but does not require [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) to flush stream buffers. See the documentation of the function in [The Open Group Base Specifications Issue 7,](http://www.opengroup.org/onlinepubs/9699919799) IEEE Std 1003.1, 2013 Edition \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \].

### Summary

The following table summarizes the exit behavior of the program termination functions.

| Function | Closes <br> Open <br> Streams | Flushes <br> Stream <br> Buffers | Removes <br> Temporary <br> Files | Calls <br> `atexit()` <br> Handlers | Program <br> Termination |
| --- | --- | --- | --- | --- | --- |
| `abort()` | ℹ️ | ℹ️\[[2](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err04-c#ERR04C.Chooseanappropriateterminationstrategy-2)\] | ℹ️  | ❌ | [Abnormal](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) |
| `_Exit()` | ℹ️ | ℹ️\[[1](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err04-c#ERR04C.Chooseanappropriateterminationstrategy-1)\] | ℹ️ | ❌ | [Normal](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-normaltermination) |
| `exit()`  |️ ✅ | ✅ | ✅  | ✅ | [Normal](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-normaltermination) |
| Return from `main()` | ✅ | ✅ | ✅ | ✅ | [Normal](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-normaltermination) |

Table legend:

-   ✅ – Yes. The specified action is performed.
-   ❌ – No. The specified action is not performed.
-   ℹ️ – [Implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) . Whether the specified action is performed depends on the implementation.

### Noncompliant Code Example

The `abort()` function should not be called if it is important to perform application-specific cleanup before exiting. In this noncompliant code example, `abort()` is called after data is sent to an open file descriptor. The data may or may not be written to the file.

### Compliant Solution

In this compliant solution, the call to `abort()` is replaced with `exit()` , which guarantees that buffered I/O data is flushed to the file descriptor and the file descriptor is properly closed:

Although this particular example benefits from calling `exit()` over `abort()` , in some situations, `abort()` is the better choice. Usually, `abort()` is preferable when a programmer does not need to close any file descriptors or call any handlers registered with `atexit()` , for instance, if the speed of terminating the program is critical.

For more details on proper usage of `abort()` , see [ERR06-C. Understand the termination behavior of assert() and abort()](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err06-c) .

## ERR05-C. Application-independent code should provide error detection without dictating error handling

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/08.error-handling-err/6.err05-c.md) — SHA-256 `fc281e938bd72c0a062ffb9fb6b77cc03321cafcf7e2051dd69f9a0fac7dac55`。

Application-independent code includes code that is

-   Shipped with the compiler or operating system
-   From a third-party library
-   Developed in-house

When application-specific code detects an error, it can immediately respond with a specific action, as in

This response occurs because the application must both detect errors and provide a mechanism for handling errors. Application-independent code, by contrast, is not associated with any application, so it cannot handle errors. However, it must still detect errors and report them to an application so that the application may handle them.

Error detection and reporting can take several forms:

-   A return value (especially of type `errno_t` )
-   An argument passed by address
-   A global object (such as `errno` )
-   `longjmp()`
-   Some combination of the above

### Noncompliant Code Example

This noncompliant code example consists of two application-independent functions, `f()` and `g()` . The `f()` function is part of the external API for the module; the `g()` function is an internal function.

If `something_really_bad_happens` in `g()` , the function prints an error message to `stderr` and then calls `abort()` . The problem is that this application-independent code does not know the context in which it is being called, so it is erroneous to handle the error.

“ Smart Libraries, ” Practice 23 \[ [Miller 2004](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Miller04) \], says:

> When a library aborts due to some kind of anomaly, it is saying there is no hope for execution to proceed normally beyond the point where the anomaly is detected. Nonetheless, it is dictatorially making this decision on behalf of the client. Even if the anomaly turns out to be some kind of internal bug in the library, which obviously cannot be resolved in the current execution, aborting is a bad thing to do. The fact is, a library developer cannot possibly know the fault-tolerant context in which his/her library is being used. The client may indeed be able to recover from the situation even if the library cannot.

It is equally bad to eliminate the call to `abort()` from `g()` . In this case, the calling function has no indication that an error has occurred.

### Compliant Solution (Return Value)

One way to inform the calling function of errors is to return a value indicating success or failure. This compliant solution ensures each function returns a value of type `errno_t` , where 0 indicates that no error has occurred:

A call to `f()` returns a status indicator, which is 0 upon success and a nonzero value upon failure indicating what went wrong.

A return type of `errno_t` indicates that the function returns a status indicator (see [DCL09-C. Declare functions that return errno with a return type of errno_t](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl09-c) ).

This error-handling approach is secure, but it has the following drawbacks:

-   Source and object code can significantly increase in size, some sources suggest by as much as 30 to 40 percent \[ [Saks 2007b](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Saks07b) \].
-   All function return values must be checked (see [ERR33-C. Detect and handle standard library errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err33-c) ).
-   Functions should not return other values if they return error indicators (see [ERR02-C. Avoid in-band error indicators](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err02-c) ).
-   Any function that allocates resources must ensure they are freed in cases where errors occur.

### Compliant Solution (Address Argument)

Instead of encoding status indicators in the return value, each function can take a pointer as an argument, which is used to indicate errors. In the following example, each function uses an `   errno_t\ *  ` argument to report errors:

A call to `f()` provides a status indicator that is 0 upon success and a nonzero value upon failure, assuming the user provided a valid pointer to an object of type `errno_t` .

This solution is secure, but it has the following drawbacks:

-   A return status can be returned only if the caller provides a valid pointer to an object of type `errno_t` . If this argument is `NULL` , there is no way to indicate *this* error.
-   Source code becomes even larger because of the possibilities of receiving a null pointer.
-   All error indicators must be checked after calling functions.
-   Any function that allocates resources must ensure they are freed in cases where errors occur.
-   Unlike return values, [static analysis](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-staticanalysis) tools generally do not diagnose a failure to check error indicators passed as argument pointers.

### Compliant Solution (Global Error Indicator)

Instead of encoding error indicators in the return value or arguments, a function can indicate its status by assigning a value to a global variable. In the following example, each function uses a static indicator called `my_errno` .

The original `errno` variable was the standard C library's implementation of error handling using this approach.

The call to `f()` provides a status indicator that is 0 upon success and a nonzero value upon failure.

This solution has many of the same properties as those observed with `errno` , including advantages and drawbacks.

-   Source code size is inflated, though not by as much as in other approaches.
-   All error indicators must be checked after calling functions.
-   Nesting of function calls that all use this mechanism is problematic.
-   Any function that allocates resources must ensure they are freed in cases where errors occur.
-   In general, combining registries of different sets of errors is difficult. For example, changing this compliant solution code to use `errno` is difficult and bug-prone because the programmer must be precisely aware of when C library functions set and clear `errno` and also must be aware of all valid `errno` values before adding new ones.
-   Calling `f()` from other application-independent code has major limitations. Because `f()` sets `my_errno` to 0, it may be overwriting a nonzero error value set by another application-independent calling function.

For these reasons, among others, this approach is generally discouraged.

### Compliant Solution ( `setjmp()` and `longjmp()` )

C provides two functions, `setjmp()` and `longjmp()` , that can be used to alter control flow. Using these functions, a user can ignore error values and trust that control flow will be correctly diverted in the event of error.

The following example uses `setjmp()` and `longjmp()` to ensure that control flow is disrupted in the event of error; it also uses the `my_errno` indicator from the previous example. See [MSC22-C. Use the setjmp(), longjmp() facility securely](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc22-c) for more information on `setjmp()` and `longjmp()` .

Calls to `f()` will either succeed or divert control to an `if` clause designed to catch the error.

-   The source code is not significantly larger because the function signatures do not change, and neither do functions that neither detect nor handle the error.
-   Allocated resources must still be freed despite the error.
-   The application must call `setjmp()` before invoking application-independent code.
-   Signals are not necessarily preserved through `longjmp()` calls.
-   The use of `setjmp()` / `longjmp()` bypasses the normal function call and return discipline.
-   Any function that allocates resources must ensure they are freed in cases where errors occur.

### Summary

The following table summarizes the characteristics of error-reporting and error-detection mechanisms.

|                          |               |                             |                           |
|--------------------------|---------------|-----------------------------|---------------------------|
| Method                   | Code Increase | Manages Allocated Resources | Automatically Enforceable |
| Return value             | Big (30–40%)  | No                          | Yes                       |
| Address argument         | Bigger        | No                          | No                        |
| Global indicator         | Medium        | No                          | Yes                       |
| `longjmp()` | Small         | No                          | n/a                       |

## ERR06-C. Understand the termination behavior of assert() and abort()

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/08.error-handling-err/7.err06-c.md) — SHA-256 `02dc53ee28b9360daa080d4de5efaebd5140de7eacbc6effee32bcfbaeb9ac13`。

The C Standard, subclause 7.2.1.1 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], defines `assert()` to have the following behavior:

> The `assert` macro puts diagnostic tests into programs; it expands to a void expression. When it is executed, if `expression` (which shall have a scalar type) is false (that is, compares equal to 0), the `assert` macro writes information about the particular call that failed (including the text of the argument, the name of the source file, the source line number, and the name of the enclosing function—the latter are respectively the values of the pre-processing macros `    __         FILE_        _   `and`    __         LINE_        _   ` and of the identifier `    __         func_        _)   ` on the standard error stream in an implementation-defined format. It then calls the `abort` function.

Because `assert()` calls `abort()` , cleanup functions registered with `atexit()` are not called. If the intention of the programmer is to properly clean up in the case of a failed assertion, then runtime assertions should be replaced with static assertions where possible. (See [DCL03-C. Use a static assertion to test the value of a constant expression](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl03-c) .) When the assertion is based on runtime data, the `assert` should be replaced with a runtime check that implements the adopted error strategy (see [ERR00-C. Adopt and implement a consistent and comprehensive error-handling policy](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err00-c) ).

See [ERR04-C. Choose an appropriate termination strategy](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err04-c) for more information on program termination strategies and [MSC11-C. Incorporate diagnostic tests using assertions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc11-c) for more information on using the `assert()` macro.

### Noncompliant Code Example

This noncompliant code example defines a function that is called before the program exits to clean up:

However, the code also has an `assert` , and if the assertion fails, the `cleanup()` function is *not* called.

### Compliant Solution

In this compliant solution, the call to `assert()` is replaced with an `if` statement that calls `exit()` to ensure that the proper termination routines are run:

## ERR07-C. Prefer functions that support error checking over equivalent functions that don't

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/08.error-handling-err/8.err07-c.md) — SHA-256 `0d8d0e01c41b896254c1596db237f266b4fdb7cfb76ff00fc3888660bff512a6`。

When you have a choice of two functions to accomplish the same task, prefer the one with better error checking and reporting.

The following table shows a list of C standard library functions that provide limited or no error checking and reporting along with preferable alternatives:

| Function | Preferable Alternative | Comments |
| --- | --- | --- |
| `atof` | `strtod` | No error indication, [undefined behavior](http://BB.%20Definitions#undefined%20behavior) on error |
| `atoi` | `strtol` | No error indication, undefined behavior on error |
| `atol` | `strtol` | No error indication, undefined behavior on error |
| `atoll` | `strtoll` | No error indication, undefined behavior on error |
| `rewind` | `fseek` | No error indication, silent failure on error |
| `setbuf` | `setvbuf` | No error indication, silent failure on error |
| `ctime` | `asctime` / `localtime` | Undefined behavior if `localtime` fails |

### Noncompliant Code Example ( `atoi()` )

This noncompliant code example converts the string token stored in the static array `buff` to a signed integer value using the `atoi()` function:

The `atoi()` , `atol()` , and `atoll()` functions convert the initial portion of a string token to `int` , `   long int  ` , and `   long long int  ` representation respectively. Except for the behavior on error, they are equivalent as follows:

|                            |                                                     |
|----------------------------|-----------------------------------------------------|
| Call                       | Equivalent on Success                               |
| `atoi(nptr)`  | `       (int)strtol(nptr, (char **)NULL, 10)      ` |
| `atol(nptr)`  | `       strtol(nptr, (char **)NULL, 10)      `      |
| `atoll(nptr)` | `       strtoll(nptr, (char **)NULL, 10)      `     |

Unfortunately, `atoi()` and related functions lack a mechanism for reporting errors for invalid values. Specifically, the `atoi()` , `atol()` , and `atoll()` functions

-   Do not need to set `errno` on an error.
-   Have [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) if the value of the result cannot be represented. (See [undefined behavior 118](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_118) of Annex J of the C Standard.)
-   Return 0 if the string does not represent an integer (which is indistinguishable from a correctly formatted, zero-denoting input string), but the C Standard only specifies the behavior of these functions on success.

See also [MSC24-C. Do not use deprecated or obsolescent functions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc24-c) .

### Compliant Solution ( `strtol()` )

The `strtol()` , `strtoll()` , `strtoul()` , and `strtoull()` functions convert a null-terminated byte string to `   long int  `,`   long long int  `,`   unsigned long int  ` , and `   unsigned long long int  ` representation respectively.

This compliant solution uses `strtol()` to convert a string token to an integer and ensures that the value is in the range of `int` :

Both the noncompliant code example and the compliant solution are taken from [ERR34-C. Detect errors when converting a string to a number](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err34-c) .

### Noncompliant Code Example ( `rewind()` )

This noncompliant code example sets the file position indicator of an input stream back to the beginning using `rewind()` :

It is impossible to determine if `rewind()` succeeded.

### Compliant Solution ( `fseek()` )

This compliant solution uses `fseek()` instead of `rewind()` and checks to see if the operation succeeded:

### Noncompliant Code Example ( `setbuf()` )

This noncompliant code example calls `setbuf()` with a `buf` argument of `NULL` :

It is not possible to determine if the call to `setbuf()` succeeded.

#### Implementation Details

On 4.2BSD and 4.3BSD systems, `setbuf()` always uses a suboptimal buffer size and should be avoided.

### Compliant Solution ( `setvbuf()` )

This compliant solution calls `setvbuf()` , which returns nonzero if the operation failed:

## ERR30-C. Take care when reading errno

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/08.error-handling-err/2.err30-c.md) — SHA-256 `88131505129958e84d6db0cd3a267a3278c7b9cb399158a6625456acda8f4ac0`。

The value of `errno` is initialized to zero at program startup, but it is never subsequently set to zero by any C standard library function. The value of `errno` may be set to nonzero by a C standard library function call whether or not there is an error, provided the use of `errno` is not documented in the description of the function. It is meaningful for a program to inspect the contents of `errno` only after an error might have occurred. More precisely, `errno` is meaningful only after a library function that sets `errno` on error has returned an error code.

According to Question 20.4 of C-FAQ \[ [Summit 2005](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Summit05) \],

> In general, you should detect errors by checking return values, and use `errno` only to distinguish among the various causes of an error, such as "File not found" or "Permission denied." (Typically, you use `perror` or `strerror` to print these discriminating error messages.) It's only necessary to detect errors with `errno` when a function does not have a unique, unambiguous, out-of-band error return (that is, because all of its possible return values are valid; one example is `    atoi [         sic        ]   ` ). In these cases (and in these cases only; check the documentation to be sure whether a function allows this), you can detect errors by setting `errno` to 0, calling the function, and then testing `errno` . (Setting `errno` to 0 first is important, as no library function ever does that for you.)

Note that `atoi()` is not required to set the value of `errno` .

Library functions fall into the following categories:

-   Those that set `errno` and return an [out-of-band error indicator](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-out-of-banderrorindicator)
-   Those that set `errno` and return an [in-band error indicator](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-in-banderrorindicator)
-   Those that do not promise to set `errno`
-   Those with differing standards documentation

### Library Functions that Set `errno` and Return an Out-of-Band Error Indicator

The C Standard specifies that the functions listed in the following table set `errno` and return an [out-of-band error indicator](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-out-of-banderrorindicator) . That is, their return value on error can never be returned by a successful call.

A program may check `errno` after invoking these library functions but is not required to do so. The program should not check the value of `errno` without first verifying that the function returned an error indicator. For example, `errno` should not be checked after calling `signal()` without first ensuring that `signal()` actually returned `SIG_ERR` .

**Functions That Set `errno` and Return an Out-of-Band Error Indicator**

|                                                              |                             |                                               |
|--------------------------------------------------------------|-----------------------------|-----------------------------------------------|
| **Function Name**                                            | **Return Value**            | `errno` **Value** |
| `ftell()`                                         | `-1L`            | Positive                                      |
| `fgetpos()` , `fsetpos()`          | Nonzero                     | Positive                                      |
| `       mbrtowc       ()      `,`       mbsrtowcs()      `|`       (size_t)(-1)      `|`       EILSEQ      `                         |
| `       signal       ()      `|`       SIG_ERR      `      | Positive                                      |
| `       wcrtomb       ()      `,`       wcsrtombs()      `|`       (size_t)(-1)      `|`       EILSEQ      `                         |
| `mbrtoc16()` , `mbrtoc32()`            | `(size_t)(-1)`   | `EILSEQ`                           |
| `c16rtomb()` , `      c32rtomb      ()     `|`      (size_t)(-1)     `|`      EILSEQ     `                           |

### Library Functions that Set `errno` and Return an In-Band Error Indicator

The C Standard specifies that the functions listed in the following table set `errno` and return an [in-band error indicator](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-in-banderrorindicator) . That is, the return value when an error occurs is also a valid return value for successful calls. For example, the `strtoul()` function returns `ULONG_MAX` and sets `errno` to `ERANGE` if an error occurs. Because `ULONG_MAX` is a valid return value, the only way to confirm that an error occurred when LONG_MAX is returned is to check `errno` .

The `fgetwc()` and `fputwc()` functions return `WEOF` in multiple cases, only one of which results in setting `errno` . The string conversion functions will return the maximum or minimum representable value and set `errno` to `ERANGE` if the converted value cannot be represented by the data type. However, if the conversion cannot happen because the input is invalid, the function will return `0` , and the output pointer parameter will be assigned the value of the input pointer parameter, provided the output parameter is non-null.

A program that uses `errno` for error checking  a function that returns an in-band error indicator must set `errno` to `0` before calling one of these library functions and then inspect `errno` before a subsequent library function call.

**Functions that Set `errno` and Return an In-Band Error Indicator**

|                                                                     |                                                       |                                               |
|---------------------------------------------------------------------|-------------------------------------------------------|-----------------------------------------------|
| **Function Name**                                                   | **Return Value**                                      | `errno` **Value** |
| `fgetwc()` , `fputwc()`                   | `WEOF`                                   | `EILSEQ`                         |
| `       strtol       ()      `,`       wcstol       ()      `|`       LONG_MIN      `or`       LONG_MAX      `|`       ERANGE      `                         |
| `       strtoll       ()      `,`       wcstoll       ()      `|`       LLONG_MIN      `or`       LLONG_MAX      `|`       ERANGE      `                         |
| `       strtoul       ()      `,`       wcstoul       ()      `|`       ULONG_MAX      `|`       ERANGE      `                         |
| `       strtoull       ()      `,`       wcstoull       ()      `|`       ULLONG_MAX      `|`       ERANGE      `                         |
| `       strtoumax       ()      `,`       wcstoumax()      `|`       UINTMAX_MAX      `|`       ERANGE      `                         |
| `       strtod       ()      `,`       wcstod       ()      `|`       0      `or`       ±HUGE_VAL      `|`       ERANGE      `                         |
| `       strtof       ()      `,`       wcstof()      `|`       0      `or`       ±HUGE_VALF      `|`       ERANGE      `                         |
| `       strtold       ()      `,`       wcstold       ()      `|`      0     `or`      ±HUGE_VALL     `|`       ERANGE      `                         |
| `       strtoimax       ()      `,`       wcstoimax()      `|`       INTMAX_MIN      `,`       INTMAX_MAX      `|`       ERANGE      `                         |

### Library Functions that Do Not Promise to Set `errno`

The C Standard fails to document the behavior of `errno` for some functions. For example, the `setlocale()` function normally returns a null pointer in the event of an error, but no guarantees are made about setting `errno` .

After calling one of these functions, a program should not rely solely on the value of `errno` to determine if an error occurred. The function might have altered `errno` , but this does not ensure that `errno` will properly indicate an error condition. If the program does check `errno` after calling one of these functions, it should set `errno` to 0 before the function call.

### Library Functions with Differing Standards Documentation

Some functions behave differently regarding `errno` in various standards. The `fopen()` function is one such example. When `fopen()` encounters an error, it returns a null pointer. The C Standard makes no mention of `errno` when describing `fopen()` . However, POSIX.1 declares that when `fopen()` encounters an error, it returns a null pointer and sets `errno` to a value indicating the error \[ [IEEE Std 1003.1-2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \]. The implication is that a program conforming to C but not to POSIX (such as a Windows program) should not check `errno` after calling `fopen()` , but a POSIX program may check `errno` if `fopen()` returns a null pointer.

### Library Functions and `errno`

The following uses of `errno` are documented in the C Standard:

-   Functions defined in `<complex.h>` may set `errno` but are not required to.
-   For numeric conversion functions in the `strtod` , `strtol` , `wcstod` , and `wcstol` families, if the correct result is outside the range of representable values, an appropriate minimum or maximum value is returned and the value `ERANGE` is stored in `errno` . For floating-point conversion functions in the `strtod` and `wcstod` families, if an underflow occurs, whether `errno` acquires the value `ERANGE` is [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) . If the conversion fails, `0` is returned and `errno` is not set.
-   The numeric conversion function `atof()` and those in the `atoi` family "need not affect the value of" `errno` .
-   For mathematical functions in `<math.h>` , if the integer expression `    math_errhandling & MATH_ERRNO   ` is nonzero, on a domain error, `errno` acquires the value `EDOM` ; on an overflow with default rounding or if the mathematical result is an exact infinity from finite arguments, `errno` acquires the value `ERANGE` ; and on an underflow, whether `errno` acquires the value `ERANGE` is implementation-defined.
-   If a request made by calling `signal()` cannot be honored, a value of `SIG_ERR` is returned and a positive value is stored in `errno` .
-   The byte I/O functions, wide-character I/O functions, and multibyte conversion functions store the value of the macro `EILSEQ` in `errno` if and only if an encoding error occurs.
-   On failure, `fgetpos()` and `fsetpos()` return nonzero and store an [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) positive value in `errno` .
-   On failure, `ftell()` returns `-1L` and stores an [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) positive value in `errno` .
-   The `perror()` function maps the error number in `errno` to a message and writes it to `stderr` .

The POSIX.1 standard defines the use of `errno` by many more functions (including the C standard library function). POSIX also has a small set of functions that are exceptions to the rule. These functions have no return value reserved to indicate an error, but they still set `errno` on error. To detect an error, an application must set `errno` to `0` before calling the function and check whether it is nonzero after the call. Affected functions include `strcoll()` , `strxfrm()` , `strerror()` , `wcscoll()` , `wcsxfrm()` , and `fwide()` . The C Standard allows these functions to set `errno` to a nonzero value on success. Consequently, this type of error checking should be performed only on POSIX systems.

### Noncompliant Code Example ( `strtoul()` )

This noncompliant code example fails to set `errno` to `0` before invoking `strtoul()` . If an error occurs, `strtoul()` returns a valid value ( `ULONG_MAX` ), so `errno` is the only means of determining if `strtoul()` ran successfully.

Any error detected in this manner may have occurred earlier in the program or may not represent an actual error.

### Compliant Solution ( `strtoul()` )

This compliant solution sets `errno` to `0` before the call to `strtoul()` and inspects `errno` after the call:

### Noncompliant Code Example ( `ftell()` )

This noncompliant code example, after calling `ftell()` , examines `errno` without first checking whether the out-of-band indicator returned by `ftell()` indicates an error.

### Compliant Solution ( `ftell()` )

This compliant solution first detects that `ftell()` failed using its out-of-band error indicator. Once an error has been confirmed, reading `errno` (implicitly by using the `perror()` function) is permitted.

### Noncompliant Code Example ( `fopen()` )

This noncompliant code example may fail to diagnose errors because `fopen()` might not set `errno` even if an error occurs:

### Compliant Solution ( `fopen()` , C)

The C Standard makes no mention of `errno` when describing `fopen()` . In this compliant solution, the results of the call to `fopen()` are used to determine failure and `errno` is not checked:

### Compliant Solution ( `fopen()` , POSIX)

In this compliant solution, `errno` is checked only after an error has already been detected by another means:

## ERR32-C. Do not rely on indeterminate values of errno

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/08.error-handling-err/3.err32-c.md) — SHA-256 `c70bc0b576def215555716c2f297a7d41619eda19c2af8d0b87ac520026ae98f`。

According to the C Standard Annex J.2 (133) \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], the behavior of a program is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) when

> the value of `errno` is referred to after a signal occurred other than as the result of calling the `abort` or `raise` function and the corresponding signal handler obtained a `SIG_ERR` return from a call to the `signal` function.

See [undefined behavior 133](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_133) .

A signal handler is allowed to call `signal();` if that fails, `signal()` returns `SIG_ERR` and sets `errno` to a positive value. However, if the event that caused a signal was external (not the result of the program calling `abort()` or `raise()` ), the only functions the signal handler may call are `_Exit()` or `abort()` , or it may call `signal()` on the signal currently being handled; if `signal()` fails, the value of `errno` is [indeterminate](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) .

This rule is also a special case of [SIG31-C. Do not access shared objects in signal handlers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/signals-sig/sig31-c) . The object designated by `errno` is of static storage duration and is not a `   volatile sig_atomic_t  ` . As a result, performing any action that would require `errno` to be set would normally cause [undefined behavior 132](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_132) . The C Standard, 7.14.1.1, paragraph 5, makes a special exception for `errno` in this case, allowing `errno` to take on an indeterminate value but specifying that there is no other [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . This special exception makes it possible to call `signal()` from within a signal handler without risking [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) , but the handler, and any code executed after the handler returns, must not depend on the value of `errno` being meaningful.

### Noncompliant Code Example

The `handler()` function in this noncompliant code example attempts to restore default handling for the signal indicated by `signum` . If the request to set the signal to default can be honored, the `signal()` function returns the value of the signal handler for the most recent successful call to the `signal()` function for the specified signal. Otherwise, a value of `SIG_ERR` is returned and a positive value is stored in `errno` . Unfortunately, the value of `errno` is indeterminate because the `handler()` function is called when an external signal is raised, so any attempt to read `errno` (for example, by the `perror()` function) is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) :

The call to `perror()` from `handler()` also violates [SIG30-C. Call only asynchronous-safe functions within signal handlers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/signals-sig/sig30-c) .

### Compliant Solution

This compliant solution does not reference `errno` and does not return from the signal handler if the `signal()` call fails:

### Noncompliant Code Example (POSIX)

POSIX is less restrictive than C about what applications can do in signal handlers. It has a long list of [asynchronous-safe](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-asynchronous-safe) functions that can be called. (See [SIG30-C. Call only asynchronous-safe functions within signal handlers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/signals-sig/sig30-c) .) Many of these functions set `errno` on error, which can lead to a signal handler being executed between a call to a failed function and the subsequent inspection of `errno` . Consequently, the value inspected is not the one set by that function but the one set by a function call in the signal handler. POSIX applications can avoid this problem by ensuring that signal handlers containing code that might alter `errno` ; always save the value of `errno` on entry and restore it before returning.

The signal handler in this noncompliant code example alters the value of `errno` . As a result, it can cause incorrect error handling if executed between a failed function call and the subsequent inspection of `errno` :

### Compliant Solution (POSIX)

This compliant solution saves and restores the value of `errno` in the signal handler:

## ERR33-C. Detect and handle standard library errors

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/08.error-handling-err/4.err33-c.md) — SHA-256 `e866ad9cb23062edaba3dc025288e3e75d801b8ff2f1d6d805cfda29bc8d2b20`。

The majority of the standard library functions, including I/O functions and memory allocation functions, return either a valid value or a value of the correct return type that indicates an error (for example, −1 or a null pointer). Assuming that all calls to such functions will succeed and failing to check the return value for an indication of an error is a dangerous practice that may lead to [unexpected](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unexpectedbehavior) or [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) when an error occurs. It is essential that programs detect and appropriately handle all errors in accordance with an error-handling policy.

The successful completion or failure of each of the standard library functions listed in the following table shall be determined either by comparing the function’s return value with the value listed in the column labeled “Error Return” or by calling one of the library functions mentioned in the footnotes.

**Standard Library Functions**

|                                                    |                                                          |                                                                                                                                    |
|----------------------------------------------------|----------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| Function                                           | Successful Return                                        | Error Return                                                                                                                       |
| `       aligned_alloc       ()      `              | Pointer to space                                         | `NULL`                                                                                                                |
| `       asctime_s       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       at_quick_exit       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       atexit       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       bsearch       ()      `                    | Pointer to matching element                              | `NULL`                                                                                                                |
| `       bsearch_s       ()      `                  | Pointer to matching element                              | `NULL`                                                                                                                |
| `       btowc       ()      `                      | Converted wide character                                 | `WEOF`                                                                                                                |
| `       c16rtomb       ()      `                   | Number of bytes                                          | `(size_t)(-1)`                                                                                                        |
| `       c32rtomb       ()      `                   | Number of bytes                                          | `(size_t)(-1)`                                                                                                        |
| `       c8rtomb       ()      `                    | Number of bytes                                          | `(size_t)(-1)`                                                                                                        |
| `       calloc       ()      `                     | Pointer to space                                         | `NULL`                                                                                                                |
| `       clock       ()      `                      | Processor time                                           | `(clock_t)(-1)`                                                                                                       |
| `       cnd_broadcast       ()      `|`       thrd_success      `|`       thrd_error      `                                                                                                          |
| `       cnd_init       ()      `|`       thrd_success      `|`       thrd_nomem      `or`       thrd_error      `                                                                             |
| `       cnd_signal       ()      `|`       thrd_success      `|`       thrd_error      `                                                                                                          |
| `       cnd_timedwait       ()      `|`       thrd_success      `|`       thrd_timedout      `or`       thrd_error      `                                                                          |
| `       cnd_wait       ()      `|`       thrd_success      `|`       thrd_error      `                                                                                                          |
| `       ctime_s       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       fclose       ()      `|`       0      `|`       EOF      ` (negative)                                                                                                      |
| `       fflush       ()      `|`       0      `|`       EOF      ` (negative)                                                                                                      |
| `       fgetc       ()      `                      | Character read                                           | `EOF` <sup>1</sup>                                                                                                    |
| `       fgetpos       ()      `|`       0      `                                         | Nonzero, `       errno > 0      `                                                                                                  |
| `       fgets       ()      `                      | Pointer to string                                        | `NULL`                                                                                                                |
| `       fgetwc       ()      `                     | Wide character read                                      | `WEOF` <sup>1</sup>                                                                                                   |
| `       fgetws       ()      `                     | Pointer to string                                        | `NULL`                                                                                                                |
| `       fopen       ()      `                      | Pointer to stream                                        | `NULL`                                                                                                                |
| `       fopen_s       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       fprintf       ()      `                    | Number of characters (nonnegative)                       | Negative                                                                                                                           |
| `       fprintf_s       ()      `                  | Number of characters (nonnegative)                       | Negative                                                                                                                           |
| `       fputc       ()      `                      | Character written                                        | `EOF` <sup>2</sup>                                                                                                    |
| `       fputs       ()      `                      | Nonnegative                                              | `EOF` (negative)                                                                                                      |
| `fputwc()`                              | Wide character written                                   | `WEOF`                                                                                                                  |
| `       fputws       ()      `                     | Nonnegative                                              | `EOF` (negative)                                                                                                      |
| `       fread       ()      `                      | Elements read                                            | Elements read                                                                                                                      |
| `       freopen       ()      `                    | Pointer to stream                                        | `NULL`                                                                                                                |
| `       freopen_s       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       fscanf       ()      `                     | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       fscanf_s       ()      `                   | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       fseek       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       fsetpos       ()      `|`       0      `                                         | Nonzero, `       errno > 0      `                                                                                                  |
| `       ftell       ()      `                      | File position                                            | `−1L` , `       errno > 0      `                                                                                      |
| `       fwprintf       ()      `                   | Number of wide characters (nonnegative)                  | Negative                                                                                                                           |
| `       fwprintf_s       ()      `                 | Number of wide characters (nonnegative)                  | Negative                                                                                                                           |
| `       fwrite       ()      `                     | Elements written                                         | Elements written                                                                                                                   |
| `       fwscanf       ()      `                    | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       fwscanf_s       ()      `                  | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       getc       ()      `                       | Character read                                           | `EOF` <sup>1</sup>                                                                                                    |
| `       getchar       ()      `                    | Character read                                           | `EOF` <sup>1</sup>                                                                                                    |
| `       getenv       ()      `                     | Pointer to string                                        | `NULL`                                                                                                                |
| `       getenv_s       ()      `                   | Pointer to string                                        | `NULL`                                                                                                                |
| `       gets_s       ()      `                     | Pointer to string                                        | `NULL`                                                                                                                |
| `       getwc       ()      `                      | Wide character read                                      | `WEOF`                                                                                                                |
| `       getwchar       ()      `                   | Wide character read                                      | `WEOF`                                                                                                                |
| `       gmtime       ()      `                     | Pointer to broken-down time                              | `NULL`                                                                                                                |
| `       gmtime_r       ()      `                   | Pointer to broken-down time                              | `NULL`                                                                                                                |
| `       gmtime_s       ()      `                   | Pointer to broken-down time                              | `NULL`                                                                                                                |
| `       localtime       ()      `                  | Pointer to broken-down time                              | `NULL`                                                                                                                |
| `       localtime_r       ()      `                | Pointer to broken-down time                              | `NULL`                                                                                                                |
| `       localtime_s       ()      `                | Pointer to broken-down time                              | `NULL`                                                                                                                |
| `       malloc       ()      `                     | Pointer to space                                         | `NULL`                                                                                                                |
| `       mblen       ()       , s != NULL      `    | Number of bytes                                          | `−1`                                                                                                                  |
| `       mbrlen       ()       , s != NULL      `   | Number of bytes or status                                | `(size_t)(-1)`                                                                                                        |
| `       mbrtoc16       ()      `                   | Number of bytes or status                                | `(size_t)(-1)` , `       errno == EILSEQ      `                                                                       |
| `       mbrtoc32       ()      `                   | Number of bytes or status                                | `(size_t)(-1)` , `       errno == EILSEQ      `                                                                       |
| `       mbrtoc8       ()      `                    | Number of bytes or status                                | `(size_t)(-1)` , `       errno == EILSEQ      `                                                                       |
| `       mbrtowc       ()       , s != NULL      `  | Number of bytes or status                                | `(size_t)(-1)` , `       errno == EILSEQ      `                                                                       |
| `       mbsrtowcs       ()      `                  | Number of non-null elements                              | `(size_t)(-1)` , `       errno == EILSEQ      `                                                                       |
| `       mbsrtowcs_s       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       mbstowcs       ()      `                   | Number of non-null elements                              | `(size_t)(-1)`                                                                                                        |
| `       mbstowcs_s       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       mbtowc       ()       , s != NULL      `   | Number of bytes                                          | `−1`                                                                                                                  |
| `       memccpy       ()      `                    | Pointer after the copied character                       | `NULL`                                                                                                                |
| `       memchr       ()      `                     | Pointer to located character                             | `NULL`                                                                                                                |
| `       mktime       ()      `                     | Calendar time                                            | `(time_t)(-1)`                                                                                                        |
| `       mtx_init       ()      `|`       thrd_success      `|`       thrd_error      `                                                                                                          |
| `       mtx_lock       ()      `|`       thrd_success      `|`       thrd_error      `                                                                                                          |
| `       mtx_timedlock       ()      `|`       thrd_success      `|`       thrd_timedout      `or`       thrd_error      `                                                                          |
| `       mtx_trylock       ()      `|`       thrd_success      `|`       thrd_busy      `or`       thrd_error      `                                                                              |
| `       mtx_unlock       ()      `|`       thrd_success      `|`       thrd_error      `                                                                                                          |
| `       printf_s       ()      `                   | Number of characters (nonnegative)                       | Negative                                                                                                                           |
| `       putc       ()      `                       | Character written                                        | `EOF` <sup>2</sup>                                                                                                    |
| `       putwc       ()      `                      | Wide character written                                   | `WEOF`                                                                                                                |
| `       raise       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       realloc       ()      `                    | Pointer to space                                         | `NULL`                                                                                                                |
| `       remove       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       rename       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       setlocale       ()      `                  | Pointer to string                                        | `NULL`                                                                                                                |
| `       setvbuf       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       scanf       ()      `                      | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       scanf_s       ()      `                    | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       signal       ()      `                     | Pointer to previous function                             | `SIG_ERR` , `       errno > 0      `                                                                                  |
| `       snprintf       ()      `                   | Number of characters that would be written (nonnegative) | Negative                                                                                                                           |
| `       snprintf_s       ()      `                 | Number of characters that would be written (nonnegative) | Negative                                                                                                                           |
| `       sprintf       ()      `                    | Number of non-null characters written                    | Negative                                                                                                                           |
| `       sprintf_s       ()      `                  | Number of non-null characters written                    | Negative                                                                                                                           |
| `       sscanf       ()      `                     | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       sscanf_s       ()      `                   | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       strchr       ()      `                     | Pointer to located character                             | `NULL`                                                                                                                |
| `       strdup       ()      `                     | Pointer to duplicate string                              | `NULL`                                                                                                                |
| `       strerror_s       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       strftime       ()      `                   | Number of non-null characters                            | `0`                                                                                                                   |
| `       strndup       ()      `                    | Pointer to created string                                | `NULL`                                                                                                                |
| `       strpbrk       ()      `                    | Pointer to located character                             | `NULL`                                                                                                                |
| `       strrchr       ()      `                    | Pointer to located character                             | `NULL`                                                                                                                |
| `       strstr       ()      `                     | Pointer to located string                                | `NULL`                                                                                                                |
| `       strtod       ()      `                     | Converted value                                          | `0` , `       errno == ERANGE      `                                                                                  |
| `       strtof       ()      `                     | Converted value                                          | `0` , `       errno == ERANGE      `                                                                                  |
| `       strtoimax       ()      `                  | Converted value                                          | `INTMAX_MAX` or `INTMAX_MIN` , `       errno == ERANGE      `                                            |
| `       strtok       ()      `                     | Pointer to first character of a token                    | `NULL`                                                                                                                |
| `       strtok_s       ()      `                   | Pointer to first character of a token                    | `NULL`                                                                                                                |
| `       strtol       ()      `                     | Converted value                                          | `LONG_MAX` or `LONG_MIN` , `       errno == ERANGE      `                                                |
| `       strtold       ()      `                    | Converted value                                          | `       0, errno == ERANGE      `                                                                                                  |
| `       strtoll       ()      `                    | Converted value                                          | `LLONG_MAX` or `LLONG_MIN` , `       errno == ERANGE      `                                              |
| `       strtoumax       ()      `                  | Converted value                                          | `UINTMAX_MAX` , `       errno == ERANGE      `                                                                        |
| `       strtoul       ()      `                    | Converted value                                          | `ULONG_MAX` , `       errno == ERANGE      `                                                                          |
| `       strtoull       ()      `                   | Converted value                                          | `ULLONG_MAX` , `       errno == ERANGE      `                                                                         |
| `       strxfrm       ()      `                    | Length of transformed string                             | `       >= n      `                                                                                                                |
| `       swprintf       ()      `                   | Number of non-null wide characters                       | Negative                                                                                                                           |
| `       swprintf_s       ()      `                 | Number of non-null wide characters                       | Negative                                                                                                                           |
| `       swscanf       ()      `                    | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       swscanf_s       ()      `                  | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       thrd_create       ()      `|`       thrd_success      `|`       thrd_nomem      `or`       thrd_error      `                                                                             |
| `       thrd_detach       ()      `|`       thrd_success      `|`       thrd_error      `                                                                                                          |
| `       thrd_join       ()      `|`       thrd_success      `|`       thrd_error      `                                                                                                          |
| `       thrd_sleep       ()      `|`       0      `                                         | Negative                                                                                                                           |
| `       time       ()      `                       | Calendar time                                            | `(time_t)(-1)`                                                                                                        |
| `       timegm       ()      `                     | Calendar time                                            | `(time_t)(-1)`                                                                                                        |
| `       timespec_get       ()      `               | Base                                                     | `0`                                                                                                                   |
| `       timespec_getres       ()      `            | Base                                                     | `0`                                                                                                                   |
| `       tmpfile       ()      `                    | Pointer to stream                                        | `NULL`                                                                                                                |
| `       tmpfile_s       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       tmpnam       ()      `                     | Non-null pointer                                         | `NULL`                                                                                                                |
| `       tmpnam_s       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       tss_create       ()      `|`       thrd_success      `|`       thrd_error      `                                                                                                          |
| `       tss_get       ()      `                    | Value of thread-specific storage                         | `0`                                                                                                                   |
| `       tss_set       ()      `|`       thrd_success      `|`       thrd_error      `                                                                                                          |
| `       ungetc       ()      `                     | Character pushed back                                    | `EOF` (see [below](https://cmu-sei.github.io/secure-coding-standards/) ) |
| `       ungetwc       ()      `                    | Character pushed back                                    | `WEOF`                                                                                                                |
| `       vfprintf       ()      `                   | Number of characters (nonnegative)                       | Negative                                                                                                                           |
| `       vfprintf_s       ()      `                 | Number of characters (nonnegative)                       | Negative                                                                                                                           |
| `       vfscanf       ()      `                    | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       vfscanf_s       ()      `                  | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       vfwprintf       ()      `                  | Number of wide characters (nonnegative)                  | Negative                                                                                                                           |
| `       vfwprintf_s       ()      `                | Number of wide characters (nonnegative)                  | Negative                                                                                                                           |
| `       vfwscanf       ()      `                   | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       vfwscanf_s       ()      `                 | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       vprintf_s       ()      `                  | Number of characters (nonnegative)                       | Negative                                                                                                                           |
| `       vscanf       ()      `                     | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       vscanf_s       ()      `                   | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       vsnprintf       ()      `                  | Number of characters that would be written (nonnegative) | Negative                                                                                                                           |
| `       vsnprintf_s       ()      `                | Number of characters that would be written (nonnegative) | Negative                                                                                                                           |
| `       vsprintf       ()      `                   | Number of non-null characters (nonnegative)              | Negative                                                                                                                           |
| `       vsprintf_s       ()      `                 | Number of non-null characters (nonnegative)              | Negative                                                                                                                           |
| `       vsscanf       ()      `                    | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       vsscanf_s       ()      `                  | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       vswprintf       ()      `                  | Number of non-null wide characters                       | Negative                                                                                                                           |
| `       vswprintf_s       ()      `                | Number of non-null wide characters                       | Negative                                                                                                                           |
| `       vswscanf       ()      `                   | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       vswscanf_s       ()      `                 | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       vwprintf_s       ()      `                 | Number of wide characters (nonnegative)                  | Negative                                                                                                                           |
| `       vwscanf       ()      `                    | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       vwscanf_s       ()      `                  | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       wcrtomb       ()      `                    | Number of bytes stored                                   | `(size_t)(-1)`                                                                                                        |
| `       wcschr       ()      `                     | Pointer to located wide character                        | `NULL`                                                                                                                |
| `       wcsftime       ()      `                   | Number of non-null wide characters                       | `0`                                                                                                                   |
| `       wcspbrk       ()      `                    | Pointer to located wide character                        | `NULL`                                                                                                                |
| `       wcsrchr       ()      `                    | Pointer to located wide character                        | `NULL`                                                                                                                |
| `       wcsrtombs       ()      `                  | Number of non-null bytes                                 | `(size_t)(-1)` , `       errno == EILSEQ      `                                                                       |
| `       wcsrtombs_s       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       wcsstr       ()      `                     | Pointer to located wide string                           | `NULL`                                                                                                                |
| `       wcstod       ()      `                     | Converted value                                          | `0` , `       errno == ERANGE      `                                                                                  |
| `       wcstof       ()      `                     | Converted value                                          | `0` , `       errno == ERANGE      `                                                                                  |
| `       wcstoimax       ()      `                  | Converted value                                          | `INTMAX_MAX` or `INTMAX_MIN` , `       errno == ERANGE      `                                            |
| `       wcstok       ()      `                     | Pointer to first wide character of a token               | `NULL`                                                                                                                |
| `       wcstok_s       ()      `                   | Pointer to first wide character of a token               | `NULL`                                                                                                                |
| `       wcstol       ()      `                     | Converted value                                          | `LONG_MAX` or `LONG_MIN` , `       errno == ERANGE      `                                                |
| `       wcstold       ()      `                    | Converted value                                          | `0` , `       errno == ERANGE      `                                                                                  |
| `       wcstoll       ()      `                    | Converted value                                          | `LLONG_MAX` or `LLONG_MIN` , `       errno == ERANGE      `                                              |
| `       wcstombs       ()      `                   | Number of non-null bytes                                 | `(size_t)(-1)`                                                                                                        |
| `       wcstombs_s       ()      `|`       0      `                                         | Nonzero                                                                                                                            |
| `       wcstoumax       ()      `                  | Converted value                                          | `UINTMAX_MAX` , `       errno == ERANGE      `                                                                        |
| `       wcstoul       ()      `                    | Converted value                                          | `ULONG_MAX` , `       errno == ERANGE      `                                                                          |
| `       wcstoull       ()      `                   | Converted value                                          | `ULLONG_MAX` , `       errno == ERANGE      `                                                                         |
| `       wcsxfrm       ()      `                    | Length of transformed wide string                        | `       >= n      `                                                                                                                |
| `       wctob       ()      `                      | Converted character                                      | `EOF`                                                                                                                 |
| `       wctomb       ()       , s != NULL      `   | Number of bytes stored                                   | `−1`                                                                                                                  |
| `       wctomb_s       ()       , s != NULL      ` | Number of bytes stored                                   | `−1`                                                                                                                  |
| `       wctrans       ()      `                    | Valid argument to `towctrans`               | `0`                                                                                                                   |
| `       wctype       ()      `                     | Valid argument to `iswctype`                | `0`                                                                                                                   |
| `       wmemchr       ()      `                    | Pointer to located wide character                        | `NULL`                                                                                                                |
| `       wprintf_s       ()      `                  | Number of wide characters (nonnegative)                  | Negative                                                                                                                           |
| `       wscanf       ()      `                     | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |
| `       wscanf_s       ()      `                   | Number of conversions (nonnegative)                      | `EOF` (negative)                                                                                                      |

Note: According to [FIO35-C](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/the-void/void-fio35-c-use-feof-and-ferror-to-detect-end-of-file-and-file-errors-when-sizeofint-sizeofchar) [. Use feof() and ferror() to detect end-of-file and file errors when sizeof(int) == sizeof(char)](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/the-void/void-fio35-c-use-feof-and-ferror-to-detect-end-of-file-and-file-errors-when-sizeofint-sizeofchar) , callers should verify end-of-file and file errors for the functions in this table as follows:

<sup>1</sup> By calling `ferror()` and `feof()` <sup>2</sup> By calling `ferror()`

The `ungetc()` function does not set the error indicator even when it fails, so it is not possible to check for errors reliably unless it is known that the argument is not equal to `EOF` .

The C Standard 7.31.3.10 paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \] states that

> )ne wide character of pushback is guaranteed...

so this should not be an issue if, at most, one character is ever pushed back before reading again. (See [FIO13-C](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio13-c) [. Never push back anything other than one read character](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio13-c) .)

### Noncompliant Code Example ( `setlocale()` )

In this noncompliant code example, the function `utf8_to_wcs()` attempts to convert a sequence of UTF-8 characters to wide characters . It first invokes `setlocale()` to set the global locale to the implementation-defined `en_US.UTF-8` but does not check for failure. The `setlocale()` function will fail by returning a null pointer, for example, when the locale is not installed. The function may fail for other reasons as well, such as the lack of resources.  Depending on the sequence of characters pointed to by `utf8` , the subsequent call to `mbstowcs()` may fail or result in the function storing an unexpected sequence of wide characters in the supplied buffer `wcs` .

### Compliant Solution ( `setlocale()` )

This compliant solution checks the value returned by `setlocale()` and avoids calling `mbstowcs()` if the function fails. The function also takes care to restore the locale to its initial setting before returning control to the caller.

### Noncompliant Code Example ( `calloc()` )

In this noncompliant code example, `temp_num` , `tmp2` , and `num_of_records` are derived from a [tainted source](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-taintedsource) . Consequently, an attacker can easily cause `calloc()` to fail by providing a large value for `num_of_records` .

When `calloc()` fails, it returns a null pointer that is assigned to `start` . If `start` is null, an attacker can provide a value for `temp_num` that, when scaled by `sizeof(signal_info)` , references a writable address to which control is eventually transferred. The contents of the string referenced by `tmp2` can then be used to overwrite the address, resulting in an arbitrary code execution [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) .

### Compliant Solution ( `calloc()` )

To correct this error, ensure the pointer returned by `calloc()` is not null:

### Noncompliant Code Example ( `realloc()` )

This noncompliant code example calls `realloc()` to resize the memory referred to by `p` . However, if `realloc()` fails, it returns a null pointer and the connection between the original block of memory and `p` is lost, resulting in a memory leak.

This code example complies with [MEM04-C](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/memory-management-mem/mem04-c) [. Do not perform zero-length allocations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/memory-management-mem/mem04-c) .

### Compliant Solution ( `realloc()` )

In this compliant solution, the result of `realloc()` is assigned to the temporary pointer `q` and validated before it is assigned to the original pointer `p` :

### Noncompliant Code Example ( `fseek()` )

In this noncompliant code example, the `fseek()` function is used to set the file position to a location `offset` in the file referred to by `file` prior to reading a sequence of bytes from the file. However, if an I/O error occurs during the seek operation, the subsequent read will fill the buffer with the wrong contents.

### Compliant Solution ( `fseek()` )

According to the C Standard, the `fseek()` function returns a nonzero value to indicate that an error occurred. This compliant solution tests for this condition before reading from a file to eliminate the chance of operating on the wrong portion of the file if `fseek()` fails:

### Noncompliant Code Example ( `snprintf()` )

In this noncompliant code example, `snprintf()` is assumed to succeed. However, if the call fails (for example, because of insufficient memory, as described in GNU libc bug [441945](https://bugzilla.redhat.com/show_bug.cgi?id=441945) ), the subsequent call to `log_message()` has [undefined behavior 174](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_174) because the character buffer is uninitialized and need not be null-terminated.

### Compliant Solution ( `snprintf()` )

This compliant solution does not assume that `snprintf()` will succeed regardless of its arguments. It tests the return value of `snprintf()` before subsequently using the formatted buffer. This compliant solution also treats the case where the static buffer is not large enough for `snprintf()` to append the terminating null character as an error.

### Compliant Solution ( `snprintf(0)` )

If unknown, the length of the potential string to output can be discovered by invoking `snprintf()` with `n=0` to determine the size required for the output, then dynamically allocating a buffer of sufficient size, and finally calling `snprintf()` again to format the output into the dynamically allocated buffer. Even with this approach, the success of all calls still needs to be tested, and any errors must be appropriately handled. A possible optimization is to first attempt to format the string into a reasonably small buffer allocated on the stack and, only when the buffer turns out to be too small, dynamically allocate one of a sufficient size:

This solution uses the `goto` statement, as suggested in [MEM12-C](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/memory-management-mem/mem12-c) [. Consider using a goto chain when leaving a function on error when using and releasing resources](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/memory-management-mem/mem12-c) .

### Exceptions

**ERR33-C-EX1:** It is acceptable to ignore the return value of a function if:

-   that function cannot fail.
-   its return value is inconsequential; that is, it does not indicate an error.
-   it is one of a handful of functions whose return values are not traditionally checked.

These functions are listed in the following table:

**Functions for which Return Values Need Not Be Checked**

|                                                                    |                                         |                               |
|--------------------------------------------------------------------|-----------------------------------------|-------------------------------|
| Function                                                           | Successful Return                       | Error Return                  |
| `       putchar       ()      `                                    | Character written                       | `EOF`            |
| `       putwchar       ()      `                                   | Wide character written                  | `WEOF`           |
| `       puts       ()      `                                       | Nonnegative                             | `EOF` (negative) |
| `putws()`                                             | Nonnegative                             | `WEOF`           |
| `       printf       ()      `,`       vprintf       ()      `   | Number of characters (nonnegative)      | Negative                      |
| `       wprintf       ()      `,`       vwprintf       ()      ` | Number of wide characters (nonnegative) | Negative                      |
| `kill_dependency()`                                     | The input parameter                     | NA                            |
| `memcpy()` , `wmemcpy()`                     | The destination input parameter         | NA                            |
| `memmove()` , `wmemmove()`                   | The destination input parameter         | NA                            |
| `strcpy()` , `wcscpy()`                      | The destination input parameter         | NA                            |
| `strncpy()` , `wcsncpy()`                    | The destination input parameter         | NA                            |
| `strcat()` , `wcscat()`                      | The destination input parameter         | NA                            |
| `strncat()` , `wcsncat()`                    | The destination input parameter         | NA                            |
| `memset()` , `wmemset()`                     | The destination input parameter         | NA                            |
| `memset_explicit()`                          | The destination input parameter         | NA                            |

The return value of a call to `fprintf()` or one of its variants ( `vfprintf()` , `wfprintf()` , `vwfprintf()` ) or one of the file output functions `fputc()` , `fputwc()` , `fputs()` , `fputws()` may be ignored if the output is being directed to `stdout` or `stderr` . Otherwise, the return value must be checked.

If a function's return value is to be ignored, it is recommended that the function's return value should be explicitly cast to void to signify the programmer's intent:

## ERR34-C. Detect errors when converting a string to a number

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/08.error-handling-err/5.err34-c.md) — SHA-256 `ae85f88abdcd0e1144b109c50405a7f26f503cef9faefc7f659a8d7e6bec5320`。

The process of parsing an integer or floating-point number from a string can produce many errors. The string might not contain a number. It might contain a number of the correct type that is out of range (such as an integer that is larger than `INT_MAX` ). The string may also contain extra information after the number, which may or may not be useful after the conversion. These error conditions must be detected and addressed when a string-to-number conversion is performed using a C Standard Library function.

The `strtol()` , `strtoll()` , `strtoimax()` , `   strtoul(), strtoull(),       strtoumax()   ` , strtof(), `strtod()` , and `strtold()` functions convert the initial portion of a null-terminated byte string to a `   long int  `,`   long long int  `,`   intmax_t  `,`   unsigned long int  `,`   unsigned long long int, uintmax_t, float, double  ` , and `   long double  ` representation, respectively.

Use one of the C Standard Library `strto*()` functions to parse an integer or floating-point number from a string. These functions provide more robust error handling than alternative solutions. Also, use the `strtol()` function to convert to a smaller signed integer type such as `   signed int  `,`   signed short  ` , and `   signed char  ` , testing the result against the range limits for that type. Likewise, use the `strtoul()` function to convert to a smaller unsigned integer type such as `   unsigned int  `,`   unsigned short  ` , and `   unsigned char  ` , and test the result against the range limits for that type. These range tests do nothing if the smaller type happens to have the same size and representation for a particular implementation.

### Noncompliant Code Example ( `atoi()` )

This noncompliant code example converts the string token stored in the `buff` to a signed integer value using the `atoi()` function:

The `atoi()` , `atol()` , `atoll()` , and `atof()` functions convert the initial portion of a string token to `int` , `   long int, long long int  ` , and `double` representation, respectively. Except for the behavior on error ( \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], s7.24.1.2) , they are equivalent to

Unfortunately, `atoi()` and related functions lack a mechanism for reporting errors for invalid values. Specifically, these functions:

-   do not need to set `errno` on an error;
-   have [undefined behavior 118](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_118) if the value of the result cannot be represented;
-   return 0 (or 0.0) if the string does not represent an integer (or decimal), which is indistinguishable from a correctly formatted, zero-denoting input string.

### Noncompliant Example ( `sscanf()` )

This noncompliant example uses the `sscanf()` function to convert a string token to an integer. The `sscanf()` function has the same limitations as `atoi()` :

The `sscanf()` function returns the number of input items successfully matched and assigned, which can be fewer than provided for, or even 0 in the event of an early matching failure. However, `sscanf()` fails to report the other errors reported by `strtol()` , such as numeric overflow.

### Compliant Solution ( `strtol()` )

The `strtol()` , `strtoll()` , `strtoimax())` , `   strtoul(), strtoull(), strtoumax(), strtof(),  ` `strtod()` , and `strtold()` functions convert a null-terminated byte string to `   long int  `,`   long long int  `,`   intmax_t  `,`   unsigned long int  `,`   unsigned long long int, uintmax_t, float, double  ` , and `   long double  ` representation, respectively.

This compliant solution uses `strtol()` to convert a string token to an integer and ensures that the value is in the range of `int` :

公開用編集（2026-11-09）：原典内のローカル端末パスを公式サイトへのリンクに置換。エラー条件の本文は変更していない。
