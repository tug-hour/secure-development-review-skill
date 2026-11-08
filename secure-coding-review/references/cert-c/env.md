# CERT C — ENV

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## ENV01-C. Do not make assumptions about the size of an environment variable

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/07.environment-env/2.env01-c.md) — SHA-256 `8f215bf2349785767812fe678b870dfeafcdde75bbd27978f3779620283b4707`。

Do not make any assumptions about the size of environment variables because an adversary might have full control over the environment. If the environment variable needs to be stored, the length of the associated string should be calculated and the storage dynamically allocated (see [STR31-C. Guarantee that storage for strings has sufficient space for character data and the null terminator](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/characters-and-strings-str/str31-c) ).

### Noncompliant Code Example

This noncompliant code example copies the string returned by `getenv()` into a fixed-size buffer:

Even if your platform assumes that `$PATH` is defined, defines `PATH_MAX` , and enforces that paths not have more than `PATH_MAX` characters, the `$PATH` environment variable still is not required to have less than `PATH_MAX` chars. And if it has more than `PATH_MAX` chars, a buffer overflow will result. Also, if `$PATH` is not defined, then `strcpy()` will attempt to dereference a null pointer.

### Compliant Solution

In this compliant solution, the `strlen()` function is used to calculate the size of the string, and the required space is dynamically allocated:

### Compliant Solution (POSIX or C23)

In this compliant solution, the `strdup()` function is used to dynamically allocate a duplicate of the string:

## ENV02-C. Beware of multiple environment variables with the same effective name

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/07.environment-env/3.env02-c.md) — SHA-256 `dd5c789fa5e07f587620d46c0060f6fa2e9adb0f4abf852c8672ca5c1bece9c0`。

The `getenv()` function searches an environment list for a string that matches a specified name and returns a pointer to a string associated with the matched list member.

Subclause 7.22.4.6 of the C Standard \[ [ISO/IEC 9899:2011](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AABibliography-ISOIEC9899-2011) \] states:

> The set of environment names and the method for altering the environment list are [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementationdefinedbehavior) .

Depending on the [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) , multiple environment variables with the same name may be allowed and can cause unexpected results if a program cannot consistently choose the same value. The GNU glibc library addresses this issue in `getenv()` and `setenv()` by always using the first variable it encounters and ignoring the rest. However, it is unwise to rely on this behavior.

One common difference between implementations is whether or not environment variables are case sensitive. Although UNIX-like implementations are generally case sensitive, environment variables are "not case sensitive in Windows 98/Me and Windows NT/2000/XP" \[ [MSDN](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MSDN) \].

### Duplicate Environment Variable Detection (POSIX)

The following code defines a function that uses the POSIX `environ` array to manually search for duplicate key entries. Any duplicate environment variables are considered an attack, so the program immediately terminates if a duplicate is detected.

### Noncompliant Code Example

This noncompliant code example behaves differently when compiled and run on Linux and Microsoft Windows platforms:

On an IA-32 Linux machine with GCC 3.4.4, this code prints

whereas, on an IA-32 Windows XP machine with Microsoft Visual C++ 2008 Express, it prints

### Compliant Solution

Portable code should use environment variables that differ by more than capitalization:

## ENV03-C. Sanitize the environment when invoking external programs

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/07.environment-env/4.env03-c.md) — SHA-256 `7a8d8e7eb05b0ce9bdc19b3e2d51e5ba15374c3fc0a63523c66ba8dc6279db03`。

Many programs and libraries, including the shared library loader on both UNIX and Windows systems, depend on environment variable settings. Because environment variables are inherited from the parent process when a program is executed, an attacker can easily sabotage variables, causing a program to behave in an unexpected and insecure manner \[ [Viega 2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Viega03) \].

All programs, particularly those running with higher privileges than the caller (such as those with `setuid/setgid` flags), should treat their environment as untrusted user input. Because the environment is inherited by processes spawned by calls to the `fork()` , `system()` , or `exec()` functions, it is important to verify that the environment does not contain any values that can lead to unexpected behavior.

The best practice for such programs is to

-   Drop privileges once they are no longer necessary. (See [POS02-C. Follow the principle of least privilege](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/posix-pos/pos02-c) .)
-   Avoid calling `system()` . (See [ENV33-C. Do not call system()](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/environment-env/env33-c) .)
-   Clear the environment and fill it with trusted or default values.

