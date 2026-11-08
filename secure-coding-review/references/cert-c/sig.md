# CERT C — SIG

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## SIG00-C. Mask signals handled by noninterruptible signal handlers

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/18.signals-sig/2.sig00-c.md) — SHA-256 `253e7d9ed5e04e847d62772a2702c7e1cdf5ee0e2a328c801e843b9def881a32`。

A signal is a mechanism for transferring control that is typically used to notify a process that an event has occurred. That process can then respond to the event accordingly. The C Standard provides functions for sending and handling signals within a C program.

Processes handle signals by registering a signal handler using the `signal()` function, which is specified as

This signal handler is conceptually equivalent to

Signal handlers can be interrupted by signals, including their own. If a signal is not reset before its handler is called, the handler can interrupt its own execution. A handler that always successfully executes its code despite interrupting itself or being interrupted is [async-signal-safe](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-async-signal-safe) .

Some platforms provide the ability to mask signals while a signal handler is being processed. If a signal is masked while its own handler is processed, the handler is noninterruptible and need not be async-signal-safe. However, even when a signal is masked while its own handler is processed, the handler must still avoid invoking async-signal-safe unsafe functions because their execution may be (or have been) interrupted by another signal.

[Vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) can arise if a signal handler that is not async-signal-safe is interrupted with any unmasked signal, including its own.

### Noncompliant Code Example

This noncompliant code example registers a single signal handler to process both `SIGUSR1` and `SIGUSR2` . The variable `sig2` should be set to `1` if one or more `SIGUSR1` signals are followed by `SIGUSR2` , essentially implementing a finite state machine within the signal handler.

Unfortunately, a race condition occurs in the implementation of `handler()` . If `handler()` is called to handle `SIGUSR1` and is interrupted to handle `SIGUSR2` , it is possible that `sig2` will not be set.

### Compliant Solution (POSIX)

The POSIX `sigaction()` function assigns handlers to signals in a similar manner to the C `signal()` function, but it also allows signal masks to be set explicitly. Consequently, `sigaction()` can be used to prevent a signal handler from interrupting itself.

POSIX recommends `sigaction()` and deprecates the use of `signal()` to register signal handlers. Unfortunately, `sigaction()` is not defined in the C Standard and is consequently not as portable a solution.

## SIG01-C. Understand implementation-specific details regarding signal handler persistence

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/18.signals-sig/3.sig01-c.md) — SHA-256 `29e5f178a141afe4041ab11acfdfa01aa68f79da0058f96105681cd602a57c7c`。

The `signal()` function has [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) behavior and behaves differently on Windows, for example, than it does on many UNIX systems.

The following code example shows this behavior:

Many UNIX (and UNIX-like) systems automatically reinstall signal handlers upon handler execution, meaning that the signal handler defined by the user is left in place until it is explicitly removed. For example, when this code is compiled with GCC 3.4.4 and executed under Red Hat Linux, `SIGINT` is captured both times by `handler` :

When a signal handler is installed with the `signal()` function in Windows and some UNIX systems, the default action is restored for that signal after the signal is triggered. This means that signal handlers are not automatically reinstalled. For example, when this code is compiled with Microsoft Visual Studio 2005, version 8.0, only the first `SIGINT` is captured by `handler` :

The second `SIGINT` executes the default action, which is to terminate program execution.

Different actions must be taken depending on whether or not the application requires signal handlers to be persistent.

## Persistent Handlers

