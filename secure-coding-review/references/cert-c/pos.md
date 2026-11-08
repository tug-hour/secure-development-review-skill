# CERT C — POS

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## POS01-C. Check for the existence of links when dealing with files

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/16.posix-pos/2.pos01-c.md) — SHA-256 `0dcde47bba99a4477cf5426611ec8f47e8b29e2c379c83aa30f7cd9fc0dceffc`。

Many common operating systems such as Windows and UNIX support file links, including hard links, symbolic (soft) links, and virtual drives. Hard links can be created in UNIX with the `ln` command or in Windows operating systems by calling the [`CreateHardLink()`](https://msdn.microsoft.com/en-us/library/aa363860(VS.85).aspx) function. Symbolic links can be created in UNIX using the `   ln -s  ` command or in Windows by using directory junctions in NTFS or the Linkd.exe (Win 2K resource kit) or "junction" freeware. Virtual drives can also be created in Windows using the `subst` command.

File links can create security issues for programs that fail to consider the possibility that the file being opened may actually be a link to a different file. This is especially dangerous when the vulnerable program is running with elevated privileges.

Frequently, there is no need to check for the existence of symbolic links because this problem can be solved using other techniques. When opening an existing file, for example, the simplest solution is often to drop privileges to the privileges of the user. This solution permits the use of links while preventing access to files for which the user of the application is not privileged.

When creating new files, it may be possible to use functions that create a new file only where a file does not already exist. This prevents the application from overwriting an existing file during file creation. (See [FIO03-C. Do not make assumptions about fopen() and file creation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio03-c) .)

In rare cases, it is necessary to check for the existence of symbolic or hard links to ensure that a program is reading from an intended file and not a different file in another directory. In these cases, avoid creating a race condition when checking for the existence of symbolic links. (See [POS35-C. Avoid race conditions while checking for the existence of a symbolic link](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/posix-pos/pos35-c) .)

### Noncompliant Code Example

This noncompliant code example opens the file specified by the string `file_name` for read/write access and then writes user-supplied data to the file:

If the process is running with elevated privileges, an attacker can exploit this code, for example, by replacing the file with a symbolic link to the `/etc/passwd` authentication file. The attacker can then overwrite data stored in the password file to create a new root account with no password. As a result, this attack can be used to gain root privileges on a vulnerable system.

### Compliant Solution (Linux 2.1.126+, FreeBSD, Solaris 10, POSIX.1-2008 `O_NOFOLLOW` )

Some systems provide the `O_NOFOLLOW` flag to help mitigate this problem. The flag is required by the POSIX.1-2008 standard and so will become more portable over time \[ [Open Group 2008](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography) \]. If the flag is set and the supplied `file_name` is a symbolic link, then the open will fail.

Note that this compliant solution does not check for hard links.

### Compliant Solution ( `lstat` - `open` - `fstat` )

This compliant solution uses the `lstat` - `open` - `fstat` idiom illustrated in [FIO05-C. Identify files using multiple file attributes](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio05-c) :

This code is still subject to a time-of-check, time-of-use (TOCTOU) race condition, but before doing any operation on the file, it verifies that the file opened is the same file as was previously checked (by checking the file's device and i-node.) As a result, the code will recognize if an attacker has tampered with the file during the race window and can operate accordingly.

Note that this code does not check for hard links.

### Hard Links

Hard links are problematic because if a file has multiple hard links, it is impossible to distinguish the original link from one that might have been created by a malicious attacker.

One way to deal with hard links is simply to disallow opening of any file with two or more hard links. The following code snippet, when inserted into the previous example, will identify if a file has multiple hard links:

Because a hard link may not be created if the link and the linked-to file are on different devices, many platforms place system-critical files on a different device from the one where user-editable files are kept. For instance, the `/` directory, which contains critical system files like `/etc/passwd` , would live on one hard drive, while the `/home` directory, which contains user-editable files, would reside on a separate hard drive. This prevents users, for example, from creating hard links to `/etc/passwd` .

## POS02-C. Follow the principle of least privilege

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/16.posix-pos/3.pos02-c.md) — SHA-256 `48bd3659ef58ef83c0d7ee169072419c7d077b9c138bc3d8d9fabc3bb28819cc`。

The principle of least privilege states that every program and every user of the system should operate using the least set of privileges necessary to complete the job \[ [Saltzer 1974](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Saltzer74) , [Saltzer 1975](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Saltzer75) \]. The Build Security In website \[ [DHS 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-DHS06) \] provides additional definitions of this principle. Executing with minimal privileges mitigates against exploitation in case a vulnerability is discovered in the code.

### Noncompliant Code Example

Privileged operations are often required in a program, though the program might not need to retain the special privileges. For instance, a network program may require superuser privileges to capture raw network packets but may not require the same set of privileges for carrying out other tasks such as packet analysis. Dropping or elevating privileges alternately according to program requirements is a good design strategy. Moreover, assigning only the required privileges limits the window of exposure for any privilege escalation exploit to succeed.

Consider a custom service that must bind to a well-known port (below 1024). To prevent malicious entities from hijacking client connections, the kernel imposes a condition so that only the superuser can use the `bind()` system call to bind to these ports.

This noncompliant code example is configured as setuid-superuser. It calls `bind()` and later forks out a child to perform the bookkeeping tasks. The program continues to run with superuser privileges even after the `bind()` operation is completed.

If a vulnerability is exploited in the main body of the program that allows an attacker to execute arbitrary code, this malicious code will run with elevated privileges.

### Compliant Solution

The program must follow the principle of least privilege while carefully separating the binding and bookkeeping tasks. To minimize the chance of a flaw in the program from compromising the superuser-level account, it should drop superuser privileges as soon as the privileged operations are completed. In the following code, privileges are permanently dropped as soon as the `bind()` operation is carried out. The code also ensures privileges may not be regained after being permanently dropped, as in [POS37-C. Ensure that privilege relinquishment is successful](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/posix-pos/pos37-c) .

## POS04-C. Avoid using PTHREAD_MUTEX_NORMAL type mutex locks

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/16.posix-pos/4.pos04-c.md) — SHA-256 `13bdecd5859f942e063bca6b23cfec3fd898f818848a0a1ad85fc968390fccce`。