This recommendation is a more specific instance of [STR02-C. Sanitize data passed to complex subsystems](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/characters-and-strings-str/str02-c) .

Subclause 7.22.4.6 of the C Standard states that "the set of environment names and the method for altering the environment list are [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) ." Consequently, it is important to understand which functions are available for clearing, modifying, and looking up default values for environment variables. Because some programs may behave in unexpected ways when certain environment variables are not set, it is important to understand which variables are necessary on your system and what are safe values for them.

### Noncompliant Code Example (POSIX, `ls` )

This noncompliant code example invokes the C `system()` function to execute the `/bin/ls` program. The `system()` function passes a string to the command processor in the host environment to be executed.

Although `IFS` does not affect the command portion of this string, `/bin/ls` , it does determine how the argument is built after calling `date` . If the default shell does not ignore the incoming value of the `IFS` environment value, and an attacker sets `IFS` to `"."` , the intended directory will not be found.

### Compliant Solution (POSIX, `ls` )

The nonstandard function `clearenv()` may be used to clear out the environment where available: otherwise, the environment can be cleared by obtaining the environment variable names from `environ` and removing each one using `unsetenv()` .

In this compliant solution, the environment is cleared by `clearenv()` , and then the `PATH` and `IFS` variables are set to safe values before `system()` is invoked. [Sanitizing](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sanitization) shell commands can be difficult, and doing so can adversely affect the power and flexibility associated with them.

POSIX also specifies the `confstr()` function, which can be used to look up default values for environment variables \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \]. The `_CS_V7_ENV` argument to `confstr()` retrieves a list of environment variable settings required for a default [conforming](https://www.securecoding.cert.org/confluence/display/seccode/BB.+Definitions#BB.Definitions-conforming) environment \[ [IEEE Std 1003.1:2013](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AA.Bibliography-IEEEStd1003.1-2013) \]. A space-separated list of `variable=value` pairs is returned, with variable names guaranteed not to contain equal signs (=), and `variable=value` pairs guaranteed not to contain spaces. Used together with the `_CS_PATH` request, this completely describes the minimum environment variable settings required to obtain a clean, conforming environment. On systems conforming to the POSIX.1-2008 standard, this should be used to create a sanitized environment.

On systems that have no `clearenv()` function, the following implementation can be used:

### Compliant Solution (Windows)

There is no portable or guaranteed way to clear out the environment under Windows. Following [ENV33-C. Do not call system()](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/environment-env/env33-c) , care should be taken to use `_execle()` , `_execlpe()` , `_execve()` , or `_execvpe()` instead of `system()` , because they allow the environment to be explicitly specified.

If it is explicitly known which environment variables need to be kept, *Secure Programming Cookbook for C and C++* \[ [Viega 2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Viega03) \] defines a function, `spc_sanitize_environment()` , that will remove everything else.

## ENV04-C. Protect programs whose behavior can be controlled by environment variables

**Recommendation / 未確定**。原典がunder construction / incomplete / stub。記載済み条件を手掛かりとして残すが、完成した規範や適合認定に補作しない。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/07.environment-env/5.env04-c.md) — SHA-256 `34e80b7e5e1cef4beef56a5fee85591ad06e2a37fe3c4f8da95feab3f04d54d1`。

## (THIS CODING RULE OR GUIDELINE IS UNDER CONSTRUCTION)

The recommended way to use OpenMP is to specify most of the performance parameters in environment variables so that administrators can tweak them for their environment:

For example,

-   Set the default number of threads to use.

– **OMP_NUM_THREADS *int_literal***

-   OpenMP added an environment variable to control the size of

child threads’ stack

– **OMP_STACKSIZE**

-   Also added an environment variable to hint to runtime how to

treat idle threads

– **OMP_WAIT_POLICY**

–     Process binding is enabled if this variable is true … i.e. if true

the runtime will not move threads around between processors.

– **OMP_PROC_BIND true \| false**

The downside of this is that if an attacker can manage to control these variables, they can devastate the performance, for example, they can serialize the program by setting the number of threads to 1.

CAPEC has a generic description of a similar attack pattern: <https://capec.mitre.org/data/definitions/10.html>

The risk is not limited to OpenMP, and the preferred mitigation is most likely to make sure that environment variables cannot be manipulated.  This would probably make the use of environment variables in OpenMP program ill-advised for set-uid-root programs, for example, where a less privileged user controls the environment.

