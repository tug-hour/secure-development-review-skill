# CERT C — FIO

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## FIO01-C. Be careful using functions that use file names for identification

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/02.fio01-c.md) — SHA-256 `d190225bd26a9a21234636918adddf178f789bdf2396a38b51feeed1e216dbdd`。

Many file-related security vulnerabilities result from a program accessing an unintended file object because file names are only loosely bound to underlying file objects. File names provide no information regarding the nature of the file object itself. Furthermore, the binding of a file name to a file object is reasserted every time the file name is used in an operation. File descriptors and `FILE` pointers are bound to underlying file objects by the operating system. (See [FIO03-C. Do not make assumptions about fopen() and file creation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio03-c) .)

Accessing files via file descriptors or `FILE` pointers rather than file names provides a greater degree of certainty as to which object is actually acted upon. It is recommended that files be accessed through file descriptors or `FILE` pointers where possible.

The following C functions rely solely on file names for file identification:

-   `remove()`
-   `rename()`
-   `fopen()`
-   `freopen()`

Use these functions with caution. See [FIO10-C. Take care when using the rename() function](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio10-c) , and [FIO08-C. Take care when calling remove() on an open file](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio08-c) .

### Noncompliant Code Example

In this noncompliant code example, the file identified by `file_name` is opened, processed, closed, and removed. However, it is possible that the file object identified by `file_name` in the call to `remove()` is not the same file object identified by `file_name` in the call to `fopen()` .

### Compliant Solution

Not much can be done programmatically to ensure the file removed is the same file that was opened, processed, and closed except to make sure that the file is opened in a secure directory with privileges that would prevent the file from being manipulated by an untrusted user. (See [FIO15-C. Ensure that file operations are performed in a secure directory](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio15-c) .)

### Noncompliant Code Example (POSIX)

In this noncompliant code example, the function `chmod()` is called to set the permissions of a file. However, it is not clear whether the file object referred to by `file_name` refers to the same object in the call to `fopen()` and in the call to `chmod()` .

### Compliant Solution (POSIX)

This compliant solution uses the POSIX `fchmod()` and `open()` functions \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \]. Using these functions guarantees that the file opened is the same file that is operated on.

### Mitigation Strategies (POSIX)

Many file-related race conditions can be eliminated by using

-   `fchown()` rather than `chown()`
-   `fstat()` rather than `stat()`
-   `fchmod()` rather than `chmod()`

or simply by ensuring the security of the working directory per [FIO15-C. Ensure that file operations are performed in a secure directory](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio15-c) .

POSIX functions that have no file descriptor counterpart should be used with caution:

-   `link()` and `unlink()`
-   `mkdir()` and `rmdir()`
-   `mount()` and `unmount()`
-   `lstat()`
-   `mknod()`
-   `symlink()`
-   `utime()`

## FIO02-C. Canonicalize path names originating from tainted sources

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/03.fio02-c.md) — SHA-256 `e3ee750a4da174fea94edfeef121a263752680d710539bcc4c4e554c750f7d1e`。

Path names, directory names, and file names may contain characters that make [validation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-validation) difficult and inaccurate. Furthermore, any path name component can be a symbolic link, which further obscures the actual location or identity of a file. To simplify file name validation, it is recommended that names be translated into their *canonical* form. Canonicalizing file names makes it much easier to verify a path, directory, or file name by making it easier to compare names.

Because the canonical form can vary between operating systems and file systems, it is best to use operating-system-specific mechanisms for canonicalization.

As an illustration, here is a function that ensures that a path name refers to a file in the user's home directory on POSIX systems:

The `verify_file()` function requires that the file name be an absolute path name. Furthermore, it can be deceived if the file name being referenced is actually a symbolic link to a file name not in the users's home directory.

### Noncompliant Code Example

In this noncompliant example, `argv[1]` contains a file name that originates from a [tainted source](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-taintedsource) and is opened for writing. Before this file name is used in file operations, it should be validated to ensure that it refers to an expected and valid file. Unfortunately, the file name referenced by `argv[1]` may contain special characters, such as directory characters, that make [validation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-validation) difficult if not impossible. Furthermore, any path name component in `argv[1]` may be a symbolic link, resulting in the file name referring to an invalid file even though it passes validation.

If validation is not performed correctly, the call to `fopen()` may result in an unintended file being accessed.

### Compliant Solution (POSIX)

Canonicalizing file names is difficult and involves an understanding of the underlying file system.

The POSIX `realpath()` function can assist in converting path names to their canonical form. According to *Standard for Information Technology—Portable Operating System Interface (POSIX®), Base Specifications, Issue 7* (IEEE Std 1003.1, 2013 Edition) \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \],

> The `realpath()` function shall derive, from the pathname pointed to by `file_name` , an absolute pathname that names the same file, whose resolution does not involve ' `.` ', ' `..` ', or symbolic links.

Further verification, such as ensuring that two successive slashes or unexpected special files do not appear in the file name, must be performed. See [Section 4.12, "Pathname Resolution,"](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap04.html#tag_04_12) of IEEE Std 1003.1, 2013 Edition, for more details on how path name resolution is performed \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \].

Many manual pages for the `realpath()` function come with an alarming warning, such as this one from the *Linux Programmer's Manual* \[ [Linux 2008](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Linux08) \]:

> Avoid using this function. It is broken by design since (unless using the non-standard `    resolved_path == NULL   ` feature) it is impossible to determine a suitable size for the output buffer, `resolved_path` . According to POSIX a buffer of size `PATH_MAX` suffices, but `PATH_MAX` need not be a defined constant, and may have to be obtained using `pathconf(3)` . And asking `pathconf(3)` does not really help, since on the one hand POSIX warns that the result of `pathconf(3)` may be huge and unsuitable for mallocing memory. And on the other hand `pathconf(3)` may return −1 to signify that `PATH_MAX` is not bounded.
>
> The libc4 and libc5 implementation contains a buffer overflow (fixed in libc-5.4.13). As a result, set-user-ID programs like `mount(8)` need a private version.

The `realpath()` function was changed in POSIX.1-2008. Older versions of POSIX allow [implementation-defined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) in situations where the `resolved_name` is a null pointer. The current POSIX revision and many current [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) (led by glibc and Linux) allocate memory to hold the resolved name if a null pointer is used for this argument.

The following statement can be used to conditionally include code that depends on this revised form of the `realpath()` function:

Consequently, despite the alarming warnings, it is safe to call `realpath()` with `resolved_name` assigned the value `NULL` (on systems that support it), as shown in this compliant solution:

It is also safe to call `realpath()` with a non-null `resolved_path` provided that `PATH_MAX` is defined as a constant in `<limits.h>` . In this case, the `realpath()` function expects `resolved_path` to refer to a character array that is large enough to hold the canonicalized path. If `PATH_MAX` is defined, allocate a buffer of size `PATH_MAX` to hold the result of `realpath()` , as shown in this compliant solution:

Care still must be taken to avoid creating a [time-of-check, time-of-use (TOCTOU)](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-TOCTOU) condition by using `realpath()` to check a file name.

### Noncompliant Code Example (POSIX)

Calling the `realpath()` function with a non-null `resolved_path` when `PATH_MAX` is not defined as a constant is not safe. IEEE Std 1003.1, 2013 Edition \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \], effectively forbids such uses of `realpath()` :

> If *resolved_name* is not a null pointer and {PATH_MAX} is not defined as a constant in the **\<limits.h>** header, the behavior is undefined.

The rationale from IEEE Std 1003.1, 2013 Edition, explains why this case is unsafe:

> Since *realpath* ( ) has no *length* argument, if {PATH_MAX} is not defined as a constant in **\<limits.h>** , applications have no way of determining how large a buffer they need to allocate for it to be safe to pass to *realpath* ( ). A {PATH_MAX} value obtained from a prior *pathconf* ( ) call is out-of-date by the time *realpath* ( ) is called. Hence the only reliable way to use *realpath* ( ) when {PATH_MAX} is not defined in **\<limits.h>** is to pass a null pointer for *resolved_name* so that *realpath* ( ) will allocate a buffer of the necessary size.

`PATH_MAX` can vary among file systems (which is the reason for obtaining it with `pathconf()` and not `sysconf()` ). A `PATH_MAX` value obtained from a prior `pathconf()` call can be invalidated, for example, if a directory in the path is replaced with a symlink to a different file system or if a new file system is mounted somewhere along the path.

#### Implementation Details (Linux)

The `libc4` and `libc5` implementations of `realpath()` contain a buffer overflow (fixed in libc-5.4.13) \[ [VU#743092](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-743092) \]. Consequently, programs need a private version of this function in which this issue is known to be fixed.

### Compliant Solution (glibc)

The `realpath()` function can be difficult to use and inefficient. Another solution, available as a GNU extension, is `canonicalize_file_name()` . This function has the same effect as `realpath()` , but the result is always returned in a newly allocated buffer \[ [Drepper 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Drepper06) \].

Because memory is allocated by `canonicalize_file_name()` , the programmer must remember to free the allocated memory.

### Noncompliant Code Example (Windows)

This noncompliant code example uses the Windows function `GetFullPathName()` for canonicalization \[ [MSDN](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MSDN) \]:

The `GetFullPathName()` function can be used to eliminate the `..` and `/./` components from a path name, but there are numerous other canonicalization issues that are not addressed by use of `GetFullPathName()` , including universal naming convention (UNC) shares, short (8.3) names, long names, Unicode names, trailing dots, forward slashes, backslashes, short cuts, and so on.

Care also must be taken to avoid creating a [TOCTOU](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-TOCTOU) condition by using `GetFullPathName()` to check a file name.

### Compliant Solution (Windows)

Producing canonical file names for Windows operating systems is extremely complex and beyond the scope of this standard. The best advice is to try to avoid making decisions based on a path, directory, or file name \[ [Howard 2002](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Howard02) \]. Alternatively, use operating-system-based mechanisms, such as access control lists (ACLs) or other authorization techniques.

## FIO03-C. Do not make assumptions about fopen() and file creation

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/04.fio03-c.md) — SHA-256 `2d48d97c717bad6e85246e2a7bb9f06cd6b423f14e4d05345289db8b42527600`。

The C `fopen()` function is used to open an existing file or create a new one. The C11 version of the `fopen()` function provides a mode flag, `x` , that provides the mechanism needed to determine if the file that is to be opened exists. Not using this mode flag can lead to a program overwriting or accessing an unintended file.

### Noncompliant Code Example ( `fopen()` )

In this noncompliant code example, the file referenced by `file_name` is opened for writing. This example is noncompliant if the programmer's intent was to create a new file, but the referenced file already exists.

### Compliant Solution (fopen("x"), C11)

Starting in C11 a new mode suffix ("x") was added to the `fopen()` function which causes `fopen()` to return NULL if the file already exists or cannot be created \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \].

### Compliant Solution ( `open()` , POSIX)

The `open()` function, as defined in the *Standard for Information Technology—Portable Operating System Interface (POSIX®), Base Specifications, Issue 7* \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \], is available on many platforms and provides finer control than `fopen()` . In particular, `open()` accepts the `O_CREAT` and `O_EXCL` flags. When used together, these flags instruct the `open()` function to fail if the file specified by `file_name` already exists.

Care should be taken when using `O_EXCL` with remote file systems because it does not work with NFS version 2. NFS version 3 added support for `O_EXCL` mode in `open()` . IETF RFC 1813 \[ [Callaghan 1995](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Callaghan95) \] defines the `EXCLUSIVE` value to the `mode` argument of `CREATE` :

> `EXCLUSIVE` specifies that the server is to follow exclusive creation semantics, using the verifier to ensure exclusive creation of the target. No attributes may be provided in this case, since the server may use the target file metadata to store the createverf3 verifier.

For examples of how to check for the existence of a file without opening it, see recommendation [FIO10-C. Take care when using the rename() function](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio10-c) .

### Compliant Solution ( `fdopen()` , POSIX)

For code that operates on `FILE` pointers and not file descriptors, the POSIX `fdopen()` function can be used to associate an open stream with the file descriptor returned by `open()` , as shown in this compliant solution \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \]:

### Compliant Solution (Windows)

The Win32 API `CreateFile()` allows a programmer to create or open a file depending on the flags passed in. Passing in the `CREATE_NEW` flag ensures the call fails if the file already exists. This compliant solution demonstrates how to open a file for reading and writing without sharing access to the file such that the call fails if the file already exists.

## FIO05-C. Identify files using multiple file attributes

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/05.fio05-c.md) — SHA-256 `d3bb2a1c3818a5ad849915b8fb82c6d5c1cd2af81fee6f50fc6ca8d946266070`。