Pthread mutual exclusion (mutex) locks are used to avoid simultaneous usage of common resources. Several types of mutex locks are defined by pthreads: `NORMAL` , `ERRORCHECK` , `RECURSIVE` , and `DEFAULT` .

POSIX describes `PTHREAD_MUTEX_NORMAL` locks as having the following undefined behavior \[ [Open Group 2004](https://www.securecoding.cert.org/confluence/display/seccode/AA.+C+References#AA.CReferences-OpenGroup04) \]:

> This type of mutex does not provide deadlock detection. A thread attempting to relock this mutex without first unlocking it shall deadlock. An error is not returned to the caller. Attempting to unlock a mutex locked by a different thread results in undefined behavior. Attempting to unlock an unlocked mutex results in undefined behavior.

The `DEFAULT` mutex pthread is also generally mapped to `PTHREAD_MUTEX_NORMAL` but is known to vary from platform to platform \[ [SOL 2010](http://docs.sun.com/app/docs/doc/816-5137/sync-28983?a=view) \]. Consequently, `NORMAL` locks should not be used, and `ERRORCHECK` or `RECURSIVE` locks should be defined explicitly when mutex locks are used.

### Noncompliant Code Example

This noncompliant code example shows a simple mutex being created using `PTHREAD_MUTEX_NORMAL` . Note that the caller does not expect a return code when `NORMAL` mutex locks are used.

### Compliant Solution

This compliant solution shows an `ERRORCHECK` mutex lock being created so that return codes will be available during locking and unlocking:

## POS05-C. Limit access to files by creating a jail

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/16.posix-pos/5.pos05-c.md) — SHA-256 `dcb0fa97e3b2af879ecde2e94fdf0dc87fe0a0c67fc378c75834f2294588e8fd`。

Creating a jail isolates a program from the rest of the file system. The idea is to create a sandbox, so entities the program does not need to access under normal operation are made inaccessible. This makes it much harder to abuse any vulnerability that can otherwise lead to unconstrained system compromise and consequently functions as a defense-in-depth strategy. A jail may consist of world-viewable programs that require fewer resources to execute than those that exist on that system. Jails are useful only when there is no way to elevate privileges in the event of program failure.

Additionally, care must be taken to ensure that all the required resources (libraries, files, and so on) are replicated within the jail directory and that no reference is made to other parts of the file system from within this directory. It is also advisable to administer restrictive read/write permissions on the jail directories and resources on the basis of the program's privilege requirements. Although creating jails is an effective security measure when used correctly, it is not a surrogate for compliance with the other rules and recommendations in this standard.

### Noncompliant Code Example

A security flaw exists in this noncompliant code example resulting from the absence of proper canonicalization measures on the file path. This allows an attacker to traverse the file system and possibly write to a file of the attacker's choice with the privileges of the vulnerable program. For example, it may be possible to overwrite the password file (such as the `/etc/passwd` , common to many POSIX-based systems) or a device file, such as the mouse, which in turn can aid further exploitation or cause a denial of service to occur.

An attacker can control the value of `argv[1]` and consequently access any resource on the file system.

This noncompliant code example also violates [FIO02-C. Canonicalize path names originating from tainted sources](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio02-c) and [FIO03-C. Do not make assumptions about fopen() and file creation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio03-c) .

### Compliant Solution (UNIX)

Some UNIX-based systems (such as OpenBSD) can restrict file system access by creating a `chroot()` jail. The `chroot()` jail requires care to implement securely \[ [Wheeler 2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Wheeler03) \]. It is achieved by passing a predefined directory name as an argument to `chroot()` . The call to `chroot()` requires superuser privileges. However, this call does not *leave* the process inside the jail directory as might be expected. A subsequent `chdir()` is required to restrict access to the jail boundaries.

Another essential step is to drop superuser privileges permanently after these calls (see [POS02-C. Follow the principle of least privilege](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/posix-pos/pos02-c) ). The `chroot()` system call is not secure against the superuser changing the current root directory (if privileges are not dropped). Successful jail creation prevents unintentional file system access even if an attacker gives malicious input, such as through command-line arguments.

An alternative sequence is to first call `chdir("chroot/jail")` and then call `chroot(".")` . However, calling `chdir("/some/path")` , then `chroot("/some/path")` , should be avoided because this sequence may be susceptible to a race condition: an attacker with sufficient privileges can arrange for `/some/path` to refer to different directories in the two system calls. Consequently, the program will not have its current working directory set to the new root directory. Using either `chdir("/")` after `chroot()` or `chroot(".")` after `chdir()` guarantees that the current working directory will be the same directory as the new root.

## POS30-C. Use the readlink() function properly

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/02.pos30-c.md) — SHA-256 `aae26c03d36317e295d67b961ca37d5217089203f2e1966a919d3bcca4d50b56`。

The `readlink()` function reads where a link points to. It makes **no** effort to null-terminate its second argument, `buffer` . Instead, it just returns the number of characters it has written.

### Noncompliant Code Example

If `len` is equal to `sizeof(buf)` , the null terminator is written 1 byte past the end of `buf` :

An incorrect solution to this problem is to try to make `buf` large enough that it can always hold the result:

This modification incorrectly assumes that the symbolic link cannot be longer than the value of `SYMLINK_MAX` returned by `pathconf()` . However, the value returned by `pathconf()` is out of date by the time `readlink()` is called, so the off-by-one buffer-overflow risk is still present because, between the two calls, the location of `/usr/bin/perl` can change to a file system with a larger `SYMLINK_MAX` value. Also, if `SYMLINK_MAX` is indeterminate (that is, if `pathconf()` returned `-1` without setting `errno` ), the code uses an arbitrary large buffer size (10,000) that it hopes will be sufficient, but there is a small chance that `readlink()` can return exactly this size.

An additional issue is that `readlink()` can return `-1` if it fails, causing an off-by-one underflow.

### Compliant Solution

This compliant solution ensures there is no overflow by reading in only `sizeof(buf)-1` characters. It also properly checks to see if an error has occurred:

## POS34-C. Do not call putenv() with a pointer to an automatic variable as the argument

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/03.pos34-c.md) — SHA-256 `46037d995fb86dc1176e98348bddf20f5df1a69c005f2db7884416462e53c5cc`。

The POSIX function `putenv()` is used to set environment variable values. The `putenv()` function does not create a copy of the string supplied to it as an argument; rather, it inserts a pointer to the string into the environment array. If a pointer to a buffer of automatic storage duration is supplied as an argument to `putenv()` , the memory allocated for that buffer may be overwritten when the containing function returns and stack memory is recycled. This behavior is noted in the Open Group Base Specifications, Issue 6 \[ [Open Group 2004](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-OpenGroup04) \]:

> A potential error is to call `putenv()` with an automatic variable as the argument, then return from the calling function while `string` is still part of the environment.

The actual problem occurs when passing a *pointer* to an automatic variable to `putenv()` . An automatic pointer to a static buffer would work as intended.

### Noncompliant Code Example

In this noncompliant code example, a pointer to a buffer of automatic storage duration is used as an argument to `putenv()` \[ [Dowd 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Dowd06) \]. The `TEST` environment variable may take on an unintended value if it is accessed after `func()` has returned and the stack frame containing `env` has been recycled.

Note that this example also violates [DCL30-C. Declare objects with appropriate storage durations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl30-c) .

### Compliant Solution ( `static` )

This compliant solution uses a static array for the argument to `putenv()` .

According to the \[ [Open Group 2004](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-OpenGroup04) \] entry for `putenv()` :

> ...the string pointed to by *string* shall become part of the environment, so altering the string shall change the environment.

This means that the call to `putenv()` is only necessary the first time `func()` is called, since subsequent changes to the string update the environment. If `func()` were called more than once, an additional variable could be added to avoid calling it unnecessarily.

### Compliant Solution (Heap Memory)

This compliant solution dynamically allocates memory for the argument to `putenv()` :

The POSIX `setenv()` function is preferred over this function \[ [Open Group 2004](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-OpenGroup04) \]. In particular, using `putenv()` will necessarily leak memory if called multiple times for the same environment variable, due to restrictions on when you can safely free the old value. According to the \[ [Open Group 2004](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-OpenGroup04) \] entry for `putenv()` :

> Although the space used by *string* is no longer used once a new string which defines *name* is passed to *putenv* (), if any thread in the application has used [*getenv* ()](https://pubs.opengroup.org/onlinepubs/9699919799/functions/getenv.html) to retrieve a pointer to this variable, it should not be freed by calling [*free* ()](https://pubs.opengroup.org/onlinepubs/9699919799/functions/free.html) . If the changed environment variable is one known by the system (such as the locale environment variables) the application should never free the buffer used by earlier calls to *putenv* () for the same variable.

### Compliant Solution ( `setenv()` )

The `setenv()` function allocates heap memory for environment variables, which eliminates the possibility of accessing volatile stack memory:

Using `setenv()` is easier and consequently less error prone than using `putenv()` .

## POS35-C. Avoid race conditions while checking for the existence of a symbolic link

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/04.pos35-c.md) — SHA-256 `e1eaf78aebc278b1b3b93308d1650175259991361c41fec60d3444b31e053cd8`。

Many common operating systems, such as Windows and UNIX, support symbolic (soft) links. Symbolic links can be created in UNIX using the `   ln -s  ` command or in Windows by using directory junctions in NTFS or the Linkd.exe (Win 2K resource kit) or "junction" freeware.

If not properly performed, checking for the existence of symbolic links can lead to race conditions.

This rule is a specific instance of rule [FIO45-C. Avoid TOCTOU race conditions while accessing files](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/input-output-fio/fio45-c) .

### Noncompliant Code Example

The POSIX `lstat()` function collects information about a symbolic link rather than its target. This noncompliant code example uses the `lstat()` function to collect information about the file, checks the `st_mode` field to determine if the file is a symbolic link, and then opens the file if it is not a symbolic link:

This code contains a time-of-check, time-of-use (TOCTOU) race condition between the call to `lstat()` and the subsequent call to `open()` because both functions operate on a file name that can be manipulated asynchronously to the execution of the program. (See [FIO01-C. Be careful using functions that use file names for identification](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio01-c) .)

### Compliant Solution (POSIX.1-2008 or newer)

This compliant solution eliminates the race condition by using `O_NOFOLLOW` to cause `open()` to fail if passed a symbolic link, avoiding the TOCTOU by not having a separate "check" and "use":

### Compliant Solution (POSIX.1-2001 or older)

This compliant solution detects a file substituted during the race window by

1.  Calling `lstat()` on the file name.
2.  Calling `open()` to open the file.
3.  Calling `fstat()` on the file descriptor returned by `open()` .
4.  Comparing the file information returned by the calls to `lstat()` and `fstat()` to ensure that the files are the same.

This code is still subject to a TOCTOU race condition between `lstat()` and `open()` , but it detects a substituted file before writing to it. Because `fstat()` is applied to file descriptors, not file names, the file passed to `fstat()` must be identical to the file that was opened. The `lstat()` function does not follow symbolic links, but `open()` does. Comparing modes using the `st_mode` field is sufficient to check for a symbolic link.

Comparing i-nodes, using the `st_ino` fields, and devices, using the `st_dev` fields, ensures that the file passed to `lstat()` is the same as the file passed to `fstat()` . (See [FIO05-C. Identify files using multiple file attributes](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio05-c) .)

## POS36-C. Observe correct revocation order while relinquishing privileges

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/05.pos36-c.md) — SHA-256 `7af4a6751f12f7a6defc0ec9bd464ec630225725d756607abf92465a278ed12e`。