## ENV30-C. Do not modify the object referenced by the return value of certain functions

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/07.environment-env/2.env30-c.md) — SHA-256 `ea8ef99462ea82c1fafb425ff2f2fc43b04a345d263152fd8a948e4bb7d769e8`。

Some functions return a pointer to an object that cannot be modified without causing [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . These functions include `getenv()` , `setlocale()` , `localeconv()` , and `strerror()` . In such cases, the function call results must be treated as being `const` -qualified.

The C Standard, 7.24.4.6, paragraph 4 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], defines `getenv()` as follows:

> The `getenv` function returns a pointer to a string associated with the matched list member. The string pointed to shall not be modified by the program, but may be overwritten by a subsequent call to the `getenv` function. If the specified name cannot be found, a null pointer is returned.

If the string returned by `getenv()` must be altered, a local copy should be created. Altering the string returned by `getenv()` is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See [undefined behavior 189](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_189) .)

Similarly, subclause 7.11.1.1, paragraph 8 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], defines `setlocale()` as follows:

> The pointer to string returned by the `setlocale` function is such that a subsequent call with that string value and its associated category will restore that part of the program's locale. The string pointed to shall not be modified by the program. The behavior is undefined if the returned value is used after a subsequent call to the `setlocale` function, or after the thread which called the `setlocale` function to obtain the returned value has exited.

And subclause 7.11.2.1, paragraph 8 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], defines `localeconv()` as follows:

> The `localeconv` function returns a pointer to the filled-in object. The structure pointed to by the return value shall not be modified by the program, but may be overwritten by a subsequent call to the `localeconv` function. In addition, calls to the `setlocale` function with categories `LC_ALL` , `LC_MONETARY` , or `LC_NUMERIC` may overwrite the contents of the structure.
>
>   

Altering the string returned by `setlocale()` or the structure returned by `localeconv()` are [undefined behaviors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See [undefined behaviors 119](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_119) and [121](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_121) .) Furthermore, the C Standard imposes no requirements on the contents of the string by `setlocale()` . Consequently, no assumptions can be made as to the string's internal contents or structure.

Finally, subclause 7.26.6.3, paragraph 4 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> The `strerror` function returns a pointer to the string, the contents of which are locale-specific. The array pointed to shall not be modified by the program. The behavior is undefined if the returned value is used after a subsequent call to the `strerror` function, or after the thread which called the function to obtain the returned value has exited.

Altering the string returned by `strerror()` is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See [undefined behavior 189](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_189) .)

### Noncompliant Code Example ( `getenv()` )

This noncompliant code example modifies the string returned by `getenv()` by replacing all double quotation marks ( `"` ) with underscores ( `_` ):

### Compliant Solution ( `getenv()` ) (Environment Not Modified)

If the programmer does not intend to modify the environment, this compliant solution demonstrates how to modify a copy of the return value:

### Compliant Solution ( `getenv()` ) (Modifying the Environment in POSIX)

If the programmer's intent is to modify the environment, this compliant solution, which saves the altered string back into the environment by using the POSIX `setenv()` and `strdup()` functions, can be used:

### Noncompliant Code Example ( `localeconv()` )

In this noncompliant example, the object returned by `localeconv()` is directly modified:

### Compliant Solution ( `localeconv()` ) (Copy)

This compliant solution modifies a copy of the object returned by `localeconv()` :

## ENV31-C. Do not rely on an environment pointer following an operation that may invalidate it

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/07.environment-env/3.env31-c.md) — SHA-256 `f4afd315edd157327068a67fa4f75fbdaac03d2f905105df0b7259fb2a99bc66`。

Some implementations provide a nonportable environment pointer that is valid when `main()` is called but may be invalidated by operations that modify the environment.

The C Standard, J.5.2 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2024) \], states

> In a hosted environment, the main function receives a third argument, char \* envp \[\] , that points to a null-terminated array of pointers to char , each of which points to a string that provides information about the environment for this execution of the program (5.1.2.3.2).

Consequently, under a [hosted environment](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions) supporting this common extension, it is possible to access the environment through a modified form of `main()` :

However, modifying the environment by any means may cause the environment memory to be reallocated, with the result that `envp` now references an incorrect location. For example, when compiled with GCC 4.8.1 and run on a 32-bit Intel GNU/Linux machine, the following code,

yields

It is evident from these results that the environment has been relocated as a result of the call to `setenv()` . The external variable `environ` is updated to refer to the current environment; the `envp` parameter is not.

