# CERT C — WIN

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## WIN00-C. Be specific when dynamically loading libraries

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/14.microsoft-windows-win/2.win00-c.md) — SHA-256 `15c86613dbf9b92d86f54555f0933624ed5d3395590ebdd762e2b430943c04ee`。

The `LoadLibrary()` or `LoadLibraryEx()` function calls \[ [MSDN](https://msdn.microsoft.com/en-us/library/windows/desktop/ms684175(v=vs.85).aspx) \] allow you to dynamically load a library at runtime and use a specific algorithm to locate the library within the file system \[ [MSDN](https://msdn.microsoft.com/en-us/library/windows/desktop/ms682586(v=vs.85).aspx) \]. It is possible for an attacker to place a file on the DLL search path such that your application inadvertently loads and executes arbitrary source code.

### Noncompliant Code Example

If an attacker were to place a malicious DLL named MyLibrary.dll higher on the search path than where the library resides, she could trigger arbitrary code to execute either via the `DllMain()` entrypoint (which is called automatically by the system loader) or by providing an implementation for `MyFunction()` , either of which would run within the security context of your application. If your application runs with elevated privileges (such as a service application), an escalation of privileges could result.

### Compliant Solution

By refusing to load a library unless it is located precisely where expected, you reduce the chance of executing arbitrary code when dynamically loading libraries. This compliant solution uses `LoadLibraryEx()` to ensure that only the application and System32 directories are searched (eliminating other search paths such as the current directory or `PATH` environment variable):

## WIN01-C. Do not forcibly terminate execution

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/14.microsoft-windows-win/3.win01-c.md) — SHA-256 `2ee63ebd4dcbee03b9b4773f17048ec6bccffe65e17ec74d801936af87313711`。

When a thread terminates under normal conditions, thread-specific resources such as the initial stack space and thread-specific `HANDLE` objects are released automatically by the system and notifications are sent to other parts of the application, such as `DLL_THREAD_DETACH` messages being sent to DLLs.  However, if a thread is forcibly terminated by calling [`TerminateThread()`](https://msdn.microsoft.com/en-us/library/windows/desktop/ms686717(v=vs.85).aspx) , the cleanup and notifications do not have the chance to run.  MSDN states

> **TerminateThread** is a dangerous function that should only be used in the most extreme cases. You should call **TerminateThread** only if you know exactly what the target thread is doing, and you control all of the code that the target thread could possibly be running at the time of the termination. For example, **TerminateThread** can result in the following problems:
>
> -   If the target thread owns a critical section, the critical section will not be released.
> -   If the target thread is allocating memory from the heap, the heap lock will not be released.
> -   If the target thread is executing certain kernel32 calls when it is terminated, the kernel32 state for the thread's process could be inconsistent.
> -   If the target thread is manipulating the global state of a shared DLL, the state of the DLL could be destroyed, affecting other users of the DLL.

On some platforms (such as Microsoft Windows XP and Microsoft Windows Server 2003), the thread's initial stack is not freed, causing a resource leak.

Processes behave similar to threads, and so share the same concerns.  Do not use the `TerminateThread()` or `TerminateProcess()` APIs.  Instead, you should prefer to exit threads and processes by returning from the entrypoint, by calling `ExitThread()` , or by calling `ExitProcess()` .

### Noncompliant Code Example

In the following example, `TerminateThread()` is used to forcibly terminate another thread, which can leak resources and leave the application in an indeterminate state.

### Compliant Solution

The compliant solution avoids calling `TerminateThread()` by requesting the thread terminate itself by exiting its entrypoint.  It does so in a lock-free, thread-safe manner by using the `InterlockedCompareExchange()` and `InterlockedExchange()` Win32 APIs.

## WIN02-C. Restrict privileges when spawning child processes

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/14.microsoft-windows-win/4.win02-c.md) — SHA-256 `262531c9cadaa36a9874f8e8edecf823538f7d3a479b3125441cb325a74631be`。

The principle of least privilege states that every program and every user of the system should operate using the least set of privileges necessary to complete the job \[ [Saltzer 1974](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Saltzer74) , [Saltzer 1975](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Saltzer75) \]. The Build Security In website \[ [DHS 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-DHS06) \] provides additional definitions of this principle. Executing with minimal privileges mitigates against exploitation in case a vulnerability is discovered in the code.

### Noncompliant Code Example

An application may spawn another process as part of its normal course of action. On Windows, the newly-spawned process automatically receives the same privileges as the parent process \[ [MSDN](https://msdn.microsoft.com/en-us/library/windows/desktop/ms682425(v=vs.85).aspx) \]. By allowing the child process to run in the same security context as the parent process, the attack surface for the application is extended to the child process. Furthermore, this example allows the child process to inherit handles from the parent process by passing `TRUE` to the `bInheritsHandles` parameter.

It is possible that the act of calling `launch_notepad()` will give the user an elevated Notepad application (from which the user could execute Explorer.exe), allowing the user access to all user's files, change system settings, and so on.

### Compliant Solution

By using the Windows Integrity Mechanism \[ [MSDN](https://msdn.microsoft.com/en-us/library/bb625963.aspx) \] when creating the process, you can assign an integrity level to the launched child process. Doing so allows you to execute the child process with a specific set of privileges instead of defaulting to the parent process's security level.

The compliant solution demonstrates how to launch notepad.exe using a low integrity level, regardless of what privilege level the parent process is running from. It also disallows handle inheritance by passing `FALSE` to the `bInheritsHandles` parameter, because notepad.exe does not require access to any of the process's handles.

Possible values for the integrity level SID strings are listed in the following table:

|                     |                                         |
|:--------------------|:----------------------------------------|
| Integrity level SID | Name                                    |
| S-1-16-4096         | Mandatory Label\\Low Mandatory Level    |
| S-1-16-8192         | Mandatory Label\\Medium Mandatory Level |
| S-1-16-12288        | Mandatory Label\\High Mandatory Level   |
| S-1-16-16384        | Mandatory Label\\System Mandatory Level |

## WIN03-C. Understand HANDLE inheritance

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/14.microsoft-windows-win/5.win03-c.md) — SHA-256 `326be5e1a3301c3f0ba1b30f2894b60dd9118627099a2c7e3e2bd2c748ebb1aa`。

Securable resources such as access tokens, events, files, threads, and others are represented via `HANDLE` objects on Windows \[ [MSDN](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AA.Bibliography-MSDN) \]. Handle inheritance is a two-step process.  When obtaining a `HANDLE` , an option is given to specify whether the object is inheritable or not. This option is usually in the form of a `BOOL` parameter (as in the case of `OpenMutex()` ), or a `SECURITY_DESCRIPTOR` parameter (as in the case of `CreateFile()` ). When creating a process via the `CreateProcess()` family of APIs, a parameter is given specifying whether the spawned process will inherit handles previously flagged as being inheritable. Any handles that were opened as being inheritable will be opened in the child process using the same handle value and access privileges as in the parent process. The parent process can then alert the child process of the handle values via an inter-process communication mechanism, and the child process can use those values as though it had opened the handle \[ [MSDN](https://msdn.microsoft.com/en-us/library/windows/desktop/ms724466(v=vs.85).aspx) \].

When opening handles to securable resources or spawning child processes, prohibit handle inheritance by default to prevent accidental information leakage. If obtaining an inherited handle from a parent process, prevent leakage to subsequent child processes by duplicating the handle without inheritance.

### Noncompliant Code Example (Mutex)

This noncompliant code example attempts to open an existing mutex handle that can be inherited by a child process:

Even if the process does not currently spawn child processes, this code example is noncompliant because future changes involving child processes could leak this handle accidentally.

### Compliant Solution (Mutex)

This compliant solution opens the same mutex without specifying the handle can be inherited by a child process:

### Noncompliant Code Example (Further Inheritance)

In this noncompliant example, the child process is spawned and inherits a single file handle from its parent process. The first argument to the main function is the handle's integer value as a hexadecimal string. However, the child process is not validating that the handle is a valid file handle, and it is not restricting further inheritance of the handle. Additionally, a portability concern arises if the parent process and the child process are the same architecture (for example, if one is 32-bit and the other is 64-bit).

### Compliant Solution (Further Inheritance)

This compliant solution receives the inherited handle via the command line but prevents further inheritance by duplicating the handle. It also ensures that the value passed is a valid `HANDLE` value. Then it validates the handle as a proper file handle by calling `GetFileInformationByHandle()` . This solution also properly handles cross-architecture situations between the processes.

### Noncompliant Code Example ( `fopen()` )

By default, all files on Windows that are opened using `fopen()` will allow handle inheritance, and processes spawned via the `system()` API automatically inherit handles. In this noncompliant code example, `SomeProcess.exe` inherits the file handle for `SomeFile.txt` :

### Compliant Solution ( `fopen()` )

In this compliant solution, the Windows-specific `'N'` mode parameter is passed  to the call to `fopen()` , which ensures the file is opened without allowing handle inheritance:

## WIN04-C. Consider encrypting function pointers

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/14.microsoft-windows-win/6.win04-c.md) — SHA-256 `4b2b767699acc892ae6b39e4c8bc409140c5f929800a0daae51c34808093393d`。

If an attacker can overwrite memory containing function pointers, they may be able to execute arbitrary code. To mitigate the effects of such attacks, pointers to functions can be encrypted at runtime on the basis of some characteristics of the execution process so that only a running process will be able to decode them.  This is only required for stored function pointers stored to writable memory, including the stack.  The Microsoft SDL \[Microsoft 2012\] recommends encoding long-lived pointers in your code.

### Noncompliant Code Example

This noncompliant code example assigns the address of the `printf()` function to the `log_fn` function pointer, which can be allocated in the stack or data segment:

If a vulnerability exists in this program that allows an attacker to overwrite the `log_fn` function pointer, such as a buffer overflow or arbitrary memory write, the attacker may be able to overwrite the value of `printf` with the location of an arbitrary function.

### Compliant Solution (Windows)

Microsoft Windows provides the [`EncodePointer()`](https://msdn.microsoft.com/en-us/library/bb432254(VS.85).aspx) and [`DecodePointer()`](https://msdn.microsoft.com/en-us/library/bb432242(VS.85).aspx) functions that encrypt and decrypt pointers using a secret that is unique to the given process:

Note that `DecodePointer()` does not return success or failure.  If an attacker has overwritten the pointer contained in `log_fn` , the pointer returned will be invalid and cause your application to crash.  However, this is preferable to giving an attacker the ability to execute arbitrary code.

## WIN30-C. Properly pair allocation and deallocation functions

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/14.microsoft-windows-win/2.win30-c.md) — SHA-256 `6630f88456bd966735f903933a53197be23dd729e148af6012ea878e14c21a04`。

Windows provides several APIs for allocating memory.  While some of these functions have converged over time, it is still important to always properly pair allocations and deallocations.  The following table shows the proper pairings.

|                                              |                                      |
|----------------------------------------------|--------------------------------------|
| Allocator                                    | Deallocator                          |
| `malloc()`                        | `free()`                  |
| `realloc()`                       | `free()`                  |
| `LocalAlloc()`                    | `LocalFree()`             |
| `LocalReAlloc()`                  | `LocalFree()`             |
| `GlobalAlloc()`                   | `GlobalFree()`            |
| `GlobalReAlloc()`                 | `GlobalFree()`            |
| `VirtualAlloc()`                  | `VirtualFree()`           |
| `VirtualAllocEx()`                | `VirtualFreeEx()`         |
| `VirtualAllocExNuma()`            | `VirtualFreeEx()`         |
| `AllocateUserPhysicalPages()`     | `FreeUserPhysicalPages()` |
| `AllocateUserPhysicalPagesNuma()` | `FreeUserPhysicalPages()` |
| `HeapAlloc()`                     | `HeapFree()`              |
| `HeapReAlloc()`                   | `HeapFree()`              |

### Noncompliant Code Example

In this example, the `FormatMessage()` function allocates a buffer and stores it in the `buf` parameter.  From the documentation of `FORMAT_MESSAGE_ALLOCATE_BUFFER` \[ [MSDN](https://msdn.microsoft.com/en-us/library/ms679351(v=vs.85).aspx) \]:

> The function allocates a buffer large enough to hold the formatted message, and places a pointer to the allocated buffer at the address specified by lpBuffer.  The lpBuffer parameter is a pointer to anL PTSTR; you must cast the pointer to an LPTSTR (for example, (LPTSTR)&lpBuffer). The nSize parameter specifies the minimum number of TCHARs to allocate for an output message buffer. The caller should use the LocalFree function to free the buffer when it is no longer needed.

Instead of freeing the memory using `LocalFree()` , this code example uses `GlobalFree()` erroneously.

### Compliant Solution

The compliant solution uses the proper deallocation function as described by the documentation.