In case of set-user-ID and set-group-ID programs, when the effective user ID and group ID are different from those of the real user, it is important to drop not only the user-level privileges but also the group privileges. While doing so, the order of revocation must be correct.

POSIX defines `setgid()` to have the following behavior \[ [Open Group 2004](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-OpenGroup04) \]:

> If the process has appropriate privileges, `setgid()` shall set the real group ID, effective group ID, and the saved set-group-ID of the calling process to `gid` .
>
> If the process does not have appropriate privileges, but `gid` is equal to the real group ID or the saved set-group-ID, `setgid()` shall set the effective group ID to `gid` ; the real group ID and saved set-group-ID shall remain unchanged.

### Noncompliant Code Example

This noncompliant code example drops privileges to those of the real user and similarly drops the group privileges. However, the order is incorrect because the `setgid()` function must be run with superuser privileges, but the call to `setuid()` leaves the effective user ID as nonzero. As a result, if a vulnerability is discovered in the program that allows for the execution of arbitrary code, an attacker can regain the original group privileges.

### Compliant Solution

This compliant solution relinquishes group privileges before taking away the user-level privileges so that both operations execute as intended.

### Supplementary Group IDs

A process may have a number of supplementary group IDs in addition to its effective group ID, and the supplementary groups can allow privileged access to files. The `getgroups()` function returns an array that contains the supplementary group IDs and may also contain the effective group ID. The `setgroups()` function can set the supplementary group IDs and may also set the effective group ID on some systems. Using `setgroups()` usually requires privileges. Although POSIX defines the `getgroups()` function, it does not define `setgroups()` .

