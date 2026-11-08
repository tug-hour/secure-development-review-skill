# CERT C — CON

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## CON01-C. Acquire and release synchronization primitives in the same module, at the same level of abstraction

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/05.concurrency-con/02.con01-c.md) — SHA-256 `2dd6227cc8170e115a6e7e74cc19c310cfbe739e1db2400e1eef6b683d1b85ac`。

All locking and unlocking of mutexes should be performed in the same module and at the same level of abstraction. Failure to follow this recommendation can lead to some lock or unlock operations not being executed by the multithreaded program as designed, eventually resulting in deadlock, race conditions, or other security [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) , depending on the mutex type.

A common consequence of improper locking is for a mutex to be unlocked twice, via two calls to `mtx_unlock()` . This can cause the unlock operation to return errors. In the case of recursive mutexes, an error is returned only if the lock count is 0 (making the mutex available to other threads) and a call to `mtx_unlock()` is made.

### Noncompliant Code Example

In this noncompliant code example for a simplified multithreaded banking system, imagine an account with a required minimum balance. The code would need to verify that all debit transactions are allowable. Suppose a call is made to `debit()` asking to withdraw funds that would bring `account_balance` below `MIN_BALANCE` , which would result in two calls to `mtx_unlock()` . In this example, because the mutex is defined statically, the mutex type is [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) .

### Compliant Solution

This compliant solution unlocks the mutex only in the same module and at the same level of abstraction at which it is locked. This technique ensures that the code will not attempt to unlock the mutex twice.

## CON02-C. Do not use volatile as a synchronization primitive

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/05.concurrency-con/03.con02-c.md) — SHA-256 `afe59f1e921bb00a4496cd151cdf48569177378f450b6e67c700f09f1e4b87bb`。

The C Standard, subclause 5.1.2.3, paragraph 2 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], says:

> Accessing a volatile object, modifying an object, modifying a file, or calling a function that does any of those operations are all side effects, which are changes in the state of the execution environment. Evaluation of an expression in general includes both value computations and initiation of side effects. Value computation for an lvalue expression includes determining the identity of the designated object.

The `volatile` keyword informs the compiler that the qualified variable may change in ways that cannot be determined; consequently, compiler optimizations must be restricted for memory areas marked as `volatile` . For example, the compiler is forbidden to load the value into a register and subsequently reuse the loaded value rather than accessing memory directly. This concept relates to multithreading because incorrect caching of a shared variable may interfere with the propagation of modified values between threads, causing some threads to view stale data.

The `volatile` keyword is sometimes misunderstood to provide atomicity for variables that are shared between threads in a multithreaded program. Because the compiler is forbidden to either cache variables declared as `volatile` in registers or to reorder the sequence of reads and writes to any given volatile variable, many programmers mistakenly believe that volatile variables can correctly serve as synchronization primitives. Although the compiler is forbidden to reorder the sequence of reads and writes to a particular volatile variable, it may legally reorder these reads and writes with respect to reads and writes to *other* memory locations. This reordering alone is sufficient to make volatile variables unsuitable for use as synchronization primitives.

Further, the `volatile` qualifier lacks any guarantees regarding the following desired properties necessary for a multithreaded program:

-   **Atomicity:** Indivisible memory operations.
-   **Visibility:** The effects of a write action by a thread are visible to other threads.
-   **Ordering:** Sequences of memory operations by a thread are guaranteed to be seen in the same order by other threads.

The `volatile` qualifier lacks guarantees for any of these properties, both by definition and by the way it is implemented in various platforms. For more information on how `volatile` is implemented, consult [DCL17-C. Beware of miscompiled volatile-qualified variables](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl17-c) .

### Noncompliant Code Example

This noncompliant code example attempts to use `flag` as a synchronization primitive:

In this example, the value of `flag` is used to determine whether the critical section can be executed. Because the `flag` variable is not declared `volatile` , it may be cached in registers. Before the value in the register is written to memory, another thread might be scheduled to run, resulting in that thread reading stale data.

### Noncompliant Code Example

This noncompliant code example uses `flag` as a synchronization primitive but qualifies `flag` as a `volatile` type:

Declaring `flag` as `volatile` solves the problem of values being cached, which causes stale data to be read. However, `   volatile flag  ` still fails to provide the atomicity and visibility guarantees needed for synchronization primitives to work correctly. The `volatile` keyword does not promise to provide the guarantees needed for synchronization primitives.

### Compliant Solution

This code uses a mutex to protect critical sections:

### Compliant Solution (Critical Section, Windows)

This compliant solution uses a Microsoft Windows critical section object to make operations involving `account_balance` atomic \[ [MSDN](https://msdn.microsoft.com/en-us/library/windows/desktop/ms682530(v=vs.85).aspx) \].

## CON03-C. Ensure visibility when accessing shared variables

**Recommendation / 未確定**。原典がunder construction / incomplete / stub。記載済み条件を手掛かりとして残すが、完成した規範や適合認定に補作しない。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/05.concurrency-con/04.con03-c.md) — SHA-256 `878b972073cf9a183950f7a6a6d681f2619be627063cdbaac59f444dc828a845`。

Reading a shared primitive variable in one thread may not yield the value of the most recent write to the variable from another thread. Consequently, the thread may observe a stale value of the shared variable. To ensure the visibility of the most recent update, the write to the variable must *happen before* the read (C Standard, subclause 5.1.2.4, paragraph 18 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \]). Atomic operations—other than relaxed atomic operations—trivially satisfy the happens before relationship. Where atomic operations are inappropriate, protecting both reads and writes with a mutex also satisfies the happens before relationship.

  

### \*\*\*\*\*\*\*\*\*\*\* Text below this note not yet converted from Java to C! \*\*\*\*\*\*\*\*\*\*\*\*

### Noncompliant Code Example (Non-volatile Flag)

This noncompliant code example uses a `shutdown()` method to set the non-volatile `done` flag that is checked in the `run()` method.

If one thread invokes the `shutdown()` method to set the flag, a second thread might not observe that change. Consequently, the second thread might observe that `done` is still `false` and incorrectly invoke the `sleep()` method. Compilers and just-in-time compilers (JITs) are allowed to optimize the code when they determine that the value of `done` is never modified by the same thread, resulting in an infinite loop.

### Compliant Solution ( `Volatile` )

In this compliant solution, the `done` flag is declared volatile to ensure that writes are visible to other threads.

### Compliant Solution ( `AtomicBoolean` )

In this compliant solution, the `done` flag is declared to be of type `java.util.concurrent.atomic.AtomicBoolean` . Atomic types also guarantee that writes are visible to other threads.

### Compliant Solution ( `synchronized` )

This compliant solution uses the intrinsic lock of the `Class` object to ensure that updates are visible to other threads.

While this is an acceptable compliant solution, intrinsic locks cause threads to block and may introduce contention. On the other hand, volatile-qualified shared variables do not block. Excessive synchronization can also make the program prone to deadlock.

Synchronization is a more secure alternative in situations where the `volatile` keyword or a `java.util.concurrent.atomic.Atomic*` field is inappropriate, such as when a variable's new value depends on its current value. See rule [VNA02-J. Ensure that compound operations on shared variables are atomic](https://cmu-sei.github.io/secure-coding-standards/sei-cert-oracle-coding-standard-for-java/rules/visibility-and-atomicity-vna/vna02-j) for more information.