Asynchronous signals may originate from malicious actors external to the process. Consequently, [vulnerabilities](http://BB.%20Definitions#vulnerability) may exist if the signal-handler-persistence behavior is inconsistent with the developer's expectations, such as when the developer expects the signal handler to persist but it does not.

### Noncompliant Code Example

This noncompliant code example fails to persist the signal handler on Windows platforms and on those UNIX systems where handlers are not persistent by default:

### Noncompliant Code Example

A common approach to create persistent signal handlers is to call `signal()` from within the handler itself, consequently *unresetting* the reset signal:

Unfortunately, this solution still contains a race window, starting when the host environment resets the signal and ending when the handler calls `signal()` . During that time, a second signal sent to the program will trigger the default signal behavior, defeating the persistent behavior. (See [SIG34-C. Do not call signal() from within interruptible signal handlers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/signals-sig/sig34-c) .)

A secure solution must prevent the environment from resetting the signal in the first place, guaranteeing persistence. Unfortunately, Windows does not provide a secure solution to this problem.

### Compliant Solution (POSIX)

The POSIX `sigaction()` function assigns handlers to signals in a manner similar to the C `signal()` function but also allows signal-handler persistence to be controlled via the `SA_RESETHAND` flag. (Leaving the flag clear makes the handler persistent.)

POSIX recommends `sigaction()` and deprecates `signal()` . Unfortunately, `sigaction()` is not defined in the C Standard and is consequently not as portable a solution.

## Nonpersistent Handlers

Errors may also occur when the developer expects the default action to be restored for a signal but the signal handler persists instead.

### Noncompliant Code Example (UNIX)

This noncompliant code example fails to reset the signal handler to its default behavior on systems where handlers are persistent by default:

### Compliant Solution (UNIX and Windows)

A C-compliant solution to reset the handler on a UNIX system is to rebind the signal to the default handler in the first line of the handler itself. Windows, however, automatically resets handlers to their default behavior.

With the compliant solution for UNIX, no race condition occurs that can be [exploited](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-exploit) by an attacker sending a second signal. This is because a second signal sent to the handler, before the latter calls `   signal(signum, SIG_DFL)  ` , will only cause the handler to restart and call `signal()` anyway.

This solution is an exception to [SIG34-C. Do not call signal() from within interruptible signal handlers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/signals-sig/sig34-c) .

### Compliant Solution (POSIX)

The POSIX `sigaction()` function assigns handlers to signals in a manner similar to the C `signal()` function but also allows signal-handler persistence to be controlled via the `SA_RESETHAND` flag. (Setting the flag makes the handler nonpersistent.)

## SIG02-C. Avoid using signals to implement normal functionality

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/18.signals-sig/4.sig02-c.md) — SHA-256 `b94d1b7250993e41c3873b8b383c6828c0142da53a62d26e2330d7222e5b2694`。

Avoid using signals to implement normal functionality. Signal handlers are severely limited in the actions they can perform in a portably secure manner. Their use should be reserved for abnormal events that can be serviced by little more than logging.

### Noncompliant Code Example

This noncompliant code example uses signals as a means to pass state changes around in a multithreaded environment:

However, using signals for such functionality often leads to nonportable or otherwise complicated solutions.

This code illustrates one thread using a signal to wake up a second thread. Using an architecture's native thread library usually allows for a more sophisticated means of sending messages between threads.

### Compliant Solution (POSIX)

A better solution, in this case, is to use condition variables. This code example uses a condition variable from the POSIX `pthread` library \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \]:

### Compliant Solution (Windows)

This compliant solution uses a condition variable from the Win32 API \[ [MSDN](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MSDN) \]:

### Noncompliant Code Example

This noncompliant code example is from a [signal race vulnerability](https://seclists.org/bugtraq/1997/Jan/0011.html) in WU-FTPD v2.4 \[ [Greenman 1997](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Greenman97) \]:

A serious [exploit](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-exploit) can occur if `SIGURG` is caught immediately following the elevation of privileges in `dologout()` . If the `longjmp()` in the `SIGURG` handler `myoob()` is invoked, execution returns to the main processing loop with an effective UID of 0.

Please note that this code sample violates [SIG30-C. Call only asynchronous-safe functions within signal handlers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/signals-sig/sig30-c) and [SIG31-C. Do not access shared objects in signal handlers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/signals-sig/sig31-c) .

An immediate fix is to ensure that `dologout()` cannot be interrupted by a `SIGURG` :

A better solution is for `myoob()` to set a failure flag of type `   volatile sig_atomic_t  ` that is periodically checked within the main loop:

This solution, however, still violates [SIG30-C. Call only asynchronous-safe functions within signal handlers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/signals-sig/sig30-c) and [SIG31-C. Do not access shared objects in signal handlers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/signals-sig/sig31-c) .

### Compliant Solution

A compliant solution (not shown) is to not use signals to signify lost connections and to design the system to have a robust error-handling mechanism (see [ERR00-C. Adopt and implement a consistent and comprehensive error-handling policy](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err00-c) ).

## SIG30-C. Call only asynchronous-safe functions within signal handlers

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/18.signals-sig/2.sig30-c.md) — SHA-256 `fb09ff15c177d4f725fe80ee675a8c4785bdc985e26d50c5f145b2acc74ff2cf`。

Call only [asynchronous-safe functions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-asynchronous-safefunction) within signal handlers. For [strictly conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-strictlyconforming) programs, only the C standard library functions `abort()` , `_Exit()` , `quick_exit()` , and `signal()` can be safely called from within a signal handler.