Under normal circumstances, `setuid()` and related calls do not alter the supplementary group IDs. However, a setuid-root program can alter its supplementary group IDs and then relinquish root privileges, in which case it maintains the supplementary group IDs but lacks the privilege necessary to relinquish them. Consequently, it is recommended that a program relinquish supplementary group IDs immediately before relinquishing root privileges. The following code defines a `set_sups()` function that will set the supplementary group IDs to a specific array on systems that support the `setgroups()` function.

## POS37-C. Ensure that privilege relinquishment is successful

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/06.pos37-c.md) — SHA-256 `67f088336cccdec85e8404081f6e16fd1add5befc78baaec779aa1e1bea64d2e`。

The POSIX `setuid()` function has complex semantics and platform-specific behavior \[ [Open Group 2004](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-OpenGroup04) \].

> If the process has appropriate privileges, `setuid()` shall set the real user ID, effective user ID, and the saved set-user-ID of the calling process to `uid` .
>
> If the process does not have appropriate privileges, but `uid` is equal to the real user ID or the saved set-user-ID, `setuid()` shall set the effective user ID to `uid` ; the real user ID and saved set-user-ID shall remain unchanged.

The meaning of "appropriate privileges" varies from platform to platform. For example, on Solaris, appropriate privileges for `setuid()` means that the `PRIV_PROC_SETID` privilege is in the effective privilege set of the process. On BSD, it means that the effective user ID (EUID) is zero (that is, the process is running as root) or that `uid=geteuid()` . On Linux, it means that the process has `CAP_SETUID` capability and that `setuid(geteuid())` will fail if the EUID is not equal to 0, the real user ID (RUID), or the saved set-user ID (SSUID).

Because of this complex behavior, desired privilege drops sometimes may fail. For example, the range of Linux Kernel versions (2.2.0–2.2.15) is vulnerable to an insufficient privilege attack wherein `setuid(getuid()` did not drop privileges as expected when the capability bits were set to zero. As a precautionary measure, subtle behavior and error conditions for the targeted implementation must be carefully noted.

### Noncompliant Code Example

This noncompliant code example compiles cleanly on most POSIX systems, but no explicit checks are made to ensure that privilege relinquishment has succeeded. This may be dangerous depending on the sequence of the preceding privilege changes.

If the program is run as a setuid root program, over time, the state of the UIDs might look like the following:

| Description | Code | EUID | RUID | SSUID |
| --- | --- | --- | --- | --- |
| Program startup |  | 0 | User | 0 |
| Temporary drop | ` seteuid(getuid()) ` | User | User | 0 |
| Restore | ` seteuid(0) ` | 0 | User | 0 |
| Permanent drop | ` setuid(getuid()) ` | User | User | User |
| Restore (attacker) | ` setuid(0) ` (fails) | User | User | User |

If the program fails to restore privileges, it will be unable to permanently drop them later:

| Description | Code | EUID | RUID | SSUID |
| --- | --- | --- | --- | --- |
| program startup |  | 0 | User | 0 |
| Temporary drop | ` seteuid(getuid()) ` | User | User | 0 |
| Restore | ` seteuid(0) ` | User | User | 0 |
| Permanent drop | ` setuid(getuid()) ` | User | User | 0 |
| Restore (attacker) | ` setuid(0) ` | 0 | 0 | 0 |

### Compliant Solution

This compliant solution was implemented in sendmail, a popular mail transfer agent, to determine if superuser privileges were successfully dropped \[ [Wheeler 2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Wheeler03) \]. If the `setuid()` call succeeds after (supposedly) dropping privileges permanently, then the privileges were not dropped as intended.

### Compliant Solution

A better solution is to ensure that proper privileges exist before attempting to perform a permanent drop:

### Supplementary Group IDs

A process may have a number of supplementary group IDs, in addition to its effective group ID, and the supplementary groups can allow privileged access to files. The `getgroups()` function returns an array that contains the supplementary group IDs and can also contain the effective group ID. The `setgroups()` function can set the supplementary group IDs and can also set the effective group ID on some systems. Using `setgroups()` usually requires privileges. Although POSIX defines the `getgroups()` function, it does not define `setgroups()` .

Under normal circumstances, `setuid()` and related calls do not alter the supplementary group IDs. However, a setuid-root program can alter its supplementary group IDs and then relinquish root privileges, in which case, it maintains the supplementary group IDs but lacks the privilege necessary to relinquish them. Consequently, it is recommended that a program immediately relinquish supplementary group IDs before relinquishing root privileges.

[POS36-C. Observe correct revocation order while relinquishing privileges](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/posix-pos/pos36-c) discusses how to drop supplementary group IDs. To ensure that supplementary group IDs are indeed relinquished, you can use the following `eql_sups` function:

### System-Specific Capabilities

Many systems have nonportable privilege capabilities that, if unchecked, can yield privilege escalation vulnerabilities. The following section describes one such capability.

#### File System Access Privileges (Linux)

Processes on Linux have two additional values called `fsuid` and `fsgid` . These values indicate the privileges used when accessing files on the file system. They normally shadow the effective user ID and effective group ID, but the `setfsuid()` and `setfsgid()` functions allow them to be changed. Because changes to the `euid` and `egid` normally also apply to `fsuid` and `fsgid` , a program relinquishing root privileges need not be concerned with setting `fsuid` or `fsgid` to safe values. However, there has been at least one kernel bug that violated this invariant (\[ [Chen 2002](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Chen02) \] and \[ [Tsafrir 2008](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Tsafrir08) \]). Consequently, a prudent program checks that `fsuid` and `fsgid` have harmless values after relinquishing privileges.