Compliance with rule [LCK00-J. Use private final lock objects to synchronize classes that may interact with untrusted code](https://cmu-sei.github.io/secure-coding-standards/sei-cert-oracle-coding-standard-for-java/rules/locking-lck/lck00-j) can reduce the likelihood of misuse by ensuring that untrusted callers cannot access the lock object.

## CON04-C. Join or detach threads even if their exit status is unimportant

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/05.concurrency-con/05.con04-c.md) — SHA-256 `22e3d8da2c3fb1d24f720955ff8dbf73c31466fe16a4d8bbb2f99efa6a28edd9`。

The `thrd_detach()` function is used to tell the underlying system that resources allocated to a particular thread can be reclaimed once it terminates. This function should be used when a thread's exit status is not required by other threads (and no other thread needs to use `thrd_join()` to wait for it to complete).

Whenever a thread terminates without detaching, the thread's stack is deallocated, but some other resources, including the thread ID and exit status, are left until it is destroyed by either `thrd_join()` or `thrd_detach()` . These resources can be vital for systems with limited resources and can lead to various "resource unavailable" errors, depending on which critical resource gets used up first. For example, if the system has a limit (either per-process or system wide) on the number of thread IDs it can keep track of, failure to release the thread ID of a terminated thread may lead to `   thrd_create() being unable to  ` create another thread.

### Noncompliant Code Example

This noncompliant code example shows a pool of threads that are not exited correctly:

### Compliant Solution

In this compliant solution, the `message_print()` function is replaced by a similar function that correctly detaches the threads so that the associated resources can be reclaimed on exit:

## CON05-C. Do not perform operations that can block while holding a lock

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/05.concurrency-con/06.con05-c.md) — SHA-256 `f2dcf60011b8d25fb166260ebef16c6b8d85de00a83efe127fac4596ca7f6e86`。

If a lock is being held and an operation that can block is performed, any other thread that needs to acquire that lock may also block. This condition can degrade system performance or cause a deadlock to occur.  Blocking calls include, but are not limited to, network, file, and console I/O.

Using a blocking operation while holding a lock may be unavoidable for a portable solution. For instance, file access could be protected via a lock to prevent multiple threads from mutating the contents of the file. Or, a thread may be required to block while holding one or more locks and waiting to acquire another lock. In these cases, attempt to hold the lock for the least time required. Additionally, if acquiring multiple locks, the order of locking must avoid deadlock, as specified in [CON35-C. Avoid deadlock by locking in a predefined order](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/concurrency-con/con35-c) .

### Noncompliant Code Example

This noncompliant example calls `fopen()` while a mutex is locked. The calls to `fopen()` and `fclose()` are blocking and may block for an extended period of time if the file resides on a network drive. While the call is blocked, other threads that are waiting for the lock are also blocked.

### Compliant Solution (Block while Not Locked)

This compliant solution performs the file operations when the lock has not been acquired. The blocking behavior consequently affects only the thread that called the blocking function.

## CON06-C. Ensure that every mutex outlives the data it protects

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/05.concurrency-con/07.con06-c.md) — SHA-256 `c534eb9bd854e869e0e77acbf4064587768d6e274baf910d6f7d9ff760030baa`。

  

### \*\*\*\*\*\*\*\*\*\* Text below this note not yet converted from Java to C! \*\*\*\*\*\*\*\*\*\*\*\*