Files can often be identified by attributes other than the file name, such as by comparing file ownership or creation time. Information about a file that has been created and closed can be stored and then used to validate the identity of the file when it is reopened.

Comparing multiple attributes of the file increases the likelihood that the reopened file is the same file that had been previously operated on.

File identification is less of an issue if applications maintain their files in secure directories, where they can be accessed only by the owner of the file and (possibly) by a system administrator. (See [FIO15-C. Ensure that file operations are performed in a secure directory](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio15-c) .)

### Noncompliant Code Example (Reopen)

This noncompliant code example opens a file for writing, closes it, opens the same named file for reading, and then closes it again. The logic relies solely on the file name to identify the file.

There is no guarantee that the file opened for reading is the same file that is opened for writing. An attacker can replace the original file (for example, with a symbolic link) between the first `fclose()` and the second `fopen()` .

### Compliant Solution (POSIX) (Device/I-node)

Reopening a file stream should generally be avoided. However, it may sometimes be necessary in long-running applications to avoid depleting available file descriptors.

This compliant solution uses a "check, use, check" pattern to ensure that the file opened for reading is the same file that was opened for writing. In this solution, the file is opened for writing using the `open()` function. If the file is successfully opened, the `fstat()` function is used to read information about the file into the `orig_st` structure. When the file is reopened for reading, information about the file is read into the `new_st` structure, and the `st_dev` and `st_ino` fields in `orig_st` and `new_st` are compared to improve identification.

This solution enables the program to recognize if an attacker has switched files between the first `close()` and the second `open()` . The program does not recognize whether the file has been modified in place, however.

Alternatively, the same solution can be implemented using the C `fopen()` function to open the file and the POSIX `fileno()` function to convert the `FILE` object pointer to a file descriptor.

The structure members `st_mode` , `st_ino` , `st_dev` , `st_uid` , `st_gid` , `st_atime` , `st_ctime` , and `st_mtime` all should have meaningful values for all file types on POSIX-compliant systems. The `st_ino` field contains the file serial number. The `st_dev` field identifies the device containing the file. The `st_ino` and `st_dev` fields, taken together, uniquely identify the file. The `st_dev` value is not necessarily consistent across reboots or system crashes, however, so this field may not be useful for file identification if a system crash or reboot may have occurred before you attempt to reopen a file.

Call the `fstat()` function on a file that is already opened instead of calling `stat()` on a file name followed by `open()` . Doing so ensures that the file for which the information is being collected is the same file that is already opened. See [FIO01-C. Be careful using functions that use file names for identification](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio01-c) for more information on avoiding race conditions resulting from the use of file names for identification.

It may also be necessary to call `open()` with `O_NONBLOCK` , as per [FIO32-C. Do not perform operations on devices that are only appropriate for files](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/input-output-fio/fio32-c) , to ensure that the program does not hang when trying to open special files.

This compliant solution may not work in some cases. For instance, a long-running service might choose to occasionally reopen a log file to add log messages but leave the file closed so that the log file may be periodically rotated. In this case, the i-node number would change, and this solution would no longer apply.

### Compliant Solution (POSIX) (Open Only Once)

A simpler solution is to not reopen the file. In this code example, the file is opened once for both writing and reading. Once writing is complete, the `fseek()` function resets the file pointer to the beginning of the file, and its contents are read back. (See [void FIO07-C. Prefer fseek() to rewind()](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/the-void/void-fio07-c-prefer-fseek-to-rewind) .)

Because the file is not reopened, the possibility of an attacker tampering with the file between the writes and subsequent reads is eliminated.

Be sure to use `fflush()` after writing data to the file, in accordance with [FIO39-C. Do not alternately input and output from a stream without an intervening flush or positioning call](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/input-output-fio/fio39-c) .

### Noncompliant Code Example (Owner)

In this noncompliant code example, the programmer's intent is to open a file for reading, but only if the user running the process owns the specified file. This requirement is more restrictive than that imposed by the operating system, which requires only that the effective user have permissions to read the file. The code, however, relies solely on the file name to identify the file.

If this code is run with superuser privileges, for example, as part of a `setuid-root` program, an attacker can exploit this program to read files for which the real user normally lacks sufficient privileges, including files not owned by the user.

### Compliant Solution (POSIX) (Owner)

In this compliant solution, the file is opened using the `open()` function. If the file is successfully opened, the `fstat()` function is used to read information about the file into the `stat` structure. This information is compared with existing information about the real user (obtained by the `getuid()` and `getgid()` functions).

By matching the file owner's user and group IDs to the process's real user and group IDs, this program now successfully restricts access to files owned by the real user of the program. This solution can be used to verify that the owner of the file is the one the program expects, reducing opportunities for attackers to replace configuration files with malicious ones, for example.

Alternatively, the same solution can be implemented using the C `fopen()` function to open the file and the POSIX `fileno()` function to convert the `FILE` object pointer to a file descriptor.

## FIO06-C. Create files with appropriate access permissions

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/06.fio06-c.md) — SHA-256 `9a5e844e14ef2e7076b36940bc8c2d4b291c9a57206c0a127bb016ba08864205`。

Creating a file with insufficiently restrictive access permissions may allow an unprivileged user to access that file. Although access permissions are heavily dependent on the file system, many file-creation functions provide mechanisms to set (or at least influence) access permissions. When these functions are used to create files, appropriate access permissions should be specified to prevent unintended access.

When setting access permissions, it is important to make sure that an attacker cannot alter them. (See [FIO15-C. Ensure that file operations are performed in a secure directory](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio15-c) .)

### Noncompliant Code Example ( `fopen()` )

The `fopen()` function does not allow the programmer to explicitly specify file access permissions. In this noncompliant code example, if the call to `fopen()` creates a new file, the access permissions are [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) :

#### Implementation Details

On POSIX-compliant systems, the permissions may be restricted by the value of the POSIX `umask()` function \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \].

The operating system modifies the access permissions by computing the intersection of the inverse of the umask and the permissions requested by the process \[ [Viega 2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Viega03) \]. For example, if the variable `requested_permissions` contained the permissions passed to the operating system to create a new file, the variable `actual_permissions` would be the actual permissions that the operating system would use to create the file:

For OpenBSD and Linux operating systems, any file created will have mode `S_IRUSR|S_IWUSR|S_IRGRP|S_IWGRP|S_IROTH|S_IWOTH` (0666), as modified by the process's umask value. (See [`fopen(3)`](http://www.openbsd.org/cgi-bin/man.cgi?query=open&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html) in the OpenBSD Manual Pages \[ [OpenBSD](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-OpenBSD) \].)

### Noncompliant Code Example ( `open()` , POSIX)

Using the POSIX `open()` function to create a file but failing to provide access permissions for that file may cause the file to be created with overly permissive access permissions. This omission has been known to lead to [vulnerabilities](http://BB.%20Definitions#vulnerability) —for example, [CVE-2006-1174](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2006-1174) .

This example also violates [EXP37-C. Call functions with the correct number and type of arguments](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp37-c) .

### Compliant Solution ( `open()` , POSIX)

Access permissions for the newly created file should be specified in the third argument to `open()` . Again, the permissions are modified by the value of `umask()` .

John Viega and Matt Messier also provide the following advice \[ [Viega 2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Viega03) \]:

> Do not rely on setting the umask to a "secure" value once at the beginning of the program and then calling all file or directory creation functions with overly permissive file modes. Explicitly set the mode of the file at the point of creation. There are two reasons to do this. First, it makes the code clear; your intent concerning permissions is obvious. Second, if an attacker managed to somehow reset the umask between your adjustment of the umask and any of your file creation calls, you could potentially create sensitive files with wide-open permissions.

## FIO08-C. Take care when calling remove() on an open file

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/07.fio08-c.md) — SHA-256 `46c7875fee456be9f86ba8082db33e602f236a88ed2bd7217e612cc3fd4983d3`。

Invoking `remove()` on an open file is [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) . Removing an open file is sometimes recommended to hide the names of temporary files that may be prone to attack. (See [FIO21-C. Do not create temporary files in shared directories](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio21-c) .)

In cases requiring the removal of an open file, a more strongly defined function, such as the POSIX `unlink()` function, should be considered. To be strictly conforming and portable, `remove()` should *not* be called on an open file.

### Noncompliant Code Example

This noncompliant code example shows a case where a file is removed while it is still open:

Some [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) will not remove the file specified by `file_name` because the stream is still open.

#### Implementation Details

Code compiled for Microsoft Windows prevents the `remove()` call from succeeding when the file is open, meaning that the file link will remain after execution completes.

### Compliant Solution (POSIX)

This compliant solution uses the POSIX `unlink()` function to remove the file. The `unlink()` function is guaranteed to unlink the file from the file system hierarchy but keep the file on disk until all open instances of the file are closed \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \].

Note that there is a race window between the `fopen()` call and the `unlink()` call, which could be exploited. This exploitation can be mitigated if the operations occur in a secure directory; see [FIO45-C. Avoid TOCTOU race conditions while accessing files](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/input-output-fio/fio45-c) for more information.

## FIO09-C. Be careful with binary data when transferring data across systems

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/08.fio09-c.md) — SHA-256 `5c409699fcc6fde3411375fb2c49db832a7a94863b131ff3b3fef2bc9c499d70`。

Portability is a concern when using the `fread()` and `fwrite()` functions across multiple, heterogeneous systems. In particular, it is never guaranteed that reading or writing of scalar data types such as integers, let alone aggregate types such as arrays or structures, will preserve the representation or value of the data. Implementations may differ in structure padding, floating-point model, number of bits per byte, endianness, and other attributes that cause binary data formats to be incompatible.

### Noncompliant Code Example

This noncompliant code example reads data from a file stream into a data structure:

However, the code makes assumptions about the layout of `myData` , which may be represented differently on a different platform.

### Compliant Solution

The best solution is to use either a text representation or a special library that ensures data integrity:

## FIO10-C. Take care when using the rename() function

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/09.fio10-c.md) — SHA-256 `b7749fd8eb333c53456216bebf297ee304fc7e66598ed9a2212ddf5c58d9d6f0`。

The `rename()` function has the following prototype:

If the file referenced by `dest_file` exists prior to calling `rename()` , the behavior is [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) . On POSIX systems, the destination file is removed. On Windows systems, the `rename()` fails. Consequently, issues arise when trying to write portable code or when trying to implement alternative behavior.

## Preserve Existing Destination File

If the desired behavior is to ensure that the destination file is not erased or overwritten, POSIX programmers must implement additional safeguards.

### Noncompliant Code Example (POSIX)

This code example is noncompliant because any existing destination file is removed by `rename()` :

### Compliant Solution (POSIX)

If the programmer's intent is not to remove an existing destination file, the POSIX `access()` function can be used to check for the existence of a file \[ [IEEE Std 1003.1:2013](https://www.securecoding.cert.org/confluence/display/seccode/AA.+Bibliography#AA.Bibliography-IEEEStd1003.1-2013) \]. This compliant solution renames the source file only if the destination file does not exist:

This code contains an unavoidable race condition between the call to `access()` and the call to `rename()` and can consequently be safely executed only when the destination file is located within a secure directory. (See [FIO15-C. Ensure that file operations are performed in a secure directory](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio15-c) .)

On file systems where the program does not have sufficient permissions in the directory to view the file, `access()` may return `-1` even when the file exists. In such cases, `rename()` will also fail because the program lacks adequate permissions to perform the operation.

In situations where the source file is supposed not to be a directory or symbolic link, an alternative solution is to use `link()` to link the source file to the destination file and then use `unlink()` (or `remove()` ) to delete the source file. Because `link()` fails if the destination file exists, the need for calling `access()` is avoided. However, this solution has two race conditions related to the source file. First, before calling `link()` , the program must use `lstat()` to check that the source file is not a directory or symbolic link. Second, the source file could change during the time window between the `link()` and the `unlink()` . Consequently, this alternative solution can be safely executed only when the source file is located within a secure directory.

### Compliant Solution (Windows)