## POS38-C. Beware of race conditions when using fork and file descriptors

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/07.pos38-c.md) — SHA-256 `42935a74b572b97fc577ca7da25f26574492b24cc276a29721e108e713e7dcc6`。

When forking a child process, file descriptors are copied to the child process, which can result in concurrent operations on the file. Concurrent operations on the same file can cause data to be read or written in a nondeterministic order, creating race conditions and unpredictable behavior.

### Noncompliant Code Example

In this example, the programmer wishes to open a file, read a character, fork, and then have both parent and child process read the second character of the file independently. However, because both processes share a file descriptor, one process might get the second character, and one might get the third. Furthermore, there is no guarantee the reads are atomic—the processes might get unpredictable results. Regardless of what the programmer is trying to accomplish with this code, this code is incorrect because it contains a race condition.

If the file accessed has contents `"abc"` , the output of this program could be either

or

This code's output cannot reliably be determined and should not be used.

### Compliant Solution

In this compliant solution, the programmer closes the file descriptor in the child after forking and then reopens it, ensuring that the file has not been modified in the meantime. See [POS01-C. Check for the existence of links when dealing with files](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/posix-pos/pos01-c) for details.

The output of this code is

## POS39-C. Use the correct byte ordering when transferring data between systems

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/08.pos39-c.md) — SHA-256 `af8ba880762d75a07ff626ba9d765c36456613f4fe6a654d928f16bbf52f3cc8`。

Different system architectures use different byte ordering, either little endian (least significant byte first) or big endian (most significant byte first). IA-32 is an example of an architecture that implements little endian byte ordering. In contrast, PowerPC and most Network Protocols (including TCP and IP) use big endian.

When transferring data between systems of different endianness, the programmer must take care to reverse the byte ordering before interpreting the data.

The functions `htonl()` , `htons()` , `ntohl()` , and `ntohs()` can be used to transfer between network byte ordering (big endian) and the host's byte ordering. On big endian systems, these functions do nothing. They may also be implemented as macros rather than functions.

### Noncompliant Code Example

In this noncompliant code example, the programmer tries to read an unsigned 32-bit integer off a previously connected network socket.

It is important to know the sizes of your data types lest they be different on architectures that are accessible over the network. Hence, we transfer a `uint32_t` rather than an `int` . For more information, see [FIO09-C. Be careful with binary data when transferring data across systems](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio09-c) .

This program prints out the number received from the socket using an incorrect byte ordering. For example, if the value 4 is sent from a big endian machine, and the receiving system is little endian, the value 67,108,864 is read. This problem can be corrected by sending and receiving using network byte ordering.

### Compliant Solution

In this compliant solution, the programmer uses the `ntohl()` function to convert the integer from network byte order to host byte ordering:

The `ntohl()` function (network to host long) translates a `uint32_t` value into the host byte ordering from the network byte ordering. This function is always appropriate to use because its implementation depends on the specific system's byte ordering. Consequently, on a big endian architecture, `ntohl()` does nothing.

The reciprocal function `htonl()` (host to network long) should be used before sending any data to another system over network protocols.

##### Portability Details

-   `ntohs()` , `ntohl()` , `htons()` , and `htonl()` are not part of the C Standard and are consequently not guaranteed to be portable to non-POSIX systems.
-   The POSIX implementations of `ntohs()` , `ntohl()` , `htons()` , and `htonl()` take arguments of types `uint16_t` and `uint32_t` and can be found in the header file `<arpa/inet.h>` .
-   The Windows implementations use `    unsigned short   `and`    unsigned long   ` and can be found in the header file `<winsock2.h>` .
-   Other variants of `ntoht()` and `htont()` , such as `ntohi()` / `htoni()` or `ntohll()` / `htonll()` , may exist on some systems.

## POS44-C. Do not use signals to terminate threads

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/09.pos44-c.md) — SHA-256 `0c260c265b9cd4be04a78fbbaef0c1e1ac9ff00f0593a5d1d377f279e903662d`。

Do not send an uncaught signal to kill a thread because the signal kills the entire process, not just the individual thread. This rule is a specific instance of [SIG02-C. Avoid using signals to implement normal functionality](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/signals-sig/sig02-c) .

In POSIX systems, using the `signal()` function in a multithreaded program falls under exception **CON37C-C-EX0** of rule [CON37-C. Do not call signal() in a multithreaded program](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/concurrency-con/con37-c) .

### Noncompliant Code Example

This code uses the `pthread_kill()` function to send a `SIGTERM` signal to the created thread. The thread receives the signal, and the entire process is terminated.

### Compliant Solution

This compliant code uses instead the `pthread_cancel()` function to terminate the thread. The thread continues to run until it reaches a cancellation point. See [*The Open Group Base Specifications Issue 6, IEEE Std 1003.1, 2004 Edition*](http://www.opengroup.org/onlinepubs/009695399/toc.htm) \[ [Open Group 2004](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-OpenGroup04) \] for lists of functions that are required and allowed to be cancellation points. If the cancellation type is set to asynchronous, the thread is terminated immediately. However, POSIX requires only the `pthread_cancel()` , `pthread_setcancelstate()` , and `pthread_setcanceltype()` functions to be async-cancel safe. An application that calls other POSIX functions with asynchronous cancellation enabled is nonconforming. Consequently, we recommend disallowing asynchronous cancellation, as explained by [POS47-C. Do not use threads that can be canceled asynchronously](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/posix-pos/pos47-c) .

## POS47-C. Do not use threads that can be canceled asynchronously

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/10.pos47-c.md) — SHA-256 `fcf1163ec44b6952b22aeb4941503273b6e07c90fef06acf384e11b9762ca24f`。

In threading, pthreads can optionally be set to cancel immediately or defer until a specific cancellation point. Canceling asynchronously (immediately) is dangerous, however, because most threads are in fact not safe to cancel immediately.

The [IEEE standards page](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-OpenGroup04) states that

> only functions that are cancel-safe may be called from a thread that is asynchronously cancelable.

Canceling asynchronously would follow the same route as passing a signal into the thread to kill it, posing problems similar to those in [CON37-C. Do not call signal() in a multithreaded program](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/concurrency-con/con37-c) , which is strongly related to [SIG02-C. Avoid using signals to implement normal functionality](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/signals-sig/sig02-c) . POS44-C and SIG02-C expand on the dangers of canceling a thread suddenly, which can create a [data race condition](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-datarace) .

### Noncompliant Code Example

In this noncompliant code example, the worker thread is doing something as simple as swapping `a` and `b` repeatedly.

This code uses one lock. The `global_lock` mutex ensures that the worker thread and main thread do not collide in accessing the `a` and `b` variables.

The worker thread repeatedly exchanges the values of `a` and `b` until it is canceled by the main thread. The main thread then prints out the current values of `a` and `b` . Ideally, one should be 5, and the other should be 10.

However, this program is subject to a race condition because an asynchronous cancel can happen at any time. If the worker thread is canceled while the `global_lock` mutex is held, it is never actually released. In this case, the main thread will wait forever trying to acquire the `global_lock` , and the program will deadlock.
It is also possible that the main thread cancels the worker thread before it has invoked `pthread_setcanceltype()` . If this happens, the cancellation will be delayed until the worker thread calls `pthread_setcanceltype()` .

### Noncompliant Code Example

In this example, the worker thread arranges to release the `global_lock` mutex if it gets interrupted:

The global variables are still subject to a race condition because an asynchronous cancel can happen at any time. For instance, the worker thread could be canceled just before the last line ( `   a = c  ` ) and thereby lose the old value of `b` . Consequently, the main thread might print that `a` and `b` have the same value.

The program is still subject to the race condition where the main thread cancels the worker thread before it has invoked `pthread_setcanceltype()` . If this happens, the cancelation will be delayed until the worker thread calls `pthread_setcanceltype()` .

Furthermore, though less likely, the program can still deadlock if the worker thread gets canceled after the `global_lock` is acquired but before `pthread_cleanup_push()` is invoked. In this case, the worker thread is canceled while holding `global_lock` , and the program will deadlock.

### Compliant Solution

From [IEEE standards page](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-OpenGroup04) :

> The cancelability state and type of any newly created threads, including the thread in which main() was first invoked, shall be PTHREAD_CANCEL_ENABLE and PTHREAD_CANCEL_DEFERRED respectively.

Because the default condition for POSIX, according to the IEEE standards, is `PTHREAD_CANCEL_DEFERRED` , it is not necessary to invoke `pthread_setcanceltype()` in the compliant solution:

Because this code limits cancellation of the worker thread to the end of the `while` loop, the worker thread can preserve the data invariant that `   a != b  ` . Consequently, the program might print that `a` is 5 and `b` is 10 or that `a` is 10 and `b` is 5, but they will always be revealed to have different values when the worker thread is canceled.

The other race conditions that plague the noncompliant code examples are not possible here. Because the worker thread does not modify its cancel type, it cannot be canceled before being improperly initialized. And because it cannot be canceled while the `global_lock` mutex is held, there is no possibility of deadlock, and the worker thread does not need to register any cleanup handlers.

## POS48-C. Do not unlock or destroy another POSIX thread's mutex

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/11.pos48-c.md) — SHA-256 `5c146543682c1278187d2355ec8a81d12f982e29d6573d290380c73fbc75d40f`。

Mutexes are used to protect shared data structures being accessed concurrently. The thread that locks the mutex owns it, and the owning thread should be the only thread to unlock the mutex. If the mutex is destroyed while still in use, critical sections and shared data are no longer protected. This rule is a specific instance of [CON31-C. Do not unlock or destroy another thread's mutex](https://www.securecoding.cert.org/confluence/display/seccode/CON31-C.+Do+not+unlock+or+destroy+another+thread%27s+mutex) using POSIX threads.

### Noncompliant Code Example

In this noncompliant code example, a race condition exists between a cleanup and a worker thread. The cleanup thread destroys the lock, which it believes is no longer in use. If there is a heavy load on the system, the worker thread that held the lock can take longer than expected. If the lock is destroyed before the worker thread has completed modifying the shared data, the program may exhibit unexpected behavior.

### Compliant Solution

This compliant solution requires that there is no chance a mutex will be needed after it has been destroyed. As always, it is important to check for error conditions when locking the mutex.

## POS49-C. When data must be accessed by multiple threads, provide a mutex and guarantee no adjacent data is also accessed

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/12.pos49-c.md) — SHA-256 `925e57880ff35ee9f8172c926b534f1fb50c68685027dcaefd9870c016a19eee`。

When multiple threads must access or make modifications to a common variable, they may also inadvertently access other variables adjacent in memory. This is an artifact of variables being stored compactly, with one byte possibly holding multiple variables, and is a common optimization on word-addressed machines. Bit-fields are especially prone to this behavior because compilers are allowed to store multiple bit-fields in one addressable byte or word. This implies that race conditions may exist not just on a variable accessed by multiple threads but also on other variables sharing the same byte or word address. This rule is a specific instance of [CON32-C. Prevent data races when accessing bit-fields from multiple threads](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/concurrency-con/con32-c) using POSIX threads.

A common tool for preventing race conditions in concurrent programming is the mutex. When properly observed by all threads, a mutex can provide safe and secure access to a common variable; however, it guarantees nothing with regard to other variables that might be accessed when a common variable is accessed.

Unfortunately, there is no portable way to determine which adjacent variables may be stored along with a certain variable.

A better approach is to embed a concurrently accessed variable inside a union, along with a `long` variable, or at least some padding to ensure that the concurrent variable is the only element to be accessed at that address. This technique would effectively guarantee that no other variables are accessed or modified when the concurrent variable is accessed or modified.

### Noncompliant Code Example (Bit-field)

In this noncompliant code example, two executing threads simultaneously access two separate members of a global `struct` :

Although this code appears to be harmless, it is likely that `flag1` and `flag2` are stored in the same byte. If both assignments occur on a thread-scheduling interleaving that ends with both stores occurring after one another, it is possible that only one of the flags will be set as intended, and the other flag will equal its previous value, because both bit-fields are represented by the same byte, which is the smallest unit the processor can work on.

For example, the following sequence of events can occur:

Even though each thread is modifying a separate bit-field, they are both modifying the same location in memory. This is the same problem discussed in [CON43-C. Do not allow data races in multithreaded code](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/concurrency-con/con43-c) but is harder to diagnose because it is not immediately obvious that the same memory location is being modified.

### Compliant Solution (Bit-field)