Programs must not use instance locks to protect static shared data because instance locks are ineffective when two or more instances of the class are created. Consequently, failure to use a static lock object leaves the shared state unprotected against concurrent access. Lock objects for classes that can interact with untrusted code must also be private and final, as shown in rule [LCK00-J. Use private final lock objects to synchronize classes that may interact with untrusted code](https://cmu-sei.github.io/secure-coding-standards/sei-cert-oracle-coding-standard-for-java/rules/locking-lck/lck00-j) .

### Noncompliant Code Example (Nonstatic Lock Object for Static Data)

This noncompliant code example attempts to guard access to the static `counter` field using a non-static lock object. When two `Runnable` tasks are started, they create two instances of the lock object and lock on each instance separately.

<div class="syntaxhighlighter nogutter java">

  

  

</div>

This example fails to prevent either thread from observing an inconsistent value of `counter` because the increment operation on volatile fields fails to be atomic in the absence of proper synchronization. (See rule [VNA02-J. Ensure that compound operations on shared variables are atomic](https://cmu-sei.github.io/secure-coding-standards/sei-cert-oracle-coding-standard-for-java/rules/visibility-and-atomicity-vna/vna02-j) for more information.)

### Noncompliant Code Example (Method Synchronization for Static Data)

This noncompliant code example uses method synchronization to protect access to a static class `counter` field.

<div class="syntaxhighlighter nogutter java">

  

  

</div>

In this case, the method synchronization uses the intrinsic lock associated with each instance of the class rather than the intrinsic lock associated with the class itself. Consequently, threads constructed using different `Runnable` instances may observe inconsistent values of `counter` .

### Compliant Solution (Static Lock Object)

This compliant solution ensures the atomicity of the increment operation by locking on a static object.

<div class="syntaxhighlighter nogutter java">

  

  

</div>

It is unnecessary to declare the `counter` variable volatile when using synchronization.

## CON07-C. Ensure that compound operations on shared variables are atomic

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/05.concurrency-con/08.con07-c.md) — SHA-256 `938b3613677997e6121794ba9cf80225a5ca48c60da0eb282ac7cfc64a9cce5e`。

Compound operations are operations that consist of more than one discrete operation. Expressions that include postfix or prefix increment ( `++` ), postfix or prefix decrement ( `--` ), or compound assignment operators always result in compound operations. Compound assignment expressions use operators such as `*=` , `/=` , `%=` , `+=` , `-=` , `<<=` , `>>=` , `^=` , and `|=` . Compound operations on shared variables must be performed atomically to prevent [data races](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-datarace) .

### Noncompliant Code Example (Logical Negation)

This noncompliant code example declares a shared `_Bool``  flag` variable and provides a `  toggle_flag()` method that negates the current value of `  flag` :  

Execution of this code may result in a [data race](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-datarace) because the value of `flag` is read, negated, and written back.

Consider, for example, two threads that call `toggle_flag()` . The expected effect of toggling `flag` twice is that it is restored to its original value. However, the following scenario leaves `flag` in the incorrect state:

|      |                      |                  |                                                                                                       |
|------|----------------------|------------------|-------------------------------------------------------------------------------------------------------|
| Time | `flag=` | Thread           | Action                                                                                                |
| 1    | `true`  | *t* <sub>1</sub> | Reads the current value of `flag` , `true` , into a cache                   |
| 2    | `true`  | *t* <sub>2</sub> | Reads the current value of `flag` , (still) `true` , into a different cache |
| 3    | `true`  | *t* <sub>1</sub> | Toggles the temporary variable in the cache to `false`                                   |
| 4    | `true`  | *t* <sub>2</sub> | Toggles the temporary variable in the different cache to `false`                         |
| 5    | `false` | *t* <sub>1</sub> | Writes the cache variable's value to `flag`                                              |
| 6    | `false` | *t* <sub>2</sub> | Writes the different cache variable's value to `flag`                                    |

As a result, the effect of the call by *t* <sub>2</sub> is not reflected in `flag` ; the program behaves as if `toggle_flag()` was called only once, not twice.

### Compliant Solution (Mutex)

This compliant solution restricts access to `flag` under a mutex lock:

This solution guards reads and writes to the `flag` field with a lock on the `flag_mutex` . This lock ensures that changes to `flag` are visible to all threads. Now, only two execution orders are possible. In one execution order, t <sub>1</sub> obtains the mutex and completes the operation before *t* <sub>2</sub> can acquire the mutex, as shown here:

|      |                      |                  |                                                                                                         |
|------|----------------------|------------------|---------------------------------------------------------------------------------------------------------|
| Time | `flag=` | Thread           | Action                                                                                                  |
| 1    | `true`  | *t* <sub>1</sub> | Reads the current value of `flag` , `true` , into a cache variable            |
| 2    | `true`  | *t* <sub>1</sub> | Toggles the cache variable to `false`                                                      |
| 3    | `false` | *t* <sub>1</sub> | Writes the cache variable's value to `flag`                                                |
| 4    | `false` | *t* <sub>2</sub> | Reads the current value of `flag` , `false` , into a different cache variable |
| 5    | `false` | *t* <sub>2</sub> | Toggles the different cache variable to `true`                                             |
| 6    | `true`  | *t* <sub>2</sub> | Writes the different cache variable's value to `flag`                                      |

The other execution order is similar, except that *t* <sub>2</sub> starts and finishes before *t* <sub>1</sub> .

### Compliant Solution ( `atomic_compare_exchange_weak()` )

This compliant solution uses atomic variables and a compare-and-exchange operation to guarantee that the correct value is stored in `flag` . All updates are visible to other threads.

An alternative solution is to use the `atomic_flag` data type for managing Boolean values atomically.

### Noncompliant Code Example (Addition of Primitives)

In this noncompliant code example, multiple threads can invoke the `set_values()` method to set the `a` and `b` fields. Because this code fails to test for integer overflow, users of this code must also ensure that the arguments to the `set_values()` method can be added without overflow (see [INT32-C. Ensure that operations on signed integers do not result in overflow](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int32-c) for more information).

The `get_sum()` method contains a race condition. For example, when `a` and `b` currently have the values `0` and `INT_MAX` , respectively, and one thread calls `get_sum()` while another calls `   set_values(INT_MAX, 0)  ` , the `get_sum()` method might return either `0` or `INT_MAX` , or it might overflow. Overflow will occur when the first thread reads `a` and `b` after the second thread has set the value of `a` to `INT_MAX` but before it has set the value of `b` to `0` .

  

Noncompliant Code Example (Addition of Atomic Integers)

In this noncompliant code example, `a` and `b` are replaced with atomic integers.

  

The simple replacement of the two `  int` fields with atomic integers fails to eliminate the race condition in the sum because the compound operation `  a.get() + b.get()` is still non-atomic. While a sum of some value of `  a` and some value of `  b` will be returned, there is no guarantee that this value represents the sum of the values of `  a`and`  b` at any particular moment.

### Compliant Solution ( `   _Atomic struct  ` )

This compliant solution uses an atomic struct, which guarantees that both numbers are read and written together.

On most modern platforms, this will compile to be lock-free.

### Compliant Solution (Mutex)

This compliant solution protects the `set_values()` and `get_sum()` methods with a mutex to ensure atomicity:

Thanks to the mutex, it is now possible to add overflow checking to the `get_sum()` function without introducing the possibility of a race condition.

## CON08-C. Do not assume that a group of calls to independently atomic methods is atomic

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/05.concurrency-con/09.con08-c.md) — SHA-256 `a73c717064f4461c03a61eb882a95ef404a8659ceb80b2f09018577f5ba6906a`。

<div class="page view">

A consistent locking policy guarantees that multiple threads cannot simultaneously access or modify shared data. When two or more operations must be performed as a single atomic operation, a consistent locking policy must be implemented using some form of locking, such as a mutex. In the absence of such a policy, the code is susceptible to race conditions.

When presented with a set of operations, where each is guaranteed to be atomic, it is tempting to assume that a single operation consisting of individually-atomic operations is guaranteed to be collectively atomic without additional locking. A grouping of calls to such methods requires additional synchronization for the group.

Compound operations on shared variables are also non-atomic. See [CON07-C. Ensure that compound operations on shared variables are atomic](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/concurrency-con/con07-c) for more information.

### Noncompliant Code Example

This noncompliant code example stores two integers atomically. It also provides atomic methods to obtain their sum and product. All methods are locked with the same mutex to provide their atomicity.

Unfortunately, the `multiply_monomials()` function is still subject to race conditions, despite relying exclusively on atomic function calls. It is quite possible for `get_sum()` and `get_product()` to work with different numbers than the ones that were set by `set_values()` . It is even possible for `get_sum()` to operate with different numbers than `get_product()` .

### Compliant Solution

This compliant solution locks the `multiply_monomials()` function with the same mutex lock that is used by the other functions. For this code to work, the mutex must be recursive. This is accomplished by making it recursive in the `init_mutex()` function.

### Noncompliant Code Example

Function chaining is a useful design pattern for building an object and setting its optional fields. The output of one function serves as an argument (typically the last) in the next function. However, if accessed concurrently, a thread may observe shared fields to contain inconsistent values. This noncompliant code example demonstrates a race condition that can occur when multiple threads can variables with no thread protection.

In this noncompliant code example, the program constructs a currency `struct` and starts two threads that use method chaining to set the optional values of the structure. This example code might result in the currency `struct` being left in an inconsistent state, for example, with two quarters and one dime or one quarter and two dimes.

### Noncompliant Code Example

This code remains unsafe even if it uses a mutex on the `set` functions to guard modification of the currency:

### Compliant Solution

This compliant solution uses a mutex, but instead of guarding the `set` functions, it guards the `init` functions, which are invoked at thread creation.

:: Consequently this compliant solution is thread-safe, and will always print out the same number of quarters as dimes.

## CON09-C. Avoid the ABA problem when using lock-free algorithms

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/05.concurrency-con/10.con09-c.md) — SHA-256 `b3de40b3184a75c1541f289e7e5e669e7751eca2f75d62df66f291fcf9a638cb`。

Lock-free programming is a technique that allows concurrent updates of shared data structures without using explicit locks. This method ensures that no threads block for arbitrarily long times, and it thereby boosts performance.

Lock-free programming has the following advantages:

-   Can be used in places where locks must be avoided, such as interrupt handlers
-   Efficiency benefits compared to lock-based algorithms for some workloads, including potential scalability benefits on multiprocessor machines
-   Avoidance of priority inversion in real-time systems

Lock-free programming requires the use of special atomic processor instructions, such as CAS (compare and swap), LL/SC (load linked/store conditional), or the C Standard `atomic_compare_exchange` generic functions.

Applications for lock-free programming include

-   Read-copy-update€? (RCU) in Linux 2.5 kernel
-   Lock-free programming on AMD multicore systems

The *ABA problem* occurs during synchronization: a memory location is read twice and has the same value for both reads. However, another thread has modified the value, performed other work, then modified the value back between the two reads, thereby tricking the first thread into thinking that the value never changed.

### Noncompliant Code Example

This noncompliant code example attempts to zero the maximum element of an array. The example is assumed to run in a multithreaded environment, where all variables are accessed by other threads.

The compare-and-swap operation sets `array[index]` to 0 if and only if it is currently set to `value` . However, this code does not necessarily zero out the maximum value of the array because

-   `index` may have changed.
-   `value` may have changed (that is, the value of the `value` variable).
-   `value` may no longer be the maximum value in the array.

### Compliant Solution (Mutex)

This compliant solution uses a mutex to prevent the data from being modified during the operation. Although this code is thread-safe, it is no longer lock-free.

### Noncompliant Code Example (GNU Glib)

This code implements a queue data structure using lock-free programming. It is implemented using glib. The function `CAS()` internally uses `g_atomic_pointer_compare_and_exchange()` .

Assume there are two threads ( `T1` and `T2` ) operating simultaneously on the queue. The queue looks like this:

> `    head -> A -> B -> C -> tail   `