An environment pointer may also become invalidated by subsequent calls to `getenv().` (See [ENV34-C. Do not store pointers returned by certain functions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/environment-env/env34-c) for more information.)

### Noncompliant Code Example (POSIX)

After a call to the POSIX `setenv()` function or to another function that modifies the environment, the `envp` pointer may no longer reference the current environment. The *Portable Operating System Interface (POSIX <sup>®</sup> ), Base Specifications, Issue 7* \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \], states

> Unanticipated results may occur if `setenv()` changes the external variable `environ` . In particular, if the optional `envp` argument to `main()` is present, it is not changed, and thus may point to an obsolete copy of the environment (as may any other copy of `environ` ).

This noncompliant code example accesses the `envp` pointer after calling `setenv()` :

Because `envp` may no longer point to the current environment, this program has unanticipated behavior.

### Compliant Solution (POSIX)

Use `environ` in place of `envp` when defined:

### Noncompliant Code Example (Windows)

After a call to the Windows [`_putenv_s()`](https://msdn.microsoft.com/en-us/library/eyw7eyfw.aspx) function or to another function that modifies the environment, the `envp` pointer may no longer reference the environment.

According to the Visual C++ reference \[ [MSDN](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MSDN) \]

> The environment block passed to `main` and `wmain` is a "frozen" copy of the current environment. If you subsequently change the environment via a call to `_putenv` or `_wputenv` , the current environment (as returned by `getenv` / `_wgetenv` and the `_environ` / `_wenviron` variable) will change, but the block pointed to by `envp` will not change.

This noncompliant code example accesses the `envp` pointer after calling `_putenv_s()` :

Because `envp` no longer points to the current environment, this program has unanticipated behavior.

### Compliant Solution (Windows)

This compliant solution uses the [`_environ`](https://msdn.microsoft.com/en-us/library/stxk41x1.aspx) variable in place of `envp` :

### Compliant Solution

This compliant solution can reduce remediation time when a large amount of noncompliant `envp` code exists. It replaces

with

This compliant solution may need to be extended to support other implementations that support forms of the external variable `environ` .

## ENV32-C. All exit handlers must return normally

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/07.environment-env/4.env32-c.md) — SHA-256 `dbdb24b0b10a62ee19ceda16356a66c4914dfe1abd58e9530bbed75721db9fea`。

The C Standard provides three functions that cause an application to terminate normally: `_Exit()` , `exit()` , and `quick_exit()` . These are collectively called *exit functions* . When the `exit()` function is called, or control transfers out of the `main()` entry point function, functions registered with `atexit()` are called (but not `at_quick_exit()` ). When the `quick_exit()` function is called, functions registered with `at_quick_exit()` (but not `atexit()` ) are called. These functions are collectively called *exit handlers* .  When the `_Exit()` function is called, no exit handlers or signal handlers are called.

Exit handlers must terminate by returning. It is important and potentially safety-critical for all exit handlers to be allowed to perform their cleanup actions. This is particularly true because the application programmer does not always know about handlers that may have been installed by support libraries. Two specific issues include nested calls to an exit function and terminating a call to an exit handler by invoking `longjmp` .

A nested call to an exit function is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See [undefined behavior 187](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_187) .) This behavior can occur only when an exit function is invoked from an exit handler or when an exit function is called from within a signal handler. (See [SIG30-C. Call only asynchronous-safe functions within signal handlers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/signals-sig/sig30-c) .)

If a call to the `longjmp()` function is made that would terminate the call to a function registered with `atexit()` , the behavior is [undefined behavior 187](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_187) .

### Noncompliant Code Example

In this noncompliant code example, the `exit1()` and `exit2()` functions are registered by `atexit()` to perform required cleanup upon program termination. However, if `some_condition` evaluates to true, `exit()` is called a second time, resulting in [undefined behavior 187](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_187) .

Functions registered by the `atexit()` function are called in the reverse order from which they were registered. Consequently, if `exit2()` exits in any way other than by returning, `exit1()` will not be executed. The same may also be true for `atexit()` handlers installed by support libraries.

### Compliant Solution

A function that is registered as an exit handler by `atexit()` must exit by returning, as in this compliant solution:

### Noncompliant Code Example

In this noncompliant code example, `exit1()` is registered by `atexit()` so that upon program termination, `exit1()` is called. The `exit1()` function jumps back to `main()` to return, with undefined results.

### Compliant Solution