This compliant solution protects all accesses of the flags with a mutex, thereby preventing any thread-scheduling interleaving from occurring. In addition, the flags are declared `volatile` to ensure that the compiler will not attempt to move operations on them outside the mutex. Finally, the flags are embedded in a union alongside a `long` , and a static assertion guarantees that the flags do not occupy more space than the `long` . This technique prevents any data not checked by the mutex from being accessed or modified with the bit-fields.

Static assertions are discussed in detail in [DCL03-C. Use a static assertion to test the value of a constant expression](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl03-c) .

## POS50-C. Declare objects shared between POSIX threads with appropriate storage durations

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/13.pos50-c.md) — SHA-256 `4be9c056cb28643e12b21d35f51cc6605a1a26ae923108d429a576c2e88afcaf`。

Accessing the stack or thread-local variables of a thread from another thread can cause invalid memory accesses because the execution of threads can be interwoven within the constraints of the synchronization model. As a result, the referenced stack frame or thread-local variable may not be valid when the other thread tries to access it. Regular shared variables should be protected by thread synchronization mechanisms. However, local variables should not be shared in the same fashion because the referenced stack frame's thread would have to stop executing, or some other way must be found to ensure that the referenced stack frame is still valid. See [CON32-C. Prevent data races when accessing bit-fields from multiple threads](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/concurrency-con/con32-c) for information on how to securely share nonautomatic and non-thread-local variables. See [DCL30-C. Declare objects with appropriate storage durations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl30-c) for information on how to declare objects with appropriate storage durations when data is not being shared between threads. Note that this is a specific instance of [CON34-C. Declare objects shared between threads with appropriate storage durations](https://www.securecoding.cert.org/confluence/display/seccode/CON34-C.+Declare+objects+shared+between+threads+with+appropriate+storage+durations) for POSIX threads.

It is important to note that local data can be used securely with threads when using other non-POSIX thread interfaces, so the programmer should not always copy data into nonlocal memory when sharing data with threads. For example, the `shared` keyword in [OpenMP](https://openmp.org/wp/) can be used in combination with OpenMP's threading interface to share local memory without having to worry about whether local automatic variables remain valid. Furthermore, copying the shared data into dynamic memory may completely negate the performance benefits of multithreading.

### Noncompliant Code Example (Automatic Storage)

The `createThread()` function allocates an integer on the stack and passes a void pointer, spawning off a new thread, `childThread()` . The order of thread execution is interleaved, so `val` can reference an object outside of its lifetime, causing the child thread to access an invalid memory location.

#### Implementation Details

The automatic storage noncompliant code example incorrectly prints 0 when compiled with GCC 4.1.2 with the `-lpthread` flag on Linux.

### Noncompliant Code Example (Thread-Local Storage)

This noncompliant code example is incorrect because `createThread()` can finish running before `childThread()` , so `childThread()` may not access a valid object:

#### Implementation Details

This thread-local storage noncompliant code example incorrectly prints 0 when compiled with GCC 4.1.2 with the `-lpthread` flag on Linux.

### Compliant Solution (Allocated Storage)

Instead of passing the pointer to the stack of the parent thread, one solution is to allocate space on the heap, copying the data to dynamic memory. Because the lifetime of an object stored in dynamic memory lasts until it is freed, the child thread ensures that memory accesses are valid.

### Compliant Solution (Static Storage)

Another solution is to store the data as a global static variable. Unlike local automatic variables, which are stored on the stack, static variables are stored in the data section of memory. Static variables last for the execution of the entire program, so a thread can safely access global static variables.

### Compliant Solution (Automatic Storage)

Another solution is to ensure that local variables shared between threads are declared in the same or previous stack frame as a call to a thread synchronization mechanism, such as `pthread_join()` . For example, in this compliant solution, `val` is declared in `main()` , where `pthread_join()` is called. Because the parent thread will wait until the child thread completes before continuing its execution, the child thread is guaranteed to access an object that is still within its lifetime.

### Compliant Solution (Thread-Local Storage)

Another solution is to declare `val` a thread-local variable combined with a call to a thread synchronization mechanism, such as `pthread_join()` . Because the parent thread will wait until the child thread completes before continuing its execution, the child thread is guaranteed to access an object that is still within its lifetime.

## POS51-C. Avoid deadlock with POSIX threads by locking in predefined order

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/14.pos51-c.md) — SHA-256 `56e28b7401e2486f6e4a383f2ed4d21a6a037ecdd71630b0993459eb2dfe5b95`。

Mutexes are often used to prevent multiple threads from accessing critical resources at the same time. Sometimes, when locking mutexes, multiple threads hold each other's lock, and the program consequently deadlocks. There are four requirements for deadlock:

-   Mutual exclusion
-   Hold and wait
-   No preemption
-   Circular wait

Deadlock requires all four conditions, so to prevent deadlock, prevent any one of the four conditions. This guideline recommends locking the mutexes in a predefined order to prevent circular wait. This rule is a specific instance of [CON35-C. Avoid deadlock by locking in predefined order](https://www.securecoding.cert.org/confluence/display/seccode/CON35-C.+Avoid+deadlock+by+locking+in+predefined+order) using POSIX threads.

### Noncompliant Code Example

This noncompliant code example has behavior that depends on the runtime environment and the platform's scheduler. However, with proper timing, the `main()` function will deadlock when running `thr1` and `thr2` , where `thr1` tries to lock `ba2` 's mutex, while `thr2` tries to lock on `ba1` 's mutex in the `deposit()` function, and the program will not progress.

### Compliant Solution

The solution to the deadlock problem is to use a predefined order for the locks in the `deposit()` function. In this compliant solution, each thread will lock on the basis of the `bank_account` ID, defined in the struct initialization. This solution prevents the circular wait problem:

## POS52-C. Do not perform operations that can block while holding a POSIX lock

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/15.pos52-c.md) — SHA-256 `8be708d5ed89511f8acbdf805855f4a40e29431630cfe7a64a3cd46fe6482def`。

If a lock is being held and an operation that can block is performed, any other thread that needs to acquire that lock may also block. This condition can degrade the performance of a system or cause a deadlock to occur.

Blocking calls include, but are not limited to: network, file, and console I/O. This rule is a specific instance of [CON05-C. Do not perform operations that can block while holding a lock](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/concurrency-con/con05-c) using POSIX threads.

### Noncompliant Code Example