The following sequence of operations occurs:

| Thread | Queue Before | Operation | Queue After |
| --- | --- | --- | --- |
| `T1` | `head -> A -> B -> C -> tail` | Enters `queue_dequeue()` function<br /><br />`head = A, tail = C`<br /><br />`next = B`<br /><br />after executing `data = next->data;`<br /><br />This thread gets preempted | `head -> A -> B -> C -> tail` |
| `T2` | `head -> A -> B -> C -> tail` | Removes node A | `head -> B -> C -> tail` |
| `T2` | `head -> B -> C -> tail` | Removes node B | `head -> C -> tail` |
| `T2` | `head -> C -> tail` | Enqueues node A back into the queue | `head -> C -> A -> tail` |
| `T2` | `head -> C -> A -> tail` | Removes node C | `head -> A -> tail` |
| `T2` | `head -> A -> tail` | Enqueues a new node D<br /><br />After enqueue operation, thread 2 gets preempted | `head -> A -> D -> tail` |
| `T1` | `head -> A -> D -> tail` | Thread 1 starts execution<br /><br />Compares the local head = `q->head` = A (true in this case)<br /><br />Updates `q->head` with node B (but node B is removed) | **undefined {}** |

According to the sequence of events in this table, `head` will now point to memory that was freed. Also, if reclaimed memory is returned to the operating system (for example, using `munmap()` ), access to such memory locations can result in fatal access violation errors. The ABA problem occurred because of the *internal reuse of nodes that have been popped off the list* or the *reclamation of memory occupied by removed nodes* .

### Compliant Solution (GNU Glib, Hazard Pointers)

According to \[ [Michael 2004](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Michael2004) \], t he core idea is to associate a number (typically one or two) of single-writer, multi-reader shared pointers, called hazard pointers, with each thread that intends to access lock-free dynamic objects. A hazard pointer either has a null value or points to a node that may be accessed later by that thread without further validation that the reference to the node is still valid. Each hazard pointer may be written only by its owner thread but may be read by other threads.

In this solution, communication with the associated algorithms is accomplished only through hazard pointers and a procedure `RetireNode()` that is called by threads to pass the addresses of retired nodes.

###### **PSEUDOCODE**

The scan consists of two stages. The first stage involves scanning the hazard pointer list for non-null values. Whenever a non-null value is encountered, it is inserted in a local list, `plist` , which can be implemented as a hash table. The second stage involves checking each node in `rlist` against the pointers in `plist` . If the lookup yields no match, the node is identified to be ready for arbitrary reuse. Otherwise, it is retained in `rlist` until the next scan by the current thread. Insertion and lookup in `plist` take constant expected time. The task of the memory reclamation method is to determine when a retired node is safely eligible for reuse while allowing memory reclamation.

In the [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) , the pointer being removed is stored in the hazard pointer, preventing other threads from reusing it and thereby avoiding the ABA problem.

###### **CODE**

### Compliant Solution (GNU Glib, Mutex)

In this compliant solution, `mtx_lock()` is used to lock the queue. When thread 1 locks on the queue to perform any operation, thread 2 cannot perform any operation on the queue, which prevents the ABA problem.

## CON30-C. Clean up thread-specific storage

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/05.concurrency-con/02.con30-c.md) — SHA-256 `12a9e821c4d1aa08ca31e0d1e89e2fc64090ee71b5295b81f3c240643755b227`。

The `tss_create()` function creates a thread-specific storage pointer identified by a key. Threads can allocate thread-specific storage and associate the storage with a key that uniquely identifies the storage by calling the `tss_set()` function. If not properly freed, this memory may be leaked. Ensure that thread-specific storage is freed.

### Noncompliant Code Example

In this noncompliant code example, each thread dynamically allocates storage in the `get_data()` function, which is then associated with the global key by the call to `tss_set()` in the `add_data()` function. This memory is subsequently leaked when the threads terminate.

### Compliant Solution

In this compliant solution, each thread explicitly frees the thread-specific storage returned by the `tss_get()` function before terminating:

### Compliant Solution

This compliant solution invokes a destructor function registered during the call to `tss_create()` to automatically free any thread-specific storage:

## CON31-C. Do not destroy a mutex while it is locked

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/05.concurrency-con/03.con31-c.md) — SHA-256 `5b16dd23ced23fc494df53548c8dd5cbd752b24e755a05ae71d23c8f3a4eb252`。

Mutexes are used to protect shared data structures being concurrently accessed. If a mutex is destroyed while a thread is blocked waiting for that mutex, [critical sections](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-criticalsections) and shared data are no longer protected.

The C Standard, 7.28.4.1, paragraph 2 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> The `mtx_destroy` function releases any resources used by the mutex pointed to by `mtx` . No threads can be blocked waiting for the mutex pointed to by `mtx` .

This statement implies that destroying a mutex while a thread is waiting on it is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

### Noncompliant Code Example

This noncompliant code example creates several threads that each invoke the `do_work()` function, passing a unique number as an ID. The `do_work()` function initializes the `lock` mutex if the argument is 0 and destroys the mutex if the argument is `   max_threads - 1  ` . In all other cases, the `do_work()` function provides normal processing. Each thread, except the final cleanup thread, increments the atomic `completed` variable when it is finished.

Unfortunately, this code contains several race conditions, allowing the mutex to be destroyed before it is unlocked. Additionally, there is no guarantee that `lock` will be initialized before it is passed to `mtx_lock()` . Each of these behaviors is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

### Compliant Solution

This compliant solution eliminates the race conditions by initializing the mutex in `main()` before creating the threads and by destroying the mutex in `main()` after joining the threads:

## CON32-C. Prevent data races when accessing bit-fields from multiple threads

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/05.concurrency-con/04.con32-c.md) — SHA-256 `d29af0253fda1d1fd92165a3367f3d1e88337547b0845d69a1b8c53e367462c8`。

When accessing a bit-field, a thread may inadvertently access a separate bit-field in adjacent memory. This is because compilers are required to store multiple adjacent bit-fields in one storage unit whenever they fit. Consequently, data races may exist not just on a bit-field accessed by multiple threads but also on other bit-fields sharing the same byte or word.  A similar problem is discussed in [CON43-C. Do not allow data races in multithreaded code](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/concurrency-con/con43-c) , but the issue described by this rule can be harder to diagnose because it may not be obvious that the same memory location is being modified by multiple threads.

One approach for preventing data races in concurrent programming is to use a mutex. When properly observed by all threads, a mutex can provide safe and secure access to a shared object. However, mutexes provide no guarantees with regard to other objects that might be accessed when the mutex is not controlled by the accessing thread. Unfortunately, there is no portable way to determine which adjacent bit-fields may be stored along with the desired bit-field.

Another approach is to insert a non-bit-field member between any two bit-fields to ensure that each bit-field is the only one accessed within its storage unit. This technique effectively guarantees that no two bit-fields are accessed simultaneously.

### Noncompliant Code Example (Bit-field)

Adjacent bit-fields may be stored in a single memory location. Consequently, modifying adjacent bit-fields in different threads is [undefined behavior 5](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_5) , as shown in this noncompliant code example:

The C Standard, 3.17, paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> Note 2 to entry: A bit-field and an adjacent non-bit-field member are in separate memory locations. The same applies to two bit-fields, if one is declared inside a nested structure declaration and the other is not, or if the two are separated by a zero-length bit-field declaration, or if they are separated by a non-bit-field member declaration. It is not safe to concurrently update two non-atomic bit-fields in the same structure if all members declared between them are also (nonzero-length) bit-fields, no matter what the sizes of those intervening bit-fields happen to be.