On Windows, the [`rename()`](https://msdn.microsoft.com/en-us/library/zw5t957f(VS.80).aspx) function fails if a

> file or directory specified by `newname` already exists or could not be created (invalid path). \[ [MSDN](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MSDN) \]

Consequently, it is unnecessary to explicitly check for the existence of the destination file before calling `rename()` .

## Remove Existing Destination File

If the desired behavior is to ensure that the destination file is erased by the `rename()` operation, Windows programmers must write additional code.

### Noncompliant Code Example (Windows)

If the intent of the programmer is to remove the file referenced by `dest_file` if it exists prior to calling `rename()` , this code example is noncompliant on Windows platforms because `rename()` will fail:

### Compliant Solution (Windows)

On Windows systems, it is necessary to explicitly remove the destination file before calling `rename()` if the programmer wants the file to be overwritten and the `rename()` operation to succeed:

This code contains unavoidable race conditions between the calls to `_access_s()` , `remove()` , and `rename()` and can consequently be safely executed only within a secure directory. (See [FIO15-C. Ensure that file operations are performed in a secure directory](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio15-c) .)  Another option would be to use the [MoveFileEx](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365240(v=vs.85).aspx) API and pass in the `MOVEFILE_REPLACE_EXISTING` flag:

Although this code is not portable, it does avoid the race condition when using \_access_s() , `remove()` , and `rename()` .

### Compliant Solution (POSIX)

On POSIX systems, if the destination file exists prior to calling `rename()` , the file is automatically removed:

## Portable Behavior

A programmer who wants an application to behave the same on any C [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) must first determine what behavior to implement.

### Compliant Solution (Remove Existing Destination File)

This compliant solution ensures that any destination file is portably removed:

This code contains an unavoidable race condition between the call to `remove()` and the call to `rename()` and consequently can be safely executed only within a secure directory. (See [FIO15-C. Ensure that file operations are performed in a secure directory](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio15-c) .)

The return value of `remove()` is deliberately not checked because it is expected to fail if the file does not exist. If the file exists but cannot be removed, the `rename()` call will also fail, and the error will be detected at that point. This is a valid exception (EXP12-C-EX1) to [EXP12-C. Do not ignore values returned by functions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/expressions-exp/exp12-c) .

### Compliant Solution (Preserve Existing Destination File)

This compliant solution renames the source file only if the destination file does not exist:

This code contains an unavoidable race condition between the call to `file_exists()` and the call to `rename()` and can consequently be safely executed only within a secure directory. (See [FIO15-C. Ensure that file operations are performed in a secure directory](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio15-c) .)

The `file_exists()` function is provided by the application and is not shown here because it must be implemented differently on different platforms. (On POSIX systems, it would use `access()` ; on Windows, `_access_s()` ; and on other platforms, whatever function is available to test file existence.)

## FIO11-C. Take care when specifying the mode parameter of fopen()

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/10.fio11-c.md) — SHA-256 `6b4e2775c4a467c0a8a2a3914eab2385e2c83a50ca510877aa41cfa78631bc00`。

The C Standard identifies specific strings to use for the `mode` on calls to `fopen()` and `fopen_s()` . C11 provides a new mode flag, `x` , that provides the mechanism needed to determine if the file that is to be opened exists. To be strictly [conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conforming) and portable, one of the strings from the following table (adapted from the C Standard, subclause 7.21.5.2 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \]) must be used:

**Strings to Use for the Mode on Calls to `fopen()` and `fopen_s()`**

|                                            |                                                                       |
|--------------------------------------------|-----------------------------------------------------------------------|
| `mode` String                 | Result                                                                |
| `r`                           | Open text file for reading                                            |
| `w`                           | Truncate to zero length or create text file for writing               |
| `wx`                          | Create text file for writing                                          |
| `a`                           | Append; open or create text file for writing at end-of-file           |
| `rb`                          | Open binary file for reading                                          |
| `wb`                          | Truncate to zero length or create binary file for writing             |
| `wbx`                         | Create binary file for writing                                        |
| `ab`                          | Append; open or create binary file for writing at end-of-file         |
| `r+`                          | Open text file for update (reading and writing)                       |
| `w+`                          | Truncate to zero length or create text file for update                |
| `w+x`                         | Create text file for update                                           |
| `a+`                          | Append; open or create text file for update, writing at end-of-file   |
| `r+b` or `rb+`   | Open binary file for update (reading and writing)                     |
| `w+b` or `wb+`   | Truncate to zero length or create binary file for update              |
| `w+bx` or `wb+x` | Create binary file for update                                         |
| `a+b` or `ab+`   | Append; open or create binary file for update, writing at end-of-file |

If the `mode` string begins with one of these sequences, the [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) might choose to ignore the remaining characters, or it might use them to select different kinds of files.

When calling `fopen_s()` , any of the mode strings used for writing ( `w` or `a` ) may be prefixed with the `u` character to give the file system default access permissions.

An implementation may define additional `mode` strings, but only the modes shown in the table are fully portable and C compliant. Beware that Microsoft Visual Studio 2012 and earlier do not support the `x` or `u` mode characters \[ [MSDN](https://msdn.microsoft.com/en-us/library/z5hh6ee9(v=vs.110).aspx) \].

## FIO13-C. Never push back anything other than one read character

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/11.fio13-c.md) — SHA-256 `c717940e8036bd1ecd1ff517231b3de93fc100ee19898554f409625546e9b3b6`。

Subclause 7.21.7.10 of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] defines `ungetc()` as follows:

> The `ungetc` function pushes the character specified by `c` (converted to an `    unsigned char   ` ) back onto the input stream pointed to by `stream` . Pushed-back characters will be returned by subsequent reads on that stream in the reverse order of their pushing. A successful intervening call (with the stream pointed to by `stream` ) to a file positioning function ( `fseek` , `fsetpos` , or `rewind` ) discards any pushed-back characters for the stream. The external storage corresponding to the stream is unchanged.
>
> One character of pushback is guaranteed.

Consequently, multiple calls to `ungetc()` on the same stream must be separated by a call to a read function or a file-positioning function (which will discard any data pushed by `ungetc()` ).

Likewise, for `ungetwc()` , C guarantees only one wide character of pushback (subclause 7.29.3.10). Consequently, multiple calls to `ungetwc()` on the same stream must be separated by a call to a read function or a file-positioning function (which will discard any data pushed by `ungetwc()` ).

### Noncompliant Code Example

In this noncompliant code example, more than one character is pushed back on the stream referenced by `fp` :

### Compliant Solution

If more than one character needs to be pushed by `ungetc()` , then `fgetpos()` and `fsetpos()` should be used before and after reading the data instead of pushing it back with `ungetc()` . Note that this solution applies only if the input is seekable.

Remember to always call `fgetpos()` before `fsetpos()` . (See [FIO44-C. Only use values for fsetpos() that are returned from fgetpos()](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/input-output-fio/fio44-c) .)

## FIO14-C. Understand the difference between text mode and binary mode with file streams

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/12.fio14-c.md) — SHA-256 `d37c5175662d870ba890af28e3c0129de48a2952ba5b8bd1f34f2111999920f0`。

Input and output are mapped into logical data streams whose properties are more uniform than their various inputs and outputs. Two forms of mapping are supported, one for text streams and one for binary streams. They differ in the actual representation of data as well as in the functionality of some C functions.

## Text Streams

### *Representation*

Characters may have to be altered to conform to differing conventions for representing text in the [host environment](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-hostedenvironment) . As a consequence, data read to or written from a text stream will not necessarily compare equal to the stream's byte content.

The following code opens the file `myfile` as a text stream:

Environments may model line breaks differently. For example, on Windows, this code writes 2 bytes (a carriage return and then a newline) to the file, whereas on POSIX systems, this code writes only 1 byte (a newline).

### *`fseek()`*

For a text stream, the offset for `fseek()` must be either 0 or a value returned by an earlier successful call to the `ftell()` function (on a stream associated with the same file) with a mode of `SEEK_SET` .

### *`ungetc()`*

The `ungetc()` function causes the file position indicator to be *unspecified* until all pushed-back characters are read. As a result, care must be taken that file-position-related functions are not used while this is true.

## Binary Streams

### *Representation*

A binary stream is an ordered sequence of characters that can transparently record internal data. As a consequence, data read from or written to a binary stream will necessarily compare equal to the stream's byte content.

The following code opens the file `myfile` as a binary stream:

Regardless of environment, this code writes exactly 1 byte (a newline).

### *`fseek()`*

According to the C Standard, a binary stream may be terminated with an unspecified number of null characters and need not meaningfully support `fseek()` calls with a mode of `SEEK_END` . Consequently, do not call `fseek()` on a binary stream with a mode of `SEEK_END` .

### *`ungetc()`*

The `ungetc()` function causes the file-position indicator to be decremented by 1 for each successful call, with the value being [indeterminate](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) if it is 0 before any call. As a result, `ungetc()` must never be called on a binary stream where the file position indicator is 0.

## FIO15-C. Ensure that file operations are performed in a secure directory

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/13.fio15-c.md) — SHA-256 `e4b973469428e09289e7c60013074f386c759b524c9285a6d9e8a0d7271aaa00`。

File operations should be performed in a *secure directory* . In most cases, a secure directory is a directory in which no one other than the user, or possibly the administrator, has the ability to create, rename, delete, or otherwise manipulate files. (Other users may read or search the directory but generally may not modify the directory's contents in any way.) Also, other users must not be able to delete or rename files in the parent of the secure directory and all higher directories, although creating new files or deleting or renaming files they own is permissible.

Performing file operations in a secure directory eliminates the possibility that an attacker might tamper with the files or file system to [exploit](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-exploit) a file system [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) in a program. These vulnerabilities often exist because there is a loose binding between the file name and the actual file. (See [FIO01-C. Be careful using functions that use file names for identification](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio01-c) .) In some cases, file operations can be performed securely anywhere. In other cases, the only way to ensure secure file operations is to perform the operation within a secure directory.

Ensuring that file systems are configured in a safe manner is typically a system administration function. However, programs can often check that a file system is securely configured before performing file operations that may lead to security vulnerabilities if the system is misconfigured. There is a slight possibility that file systems will be reconfigured in an insecure manner while a process is running and after the check has been made. As a result, it is always advisable to implement your code in a secure manner (that is, consistent with the other rules and recommendations in this section) even when running in a secure directory.

### Noncompliant Code Example

In this noncompliant code example, the file identified by `file_name` is opened, processed, closed, and removed:

An attacker can replace the file object identified by `file_name` with a link to an arbitrary file before the call to `fopen()` . It is also possible that the file object identified by `file_name` in the call to `remove()` is not the same file object identified by `file_name` in the call to `fopen()` . If the file is not in a secure directory, for example, `/tmp/app/tmpdir/passwd` , then an attacker can manipulate the location of the file as follows:

Not much can be done programmatically to ensure the file removed is the same file that was opened, processed, and closed, except to make sure that the file is opened in a secure directory with privileges that would prevent the file from being manipulated by an untrusted user.

### Compliant Solution (POSIX)

This sample [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) of the function `secure_dir()` ensures that the directory `fullpath` and all directories above it are owned by either the user or the superuser and that other users do not have write access to the directories. When checking directories, it is important to traverse from the root to the leaf to avoid a dangerous race condition in which an attacker who has privileges to at least one of the directories can rename and re-create a directory after the privilege verification.

`fullpath` need not be canonicalized (see [FIO02-C. Canonicalize path names originating from tainted sources](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio02-c) ). If the path contains a symbolic link, this routine recursively invokes itself on the linked-to directory and ensures it is also secure. A symbolically linked directory may be secure if both its source and linked-to directory are secure.

Note that this function is effective only on file systems that are fully compatible with UNIX permissions, and it may not behave normally for file systems with other permission mechanisms, such as AFS (Andrew File System).

This compliant solution uses the `secure_dir()` function to ensure that an attacker may not tamper with the file to be opened and subsequently removed. Note that once the path name of a directory has been checked using `secure_dir()` , all further file operations on that directory must be performed using the same path.

## FIO17-C. Do not rely on an ending null character when using fread()

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/14.fio17-c.md) — SHA-256 `c36bd4001198dcc8b168089e76bf88ffa50dca3f8eabc77a2e07a2ea8bcd854c`。

The `fread()` function, as defined in the C Standard, subclause 7.21.8.1 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], does not explicitly null-terminate the read character sequence.

> Synopsis  
> `    size_t fread(void * restrict ptr, size_t size, size_t nmemb, FILE * restrict stream)   `
>
> Description  
> The `fread` function reads, into the array pointed to by `ptr` , up to `nmemb` elements  
> whose size is specified by `size` , from the stream pointed to by `stream` .

Although the content of a file has a properly null-terminated character sequence, if `nmemb` is less than the total length of the characters, the `fread()` function will not read after `nmemb` characters. `fread()` will not append a null character to the end of the string being read to.

### Noncompliant Code Example

Suppose we have a null-terminated character sequence in a file, and we need to extract a null-terminated byte string:

When `size` is less than the total length of the file ( `file.txt` ), `buffer` is not properly null-terminated.

### Compliant Solution

To correct this example, the size of `buffer` must be compared with the total length of the file to identify the erroneous case where `size` differs from `length` . At this point, it is up to the programmer to handle this case.

## FIO18-C. Never expect fwrite() to terminate the writing process at a null character

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/15.fio18-c.md) — SHA-256 `9c9a4f4271a27432996c922a98a2e226146b0b8381021bd246fc880bfc26d7ab`。

The C Standard, subclause 7.21.8.2 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], defines the `fwrite()` function as follows:

> Synopsis
>
> `    size_t fwrite(const void *restrict ptr, size_t size, size_t nmemb, FILE *restrict stream);   `
>
> Description
>
> The `fwrite()` function writes, from the array pointed to by `ptr` , up to `nmemb` elements whose size is specified by `size` , to the stream pointed to by `stream` . For each object, `size` calls are made to the `fputc()` function, taking the values (in order) from an array of `    unsigned char   ` exactly overlaying the object. The file position indicator for the stream (if defined) is advanced by the number of bytes successfully written. If an error occurs, the resulting value of the file position indicator for the stream is indeterminate.

The definition does not state that the `fwrite()` function will stop copying characters into the file if a null character is encountered. Therefore, when writing a null-terminated byte string to a file using the `fwrite()` function, always use the length of the string plus 1 (to account for the null character) as the `nmemb` parameter.

### Noncompliant Code Example

In this noncompliant code example, the size of the buffer is stored in `size1` , but `size2` number of characters are written to the file. If `size2` is greater than `size1` , `write()` will not stop copying characters at the null character.

### Compliant Solution

This compliant solution ensures that the correct number of characters are written to the file:

## FIO19-C. Do not use fseek() and ftell() to compute the size of a regular file

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/16.fio19-c.md) — SHA-256 `475d98a19845ae848a29923ab3ecde8c5c1db97c8056d5faaf261ca7410f78eb`。

Understanding the difference between text mode and binary mode is important when using functions that operate on file streams. (See [FIO14-C. Understand the difference between text mode and binary mode with file streams](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio14-c) for more information.)

Subclause 7.21.9.2 of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] specifies the following behavior for `fseek()` when opening a binary file in binary mode:

> A binary stream need not meaningfully support fseek calls with a `whence` value of `SEEK_END` .

In addition, footnote 268 of subclause 7.21.3 says:

> Setting the file position indicator to end-of-file, as with `    fseek(file, 0, SEEK_END)   ` , has undefined behavior for a binary stream (because of possible trailing null characters) or for any stream with state-dependent encoding that does not assuredly end in the initial shift state.

Seeking to the end of a binary stream in binary mode with `fseek()` is not meaningfully supported and is not a recommended method for computing the size of a file.

Subclause 7.21.9.4 of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] specifies the following behavior for `ftell()` when opening a text file in text mode:

> For a text stream, its file position indicator contains unspecified information, usable by the `fseek` function for returning the file position indicator for the stream to its position at the time of the `ftell` call.

Consequently, the return value of `ftell()` for streams opened in text mode should never be used for offset calculations other than in calls to `fseek()` .

POSIX \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \] provides several guarantees that the problems described in the C Standard do not occur on POSIX systems.

First, the `fopen` page says:

> The character ' `b` ' shall have no effect, but is allowed for ISO C standard conformance.

This guarantees that binary files are treated the same as text files in POSIX.

Second, the `fwrite` page says:

> For each object, *size* calls shall be made to the *fputc()* function, taking the values (in order) from an array of **unsigned char** exactly overlaying the object. The file-position indicator for the stream (if defined) shall be advanced by the number of bytes successfully written.

This means that the file position indicator, and consequently the file size, is directly based on the number of bytes actually written to a file.

### Noncompliant Code Example (Binary File)

This code example attempts to open a binary file in binary mode and use `fseek()` and `ftell()` to obtain the file size. This code is noncompliant on systems that do not provide the same guarantees as POSIX. On these systems, setting the file position indicator to the end of the file using `fseek()` is not guaranteed to work for a binary stream, and consequently, the amount of memory allocated may be incorrect, leading to a potential [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) .

### Compliant Solution (POSIX `ftello()` )

If the code needs to handle large files, it is preferable to use `fseeko()` and `ftello()` because, for some implementations, they can handle larger file offsets than `fseek()` and `ftell()` can handle. If they are used, the `file_size` variable should have type `off_t` to avoid the possibility of overflow when assigning the return value of `ftello()` to it. This solution works only with regular files.

### Compliant Solution (POSIX `fstat()` )

This compliant solution uses the size provided by the POSIX `fstat()` function, rather than by `fseek()` and `ftell()` , to obtain the size of the binary file. This solution works only with regular files.

### Compliant Solution (Windows)

This compliant solution uses the Windows `_filelength()` function to determine the size of the file on a 32-bit operating system. For a 64-bit operating system, consider using `_filelengthi64` instead.

### Compliant Solution (Windows)

This compliant solution uses the Windows `GetFileSizeEx()` function to determine the size of the file on a 32- or 64-bit operating system:

### Noncompliant Code Example (Text File)

This noncompliant code example attempts to open a text file in text mode and use `fseek()` and `ftell()` to obtain the file size:

However, the file position indicator returned by `ftell()` with a file opened in text mode is useful only in calls to `fseek()` . As such, the value of `file_size` may not necessarily be a meaningful measure of the number of characters in the file, and consequently, the amount of memory allocated may be incorrect, leading to a potential [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) .

The Visual Studio documentation for `ftell()` \[ [MSDN](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-MSDN) \] states:

> The value returned by `ftell` may not reflect the physical byte offset for streams opened in text mode, because text mode causes carriage return-linefeed translation. Use `ftell` with `fseek` to return to file locations correctly.

Again, this indicates that the return value of `ftell()` for streams opened in text mode is useful only in calls to `fseek()` and should not be used for any other purpose.

### Compliant Solution (Windows)

The compliant solution used for binary files on Windows can also be used for text files.

### Compliant Solution (POSIX)

Because binary files are treated the same as text files in POSIX, either compliant solution can be used for determining the size of a binary file under POSIX to determine the size of a text file as well.

## FIO20-C. Avoid unintentional truncation when using fgets() or fgetws()

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/17.fio20-c.md) — SHA-256 `313dca9b72aa02158d6bf5f15bbed267a03105d86e090eacaaa7c331089b3eb5`。

The `fgets()` and `fgetws()` functions are typically used to read a newline-terminated line of input from a stream. Both functions read at most one less than the number of narrow or wide characters specified by an argument `n` from a stream to a string. Truncation errors can occur if `   n - 1  ` is less than the number of characters appearing in the input string prior to the new-line narrow or wide character (which is retained) or after end-of-file.  This can result in the accidental truncation of user input.

### Noncompliant Code Example

This noncompliant code example copies the input string into a buffer, and assumes it captured all of the user's input.

However, if the last character in `buf` is not a newline and the stream is not at the end-of-file marker, the buffer was too small to contain all of the data from the user.  For example, because the buffer is only 8 characters in length, if the user input `   "Hello World\n"  ` , the buffer would contain `   "Hello W"  ` terminated by a null character.

### Compliant Solution (Fail on Truncation)

This compliant solution examines the end-of-file marker for the stream and the last character in the buffer to determine whether it is a newline or not.  If it is the end of file, or the last character is a newline, then the buffer contains all of the user's input.  However, if the last character is not at the end-of-file and not a newline then the user's input has been truncated.

### Compliant Solution (Expanding Buffer)

This compliant solution solves the problem by expanding the buffer to read the entire contents from `stdin` instead of failing if the caller did not allocate enough space.  If the allocation fails, it will return `NULL` , but otherwise, it returns a buffer of the received data, which the caller must free.

### Compliant Solution (POSIX `getline()` )

The `getline()` function was originally a GNU extension, but is now standard in POSIX.1-2008. It also fills a string with characters from an input stream. In this case, the program passes it a NULL pointer for a string, indicating that `getline()` should allocate sufficient space for the string and the caller frees it later.

## FIO21-C. Do not create temporary files in shared directories

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/18.fio21-c.md) — SHA-256 `13983abce02c46cfd46e06640417eac3a58ffaedab507cbc97d2878fe8dcb527`。

Programmers frequently create temporary files in directories that are writable by everyone (examples are `/tmp` and `/var/tmp` on UNIX and `%TEMP%` on Windows) and may be purged regularly (for example, every night or during reboot).

Temporary files are commonly used for auxiliary storage for data that does not need to, or otherwise cannot, reside in memory and also as a means of communicating with other processes by transferring data through the file system. For example, one process will create a temporary file in a shared directory with a well-known name or a temporary name that is communicated to collaborating processes. The file then can be used to share information among these collaborating processes.

This practice is dangerous because a well-known file in a shared directory can be easily hijacked or manipulated by an attacker. [Mitigation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-mitigation) strategies include the following:

1.  Use other low-level IPC (interprocess communication) mechanisms such as sockets or shared memory.
2.  Use higher-level IPC mechanisms such as remote procedure calls.
3.  Use a secure directory or a jail that can be accessed only by application instances (ensuring that multiple instances of the application running on the same platform do not compete).

There are many different IPC mechanisms; some require the use of temporary files, and others do not. An example of an IPC mechanism that uses temporary files is the POSIX `mmap()` function. Berkeley Sockets, POSIX Local IPC Sockets, and System V Shared Memory do not require temporary files. Because the multiuser nature of shared directories poses an inherent security risk, the use of shared temporary files for IPC is discouraged.

When two or more users or a group of users have write permission to a directory, the potential for deception is far greater than it is for shared access to a few files. Consequently, temporary files in shared directories must be

-   Created unpredictable file names
-   Created with unique names
-   Opened only if the file doesn't already exist (atomic open)
-   Opened with exclusive access
-   Opened with appropriate permissions
-   Removed before the program exits

The following table lists common temporary file functions and their respective conformance to these criteria:

### Conformance of File Functions to Criteria for Temporary Files

|  | `tmpnam`<br /><br />(C) | `tmpfile`<br /><br />(C/POSIX) | `mktemp`<br /><br />(POSIX) | `mkstemp`<br /><br />(POSIX) |
| --- | --- | --- | --- | --- |
| **Unpredictable Name** | Not portably | Not portably | Not portably | Not portably |
| **Unique Name** | Yes | Yes | Yes | Yes |
| **Atomic open<br />** | No | Yes | No | Yes |
| **Exclusive Access** | Possible | No | Possible | If supported by OS |
| **Appropriate Permissions** | Possible | If supported by OS * | Possible | Yes (POSIX.1-2008 and later) |
| **File Removed** | No | Yes* | No | No |

\* If the program terminates abnormally, this behavior is [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) .

Securely creating temporary files is error prone and dependent on the version of the C runtime library used, the operating system, and the file system. Code that works for a locally mounted file system, for example, may be vulnerable when used with a remotely mounted file system. Moreover, none of these functions are without problems. The only secure solution is to not create temporary files in shared directories.

### Unique and Unpredictable File Names

Privileged programs that create temporary files in world-writable directories can be [exploited](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-exploit) to overwrite protected system files. An attacker who can predict the name of a file created by a privileged program can create a symbolic link (with the same name as the file used by the program) to point to a protected system file. Unless the privileged program is coded securely, the program will follow the symbolic link instead of opening or creating the file that it is supposed to be using. As a result, a protected system file to which the symbolic link points can be overwritten when the program is executed \[ [HP 2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-HP03) \]. Unprivileged programs can be similarly exploited to overwrite protected user files.

### Exclusive Access

Exclusive access grants unrestricted file access to the locking process while denying access to all other processes and eliminates the potential for a race condition on the locked region. (See *Secure Coding in C and* C++, Chapter 8 \[ [Seacord 2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Seacord2013) \].)

Files, or regions of files, can be locked to prevent two processes from concurrent access. Windows supports two types of file locks:

-   *Shared locks* , provided by `LockFile()` , prohibit all write access to the locked file region while allowing concurrent read access to all processes.
-   *Exclusive locks* , provided by `LockFileEx()` , grant unrestricted file access to the locking process while denying access to all other processes.

In both cases, the lock is removed by calling `UnlockFile()` .

Both shared locks and exclusive locks eliminate the potential for a race condition on the locked region. The exclusive lock is similar to a mutual exclusion solution, and the shared lock eliminates race conditions by removing the potential for altering the state of the locked file region (one of the required properties for a race).