The C Standard, 7.14.1.1, paragraph 5 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states that if the signal occurs other than as the result of calling the `abort()` or `raise()` function, the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) if

> If the signal occurs other than as the result of calling the abort or raise function, the behavior is undefined if the signal handler refers to any object with static or thread storage duration that is not a lock-free atomic object and that is not declared with the constexpr storage-class specifier other than by assigning a value to an object declared as volatile sig \_ atomic \_ t , or the signal handler calls any function in the standard library other than
>
> —  the abort function,
>
> —  the \_ Exit function,
>
> —  the quick \_ exit function,
>
> —  the functions in \<stdatomic.h> (except where explicitly stated otherwise) when the atomic arguments are lock-free,
>
> —  the atomic \_ is \_ lock \_ free function with any atomic argument, or
>
> —  the signal function with the first argument equal to the signal number corresponding to the signal that caused the invocation of the handler. Furthermore, if such a call to the signal function results in a SIG \_ ERR return, the object designated by errno has an indeterminate representation. 294)

Implementations may define a list of additional asynchronous-safe functions. These functions can also be called within a signal handler. This restriction applies to library functions as well as application-defined functions.

According to the C Rationale, 7.14.1.1 \[ [C99 Rationale 2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-C992003) \],

> When a signal occurs, the normal flow of control of a program is interrupted. If a signal occurs that is being trapped by a signal handler, that handler is invoked. When it is finished, execution continues at the point at which the signal occurred. This arrangement can cause problems if the signal handler invokes a library function that was being executed at the time of the signal.

In general, it is not safe to invoke I/O functions from within signal handlers. Programmers should ensure a function is included in the list of an implementation's asynchronous-safe functions for all implementations the code will run on before using them in signal handlers.

### Noncompliant Code Example

In this noncompliant example, the C standard library functions `fputs()` and `free()` are called from the signal handler via the function `log_message()` . Neither function is [asynchronous-safe](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-asynchronous-safefunction) .

### Compliant Solution

Signal handlers should be as concise as possible—ideally by unconditionally setting a flag and returning. This compliant solution sets a flag of type `   volatile sig_atomic_t  ` and returns; the `log_message()` and `free()` functions are called directly from `main()` :

### Noncompliant Code Example ( `longjmp()` )

Invoking the `longjmp()` function from within a signal handler can lead to [undefined behavior 132](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_132) if it results in the invocation of any non- [asynchronous-safe](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-asynchronous-safe) functions. Consequently, neither `longjmp()` nor the POSIX `siglongjmp()` functions should ever be called from within a signal handler.