For example, the following instruction sequence is possible:

### Compliant Solution (Bit-field, C11, Mutex)

This compliant solution protects all accesses of the flags with a mutex, thereby preventing any data races:

### Compliant Solution (C11)

In this compliant solution, two threads simultaneously modify two distinct non-bit-field members of a structure. Because the members occupy different bytes in memory, no concurrency protection is required.

Unlike C99, C11 and C23 explicitly define a memory location and provides the following note in subclause 3.17 paragraph 2 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \]:

> Note 1 to entry: Two threads of execution can update and access separate memory locations without interfering with each other.

It is almost certain that `flag1` and `flag2` are stored in the same word. Using a compiler that conforms to C99 or earlier, if both assignments occur on a thread-scheduling interleaving that ends with both stores occurring after one another, it is possible that only one of the flags will be set as intended. The other flag will contain its previous value because both members are represented by the same word, which is the smallest unit the processor can work on. Before the changes were made to the C Standard for C11, there were no guarantees that these flags could be modified concurrently.

## CON33-C. Avoid race conditions when using library functions

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/05.concurrency-con/05.con33-c.md) — SHA-256 `2b7d872841f92d1e2c2161dcdc39f5959131e15dccf5c986acf9a90e8a135513`。

Some C standard library functions are not guaranteed to be [reentrant](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-reentrant) with respect to threads. Functions such as `strtok()` and `asctime()` return a pointer to the result stored in function-allocated memory on a per-process basis. Other functions such as `rand()` store state information in function-allocated memory on a per-process basis. Multiple threads invoking the same function can cause concurrency problems, which often result in abnormal behavior and can cause more serious [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) , such as [abnormal termination](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) , [denial-of-service attack](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-denial-of-service) , and data integrity violations.

According to the C Standard, the library functions listed in the following table may contain data races when invoked by multiple threads.

| Functions | Remediation |
| --- | --- |
| ` rand() ` , ` srand() ` | [MSC30-C. Do not use the rand() function for generating pseudorandom numbers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/miscellaneous-msc/msc30-c) |
| ` getenv() ` | [ENV34-C. Do not store pointers returned by certain functions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/environment-env/env34-c) |
| ` strtok() ` | ` strtok_r() ` in POSIX |
| ` strerror() ` | ` strerror_r() ` in POSIX |
| ` asctime() ` , ` ctime() ` , <br> ` localtime() ` , ` gmtime() ` | ` strftime() ` |
| ` setlocale() ` | Protect multithreaded access to locale-specific functions with a mutex |
| ` ATOMIC_VAR_INIT ` , ` atomic_init() ` | Do not attempt to initialize an atomic variable from multiple threads |
| ` tmpnam() ` | ` tmpnam_r() ` in POSIX |
| ` mbrtoc16() ` , ` c16rtomb() ` , <br> ` mbrtoc32() ` , ` c32rtomb() ` | Do not call with a null ` mbstate_t * ` argument |

Section 2.9.1 of the *Portable Operating System Interface (POSIX <sup>®</sup> ), Base Specifications, Issue 7* \[ [IEEE Std 1003.1:2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \] extends the list of functions that are not required to be thread-safe.

### Noncompliant Code Example

In this noncompliant code example, the function `f()` is called from within a multithreaded application but encounters an error while calling a system function. The `strerror()` function returns a human-readable error string given an error number.

The C Standard, 7.26.6.3 paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], specifically states that `strerror()` is not required to avoid data races.

> The strerror function is not required to avoid data races with other calls to the strerror function.

An [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) could write the error string into a static array and return a pointer to it, and that array might be accessible and modifiable by other threads.

This code first sets `errno` to 0 to comply with [ERR30-C. Take care when reading errno](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err30-c) .

### Compliant Solution (POSIX, `strerror_r()` )

This compliant solution uses the POSIX `strerror_r()` function, which has the same functionality as `strerror()` but guarantees thread safety:

Linux provides two versions of `strerror_r()` , known as the *XSI-compliant version* and the *GNU-specific version* . This compliant solution assumes the XSI-compliant version, which is the default when an application is compiled as required by POSIX (that is, by defining `_POSIX_C_SOURCE` or `_XOPEN_SOURCE` appropriately). The `strerror_r()` manual page lists versions that are available on a particular system.

## CON34-C. Declare objects shared between threads with appropriate storage durations

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/05.concurrency-con/06.con34-c.md) — SHA-256 `c1e932120468f72da66c22dd8ffdb9a84c481f9505d2869e576899786f5ab91b`。

Accessing the automatic or thread-local variables of one thread from another thread is [implementation-defined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) and can cause invalid memory accesses because the execution of threads can be interwoven within the constraints of the synchronization model. As a result, the referenced stack frame or thread-local variable may no longer be valid when another thread tries to access it. Shared static variables can be protected by thread synchronization mechanisms.

However, automatic (local) variables cannot be shared in the same manner because the referenced stack frame's thread would need to stop executing, or some other mechanism must be employed to ensure that the referenced stack frame is still valid. Do not access automatic or thread-local objects from a thread other than the one with which the object is associated. See [DCL30-C. Declare objects with appropriate storage durations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl30-c) for information on how to declare objects with appropriate storage durations when data is not being shared between threads.

  

### Noncompliant Code Example (Automatic Storage Duration)

This noncompliant code example passes the address of a variable to a child thread, which prints it out. The variable has automatic storage duration. Depending on the execution order, the child thread might reference the variable after the variable's lifetime in the parent thread. This would cause the child thread to access an invalid memory location.

### Noncompliant Code Example (Automatic Storage Duration)

One practice is to ensure that all objects with automatic storage duration shared between threads are declared such that their lifetime extends past the lifetime of the threads. This can be accomplished using a thread synchronization mechanism, such as `thrd_join()` . In this code example, `val` is declared in `main()` , where `thrd_join()` is called. Because the parent thread waits until the child thread completes before continuing its execution, the shared objects have a lifetime at least as great as the thread.

### 

However, the C Standard, 6.2.4 paragraphs 4 and 5 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2024) \], states:

> The result of attempting to indirectly access an object with thread storage duration from a thread other than the one with which the object is associated is implementation-defined. . . .
>
> The result of attempting to indirectly access an object with automatic storage duration from a thread other than the one with which the object is associated is implementation-defined.

Therefore this example relies on [implementation-defined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) and is nonportable.

### Compliant Solution (Static Storage Duration)

This compliant solution stores the value in an object having static storage duration. The lifetime of this object is the entire execution of the program; consequently, it can be safely accessed by any thread.

### Compliant Solution (Allocated Storage Duration)

This compliant solution stores the value passed to the child thread in a dynamically allocated object. Because this object will persist until explicitly freed, the child thread can safely access its value.

### Noncompliant Code Example (Thread-Specific Storage)

In this noncompliant code example, the value is stored in thread-specific storage of the parent thread. However, because thread-specific data is available only to the thread that stores it, the `child_thread()` function will set `result` to a null value.

### Compliant Solution (Thread-Specific Storage)

This compliant solution illustrates how thread-specific storage can be combined with a call to a thread synchronization mechanism, such as `thrd_join()` . Because the parent thread waits until the child thread completes before continuing its execution, the child thread is guaranteed to access a valid live object.

### Compliant Solution (Thread-Local Storage, Windows, Visual Studio)