These Windows file-locking mechanisms are called *mandatory locks* because every process attempting to access a locked file region is subject to the restriction. Linux implements mandatory locks and advisory locks. An advisory lock is not enforced by the operating system, which severely diminishes its value from a security perspective. Unfortunately, the mandatory file lock in Linux is also largely impractical for the following reasons:

-   Mandatory locking works only on local file systems and does not extend to network file systems (such as NFS or AFS).
-   File systems must be mounted with support for mandatory locking, and this is disabled by default.
-   Locking relies on the group ID bit that can be turned off by another process (thereby defeating the lock).

### Removal before Termination

Removing temporary files when they are no longer required allows file names and other resources (such as secondary storage) to be recycled. In the case of [abnormal termination](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) , there is no sure method that can guarantee the removal of orphaned files. For this reason, temporary file cleaner utilities, which are invoked manually by a system administrator or periodically run by a daemon to sweep temporary directories and remove old files, are widely used. However, these utilities are themselves vulnerable to file-based [exploits](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-exploit) and often require the use of shared directories. During normal operation, it is the responsibility of the program to ensure that temporary files are removed either explicitly or through the use of library routines, such as `tmpfile_s` , which guarantee temporary file deletion upon program termination.

### Noncompliant Code Example ( `fopen()/open()` with `tmpnam()` )

This noncompliant code example creates a file with a hard-coded `file_name` (presumably in a shared directory such as `/tmp` or `C:\Temp` ):

Because the name is hard coded and consequently neither unique nor unpredictable, an attacker need only replace a file with a symbolic link, and the target file referenced by the link is opened and truncated.

This noncompliant code example attempts to remedy the problem by generating the file name at runtime using `tmpnam()` . The C `tmpnam()` function generates a string that is a valid file name and that is not the same as the name of an existing file. Files created using strings generated by the `tmpnam()` function are temporary in that their names should not collide with those generated by conventional naming rules for the [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) . The function is potentially capable of generating `TMP_MAX` different strings, but any or all of them may already be in use by existing files.

Because `tmpnam()` does not guarantee a unique name and `fopen()` does not provide a facility for an exclusive open, this code is still vulnerable.

The next noncompliant code example attempts to remedy the problem by using the POSIX `open()` function and providing a mechanism to indicate whether an existing file has been opened for writing or a new file has been created \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \]. If the `O_CREAT` and `O_EXCL` flags are used together, the `open()` function fails when the file specified by `file_name` already exists. To prevent an existing file from being opened and truncated, include the flags `O_CREAT` and `O_EXCL` when calling `open()` :

This call to `open()` fails whenever `file_name` already exists, including when it is a symbolic link, but a temporary file is presumably still required. Additionally, the method used by `tmpnam()` to generate file names is not guaranteed to be unpredictable, which leaves room for an attacker to guess the file name ahead of time.

Care should be observed when using `O_EXCL` with remote file systems because it does not work with NFS version 2. NFS version 3 added support for `O_EXCL` mode in `open()` ; see IETF RFC 1813 \[ [Callaghan 1995](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Callaghan95) \], particularly the `EXCLUSIVE` value to the `mode` argument of `CREATE` .

Moreover, the `open()` function, as specified by the *Standard for Information Technology—Portable Operating System Interface (POSIX®), Base Specifications, Issue 7* \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \], does not include support for shared or exclusive locks. However, BSD systems support two additional flags that allow you to obtain these locks:

-   `O_SHLOCK` : Atomically obtain a shared lock.
-   `O_EXLOCK` : Atomically obtain an exclusive lock.

### Noncompliant Code Example ( `mktemp()/open()` , POSIX)

The POSIX function `mktemp()` takes a given file name template and overwrites a portion of it to create a file name. The template may be any file name with exactly six X's appended to it (for example, `/tmp/temp.XXXXXX` ). The six trailing X's are replaced with the current process number and/or a unique letter combination.

The `mktemp()` function is marked "LEGACY" in the Open Group Base Specifications Issue 6 \[ [Open Group 2004](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-OpenGroup04) \]. The manual page for `mktemp()` gives more detail:

> Never use `mktemp()` . Some implementations follow BSD 4.3 and replace `XXXXXX` by the current process id and a single letter, so that at most 26 different names can be returned. Since on the one hand the names are easy to guess, and on the other hand there is a race between testing whether the name exists and opening the file, every use of `mktemp()` is a security risk. The race is avoided by `mkstemp(3)` .

### Noncompliant Code Example ( `tmpfile()` )

The `tmpfile()` function creates a temporary binary file that is different from any other existing file and that is automatically removed when it is closed or at program termination.

It should be possible to open at least `TMP_MAX` temporary files during the lifetime of the program. (This limit may be shared with `tmpnam()` .) Subclause 7.21.4.4, paragraph 6, of the C Standard allows for the value of the macro `TMP_MAX` to be as small as 25.

Most historic implementations provide only a limited number of possible temporary file names (usually 26) before file names are recycled.

### Compliant Solution ( `mkstemp()` , POSIX)

The `mkstemp()` algorithm for selecting file names has shown to be immune to attacks. The `mkstemp()` function is available on systems that support the Open Group Base Specifications Issue 4, version 2 or later.

A call to `mkstemp()` replaces the six `X` 's in the template string with six randomly selected characters and returns a file descriptor for the file (opened for reading and writing), as in this compliant solution:

This solution is not serially reusable, however, because the `mkstemp()` function replaces the `"XXXXXX"` in `template` the first time it is invoked. This is not a problem as long as `template` is reinitialized before calling `mkstemp()` again. If `template` is not reinitialized, the `mkstemp()` function will return `-1` and leave `template` unmodified because `template` did not contain six `X` 's.

The Open Group Base Specification Issue 6 \[ [Open Group 2004](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-OpenGroup04) \] does not specify the permissions the file is created with, so these are [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) . However, IEEE Std 1003.1, 2013 Edition \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \] specifies them as `S_IRUSR|S_IWUSR` (0600).

This compliant solution invokes the user-defined function `secure_dir()` (such as the one defined in [FIO15-C. Ensure that file operations are performed in a secure directory](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio15-c) ) to ensure the temporary file resides in a secure directory.

#### Implementation Details

For GLIBC, versions 2.0.6 and earlier, the file is created with permissions 0666; for GLIBC, versions 2.0.7 and later, the file is created with permissions 0600. On NetBSD, the file is created with permissions 0600. This creates a security risk in that an attacker will have write access to the file immediately after creation. Consequently, programs need a private version of the `mkstemp()` function in which this issue is known to be fixed.

In many older [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) , the name is a function of process ID and time, so it is possible for the attacker to predict the name and create a decoy in advance. FreeBSD changed the `mk*temp()` family to eliminate the process ID component of the file name and replace the entire field with base-62 encoded randomness. This raises the number of possible temporary files for the typical use of six `X` 's significantly, meaning that even `mktemp()` with six `X` 's is reasonably (probabilistically) secure against guessing except under frequent usage \[ [Kennaway 2000](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Kennaway00) \].

## FIO22-C. Close files before spawning processes

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/19.fio22-c.md) — SHA-256 `9b7dfda0e971b726602a7b3bd980325d3ca33c3be1b5fb7b4f5ea3f0da4cf6cd`。

Standard `FILE` objects and their underlying representation (file descriptors on POSIX platforms or handles elsewhere) are a finite resource that must be carefully managed. The number of files that an [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) guarantees may be open simultaneously is bounded by the `FOPEN_MAX` macro defined in `<stdio.h>` . The value of the macro is guaranteed to be at least 8. Consequently, portable programs must either avoid keeping more than `FOPEN_MAX` files at the same time or be prepared for functions such as `fopen()` to fail due to resource exhaustion.

Failing to close files when they are no longer needed may allow attackers to exhaust, and possibly manipulate, system resources. This phenomenon is sometimes called *file descriptor leakage* , although file pointers may also be used as an attack vector. In addition, keeping files open longer than necessary increases the risk that data written into in-memory file buffers will not be flushed in the event of [abnormal program termination](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) . To prevent file descriptor leaks and to guarantee that any buffered data will be flushed into permanent storage, files must be closed when they are no longer needed.

The behavior of a program is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) when it uses the value of a pointer to a `FILE` object after the associated file is closed (see [undefined behavior 153](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_153) .) Programs that close the standard streams (especially `stdout` but also `stderr` and `stdin` ) must be careful not to use the stream objects in subsequent function calls, particularly those that implicitly operate on such objects (such as `printf()` , `perror()` , and `getchar()` ).

### Noncompliant Code Example

In this noncompliant code example, derived from a [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) in OpenBSD's `chpass` program \[ [NAI 1998](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-NAI98) \], a file containing sensitive data is opened for reading. The `get_validated_editor()` function retrieves the registered editor from the `EDITOR` environment variable and [sanitizes](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sanitize) it to be a valid editor in accordance with [FIO02-C. Canonicalize path names originating from tainted sources.](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio02-c) The function returns a command for invoking the editor  which is subsequently passed as a command `system()` function. If the `system()` function is implemented in a way that spawns a child process, then the child process could inherit the file descriptors opened by its parent. If this happens, as it does in POSIX systems, the child process will be able to access the contents of the potentially sensitive file called `file_name` .

If the command returned by `get_validated_editor()` will always be a simple path (such as `/usr/bin/vim` ), and runs on a POSIX system, this program could be strengthened by using a call to `execve()` rather than `system()` , in accordance with [ENV33-C. Do not call system()](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/environment-env/env33-c) .

On UNIX-based systems, child processes are typically spawned using a form of `fork()` and `exec()` , and the child process always inherits from its parent any file descriptors that do not have the close-on-exec flag set. Under Microsoft Windows, file-handle inheritance is determined on a per-file and per-spawned process basis. See [WIN03-C. Understand HANDLE inheritance](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/microsoft-windows-win/win03-c) for more information.

### Compliant Solution

In this compliant solution, `file_name` is closed before launching the editor:

### Compliant Solution (POSIX)

Sometimes it is not practical for a program to close all active file descriptors before issuing a system call such as `system()` or `exec()` . An alternative on POSIX systems is to use the `FD_CLOEXEC` flag, or `O_CLOEXEC` when available, to set the close-on-exec flag for the file descriptor:

### Compliant Solution (Linux)

Some systems (such as those with Linux kernel versions 2.6.23 and later) have an `O_CLOEXEC` flag that provides the close-on-exec function directly in `open()` . This flag is required by IEEE Std 1003.1 \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \]. In multithreaded programs, this flag should be used, if possible, because it prevents a timing hole between `open()` and `fcntl()` when using `FD_CLOEXEC` , during which another thread can create a child process while the file descriptor does not have close-on-exec set.

## FIO23-C. Do not exit with unflushed data in stdout or stderr

**Recommendation / 条件付き採用**。原典のDeprecatedはC23だけを要求するコードへの非適用。C23より前への適合が必要なら本文どおり適用する。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/20.fio23-c.md) — SHA-256 `6ae1a93b41a25648f9c82565212f8b9c3db68e400630a6e7b692ff1b7d9593ff`。

Deprecated

This guideline does not apply to code that need conform only to C23. Code that must conform to older versions of the C standard should still comply with this guideline.

The C standard makes no guarantees as to when output to `stdout` (standard output) or `stderr` (standard error) is actually flushed. On many platforms, output to `stdout` is buffered unless `stdout` outputs to a terminal, and `stderr` output is typically not buffered. However, programs are free to modify the buffering rules for either `stdout` or `stderr` . Programs are also free to explicitly close `stdout` or `stderr` ; if they do not do so, these streams will be closed upon program termination.

Closing any output stream requires flushing any data that has not yet been written to the stream. The flushing operation (manually handled by the `fflush()` function) can fail for several reasons. The output stream may be directed to a file in a filesystem with no remaining free space, or to a network socket that fails. Checking for the success of a `fflush()` operation is mandatory for a secure program, and hence checking the result of a `fclose()` operation is also required.

Consequently, any program that sends data to `stdout` or `stderr` must take care to flush the data before it terminates. Failing to flush the data (with a call to `fflush()` or `fclose()` ), may cause the data to fail to be written and become lost.

### Noncompliant Code Example

This noncompliant code example sends some data to standard output. If standard output is directed to a file and an error occurs while flushing the data (after program termination), then the output may be lost.

### Compliant Solution

This compliant solution explicitly closes `stdout` , and handles any errors that arise.

### Noncompliant Code Example ( `atexit()` )

This noncompliant code example closes standard output before exiting `main()` , but then tries to print to standard output in an exit handler.