This noncompliant code example is similar to a [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) in an old version of Sendmail \[ [VU #834865](https://www.kb.cert.org/vuls/id/834865) \]. The intent is to execute code in a `main()` loop, which also logs some data. Upon receiving a `SIGINT` , the program transfers out of the loop, logs the error, and terminates.

However, an attacker can [exploit](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-exploit) this noncompliant code example by generating a `SIGINT` just before the second `if` statement in `log_message()` . The result is that `longjmp()` transfers control back to `main()` , where `log_message()` is called again. However, the first `if` statement would not be executed this time (because `buf` is not set to `NULL` as a result of the interrupt), and the program would write to the invalid memory location referenced by `buf0` .

### Compliant Solution

In this compliant solution, the call to `longjmp()` is removed; the signal handler sets an error flag instead:

### Noncompliant Code Example ( `raise()` )

In this noncompliant code example, the `int_handler()` function is used to carry out tasks specific to `SIGINT` and then raises `SIGTERM` . However, there is a nested call to the `raise()` function, which is [undefined behavior 131](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_131) .

### Compliant Solution

In this compliant solution, `int_handler()` invokes `term_handler()` instead of raising `SIGTERM` :

#### Implementation Details

##### POSIX

The following table from the POSIX standard \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \] defines a set of functions that are [asynchronous-signal-safe](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-asynchronous-safefunction) . Applications may invoke these functions, without restriction, from a signal handler.

|                                |                              |                                    |                                   |
|--------------------------------|------------------------------|------------------------------------|-----------------------------------|
| `_Exit()`         | `fexecve()`     | `posix_trace_event()` | `sigprocmask()`      |
| `_exit()`         | `fork()`        | `pselect()`           | `sigqueue()`         |
| `abort()`         | `fstat()`       | `pthread_kill()`      | `sigset()`           |
| `accept()`        | `fstatat()`     | `pthread_self()`      | `sigsuspend()`       |
| `access()`        | `fsync()`       | `pthread_sigmask()`   | `sleep()`            |
| `aio_error()`     | `ftruncate()`   | `raise()`             | `sockatmark()`       |
| `aio_return()`    | `futimens()`    | `read()`              | `socket()`           |
| `aio_suspend()`   | `getegid()`     | `readlink()`          | `socketpair()`       |
| `alarm()`         | `geteuid()`     | `readlinkat()`        | `stat()`             |
| `bind()`          | `getgid()`      | `recv()`              | `symlink()`          |
| `cfgetispeed()`   | `getgroups()`   | `recvfrom()`          | `symlinkat()`        |
| `cfgetospeed()`   | `getpeername()` | `recvmsg()`           | `tcdrain()`          |
| `cfsetispeed()`   | `getpgrp()`     | `rename()`            | `tcflow()`           |
| `cfsetospeed()`   | `getpid()`      | `renameat()`          | `tcflush()`          |
| `chdir()`         | `getppid()`     | `rmdir()`             | `tcgetattr()`        |
| `chmod()`         | `getsockname()` | `select()`            | `tcgetpgrp()`        |
| `chown()`         | `getsockopt()`  | `sem_post()`          | `tcsendbreak()`      |
| `clock_gettime()` | `getuid()`      | `send()`              | `tcsetattr()`        |
| `close()`         | `kill()`        | `sendmsg()`           | `tcsetpgrp()`        |
| `connect()`       | `link()`        | `sendto()`            | `time()`             |
| `creat()`         | `linkat()`      | `setgid()`            | `timer_getoverrun()` |
| `dup()`           | `listen()`      | `setpgid()`           | `timer_gettime()`    |
| `dup2()`          | `lseek()`       | `setsid()`            | `timer_settime()`    |
| `execl()`         | `lstat()`       | `setsockopt()`        | `times()`            |
| `execle()`        | `mkdir()`       | `setuid()`            | `umask()`            |
| `execv()`         | `mkdirat()`     | `shutdown()`          | `uname()`            |
| `execve()`        | `mkfifo()`      | `sigaction()`         | `unlink()`           |
| `faccessat()`     | `mkfifoat()`    | `sigaddset()`         | `unlinkat()`         |
| `fchdir()`        | `mknod()`       | `sigdelset()`         | `utime()`            |
| `fchmod()`        | `mknodat()`     | `sigemptyset()`       | `utimensat()`        |
| `fchmodat()`      | `open()`        | `sigfillset()`        | `utimes()`           |
| `fchown()`        | `openat()`      | `sigismember()`       | `wait()`             |
| `fchownat()`      | `pause()`       | `signal()`            | `waitpid()`          |
| `fcntl()`         | `pipe()`        | `sigpause()`          | `write()`            |
| `fdatasync()`     | `poll()`        | `sigpending()`        | `      `                          |

  

All functions not listed in this table are considered to be unsafe with respect to signals. In the presence of signals, all POSIX functions behave as defined when called from or interrupted by a signal handler, with a single exception: when a signal interrupts an unsafe function and the signal handler calls an unsafe function, the behavior is undefined.

The C Standard, 7.14.1.1, paragraph 4 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> If the signal occurs as the result of calling the abort or raise function, the signal handler shall not call the raise function.

However, in the description of `signal()` , POSIX \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \] states

> This restriction does not apply to POSIX applications, as POSIX.1-2008 requires `raise()` to be async-signal-safe.

See also [undefined behavior 131.](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_131)

##### OpenBSD

The OpenBSD [`signal()`](http://www.openbsd.org/cgi-bin/man.cgi?query=signal) manual page lists a few additional functions that are asynchronous-safe in OpenBSD but "probably not on other systems" \[ [OpenBSD](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-OpenBSD) \], including `snprintf()` , `vsnprintf()` , and `syslog_r()` but only when the `   syslog_data struct  ` is initialized as a local variable.

## SIG31-C. Do not access shared objects in signal handlers

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/18.signals-sig/3.sig31-c.md) — SHA-256 `ca0c4ac1281522ec514f34af3ba29af7c61558be2c1209170a19010a1d426e92`。

Accessing or modifying shared objects in signal handlers can result in race conditions that can leave data in an inconsistent state. The two exceptions (C Standard, 5.1.2.3, paragraph 5) to this rule are the ability to read from and write to lock-free atomic objects and variables of type `volatile sig_atomic_t`. Accessing any other type of object from a signal handler is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior). (See [undefined behavior 131](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_131).)