Similar to the preceding compliant solution, this compliant solution uses thread-local storage combined with thread synchronization to ensure the child thread is accessing a valid live object. It uses the Visual Studio–specific [`__declspec(thread)`](https://msdn.microsoft.com/en-us/library/9w1sdazb.aspx) language extension to provide the thread-local storage and the `WaitForSingleObject()` API to provide the synchronization.

### Noncompliant Code Example (OpenMP, `parallel` )

It is important to note that local data can be used securely with threads when using other thread interfaces, so the programmer need not always copy data into nonlocal memory when sharing data with threads. For example, the `shared` keyword in *[The OpenMP <sup>®</sup> API Specification for Parallel Programming](https://openmp.org/wp/)* \[ [OpenMP](https://openmp.org/wp/) \] can be used in combination with OpenMP's threading interface to share local memory without having to worry about whether local automatic variables remain valid.

In this noncompliant code example, a variable `j` is declared outside a `parallel` `#pragma` and not listed as a private variable. In OpenMP, variables outside a `   parallel #pragma  ` are shared unless designated as `private` .

### Compliant Solution (OpenMP, `parallel` , `private` )

In this compliant solution, the variable `j` is declared outside of the `parallel` `#pragma` but is explicitly labeled as `private` :

## CON35-C. Avoid deadlock by locking in a predefined order

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/05.concurrency-con/07.con35-c.md) — SHA-256 `9039016ec815501ed92ce33d93a0902bbd595a64284cd60e106580591617e817`。

Mutexes are used to prevent multiple threads from causing a data race by accessing shared resources at the same time. Sometimes, when locking mutexes, multiple threads hold each other's lock, and the program consequently deadlocks. Four conditions are required for deadlock to occur:

-   Mutual exclusion
-   Hold and wait
-   No preemption
-   Circular wait

Deadlock needs all four conditions, so preventing deadlock requires preventing any one of the four conditions. One simple solution is to lock the mutexes in a predefined order, which prevents circular wait.

### Noncompliant Code Example

The behavior of this noncompliant code example depends on the runtime environment and the platform's scheduler. The program is susceptible to deadlock if thread `thr1` attempts to lock `ba2` 's mutex at the same time thread `thr2` attempts to lock `ba1` 's mutex in the `deposit()` function.

### Compliant Solution

This compliant solution eliminates the circular wait condition by establishing a predefined order for locking in the `deposit()` function. Each thread will lock on the basis of the `bank_account` ID, which is set when the `   bank_account struct  ` is initialized.

## CON36-C. Wrap functions that can spuriously wake up in a loop

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/05.concurrency-con/08.con36-c.md) — SHA-256 `5e2656dba14d61c9e2dd4436e5312fbbcd17524f2fce8db4f68b6a9f640e320c`。

The `cnd_wait()` and `cnd_timedwait()` functions temporarily cede possession of a mutex so that other threads that may be requesting the mutex can proceed. These functions must always be called from code that is protected by locking a mutex. The waiting thread resumes execution only after it has been notified, generally as the result of the invocation of the `cnd_signal()` or `cnd_broadcast()` function invoked by another thread. The `cnd_wait()` function must be invoked from a loop that checks whether a [condition predicate](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conditionpredicate) holds. A condition predicate is an expression constructed from the variables of a function that must be true for a thread to be allowed to continue execution. The thread pauses execution, via `cnd_wait()` , `cnd_timedwait()` , or some other mechanism, and is resumed later, presumably when the condition predicate is true and the thread is notified.

The notification mechanism notifies the waiting thread and allows it to check its condition predicate. The invocation of `cnd_broadcast()` in another thread cannot precisely determine which waiting thread will be resumed. Condition predicate statements allow notified threads to determine whether they should resume upon receiving the notification.

### Noncompliant Code Example

This noncompliant code example monitors a linked list and assigns one thread to consume list elements when the list is nonempty.

This thread pauses execution using `cnd_wait()` and resumes when notified, presumably when the list has elements to be consumed. It is possible for the thread to be notified even if the list is still empty, perhaps because the notifying thread used `cnd_broadcast()` , which notifies all threads. Notification using `cnd_broadcast()` is frequently preferred over using `cnd_signal().` (See [CON38-C. Preserve thread safety and liveness when using condition variables](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/concurrency-con/con38-c) for more information.)

A condition predicate is typically the negation of the condition expression in the loop. In this noncompliant code example, the condition predicate for removing an element from a linked list is `   (list->next != NULL)  ` , whereas the condition expression for the `while` loop condition is `   (list->next == NULL)  ` .

This noncompliant code example nests the `cnd_wait()` function inside an `if` block and consequently fails to check the condition predicate after the notification is received. If the notification was spurious or malicious, the thread would wake up prematurely.

### Compliant Solution

This compliant solution calls the `cnd_wait()` function from within a `while` loop to check the condition both before and after the call to `cnd_wait()` :

<div class="syntaxhighlighter nogutter java">

## CON37-C. Do not call signal() in a multithreaded program

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/05.concurrency-con/09.con37-c.md) — SHA-256 `8ed4649fe76c74ab3d62a66ba7595126895c702771f816c3ccd90aad35fa8222`。

Calling the `signal()` function in a multithreaded program is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See [undefined behavior 135](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_135) .)

### Noncompliant Code Example

This noncompliant code example invokes the `signal()` function from a multithreaded program:

NOTE: The `SIGUSR1` signal value is not defined in the C Standard; consequently, this is not a C-compliant code example.

### Compliant Solution

This compliant solution uses an object of type `atomic_bool` to indicate when the child thread should terminate its loop:

### Exceptions

**CON37-C-EX1:** Implementations such as POSIX that provide defined behavior when multithreaded programs use custom signal handlers are exempt from this rule \[ [IEEE Std 1003.1-2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \].

## CON38-C. Preserve thread safety and liveness when using condition variables

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/05.concurrency-con/10.con38-c.md) — SHA-256 `9e70ff08adb5ea21c444bd436c001534fd17537984fd292c673dd3bbb0b806e9`。

Both thread safety and [liveness](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-liveness) are concerns when using condition variables. The *thread-safety* property requires that all objects maintain consistent states in a multithreaded environment \[ [Lea 2000](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Lea2000) \]. The *liveness* property requires that every operation or function invocation execute to completion without interruption; for example, there is no deadlock.

Condition variables must be used inside a `while` loop. (See [CON36-C. Wrap functions that can spuriously wake up in a loop](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/concurrency-con/con36-c) for more information.) To guarantee liveness, programs must test the `while` loop condition before invoking the `cnd_wait()` function. This early test checks whether another thread has already satisfied the [condition predicate](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conditionpredicate) and has sent a notification. Invoking the `cnd_wait()` function after the notification has been sent results in indefinite blocking.

To guarantee thread safety, programs must test the `while` loop condition after returning from the `cnd_wait()` function. When a given thread invokes the `cnd_wait()` function, it will attempt to block until its condition variable is signaled by a call to `cnd_broadcast()` or to `cnd_signal()` .

The `cnd_signal()` function unblocks one of the threads that are blocked on the specified condition variable at the time of the call. If multiple threads are waiting on the same condition variable, the scheduler can select any of those threads to be awakened (assuming that all threads have the same priority level). The `cnd_broadcast()` function unblocks all of the threads that are blocked on the specified condition variable at the time of the call. The order in which threads execute following a call to `cnd_broadcast()` is unspecified. Consequently, an unrelated thread could start executing, discover that its condition predicate is satisfied, and resume execution even though it was supposed to remain dormant. For these reasons, threads must check the condition predicate after the `cnd_wait()` function returns. A `while` loop is the best choice for checking the condition predicate both before and after invoking `cnd_wait()` .