This compliant solution does not call `longjmp()` but instead returns from the exit handler normally:

## ENV33-C. Do not call system()

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/07.environment-env/5.env33-c.md) — SHA-256 `bd4a7b9f69380d82197caf4c86027957b125a8de681b180847f09a209f9751a2`。

The C Standard `system()` function executes a specified command by invoking an [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) command processor, such as a UNIX shell or `CMD.EXE` in Microsoft Windows. The POSIX [`popen()`](https://pubs.opengroup.org/onlinepubs/9699919799/) and Windows `_popen()` functions also invoke a command processor but create a pipe between the calling program and the executed command, returning a pointer to a stream that can be used to either read from or write to the pipe \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \].

Use of the system() function can result in exploitable [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) , in the worst case allowing execution of arbitrary system commands. Situations in which calls to system() have high risk include the following:

-   When passing an unsanitized or improperly sanitized command string originating from a tainted source
-   If a command is specified without a path name and the command processor path name resolution mechanism is accessible to an attacker
-   If a relative path to an executable is specified and control over the current working directory is accessible to an attacker
-   If the specified executable program can be spoofed by an attacker

Do not invoke a command processor via `system()` or equivalent functions to execute a command.

### Noncompliant Code Example

In this noncompliant code example, the `system()` function is used to execute `any_cmd` in the host environment.

If this code is compiled and run with elevated privileges on a Linux system, for example, an attacker can create an account by entering the following string:

The shell would interpret this string as two separate commands:

and create a new user account that the attacker can use to access the compromised system.

This noncompliant code example also violates [STR02-C. Sanitize data passed to complex subsystems](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/characters-and-strings-str/str02-c) .

### Compliant Solution (POSIX)

In this compliant solution, the call to `system()` is replaced with a call to `execve()` . The `exec` family of functions does not use a full shell interpreter, so it is not vulnerable to command-injection attacks, such as the one illustrated in the noncompliant code example.

The `execlp()` , `execvp()` , and (nonstandard) `execvP()` functions duplicate the actions of the shell in searching for an executable file if the specified file name does not contain a forward slash character ( `/` ). As a result, they should be used without a forward slash character ( `/` ) only if the `PATH` environment variable is set to a safe value, as described in [ENV03-C. Sanitize the environment when invoking external programs](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/environment-env/env03-c) .

The `execl()` , `execle()` , `execv()` , and `execve()` functions do not perform path name substitution.

Additionally, precautions should be taken to ensure the external executable cannot be modified by an untrusted user, for example, by ensuring the executable is not writable by the user.

This compliant solution is significantly different from the preceding noncompliant code example. First, `input` is incorporated into the `args` array and passed as an argument to `execve()` , eliminating concerns about buffer overflow or string truncation while forming the command string. Second, this compliant solution forks a new process before executing `"/usr/bin/any_cmd"` in the child process. Although this method is more complicated than calling `system()` , the added security is worth the additional effort.

The exit status of 127 is the value set by the shell when a command is not found, and POSIX recommends that applications should do the same. XCU, Section 2.8.2, of *Standard for Information Technology—Portable Operating System Interface (POSIX®), Base Specifications, Issue 7* \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \], says

> If a command is not found, the exit status shall be 127. If the command name is found, but it is not an executable utility, the exit status shall be 126. Applications that invoke utilities without using the shell should use these exit status values to report similar errors.

### Compliant Solution (Windows)