This noncompliant code example demonstrates an occurrence of a blocking call that waits to receive data on a socket while a mutex is locked. The `recv()` call blocks until data arrives on the socket. While it is blocked, other threads that are waiting for the lock are also blocked.

Although this example is specific to network I/O, the `recv()` call could be replaced with any blocking call, and the same behavior would occur.

### Compliant Solution (Block while Not Locked)

This compliant solution performs the `recv()` call when the lock has not been acquired. The blocking behavior consequently affects only the thread that called the blocking function.

### Compliant Solution (Use a Nonblocking Call)

This compliant solution performs the `recv()` call with the parameter `MSG_DONTWAIT` , which causes the call to fail if no messages are available on the socket:

### Exceptions

**POS52-C-EX1:** A thread may block while holding one or more locks and waiting to acquire another lock. When acquiring multiple locks, the order of locking must avoid deadlock, as specified in [CON35-C. Avoid deadlock by locking in a predefined order](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/concurrency-con/con35-c) .

## POS53-C. Do not use more than one mutex for concurrent waiting operations on a condition variable

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/16.pos53-c.md) — SHA-256 `a61b594c9e3db7fe970ba5ad20db091a9319dcc602a3fef7a95df6ec6895fc84`。

`pthread_cond_wait()` and `pthread_cond_timedwait()` take a condition variable and locked mutex as arguments. These functions unlock the mutex until the condition variable is signaled and then relock the mutex before returning. While a thread is waiting on a particular condition variable and mutex, other threads may only wait on the same condition variable if they also pass the same mutex as an argument. This requirement is noted in the *Open Group Base Specifications, Issue 6* :

> As long as at least one thread is blocked on the condition variable. During this time, the effect of an attempt by any thread to wait on that condition variable using a different mutex is undefined.

It also specifies that `pthread_cond_wait()` *may* €? fail if:

> \[EINVAL\]  
> The value specified by cond or mutex is invalid.  
> \[EPERM\]  
> The mutex was not owned by the current thread at the time of the call.

### Noncompliant Code Example

In this noncompliant code example, `mutex1` protects `count1` and `mutex2` protects `count2` . A [race condition](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-racecondi) exists between the `waiter1` and `waiter2` threads because they use the same condition variable with different mutexes. If both threads attempt to call `pthread_cond_wait()` at the same time, one thread will succeed and the other thread will invoke [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

#### Implementation Details: Linux

When the system is built on the following platform,

> Red Hat Enterprise Linux Client release 5.5 (Tikanga)  
> kernel 2.6.18  
> gcc 4.3.5 with the --D_GNU_SOURCE flag

the preceding code works as expected. `waiter1` and `waiter2` increment the variable once they are signaled, and the correct mutex is acquired after `pthread_cond_wait` returns in each thread.

The man page for `pthread_cond_wait` on this configuration says that it *may* €? fail with a return value of `EINVAL` if "€œdifferent mutexes were supplied for concurrent `pthread_cond_timedwait()` or `pthread_cond_wait()` operations on the same condition variable." However, this does not happen.

#### Implementation Details: OS X

When the system is built on the following platform,

> OS X 10.6.4 (Snow Leopard)  
> gcc 4.2.1

`pthread_cond_wait()` returns `EINVAL` if it is called when another thread is waiting on the condition variable with a different mutex. This approach is arguably better because it forces the coder to fix the problem instead of allowing reliance on undefined behavior.

The man page for `   pthread_cond_wait()}] simply says that {{EINVAL  ` will be returned if "€œthe value specified by cond or the value specified by mutex is invalid,"€? but it doesn'€™t say what *invalid* means.

### Compliant Solution

This problem can be solved either by always using the same mutex whenever a particular condition variable is used or by using separate condition variables, depending on how the code is expected to work. This compliant solution uses the €œsame-mutex€? solution:

## POS54-C. Detect and handle POSIX library errors

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/16.posix-pos/17.pos54-c.md) — SHA-256 `32b8af775c68bbb1852d150bfc9bcead49d89632f1fddef71bf46f4f492fd383`。

All standard library functions, including I/O functions and memory allocation functions, return either a valid value or a value of the correct return type that indicates an error (for example, −1 or a null pointer). Assuming that all calls to such functions will succeed and failing to check the return value for an indication of an error is a dangerous practice that may lead to [unexpected](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unexpectedbehavior) or [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) when an error occurs. It is essential that programs detect and appropriately handle all errors in accordance with an error-handling policy, as discussed in [ERR00-C. Adopt and implement a consistent and comprehensive error-handling policy](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err00-c) .  In addition to the C standard library functions mentioned in [ERR33-C. Detect and handle standard library errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err33-c) , the following functions defined in POSIX require error checking (list is not all-inclusive).

The successful completion or failure of each of the standard library functions listed in the following table shall be determined either by comparing the function’s return value with the value listed in the column labeled “Error Return” or by calling one of the library functions mentioned in the footnotes to the same column.

|                                 |                                         |                     |                       |
|---------------------------------|-----------------------------------------|---------------------|-----------------------|
| Function                        | Successful Return                       | Error Return        | `errno`  |
| `fmemopen()`       | Pointer to a `FILE` object | `NULL` | `ENOMEM` |
| `open_memstream()` | Pointer to a `FILE` object | `NULL` | `ENOMEM` |
| `posix_memalign()` | `0`                        | Nonzero             | Unchanged             |

Setting `errno` is a POSIX \[ [ISO/IEC 9945:2008](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9945-2008) \] extension to the C Standard. On error, `posix_memalign()` returns a value that corresponds to one of the constants defined in the `<errno.h>` header. The function does not set `errno` . The `posix_memalign()` function is optional and is not required to be provided by POSIX-conforming implementations.

### Noncompliant Code Example ( POSIX )

In this noncompliant code example, `fmemopen()` and `open_memstream()` are assumed to succeed. However, if the calls fail, the two file pointers `in` and `out` will be null and the program will have [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

### Compliant Solution (POSIX)

A compliant solution avoids assuming that `fmemopen()` and `open_memstream()` succeed regardless of its arguments and tests the return value of the function before using the file pointers `in` and `out` :

### Exceptions

****POS54-C-EX1:**** *This exception has been removed.*

**POS54-C-EX2:** The exception from [ERR33-C. Detect and handle standard library errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err33-c) (that is ERR33-C-EX1) applies to this rule. See that exception for more information.