The use of `cnd_signal()` is safe if each thread uses a unique condition variable. If multiple threads share a condition variable, the use of `cnd_signal()` is safe only if the following conditions are met:

-   All threads must perform the same set of operations after waking up, which means that any thread can be selected to wake up and resume for a single invocation of `cnd_signal()` .
-   Only one thread is required to wake upon receiving the signal.

The `cnd_broadcast()` function can be used to unblock all of the threads that are blocked on the specified condition variable if the use of `cnd_signal()` is unsafe.

### Noncompliant Code Example ( `cnd_signal()` )

This noncompliant code example uses five threads that are intended to execute sequentially according to the step level assigned to each thread when it is created (serialized processing). The `current_step` variable holds the current step level and is incremented when the respective thread completes. Finally, another thread is signaled so that the next step can be executed. Each thread waits until its step level is ready, and the `cnd_wait()` function call is wrapped inside a `while` loop, in compliance with [CON36-C. Wrap functions that can spuriously wake up in a loop](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/concurrency-con/con36-c) .

In this example, all threads share a condition variable. Each thread has its own distinct [condition predicate](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conditionpredicate) because each thread requires `current_step` to have a different value before proceeding. When the condition variable is signaled, any of the waiting threads can wake up.

The following table illustrates a possible scenario in which the [liveness](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-liveness) property is violated. If, by chance, the notified thread is not the thread with the next step value, that thread will wait again. No additional notifications can occur, and eventually the pool of available threads will be exhausted.

**Deadlock: Out-of-Sequence Step Value**

| Time | Thread # <br> ( ` my_step ` ) | ` current_step ` | Action |
| --- | --- | --- | --- |
| 0 | 3 | 0 | Thread 3 executes first time: predicate is ` FALSE -> wait() ` |
| 1 | 2 | 0 | Thread 2 executes first time: predicate is ` FALSE -> wait() ` |
| 2 | 4 | 0 | Thread 4 executes first time: predicate is ` FALSE -> wait() ` |
| 3 | 0 | 0 | Thread 0 executes first time: predicate is ` TRUE -> current_step++; cnd_signal() ` |
| 4 | 1 | 1 | Thread 1 executes first time: predicate is ` TRUE -> current_step++; cnd_signal() ` |
| 5 | 3 | 2 | Thread 3 wakes up (scheduler choice): predicate is ` FALSE -> wait() ` |
| 6 | — | — | **Thread exhaustion!** No more threads to run, and a conditional variable signal is needed to wake up the others |

This noncompliant code example violates the [liveness](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-liveness) property.

### Compliant Solution ( `cnd_broadcast()` )

This compliant solution uses the `cnd_broadcast()` function to signal all waiting threads instead of a single random thread. Only the `run_step()` thread code from the noncompliant code example is modified, as follows:

Awakening all threads guarantees the [liveness](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-liveness) property because each thread will execute its [condition predicate](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conditionpredicate) test, and exactly one will succeed and continue execution.

### Compliant Solution (Using `cnd_signal()` with a Unique Condition Variable per Thread)

Another compliant solution is to use a unique condition variable for each thread (all associated with the same mutex). In this case, `cnd_signal()` wakes up only the thread that is waiting on it. This solution is more efficient than using `cnd_broadcast()` because only the desired thread is awakened.

The [condition predicate](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conditionpredicate) of the signaled thread must be true; otherwise, a deadlock will occur.

### Compliant Solution (Windows, Condition Variables)

This compliant solution uses  a `CONDITION_VARIABLE` object, available on Microsoft Windows (Vista and later):

## CON39-C. Do not join or detach a thread that was previously joined or detached

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/05.concurrency-con/11.con39-c.md) — SHA-256 `574da1e87274386329241e0b5426cdfe16cc8f52153f85495e67c57722e42bf2`。

The C Standard, 7.28.5.6 paragraph 2 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states that a thread shall not be joined once it was previously joined or detached.

> The termination of the other thread synchronizes with the completion of the thrd_join function. The thread identified by thr shall not have been previously detached or joined with another thread.

Similarly, subclause 7.28.5.3 paragraph 2 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states that a thread shall not be detached once it was previously joined or detached.

> The thrd_detach function tells the operating system to dispose of any resources allocated to the thread identified by thr when that thread terminates. The thread identified by thr shall not have been previously detached or joined with another thread.

Violating either of these subclauses results in [undefined behavior 211](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_211) .

### Noncompliant Code Example

This noncompliant code example detaches a thread that is later joined.

### Compliant Solution

This compliant solution does not detach the thread. Its resources are released upon successfully joining with the main thread:

## CON40-C. Do not refer to an atomic variable twice in an expression

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/05.concurrency-con/12.con40-c.md) — SHA-256 `65ca7f11056fc9807272172ec43d4d8cc6c0b4016973d82d6594437cd72fd1f8`。

A consistent locking policy guarantees that multiple threads cannot simultaneously access or modify shared data. Atomic variables eliminate the need for locks by guaranteeing thread safety when certain operations are performed on them. The thread-safe operations on atomic variables are specified in the C Standard, subclauses 7.17.7 and 7.17.8 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2024) \]. While atomic operations can be combined, combined operations do not provide the thread safety provided by individual atomic operations.

Every time an atomic variable appears on the left side of an assignment operator, including a compound assignment operator such as `*=` , an atomic write is performed on the variable. The use of the increment (++ `)` or decrement `(--)` operators on an atomic variable constitutes an atomic read-and-write operation and is consequently thread-safe. Any reference of an atomic variable anywhere else in an expression indicates a distinct atomic read on the variable.

If the same atomic variable appears twice in an expression, then two atomic reads, or an atomic read and an atomic write, are required. Such a pair of atomic operations is not thread-safe, as another thread can modify the atomic variable between the two operations. Consequently, an atomic variable must not be referenced twice in the same expression.

Referring to an atomic variable more than once in the same block (but not the same expression) can be OK, as long as there is no assumption of atomicity for both times the variable is referenced.

  

### Noncompliant Code Example ( `atomic_int` )

This noncompliant code example takes an atomic global variable `n` and computes `   n + (n - 1) + (n - 2) + ... + 1  ` , using the formula `   n * (n + 1) / 2  ` :

The value of `n` may change between the two atomic reads of `n` in the expression, yielding an incorrect result.

### Compliant Solution

This compliant solution passes the atomic variable as a function argument, forcing the variable to be copied and guaranteeing a correct result. Note that the function's formal parameter need not be atomic, and the atomic variable can still be passed as an actual argument.

### Noncompliant Code Example ( `atomic_bool` )

This noncompliant code example declares a shared `atomic_bool` `flag` variable and provides a `toggle_flag()` method that negates the current value of `flag` :

Execution of this code may result in unexpected behavior because the value of `flag` is read, negated, and written back. This occurs even though the read and write are both atomic.

Consider, for example, two threads that call `toggle_flag()` . The expected effect of toggling `flag` twice is that it is restored to its original value. However, the scenario in the following table leaves `flag` in the incorrect state.

####### `toggle_flag()` without Compare-and-Exchange

|      |                      |                  |                                                                                                             |
|------|----------------------|------------------|-------------------------------------------------------------------------------------------------------------|
| Time | `flag`  | Thread           | Action                                                                                                      |
| 1    | `true`  | *t* <sub>1</sub> | Reads the current value of `flag` , which is `true,` into a cache                 |
| 2    | `true`  | *t* <sub>2</sub> | Reads the current value of `flag` , which is still `true,` into a different cache |
| 3    | `true`  | *t* <sub>1</sub> | Toggles the temporary variable in the cache to `false`                                         |
| 4    | `true`  | *t* <sub>2</sub> | Toggles the temporary variable in the different cache to `false`                               |
| 5    | `false` | *t* <sub>1</sub> | Writes the cache variable's value to `flag`                                                    |
| 6    | `false` | *t* <sub>2</sub> | Writes the different cache variable's value to `flag`                                          |

As a result, the effect of the call by *t* <sub>2</sub> is not reflected in `flag` ; the program behaves as if `toggle_flag()` was called only once, not twice.

### Noncompliant Code Example

This noncompliant code  breaks the `toggle_flag` function into three statements, but suffers from the same lack of atomicity.

### Compliant Solution ( `atomic_compare_exchange_weak()` )

This compliant solution uses a compare-and-exchange to guarantee that the correct value is stored in `flag` . All updates are visible to other threads. The call to `atomic_compare_exchange_weak()` is in a loop in conformance with [CON41-C. Wrap functions that can fail spuriously in a loop](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/concurrency-con/con41-c) .

An alternative solution is to use the `atomic_flag` data type for managing Boolean values atomically. However, `atomic_flag` does not support a toggle operation.

### Compliant Solution (Compound Assignment)

This compliant solution uses the `^=` assignment operation to toggle `flag` . This operation is guaranteed to be atomic, according to the C Standard, 6.5.17.3, paragraph 4 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2024) \]. This operation performs a bitwise-exclusive-or between its arguments, but for Boolean arguments, this is equivalent to negation.