### Compliant Solution ( `atexit()` )

This compliant solution uses `fflush()` instead of `fclose()` , and the exit handler also uses `fflush()` to flush its output:

### Exceptions

**FIO23-C-EX1:** Programs that do not send data to either output stream need not close them.  
**FIO23-C-EX2:** Programs that never run with buffered output streams need not close them.

## FIO24-C. Do not open a file that is already open

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/11.input-output-fio/21.fio24-c.md) — SHA-256 `6bcb2a6a3c91470da0b8178d2d390314c5931087ff4e9036df6cc00cecdc7d16`。

Opening a file that is already open has [implementation-defined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) , according to the C Standard, 7.21.3, paragraph 8 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \]:

> Functions that open additional (nontemporary) ﬁles require a file name, which is a string. The rules for composing valid ﬁle names are implementation-deﬁned. Whether the same file can be simultaneously open multiple times is also implementation-deﬁned.

Some implementations do not allow multiple copies of the same file to be open at the same time. Consequently, portable code cannot depend on what will happen if this rule is violated. Even on implementations that do not outright fail to open an already-opened file, a [TOCTOU](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-TOCTOU) (time-of-check, time-of-use) race condition exists in which the second open could operate on a different file from the first due to the file being moved or deleted (see [FIO45-C. Avoid TOCTOU race conditions while accessing files](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/input-output-fio/fio45-c) for more details on TOCTOU race conditions).

### Noncompliant Code Example

This noncompliant code example logs the program's state at runtime:

Because the file `log` is opened twice (once in `main()` and again in `do_stuff()` ), this program has [implementation-defined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) .

### Compliant Solution

In this compliant solution, a reference to the file pointer is passed as an argument to functions that need to perform operations on that file. This reference eliminates the need to open the same file multiple times.

## FIO30-C. Exclude user input from format strings

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/11.input-output-fio/02.fio30-c.md) — SHA-256 `a428e7bfb8158cac0eac0b0a2c40fbae9f77cd5a8d315371aa4a52e72f63bbf9`。

Never call a formatted I/O function with a format string containing a [tainted value](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-taintedvalue) .  An attacker who can fully or partially control the contents of a format string can crash a vulnerable process, view the contents of the stack, view memory content, or write to an arbitrary memory location. Consequently, the attacker can execute arbitrary code with the permissions of the vulnerable process \[ [Seacord 2013b](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Seacord2013) \]. Formatted output functions are particularly dangerous because many programmers are unaware of their capabilities. For example, formatted output functions can be used to write an integer value to a specified address using the `%n` conversion specifier.

### Noncompliant Code Example

The `incorrect_password()` function in this noncompliant code example is called during identification and authentication to display an error message if the specified user is not found or the password is incorrect. The function accepts the name of the user as a string referenced by `user` . This is an exemplar of [untrusted data](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-untrusteddata) that originates from an unauthenticated user. The function constructs an error message that is then output to `stderr` using the C Standard `fprintf()` function.

The `incorrect_password()` function calculates the size of the message, allocates dynamic storage, and then constructs the message in the allocated memory using the `snprintf()` function. The addition operations are not checked for integer overflow because the string referenced by `user` is known to have a length of 256 or less. Because the `%s` characters are replaced by the string referenced by `user` in the call to `snprintf()` , the resulting string needs 1 byte less than is allocated. The `snprintf()` function is commonly used for messages that are displayed in multiple locations or messages that are difficult to build. However, the resulting code contains a format-string [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) because the `msg` includes untrusted user input and is passed as the format-string argument in the call to `fprintf()` .

### Compliant Solution ( `fputs()` )

This compliant solution fixes the problem by replacing the `fprintf()` call with a call to `fputs()` , which outputs `msg` directly to `stderr` without evaluating its contents:

### Compliant Solution ( `fprintf()` )

This compliant solution passes the untrusted user input as one of the variadic arguments to `fprintf()` and not as part of the format string, eliminating the possibility of a format-string vulnerability:

### Noncompliant Code Example (POSIX)

This noncompliant code example is similar to the first noncompliant code example but uses the POSIX function `syslog()` \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \] instead of the `fprintf()` function. The `syslog()` function is also susceptible to format-string vulnerabilities.

The `syslog()` function first appeared in BSD 4.2 and is supported by Linux and other modern UNIX implementations. It is not available on Windows systems.

### Compliant Solution (POSIX)

This compliant solution passes the untrusted user input as one of the variadic arguments to `syslog()` instead of including it in the format string:

## FIO32-C. Do not perform operations on devices that are only appropriate for files

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/11.input-output-fio/03.fio32-c.md) — SHA-256 `5f1d28dff1b59d13e54b2002698f4d44743f74d4b2cbaecb2ef9c491c2b75317`。