The need for the `volatile` keyword is described in [DCL22-C. Use volatile for data that cannot be cached](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl22-c).

The type `sig_atomic_t` is the integer type of an object that can be accessed as an atomic entity even in the presence of asynchronous interrupts. The type of `sig_atomic_t` is [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior), though it provides some guarantees. Integer values ranging from `SIG_ATOMIC_MIN` through `SIG_ATOMIC_MAX`, inclusive, may be safely stored to a variable of the type. In addition, when `sig_atomic_t` is a signed integer type, `SIG_ATOMIC_MIN` must be no greater than `−127` and `SIG_ATOMIC_MAX` no less than `127`. Otherwise, `SIG_ATOMIC_MIN` must be `0` and `SIG_ATOMIC_MAX` must be no less than `255`. The macros `SIG_ATOMIC_MIN` and `SIG_ATOMIC_MAX` are defined in the header `<stdint.h>`.

According to the C99 Rationale \[ [C99 Rationale 2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-C992003) \], other than calling a limited, prescribed set of library functions,

> the C89 Committee concluded that about the only thing a [strictly conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-strictlyconforming) program can do in a signal handler is to assign a value to a `volatile static` variable which can be written uninterruptedly and promptly return.

However, this issue was discussed at the April 2008 meeting of ISO/IEC WG14, and it was agreed that there are no known [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) in which it would be an error to read a value from a `volatile sig_atomic_t` variable, and the original intent of the committee was that both reading and writing variables of `volatile sig_atomic_t` would be strictly conforming.

The signal handler may also call a handful of functions, including `abort().` (See [SIG30-C. Call only asynchronous-safe functions within signal handlers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/signals-sig/sig30-c) for more information.)

### Noncompliant Code Example

In this noncompliant code example, `err_msg` is updated to indicate that the `SIGINT` signal was delivered.  The `err_msg` variable is a character pointer and not a variable of type `volatile sig_atomic_t`.

### Compliant Solution (Writing `   volatile sig_atomic_t  ` )

For maximum portability, signal handlers should only unconditionally set a variable of type `volatile sig_atomic_t` and return, as in this compliant solution:

### Compliant Solution (Lock-Free Atomic Access)

Signal handlers can refer to objects with static or thread storage durations that are lock-free atomic objects, as in this compliant solution:

### Exceptions

**SIG31-C-EX1:** The C Standard, 7.14.1.1 paragraph 5 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2024) \], makes a special exception for `errno` when a valid call to the `signal()` function results in a `SIG_ERR` return, allowing `errno` to take an indeterminate value. (See [ERR32-C. Do not rely on indeterminate values of errno](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/signals-sig/sig31-c#) .)

> the signal function with the first argument equal to the signal number corresponding to the signal that caused the invocation of the handler. Furthermore, if such a call to the signal function results in a SIG_ERR return, the object designated by errno has an indeterminate representation.

## SIG34-C. Do not call signal() from within interruptible signal handlers

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/18.signals-sig/4.sig34-c.md) — SHA-256 `cf0f131da0a97631b89bb8f0b5f534cdedd0fb246a4871bdff4426cd1960135e`。

A signal handler should not reassert its desire to handle its own signal. This is often done on *nonpersistent* platforms—that is, platforms that, upon receiving a signal, reset the handler for the signal to SIG_DFL before calling the bound signal handler. Calling `signal()` under these conditions presents a race condition. (See [SIG01-C. Understand implementation-specific details regarding signal handler persistence](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/signals-sig/sig01-c) .)

A signal handler may call `signal()` only if it does not need to be [asynchronous-safe](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-asynchronous-safefunction) (that is, if all relevant signals are masked so that the handler cannot be interrupted).

### Noncompliant Code Example (POSIX)

On nonpersistent platforms, this noncompliant code example contains a race window, starting when the host environment resets the signal and ending when the handler calls `signal()` . During that time, a second signal sent to the program will trigger the default signal behavior, consequently defeating the persistent behavior implied by the call to `signal()` from within the handler to reassert the binding.

If the environment is persistent (that is, it does not reset the handler when the signal is received), the `signal()` call from within the `handler()` function is redundant.

### Compliant Solution (POSIX)

Calling the `signal()` function from within the signal handler to reassert the binding is unnecessary for persistent platforms, as in this compliant solution:

### Compliant Solution (POSIX)

POSIX defines the `sigaction()` function, which assigns handlers to signals in a similar manner to `signal()` but allows the caller to explicitly set persistence. Consequently, the `sigaction()` function can be used to eliminate the race window on nonpersistent platforms, as in this compliant solution:

Although the handler in this example does not call `signal()` , it could do so safely because the signal is masked and the handler cannot be interrupted. If the same handler is installed for more than one signal, the signals must be masked explicitly in `act.sa_mask` to ensure that the handler cannot be interrupted because the system masks only the signal being delivered.

POSIX recommends that new applications should use `sigaction()` rather than `signal()` . The `sigaction()` function is not defined by the C Standard and is not supported on some platforms, including Windows.

### Compliant Solution (Windows)

There is no safe way to implement persistent signal-handler behavior on Windows platforms, and it should not be attempted. If a design depends on this behavior, and the design cannot be altered, it may be necessary to claim a deviation from this rule after completing an appropriate risk analysis.

The reason for this is that Windows is a nonpersistent platform as discussed above.  Just before calling the current handler function, Windows resets the handler for the next occurrence of the same signal to `SIG_DFL` . If the handler calls `signal()` to reinstall itself, there is still a race window. A signal might occur between the start of the handler and the call to `signal()` , which would invoke the default behavior instead of the desired handler.

### Exceptions

**SIG34-C-EX1:** For implementations with persistent signal handlers, it is safe for a handler to modify the behavior of its own signal. Behavior modifications include ignoring the signal, resetting to the default behavior, and having the signal handled by a different handler. A handler reasserting its binding is also safe but unnecessary.

The following code example resets a signal handler to the system's default behavior:

## SIG35-C. Do not return from a computational exception signal handler

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/18.signals-sig/5.sig35-c.md) — SHA-256 `c1c3d758a38957440dbe861753e8a4f208bd26141b7dc0fd2a9ee1c6d14b51f1`。

According to the C Standard, 7.14.1.1 paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], if a signal handler returns when it has been entered as a result of a computational exception (that is, with the value of its argument of `SIGFPE` , `SIGILL` , `SIGSEGV` , or any other implementation-defined value corresponding to such an exception) returns, then the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See [undefined behavior 130](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-130) .)

> When a signal occurs and func points to a function, it is implementation-defined whether the equivalent of signal (sig, SIG_DFL); is executed or the implementation prevents some implementation- defined set of signals (at least including sig) from occurring until the current signal handling has completed; in the case of SIGILL, the implementation may alternatively define that no action is taken. Then the equivalent of (\*func)(sig); is executed. If and when the function returns, if the value of sig is SIGFPE, SIGILL, SIGSEGV, or any other implementation-defined value corresponding to a computational exception, the behavior is undefined; otherwise the program will resume execution at the point it was interrupted.

The Portable Operating System Interface (POSIX <sup>®</sup> ), Base Specifications, Issue 7 \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \], adds `SIGBUS` to the list of computational exception signal handlers:

> The behavior of a process is undefined after it returns normally from a signal-catching function for a `SIGBUS` , `SIGFPE` , `SIGILL` , or `SIGSEGV` signal that was not generated by `kill()` , `sigqueue()` , or `raise()` .

Do not return from `SIGFPE` , `SIGILL` , `SIGSEGV` , or any other implementation-defined value corresponding to a computational exception, such as `SIGBUS` on POSIX systems, regardless of how the signal was generated.

### Noncompliant Code Example

In this noncompliant code example, the division operation has [undefined behavior 41](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-41) if `denom` equals `0.` (See [INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int33-c) )  and may result in a `SIGFPE` signal to the program.)

When compiled with some implementations, this noncompliant code example will loop infinitely if given the input `0` . It illustrates that even when a `SIGFPE` handler attempts to fix the error condition while obeying all other rules of signal handling, the program still does not behave as expected.

### Compliant Solution

The only portably safe way to leave a `SIGFPE` , `SIGILL` , or `SIGSEGV` handler is to invoke `abort()` , `quick_exit()` , or `_Exit()` . In the case of `SIGFPE` , the default action is [abnormal termination](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) , so no user-defined handler is required:

#### Implementation Details

Some implementations define useful behavior for programs that return from one or more of these signal handlers. For example, Solaris provides the `sigfpe()` function specifically to set a `SIGFPE` handler that a program may safely return from. [Oracle](https://docs.oracle.com/) also provides platform-specific computational exceptions for the `SIGTRAP` , `SIGBUS` , and `SIGEMT` signals. Finally, GNU libsigsegv takes advantage of the ability to return from a `SIGSEGV` handler to implement page-level memory management in user mode.