An alternative solution is to use a mutex to protect the atomic operation, but this solution loses the performance benefits of atomic variables.

## CON41-C. Wrap functions that can fail spuriously in a loop

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/05.concurrency-con/13.con41-c.md) — SHA-256 `45a2441e099c942b6c5d258ac7e66d77c2d2aab8be70e0f53058f523455a5835`。

Functions that can fail spuriously should be wrapped in a loop.  The `atomic_compare_exchange_weak()` and `atomic_compare_exchange_weak_explicit()` functions both attempt to set an atomic variable to a new value but only if it currently possesses a known old value. Unlike the related functions `atomic_compare_exchange_strong()` and `atomic_compare_exchange_strong_explicit()` , these functions are permitted to *fail spuriously* . This makes these functions faster on some platforms—for example, on architectures that implement compare-and-exchange using load-linked/store-conditional instructions, such as Alpha, ARM, MIPS, and PowerPC. The C Standard, 7.17.7.4, paragraph 5 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2024) \], describes this behavior:

> A weak compare-and-exchange operation may fail spuriously. That is, even when the contents of memory referred to by `expected` and `object` are equal, it may return zero and store back to `expected` the same memory contents that were originally there.

### Noncompliant Code Example

In this noncompliant code example, `reorganize_data_structure()` is to be used as an argument to `thrd_create()` .  After reorganizing, the function attempts to replace the head pointer so that it points to the new version.  If no other thread has changed the head pointer since it was originally loaded, `reorganize_data_structure()` is intended to exit the thread with a result of `true` , indicating success.  Otherwise, the new reorganization attempt is discarded and the thread is exited with a result of `false` .  However, `atomic_compare_exchange_weak()` may fail even when the head pointer has not changed. Therefore, `reorganize_data_structure()` may perform the work and then discard it unnecessarily.

### Compliant Solution ( `atomic_compare_exchange_weak()` )

To recover from spurious failures, a loop must be used.  However, `atomic_compare_exchange_weak()` might fail because the head pointer changed, or the failure may be spurious. In either case, the thread must perform the work repeatedly until the compare-and-exchange succeeds, as shown in this compliant solution:

This loop could also be part of a larger control flow; for example, the thread from the noncompliant code example could be retried if it returns `false` .

### Compliant Solution ( `atomic_compare_exchange_strong()` )

When a weak compare-and-exchange would require a loop and a strong one would not, the strong one is preferable, as in this compliant solution:

<div class="syntaxhighlighter nogutter java">

## CON43-C. Do not allow data races in multithreaded code

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/05.concurrency-con/15.con43-c.md) — SHA-256 `bed26cf82a9a95fab5be6a1b7b837f7ebc7e6a4fcbe00bb3307e905b10142b39`。

When multiple threads can read or modify the same data, use synchronization techniques to avoid software flaws that can lead to security [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) . [Data races](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-datarace) can often result in [abnormal termination](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) or [denial of service](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-denial-of-serviceattack) , but it is possible for them to result in more serious vulnerabilities. The C Standard, section 5.1.2.5, paragraph 35 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO/IEC9899-2024) \], says:

> The execution of a program contains a data race if it contains two conflicting actions in different threads, at least one of which is not atomic, and neither happens before the other. Any such data race results in undefined behavior.

### Noncompliant Code Example

Assume this simplified code is part of a multithreaded bank system. Threads call `credit()` and `debit()` as money is deposited into and withdrawn from the single account. Because the addition and subtraction operations are not atomic, it is possible that two operations can occur concurrently, but only the result of one would be saved—despite declaring the `account_balance` `volatile` . For example, an attacker can credit the account with a sum of money and make a large number of small debits concurrently. Some of the debits might not affect the account balance because of the race condition, so the attacker is effectively creating money.

### Compliant Solution (Mutex)

This compliant solution uses a mutex to make credits and debits atomic operations. All credits and debits will now affect the account balance, so an attacker cannot exploit the race condition to steal money from the bank. The mutex is created with the `mtx_init()` function. The presence of the mutex makes declaring `account_balance` `volatile` unnecessary.

### Compliant Solution (Atomic)

This compliant solution uses an atomic variable to synchronize credit and debit operations. All credits and debits will now affect the account balance, so an attacker cannot exploit the race condition to steal money from the bank. The atomic integer does not need to be initialized because default (zero) initialization of an atomic object with static or thread-local storage is guaranteed to produce a valid state. The `+=` and `-=` operators behave atomically when used with an atomic variable.

### Noncompliant Code Example (Double-Fetch)

This noncompliant code example illustrates Xen Security Advisory CVE-2015-8550 / [XSA-155](https://xenbits.xen.org/xsa/advisory-155.html) In this example, the following code is vulnerable to a data race where the integer referenced by `ps` could be modified by a second thread that ran between the two reads of the variable.

Even though there is only one read of the `*ps` variable in the source code, the compiler is permitted to produce object code that performs multiple reads of the memory location. This is permitted by the "as-if" principle, as explained by section 5.1 of the [\[C99 Rationale 2003\]](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-C992003) :

> The /as if/ principle is invoked repeatedly in this Rationale. The C89 Committee found that describing various aspects of the C language, library, and environment in terms of concrete models best serves discussion and presentation. Every attempt has been made to craft these models so that implementations are constrained only insofar as they must bring about the same result, /as if/ they had implemented the presentation model; often enough the clearest model would make for the worst implementation.

#### Implementation Details (GCC)

This code produces two reads of the `*ps` value using GCC 4.8.4 on x86, as well as GCC 5.3.0 on x86-64 ( [Compiler-Introduced Double-Fetch Vulnerabilities – Understanding XSA-155](http://tkeetch.co.uk/blog/?p=58) ).

### Noncompliant Code Example (Volatile)

The data race can be disabled by declaring the data to be volatile, because the `volatile` keyword forces the compiler to not produce two reads of the data. However, this violates [CON02-C. Do not use volatile as a synchronization primitive](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/concurrency-con/con02-c) .

### Compliant Solution (C11, Atomic)

Declaring the data to be atomic also forces the compiler to produce only one read of the data.

### Compliant Solution (C11, Fences)

The bug was actually resolved by erecting fences around the `switch` statement.