This compliant solution uses the Microsoft Windows [`CreateProcess()`](https://msdn.microsoft.com/en-us/library/windows/desktop/ms682425(v=vs.85).aspx) API:

This compliant solution relies on the `input` parameter being non- `const` . If it were `const` , the solution would need to create a copy of the parameter because the `CreateProcess()` function can modify the command-line arguments to be passed into the newly created process.

This solution creates the process such that the child process does not inherit any handles from the parent process, in compliance with [WIN03-C. Understand HANDLE inheritance](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/microsoft-windows-win/win03-c) .

### Noncompliant Code Example (POSIX)

This noncompliant code invokes the C `system()` function to remove the `.config` file in the user's home directory.

If the vulnerable program has elevated privileges, an attacker can manipulate the value of the `HOME` environment variable such that this program can remove any file named `.config` anywhere on the system.

### Compliant Solution (POSIX)

An alternative to invoking the `system()` call to execute an external program to perform a required operation is to implement the functionality directly in the program using existing library calls. This compliant solution calls the POSIX `unlink()` function to remove a file without invoking the `system()` function \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) [\]](https://pubs.opengroup.org/onlinepubs/9699919799/)

The `unlink()` function is not susceptible to a symlink attack where the final component of `pathname` (the file name) is a symbolic link because `unlink()` will remove the symbolic link and not affect any file or directory named by the contents of the symbolic link. (See [FIO01-C. Be careful using functions that use file names for identification](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio01-c) .)  While this reduces the susceptibility of the `unlink()` function to symlink attacks, it does not eliminate it.  The `unlink()` function is still susceptible if one of the directory names included in the `pathname` is a symbolic link.  This could cause the `unlink()` function to delete a similarly named file in a different directory.

### Compliant Solution (Windows)

This compliant solution uses the Microsoft Windows [`SHGetKnownFolderPath()`](https://msdn.microsoft.com/en-us/library/windows/desktop/bb762188(v=vs.85).aspx) API to get the current user's My Documents folder, which is then combined with the file name to create the path to the file to be deleted. The file is then removed using the [`DeleteFile()`](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363915(v=vs.85).aspx) API.

### Exceptions

**ENV33-C-EX1** : It is permissible to call `system()` with a null pointer argument to determine the presence of a command processor for the system.

## ENV34-C. Do not store pointers returned by certain functions

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/07.environment-env/6.env34-c.md) — SHA-256 `100821de2b76fa76c99e1a2eecf9b8017dca34db27be2f4a66042c0c3841bc71`。

The C Standard, 7.24.4.6, paragraph 4 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> The `getenv` function returns a pointer to a string associated with the matched list member. The string pointed to shall not be modified by the program but may be overwritten by a subsequent call to the `getenv` function. If the specified `name` cannot be found, a null pointer is returned.

This paragraph gives an implementation the latitude, for example, to return a pointer to a statically allocated buffer. Consequently, do not store this pointer because the string data it points to may be overwritten by a subsequent call to the `getenv()` function or invalidated by modifications to the environment. This string should be referenced immediately and discarded. If later use is anticipated, the string should be copied so the copy can be safely referenced as needed.

The `getenv()` function is not thread-safe. Make sure to address any possible race conditions resulting from the use of this function.

The `asctime()` , `localeconv()` , `setlocale()` , and `strerror()` functions have similar restrictions, as well as the time-conversion functions from C Standard, 7.29.3, such as `localtime()`.  Do not access the objects returned by any of these functions after a subsequent call to the function.  Accessing such objects is undefined behavior [120](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_120) , [201](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_201) , or [214](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_214) .

### Noncompliant Code Example

This noncompliant code example attempts to compare the value of the `TMP` and `TEMP` environment variables to determine if they are the same:

This code example is noncompliant because the string referenced by `tmpvar` may be overwritten as a result of the second call to the `getenv()` function. As a result, it is possible that both `tmpvar` and `tempvar` will compare equal even if the two environment variables have different values.

### Compliant Solution

This compliant solution uses the `malloc()` and `strcpy()` functions to copy the string returned by `getenv()` into a dynamically allocated buffer:

### Compliant Solution (C23 or POSIX)

POSIX provides the `strdup()` function, which can make a copy of the environment variable string \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \]. The `strdup()` function is also included in *Extensions to the C Library—Part II* \[ [ISO/IEC TR 24731-2:2010](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IECTR24731-2-2010), and is now part of [\[ ISO/IEC 9899:2024 \]](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024).

### Compliant Solution (Windows)

Microsoft Windows provides the `_dupenv_s()` and [`wdupenv_s()`](https://msdn.microsoft.com/en-us/library/ms175774.aspx) functions for getting a value from the current environment \[ [MSDN](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MSDN) \]. The `_dupenv_s()` function searches the list of environment variables for a specified name. If the name is found, a buffer is allocated; the variable's value is copied into the buffer, and the buffer's address and number of elements are returned. The `_dupenv_s()` and `_wdupenv_s()` functions provide more convenient alternatives to `getenv_s()` and `_wgetenv_s()` because each function handles buffer allocation directly.

The caller is responsible for freeing any allocated buffers returned by these functions by calling `free()` .

### Noncompliant Code Example

This noncompliant code example saves the pointer returned by `localtime()`, and only dereferences it after the thread has exited, resulting in undefined behavior 214.

### Compliant Solution (`localtime()`)

This compliant solution preserves the string representation of the current time, removing dependence on the `localtime()` return value.