File names on many operating systems, including Windows and UNIX, may be used to access *special files* , which are actually devices. Reserved Microsoft Windows device names include `AUX` , `CON` , `PRN` , `COM1` , and `LPT1` or paths using the `\\.\` device namespace. Device files on UNIX systems are used to apply access rights and to direct operations on the files to the appropriate device drivers.

Performing operations on device files that are intended for ordinary character or binary files can result in crashes and [denial-of-service attacks](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-denial-of-service) . For example, when Windows attempts to interpret the device name as a file resource, it performs an invalid resource access that usually results in a crash \[ [Howard 2002](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Howard02) \].

Device files in UNIX can be a security risk when an attacker can access them in an unauthorized way. For example, if attackers can read or write to the `/dev/kmem` device, they may be able to alter the priority, UID, or other attributes of their process or simply crash the system. Similarly, access to disk devices, tape devices, network devices, and terminals being used by other processes can lead to problems \[ [Garfinkel 1996](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Garfinkel96) \].

On Linux, it is possible to lock certain applications by attempting to open devices rather than files. Consider the following example:

A Web browser that failed to check for these devices would allow an attacker to create a website with image tags such as `   <IMG src="file:///dev/mouse">  ` that would lock the user's mouse \[ [Howard 2002](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Howard02) \].

### Noncompliant Code Example

In this noncompliant code example, the user can specify a locked device or a FIFO (first-in, first-out) file name, which can cause the program to hang on the call to `fopen()` :

### Compliant Solution (POSIX)

POSIX defines the `O_NONBLOCK` flag to `open()` , which ensures that delayed operations on a file do not hang the program \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \].

> When opening a FIFO with `O_RDONLY` or `O_WRONLY` set:
>
> -   If `O_NONBLOCK` is set, an `open()` for reading-only returns without delay. An `open()` for writing-only returns an error if no process currently has the file open for reading.
> -   If `O_NONBLOCK` is clear, an `open()` for reading-only blocks the calling thread until a thread opens the file for writing. An `open()` for writing-only blocks the calling thread until a thread opens the file for reading.
>
> When opening a block special or character special file that supports nonblocking opens:
>
> -   If `O_NONBLOCK` is set, the `open()` function returns without blocking for the device to be ready or available; subsequent behavior is device-specific.
> -   If `O_NONBLOCK` is clear, the `open()` function blocks the calling thread until the device is ready or available before returning.
>
> Otherwise, the behavior of `O_NONBLOCK` is unspecified.

Once the file is open, programmers can use the POSIX `lstat()` and `fstat()` functions to obtain information about a file and the `S_ISREG()` macro to determine if the file is a regular file.

Because the behavior of `O_NONBLOCK` on subsequent calls to `read()` or `write()` is [unspecified](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unspecifiedbehavior) , it is advisable to disable the flag after it has been determined that the file in question is not a special device.

When available (Linux 2.1.126+, FreeBSD, Solaris 10, POSIX.1-2008), the `O_NOFOLLOW` flag should also be used. (See [POS01-C. Check for the existence of links when dealing with files](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/posix-pos/pos01-c) .) When `O_NOFOLLOW` is not available, symbolic link checks should use the method from [POS35-C. Avoid race conditions while checking for the existence of a symbolic link](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/posix-pos/pos35-c) .

This code contains an intractable [TOCTOU (time-of-check, time-of-use)](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-TOCTOU) race condition under which an attacker can alter the file referenced by `file_name` following the call to `lstat()` but before the call to `open()` . The switch will be discovered after the file is opened, but opening the file cannot be prevented in the case where this action itself causes undesired behavior. (See [FIO45-C. Avoid TOCTOU race conditions while accessing files](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/input-output-fio/fio45-c) for more information about TOCTOU race conditions.)

Essentially, an attacker can switch out a file for one of the file types shown in the following table with the specified effect.

####### File Types and Effects

|                      |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Type                 | Note on Effect                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| Another regular file | The `fstat()` verification fails.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| FIFO                 | Either `open()` returns `-1` and sets `errno` to `ENXIO` , or `open()` succeeds and the `fstat()` verification fails.                                                                                                                                                                                                                                                                                                                                                                   |
| Symbolic link        | `open()` returns `-1` if `O_NOFOLLOW` is available; otherwise, the `fstat()` verification fails.                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Special device       | Usually the `fstat()` verification fails on `st_mode` . This can still be a problem if the device is one for which just opening (or closing) it causes a side effect. If `st_mode` compares equal, then the device is one that, after opening, appears to be a regular file. It would then fail the `fstat()` verification on `st_dev` and `st_ino` (unless it happens to be the *same* file, as can happen with `/dev/fd/*` on Solaris, but this would not be a problem). |

  

To be compliant with this rule and to prevent this TOCTOU race condition, `file_name` must refer to a file in a secure directory. (See [FIO15-C. Ensure that file operations are performed in a secure directory](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio15-c) .)

### Noncompliant Code Example (Windows)

This noncompliant code example uses the `GetFileType()` function to attempt to prevent opening a special file:

Although tempting, the Win32 `GetFileType()` function is dangerous in this case. If the file name given identifies a named pipe that is currently blocking on a read request, the call to `GetFileType()` will block until the read request completes. This provides an effective attack vector for a [denial-of-service attack](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-denial-of-service) on the application. Furthermore, the act of opening a file handle may cause side effects, such as line states being set to their default voltage when opening a serial device.

### Compliant Solution (Windows)

Microsoft documents a list of reserved identifiers that represent devices and have a device namespace to be used specifically by devices \[ [MSDN](https://msdn.microsoft.com/en-us/library/aa365247%28v=vs.85%29.aspx) \]. In this compliant solution, the `isReservedName()` function can be used to determine if a specified path refers to a device. Care must be taken to avoid a TOCTOU race condition when first testing a path name using the `isReservedName()` function and then later operating on that path name.  

### Exceptions

**FIO32-C-EX1:** These checks are intended for opening files specified by a user (or an untrusted source). Intentionally opening device files does not violate this rule.

## FIO34-C. Distinguish between characters read from a file and EOF or WEOF

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/11.input-output-fio/04.fio34-c.md) — SHA-256 `fa1b538114b90aa754e4eb81318aa7b40c74e58e291a22e94c3ba4e9730dfc95`。

The `EOF` macro represents a negative value that is used to indicate that the file is exhausted and no data remains when reading data from a file. `EOF` is an example of an [in-band error indicator](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-in-banderrorindicator) . In-band error indicators are problematic to work with, and the creation of new in-band-error indicators is discouraged by [ERR02-C. Avoid in-band error indicators](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err02-c) .

The byte I/O functions `fgetc()` , `getc()` , and `getchar()` all read a character from a stream and return it as an `int.` (See [STR00-C. Represent characters using an appropriate type](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/characters-and-strings-str/str00-c) .) If the stream is at the end of the file, the end-of-file indicator for the stream is set and the function returns `EOF` . If a read error occurs, the error indicator for the stream is set and the function returns `EOF` . If these functions succeed, they cast the character returned into an `   unsigned char  ` .

Because `EOF` is negative, it should not match any unsigned character value. However, this is only true for [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) where the `int` type is wider than `char` . On an implementation where `int` and `char` have the same width, a character-reading function can read and return a valid character that has the same bit-pattern as `EOF` . This could occur, for example, if an attacker inserted a value that looked like `EOF` into the file or data stream to alter the behavior of the program.

The C Standard requires only that the `int` type be able to represent a maximum value of +32767 and that a `char` type be no larger than an `int` . Although uncommon, this situation can result in the integer constant expression `EOF` being indistinguishable from a valid character; that is, `   (int)(unsigned char)65535 == -1  ` . Consequently, failing to use `feof()` and `ferror()` to detect end-of-file and file errors can result in incorrectly identifying the `EOF` character on rare implementations where `   sizeof(int) == sizeof(char)  ` .

This problem is much more common when reading wide characters. The `fgetwc()` , `getwc()` , and `getwchar()` functions return a value of type `wint_t` . This value can represent the next wide character read, or it can represent `WEOF` , which indicates end-of-file for wide character streams. On most implementations, the `wchar_t` type has the same width as `wint_t` , and these functions can return a character indistinguishable from `WEOF` .

In the UTF-16 character set, `0xFFFF` is guaranteed not to be a character, which allows `WEOF` to be represented as the value `-1` . Similarly, all UTF-32 characters are positive when viewed as a signed 32-bit integer. All widely used character sets are designed with at least one value that does not represent a character. Consequently, it would require a custom character set designed without consideration of the C programming language for this problem to occur with wide characters or with ordinary characters that are as wide as `int` .

The C Standard `feof()` and `ferror()` functions are not subject to the problems associated with character and integer sizes and should be used to verify end-of-file and file errors for susceptible implementations \[ [Kettlewell 2002](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Kettle02) \]. Calling both functions on each iteration of a loop adds significant overhead, so a good strategy is to temporarily trust `EOF` and `WEOF` within the loop but verify them with `feof()` and `ferror()` following the loop.

### Noncompliant Code Example

This noncompliant code example loops while the character `c` is not `EOF` :

Although `EOF` is guaranteed to be negative and distinct from the value of any unsigned character, it is not guaranteed to be different from any such value when converted to an `int` . Consequently, when `int` has the same width as `char` , this loop may terminate prematurely.

### Compliant Solution (Portable)

This compliant solution uses `feof()` and `ferror()` to test whether the `EOF` was an actual character or a real `EOF` because of end-of-file or errors:

### Noncompliant Code Example (Nonportable)

This noncompliant code example uses an assertion to ensure that the code is executed only on architectures where `int` is wider than `char` and `EOF` is guaranteed not to be a valid character value. However, this code example is noncompliant because the variable `c` is declared as a `char` rather than an `int` , making it possible for a valid character value to compare equal to the value of the `EOF` macro when `char` is signed because of sign extension:

Assuming that a `char` is a signed 8-bit type and an int is a 32-bit type, if `getchar()` returns the character value `'\xff` (decimal 255), it will be interpreted as `EOF` because this value is sign-extended to `0xFFFFFFFF` (the value of `EOF` ) to perform the comparison. (See [STR34-C. Cast characters to unsigned char before converting to larger integer sizes](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/characters-and-strings-str/str34-c) .)

### Compliant Solution (Nonportable)

This compliant solution declares `c` to be an `int` . Consequently, the loop will terminate only when the file is exhausted.

### Noncompliant Code Example (Wide Characters)

In this noncompliant example, the result of the call to the C standard library function `getwc()` is stored into a variable of type `wchar_t` and is subsequently compared with `WEOF` :

This code suffers from two problems. First, the value returned by `getwc()` is immediately converted to `wchar_t` before being compared with `WEOF` . Second, there is no check to ensure that `wint_t` is wider than `wchar_t` . Both of these problems make it possible for an attacker to terminate the loop prematurely by supplying the wide-character value matching `WEOF` in the file.

### Compliant Solution (Portable)

This compliant solution declares `wc` to be a `wint_t` to match the integer type returned by `getwc()` . Furthermore, it does not rely on `WEOF` to determine end-of-file definitively.

### Exceptions

**FIO34-C-EX1:** A number of C functions do not return characters but can return `EOF` as a status code. These functions include `fclose()` , `fflush()` , `fputs()` , `fscanf()` , `puts()` , `scanf()` , `sscanf()` , `vfscanf()` , and `vscanf()` . These return values can be compared to `EOF` without validating the result.

## FIO37-C. Do not assume that fgets() or fgetws() returns a nonempty string when successful

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/11.input-output-fio/05.fio37-c.md) — SHA-256 `58b216ccd7c1b6a1f77f3d7c27aac307570c3a3d00b018984cf3244c8af31633`。

Errors can occur when incorrect assumptions are made about the type of data being read. These assumptions may be violated, for example, when binary data has been read from a file instead of text from a user's terminal or the output of a process is piped to `stdin.` (See [FIO14-C. Understand the difference between text mode and binary mode with file streams](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio14-c) .) On some systems, it may also be possible to input a null byte (as well as other binary codes) from the keyboard.

Subclause 7.23.7.2 of the C Standard paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \] says,

> The fgets function returns s if successful. If end-of-file is encountered and no characters have been read into the array, the contents of the array remain unchanged and a null pointer is returned. If a read error occurs during the operation, the members of the array have unspecified values and a null pointer is returned.

The wide-character function `fgetws()` has the same behavior. Therefore, if `fgets()` or `fgetws()` returns a non-null pointer, it is safe to assume that the array contains data. However, it is erroneous to assume that the array contains a nonempty string because the data may contain null characters.

### Noncompliant Code Example

This noncompliant code example attempts to remove the trailing newline ( `\n` ) from an input line. The `fgets()` function is typically used to read a newline-terminated line of input from a stream. It takes a size parameter for the destination buffer and copies, at most, `   size - 1  ` characters from a stream to a character array.

The `strlen()` function computes the length of a string by determining the number of characters that precede the terminating null character. A problem occurs if the first character read from the input by `fgets()` happens to be a null character. This may occur, for example, if a binary data file is read by the `fgets()` call \[ [Lai 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Lai06) \]. If the first character in `buf` is a null character, `strlen(buf)` returns 0, the expression `   strlen(buf) - 1  ` wraps around to a large positive value, and a write-outside-array-bounds error occurs.

### Compliant Solution

This compliant solution uses `strchr()` to replace the newline character in the string if it exists:

## FIO38-C. Do not copy a FILE object

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/11.input-output-fio/06.fio38-c.md) — SHA-256 `d51887e88f616f9771dc30eb70d78c1d5aaca0fed94175d6148f58351feea8e5`。

According to the C Standard, 7.23.3, paragraph 6 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \],

> The address of the `FILE` object used to control a stream may be significant; a copy of a `FILE` object is not required to serve in place of the original.

Consequently, do not copy a `FILE` object.

### Noncompliant Code Example

This noncompliant code example can fail because a by-value copy of `stdout` is being used in the call to `fputs()` :

When compiled under Microsoft Visual Studio 2013 and run on Windows, this noncompliant example results in an "access violation" at runtime.

### Compliant Solution

In this compliant solution, a copy of the `stdout` pointer to the `FILE` object is used in the call to `fputs()` :

## FIO39-C. Do not alternately input and output from a stream without an intervening flush or positioning call

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/11.input-output-fio/07.fio39-c.md) — SHA-256 `e45cc8e2e6e9b50348616e6895ad657b1547801d377ccc03611e3b40205c6d9e`。

The C Standard, 7.23.5.3, paragraph 7 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], places the following restrictions on update streams:

> When a file is opened with update mode . . ., both input and output may be performed on the associated stream. However, output shall not be directly followed by input without an intervening call to the `fflush` function or to a file positioning function ( `fseek` , `fsetpos` , or `rewind` ), and input shall not be directly followed by output without an intervening call to a file positioning function, unless the input operation encounters end-of-file. Opening (or creating) a text file with update mode may instead open (or create) a binary stream in some implementations.

The following scenarios can result in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See [undefined behavior 156](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_156) .)

-   Receiving input from a stream directly following an output to that stream without an intervening call to `fflush()` , `fseek()` , `fsetpos()` , or `rewind()` if the file is not at end-of-file
-   Outputting to a stream after receiving input from that stream without a call to `fseek()` , `fsetpos()` , or `rewind()` if the file is not at end-of-file

Consequently, a call to `fseek()` , `fflush()` , or `fsetpos()` is necessary between input and output to the same stream. See [ERR07-C. Prefer functions that support error checking over equivalent functions that don't](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err07-c) for more information on why `fseek()` is preferred over `rewind()` .

### Noncompliant Code Example

This noncompliant code example appends data to a file and then reads from the same file:

Because there is no intervening flush or positioning call between the calls to `fread()` and `fwrite()` , the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

### Compliant Solution

In this compliant solution, `fseek()` is called between the output and input, eliminating the [undefined behavior 156](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_156) :

## FIO40-C. Reset strings on fgets() or fgetws() failure

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/11.input-output-fio/08.fio40-c.md) — SHA-256 `ce3ae99e17736de8b16347381a24aa15609fb563800804dd5ccebef888a7af23`。

If either of the C Standard `fgets()` or `fgetws()` functions fail, the contents of the array being written is [indeterminate](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) . (See [undefined behavior 175](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_175) .)  It is necessary to reset the string to a known value to avoid errors on subsequent string manipulation functions.

### Noncompliant Code Example

In this noncompliant code example, an error flag is set if `fgets()` fails. However, `buf` is not reset and has indeterminate contents when it is written to `stdout`:

### Compliant Solution

In this compliant solution, `buf` is set to an empty string if `fgets()` fails. The equivalent solution for `fgetws()` would set `buf` to an empty wide string.

### Exceptions

**FIO40-C-EX1:** If the string goes out of scope immediately following the call to `fgets()` or `fgetws()` or is not referenced in the case of a failure, it need not be reset.

## FIO41-C. Do not call getc(), putc(), getwc(), or putwc() with a stream argument that has side effects

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/11.input-output-fio/09.fio41-c.md) — SHA-256 `57ad2fd62924950dafb5239660102ecb66e73cfe052e23eae8b7953929d5ad24`。

Do not invoke `getc()` or `putc()` or their wide-character analogues `getwc()` and `putwc()` with a stream argument that has side effects. The stream argument passed to these macros may be evaluated more than once if these functions are implemented as unsafe macros. (See [PRE31-C. Avoid side effects in arguments to unsafe macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/preprocessor-pre/pre31-c) for more information.)

This rule does not apply to the character argument in `putc()` or the wide-character argument in `putwc()` , which is guaranteed to be evaluated exactly once.

### Noncompliant Code Example ( `getc()` )

This noncompliant code example calls the `getc()` function with an expression as the stream argument. If `getc()` is implemented as a macro, the file may be opened multiple times. (See [FIO24-C. Do not open a file that is already open](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio24-c) .)

This noncompliant code example also violates [ERR33-C. Detect and handle standard library errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err33-c) because the value returned by `fopen()` is not checked for errors.

### Compliant Solution ( `getc()` )

In this compliant solution, `fopen()` is called before `getc()` and its return value is checked for errors:

### Noncompliant Code Example ( `putc()` )

In this noncompliant example, `putc()` is called with an expression as the stream argument. If `putc()` is implemented as a macro, this expression might be evaluated multiple times.

This noncompliant code example might appear safe even if the `putc()` macro evaluates its stream argument multiple times, as the ternary conditional expression ostensibly prevents multiple calls to `fopen()` . However, the assignment to `fptr` and the evaluation of `fptr` as the controlling expression of the ternary conditional expression can take place between the same sequence points, resulting in [undefined behavior 34](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_34) (a violation of [EXP30-C. Do not depend on the order of evaluation for side effects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp30-c) ). This code also violates [ERR33-C. Detect and handle standard library errors](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err33-c) because it fails to check the return value from `fopen()` .

### Compliant Solution ( `putc()` )

In this compliant solution, the stream argument to `putc()` no longer has side effects:

The expression `c++` is perfectly safe because `putc()` guarantees to evaluate its character argument exactly once.

NOTE: The output of this compliant solution differs depending on the character set. For example, when run on a machine using an ASCII-derived code set such as ISO-8859 or Unicode, this solution will print out the 26 lowercase letters of the English alphabet. However, if run with an EBCDIC-based code set, such as Codepage 037 or Codepage 285, punctuation marks or symbols may be output between the letters.

## FIO42-C. Close files when they are no longer needed

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/11.input-output-fio/10.fio42-c.md) — SHA-256 `d071ffe46d8de1213fbde1094c616ee68acc7a6aef2899a40578228d02d7003a`。

A call to the `fopen()` or `freopen()` function must be matched with a call to `fclose()` before the lifetime of the last pointer that stores the return value of the call has ended or before normal program termination, whichever occurs first.

In general, this rule should also be applied to other functions with open and close resources, such as the POSIX `open()` and `close()` functions, or the Microsoft Windows `CreateFile()` and `CloseHandle()` functions.

### Noncompliant Code Example

This code example is noncompliant because the file opened by the call to `fopen()` is not closed before function `func()` returns:

### Compliant Solution

In this compliant solution, the file pointed to by `f` is closed before returning to the caller:

### Noncompliant Code Example ( `exit()` )

This code example is noncompliant because the resource allocated by the call to `fopen()` is not closed before the program terminates.  Although `exit()` closes the file, the program has no way of determining if an error occurs while flushing or closing the file.

### Compliant Solution ( `exit()` )

In this compliant solution, the program closes `f` explicitly before calling `exit()` , allowing any error that occurs when flushing or closing the file to be handled appropriately:

### Noncompliant Code Example (POSIX)

This code example is noncompliant because the resource allocated by the call to `open()` is not closed before function `func()` returns:

### Compliant Solution (POSIX)

In this compliant solution, `fd` is closed before returning to the caller:

### Noncompliant Code Example (Windows)

In this noncompliant code example, the file opened by the Microsoft Windows `CreateFile()` function is not closed before `func()` returns:

### Compliant Solution (Windows)

In this compliant solution, `hFile` is closed by invoking the `CloseHandle()` function before returning to the caller:

## FIO44-C. Only use values for fsetpos() that are returned from fgetpos()

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/11.input-output-fio/11.fio44-c.md) — SHA-256 `7e8ca72094f56d165cc42f5c710c30ac4a2a6f4b6ad5031c92340794a9258163`。

The C Standard, 7.23.9.3 paragraph 2 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], defines the following behavior for `fsetpos()` :

> The `fsetpos` function sets the `mbstate_t` object (if any) and file position indicator for the stream pointed to by `stream` according to the value of the object pointed to by `pos` , which shall be a value obtained from an earlier successful call to the `fgetpos` function on a stream associated with the same file.

Invoking the `fsetpos()` function with any other values for `pos` is [undefined behavior 181](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_181) .

### Noncompliant Code Example

This noncompliant code example attempts to read three values from a file and then set the file position pointer back to the beginning of the file:

Only the return value of an `fgetpos()` call is a valid argument to `fsetpos()` ; passing a value of type `fpos_t` that was created in any other way is [undefined behavior 181](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_181) .

### Compliant Solution

In this compliant solution, the initial file position indicator is stored by first calling `fgetpos()` , which is used to restore the state to the beginning of the file in the later call to `fsetpos()` :

## FIO45-C. Avoid TOCTOU race conditions while accessing files

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/11.input-output-fio/12.fio45-c.md) — SHA-256 `69df8e7c34f776b825a4524b32b3a52cbc9867f1f8446cb218a666f19246c522`。

A [TOCTOU](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-TOCTOU) (time-of-check, time-of-use) race condition is possible when two or more concurrent processes are operating on a shared file system \[ [Seacord 2013b](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Seacord2013) \]. Typically, the first access is a check to verify some attribute of the file, followed by a call to use the file. An attacker can alter the file between the two accesses, or replace the file with a symbolic or hard link to a different file. These TOCTOU conditions can be exploited when a program performs two or more file operations on the same file name or path name.

A program that performs two or more file operations on a single file name or path name creates a race window between the two file operations. This race window comes from the assumption that the file name or path name refers to the same resource both times. If an attacker can modify the file, remove it, or replace it with a different file, then this assumption will not hold.

### Noncompliant Code Example

If an existing file is opened for writing with the `w` mode argument, the file's previous contents (if any) are destroyed. This noncompliant code example tries to prevent an existing file from being overwritten by first opening it for reading before opening it for writing. An attacker can exploit the race window between the two calls to `fopen()` to overwrite an existing file.

### Compliant Solution

This compliant solution invokes `fopen()` at a single location and uses the `x` mode of `fopen()` , which was added in C11. This mode causes `fopen()` to fail if the file exists. This check and subsequent open is performed without creating a race window. The `x` mode provides exclusive access to the file only if the host environment provides this support.

### Compliant Solution (POSIX)

This compliant solution uses the `O_CREAT` and `O_EXCL` flags of POSIX's `open()` function. These flags cause `open()` to fail if the file exists.

### Exceptions

**FIO45-C-EX2:** Accessing a file name or path name multiple times is permitted if the file referenced resides in a secure directory. (For more information, see [FIO15-C. Ensure that file operations are performed in a secure directory](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/input-output-fio/fio15-c) .)

**FIO45-C-EX3:** Accessing a file name or path name multiple times is permitted if the program can verify that every operation operates on the same file.

This POSIX code example verifies that each subsequent file access operates on the same file. In POSIX, every file can be uniquely identified by using its device and i-node attributes. This code example checks that a file name refers to a regular file (and not a directory, symbolic link, or other special file) by invoking `lstat()` . This call also retrieves its device and i-node. The file is subsequently opened. Finally, the program verifies that the file that was opened is the same one (matching device and i-nodes) as the file that was confirmed as a regular file.

An attacker can still exploit this code if they have the ability to delete the benign file and create the malicious file within the race window between lstat() and open(). It is possible that the OS kernel will reuse the same device and i-node for both files. This can be mitigated by making sure that the attacker lacks the permissions to delete the benign file.

## FIO46-C. Do not access a closed file

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/11.input-output-fio/13.fio46-c.md) — SHA-256 `ace9a80d3404b706d57edfd42520f99e56e60f232e191f4216ffb6d75b335407`。

Using the value of a pointer to a `FILE` object after the associated file is closed is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See [undefined behavior 153](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_153) .) Programs that close the standard streams (especially `stdout` but also `stderr` and `stdin` ) must be careful not to use these streams in subsequent function calls, particularly those that implicitly operate on them (such as `printf()` , `perror()` , and `getchar()` ).

This rule can be generalized to other file representations.

### Noncompliant Code Example

In this noncompliant code example, the `stdout` stream is used after it is closed:

### Compliant Solution

In this compliant solution, `stdout` is not used again after it is closed. This must remain true for the remainder of the program, or `stdout` must be assigned the address of an open file object.

## FIO47-C. Use valid format strings

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/11.input-output-fio/14.fio47-c.md) — SHA-256 `02dc60fd0ce050d53bf214e718fbc24e36ad9a235ad4dbe05e881d4a2852b5bd`。

The formatted output functions ( `fprintf()` and related functions) convert, format, and print their arguments under control of a *format* string. The C Standard, 7.23.6.1, paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2024) \], specifies

> The format shall be a multibyte character sequence, beginning and ending in its initial shift state. The format is composed of zero or more directives: ordinary multibyte characters (not **%** ), which are copied unchanged to the output stream; and conversion specifications, each of which results in fetching zero or more subsequent arguments, converting them, if applicable, according to the corresponding conversion specifier, and then writing the result to the output stream.

Each *conversion specification* is introduced by the `%` character followed (in order) by

-   Zero or more *flags* (in any order), which modify the meaning of the conversion specification
-   An optional minimum field *width*
-   An optional *precision* that gives the minimum number of digits, the maximum number of digits, or the maximum number of bytes, etc. depending on the conversion specifier
-   An optional *length modifier* that specifies the size of the argument
-   A *conversion specifier character* that indicates the type of conversion to be applied

Common mistakes in creating format strings include

-   Providing an incorrect number of arguments for the format string
-   Using invalid conversion specifiers
-   Using a flag character that is incompatible with the conversion specifier
-   Using a length modifier that is incompatible with the conversion specifier
-   Mismatching the argument type and conversion specifier
-   Using an argument of type other than `int` for *width* or *precision*

The following table summarizes the compliance of various conversion specifications. The first column contains one or more conversion specifier characters. The next four columns consider the combination of the specifier characters with the various flags (the apostrophe \[ `'` \], `-` , `+` , the space character, `#` , and `0` ). The next eight columns consider the combination of the specifier characters with the various length modifiers ( `h` , `hh` , `l` , `ll` , `j` , `z` , `t` , and `L` ).

Valid combinations are marked with a type name; arguments matched with the conversion specification are interpreted as that type. For example, an argument matched with the specifier `%hd` is interpreted as a `short` , so `short` appears in the cell where `d` and `h` intersect. The last column denotes the expected types of arguments matched with the original specifier characters.

Valid and meaningful combinations are marked by the ✅ symbol (save for the length modifier columns, as described previously). Valid combinations that have no effect are labeled *N/E* . Using a combination marked by the ❌ symbol, using a specification not represented in the table, or using an argument of an unexpected type is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See undefined behaviors [158](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_158) , [160](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_1560) , [162](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_162) , [163](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_163) , [166](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_166) , and [167](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_167) .)

| Conversion <br> Specifier <br> Character | **`'`** <br> *<sup>XSI</sup>* | **`-`** <br> **+** <br> **SPACE** | **#** | 0 | `h` | `hh` | `l` | `ll` | `j` | `z` | `t` | `L` | Argument <br> Type |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `d`, `i` | ✅ | ✅ | ❌ | ✅ | `short` | `signed char` | `long` | `long long` | `intmax_t` | `size_t` | `ptrdiff_t` | ❌ | Signed integer |
| `o` | ❌ | ✅ | ✅ | ✅ | `unsigned short` | `unsigned char` | `unsigned long` | `unsigned long long` | `uintmax_t` | `size_t` | `ptrdiff_t` | ❌ | Unsigned integer |
| `u` | ✅ | ✅ | ❌ | ✅ | `unsigned short` | `unsigned char` | `unsigned long` | `unsigned long long` | `uintmax_t` | `size_t` | `ptrdiff_t` | ❌ | Unsigned integer |
| `x`, `X` | ❌ | ✅ | ✅ | ✅ | `unsigned short` | `unsigned char` | `unsigned long` | `unsigned long long` | `uintmax_t` | `size_t` | `ptrdiff_t` | ❌ | Unsigned integer |
| `f`, `F` | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | *N/E* | ❌ | ❌ | ❌ | ❌ | `long double` | `double` *or* `long double` |
| `e`, `E` | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ | *N/E* | ❌ | ❌ | ❌ | ❌ | `long double` | `double` *or* `long double` |
| `g`, `G` | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | *N/E* | ❌ | ❌ | ❌ | ❌ | `long double` | `double` *or* `long double` |
| `a`, `A` | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | *N/E* | ❌ | ❌ | ❌ | ❌ | `long double` | `double` *or* `long double` |
| `c` | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | `wint_t` | ❌ | ❌ | ❌ | ❌ | ❌ | `int` *or* `wint_t` |
| `s` | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | NTWS | ❌ | ❌ | ❌ | ❌ | ❌ | NTBS *or* NTWS |
| `p` | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | `void*` |
| `n` | ❌ | ❌ | ❌ | ❌ | `short*` | `char*` | `long*` | `long long*` | `intmax_t*` | `size_t*` | `ptrdiff_t*` | ❌ | Pointer to integer |
| `C` *<sup>XSI</sup>* | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | `wint_t` |
| `S` *<sup>XSI</sup>* | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | NTWS |
| `%` | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | *None* |

SPACE: The space ( `   " "  ` ) character
*N/E* : No effect
NTBS: `char*` argument pointing to a null-terminated character string
NTWS: `wchar_t*` argument pointing to a null-terminated wide character string
XSI: [ISO/IEC 9945-2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9945-2003) XSI extension

The formatted input functions ( `fscanf()` and related functions) use similarly specified format strings and impose similar restrictions on their format strings and arguments.

Do not supply an unknown or invalid conversion specification or an invalid combination of flag character, precision, length modifier, or conversion specifier to a formatted IO function. Likewise, do not provide a number or type of argument that does not match the argument type of the conversion specifier used in the format string.

Note that C23, section 7.16.1.1, paragraph 2 indicates that it is safe to specify a signed integer conversion specifier with an unsigned integer argument, or an unsigned specifier with a signed argument, as long as both types are compatible and the integer value is representable in both types.  In contrast, the `%n` specifier requires a pointer to signed int. Passing the `%n` specifier an unsigned int or any other type of integer is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

Format strings are usually string literals specified at the call site, but they need not be. However, they should not contain [tainted values](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-taintedvalue) . (See [FIO30-C. Exclude user input from format strings](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/input-output-fio/fio30-c) for more information.)

### Noncompliant Code Example

Mismatches between arguments and conversion specifications may result in [undefined behavior 163](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_163) . Compilers may diagnose type mismatches in formatted output function invocations. In this noncompliant code example, the `error_type` argument to `printf()` is incorrectly matched with the `s` specifier rather than with the `d` specifier. Likewise, the `error_msg` argument is incorrectly matched with the `d` specifier instead of the `s` specifier. These usages result in [undefined behavior 163](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_163) . One possible result of this invocation is that `printf()` will interpret the `error_type` argument as a pointer and try to read a string from the address that `error_type` contains, possibly resulting in an access violation.

### Compliant Solution

This compliant solution ensures that the arguments to the `printf()` function match their respective conversion specifications:
