# CERT C — DCL

Carnegie Mellon University, © 2026。CC BY 4.0。[帰属・編集範囲](../cert-c.md)。原規範と例外・説明の抜粋であり、新しい要約規範ではない。

共通条件: C実装・C境界に条件付きで適用し、言語版/API/型/環境と本文の例外に照合する。コード例は収録しない。本文で決まらないコード依存条件は各IDの固定原典を確認し、確認前は未確認にする。元hashは未加工bytesのSHA-256。未完成/廃止/版依存の差分は各IDに示す。

## DCL00-C. Const-qualify immutable objects

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/02.dcl00-c.md) — SHA-256 `428aad48d92b62815f145d408608d420e67e124032fee1bb78f2875ed7e13ed2`。

Immutable objects should be `const` -qualified. Enforcing object immutability using `const` qualification helps ensure the correctness and security of applications. ISO/IEC TR 24772, for example, recommends labeling parameters as constant to avoid the unintentional modification of function arguments \[ [ISO/IEC TR 24772](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECTR24772-2010) \]. [STR05-C. Use pointers to const when referring to string literals](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/characters-and-strings-str/str05-c) describes a specialized case of this recommendation.

Adding `const` qualification may propagate through a program; as you add `const` , qualifiers become still more necessary. This phenomenon is sometimes called *`const` poisoning* , which can frequently lead to violations of [EXP05-C. Do not cast away a const qualification](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/expressions-exp/exp05-c) . Although `const` qualification is a good idea, the costs may outweigh the value in the remediation of existing code.

A macro or an enumeration constant may also be used instead of a `const` -qualified object. [DCL06-C. Use meaningful symbolic constants to represent literal values](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl06-c) describes the relative merits of using `const` -qualified objects, enumeration constants, and object-like macros. However, adding a `const` qualifier to an existing variable is a better first step than replacing the variable with an enumeration constant or macro because the compiler will issue warnings on any code that changes your `const` -qualified variable. Once you have verified that a `const` -qualified variable is not changed by any code, you may consider changing it to an enumeration constant or macro, as best fits your design.

### Noncompliant Code Example

In this noncompliant code, `pi` is declared as a `float` . Although pi is a mathematical constant, its value is not protected from accidental modification.

### Compliant Solution

In this compliant solution, `pi` is declared as a `const` -qualified object:

## DCL01-C. Do not reuse variable names in subscopes

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/03.dcl01-c.md) — SHA-256 `384b83dbcb3841acd397e28a724873878907db3b63fb5469c4d48fd4c9fbf04a`。

Do not use the same variable name in two scopes where one scope is contained in another. For example,

-   No other variable should share the name of a global variable if the other variable is in a subscope of the global variable.
-   A block should not declare a variable with the same name as a variable declared in any block that contains it.

Reusing variable names leads to programmer confusion about which variable is being modified. Additionally, if variable names are reused, generally one or both of the variable names are too generic.

### Noncompliant Code Example

This noncompliant code example declares the `msg` identifier at file scope and reuses the same identifier to declare a character array local to the `report_error()` function. The programmer may unintentionally copy the function argument to the locally declared `msg` array within the `report_error()` function. Depending on the programmer's intention, it either fails to initialize the global variable `msg` or allows the local `msg` buffer to overflow by using the global value `msgsize` as a bounds for the local buffer.

### Compliant Solution

This compliant solution uses different, more descriptive variable names:

When the block is small, the danger of reusing variable names is mitigated by the visibility of the immediate declaration. Even in this case, however, variable name reuse is not desirable. In general, the larger the declarative region of an identifier, the more descriptive and verbose should be the name of the identifier.

By using different variable names globally and locally, the compiler forces the developer to be more precise and descriptive with variable names.

### Noncompliant Code Example

This noncompliant code example declares two variables with the same identifier, but in slightly different scopes. The scope of the identifier `i` declared in the `for` loop's initial clause terminates after the closing curly brace of the for loop. The scope of the identifier i declared in the `for` loop's compound statement terminates before the closing curly brace. Thus, the inner declaration of `i` hides the outer declaration of `i` , which can lead to unintentionally referencing the wrong object.

### Compliant Solution

This compliant solution uses a unique identifier for the variable declared within the `for` loop.

### Exceptions

**DCL01-C-EX1:** A function argument in a function declaration may clash with a variable in a containing scope provided that when the function is defined, the argument has a name that clashes with no variables in any containing scopes.

**DCL01-C-EX2:** A temporary variable within a new scope inside of a macro can override an identifier in a containing scope. However,this exception does not apply to to the arguments of the macro itself.

## DCL02-C. Use visually distinct identifiers

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/04.dcl02-c.md) — SHA-256 `f8b61caa4bc7cbce98f69bbe0db2fba41dbf2c359b4f0a31e422fb0a0f5b035d`。

Use visually distinct identifiers with meaningful names to eliminate errors resulting from misreading the spelling of an identifier during the development and review of code. An identifier can denote an object; a function; a tag or a member of a structure, union, or enumeration; a typedef name; a label name; a macro name; or a macro parameter.

Depending on the fonts used, certain characters appear visually similar or even identical:

|                                                           |                                                                                                                                          |
|-----------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------|
| Character                                                 | Similar Characters                                                                                                                       |
| `0` (zero)                                   | `O` (capital `o` ), `Q` (capital `q` ), `D` (capital `d` ) |
| `1` (one)                                    | `I` (capital `i` ), `l` (lowercase `L` )                                             |
| `2` (two)                                    | `Z` (capital `z` )                                                                                             |
| `5` (five)                                   | `S` (capital `s` )                                                                                             |
| `8` (eight)                                  | `B` (capital `b` )                                                                                             |
| `n` (lowercase `N` )            | `h` (lowercase `H` )                                                                                           |
| `m` (lowercase `M` ) | `rn` (lowercase `R` , lowercase `N` )                                                |

Do not define multiple identifiers that vary only with respect to one or more visually similar characters.

Make the initial portions of long identifiers unique for easier recognition and to help prevent errors resulting from nonunique identifiers. (See [DCL23-C. Guarantee that mutually visible identifiers are unique](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl23-c) .)

In addition, the larger the scope of an identifier, the more descriptive its name should be. It may be perfectly appropriate to name a loop control variable `i` , but the same name would likely be confusing if it named a file scope object or a variable local to a function more than a few lines long. See also [DCL01-C. Do not reuse variable names in subscopes](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl01-c) and [DCL19-C. Use as minimal a scope as possible for all variables and functions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl19-c) .

### Noncompliant Code Example (Source Character Set)

DCL02-C implicitly assumes *global scope* , which can be confused with *scope within the same file* . Although it may not generate any errors, a possible violation of the rule may occur, as in the following example. Note this example does not violate [DCL23-C. Guarantee that mutually visible identifiers are unique](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl23-c) .

In file `foo.h` :

In file `bar.h` :

If a file `foobar.c` includes both `foo.h` and `bar.h` , then both `id_0` and `id_0` come in the same scope, violating this rule.

### Compliant Solution (Source Character Set)

In a compliant solution, use of visually similar identifiers should be avoided in the same project scope.

In file `foo.h` :

In file `bar.h` :

## DCL03-C. Use a static assertion to test the value of a constant expression

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/05.dcl03-c.md) — SHA-256 `55fe8f7da981a41695358bf9ffccb7c4f6bdf7ae426222a424fdee8e9abedf9d`。

Assertions are a valuable diagnostic tool for finding and eliminating software defects that may result in [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) (see [MSC11-C. Incorporate diagnostic tests using assertions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc11-c) ). The runtime `assert()` macro has some limitations, however, in that it incurs a runtime overhead and because it calls `abort()` . Consequently, the runtime `assert()` macro is useful only for identifying incorrect assumptions and not for runtime error checking. As a result, runtime assertions are generally unsuitable for server programs or embedded systems.

Static assertion is a new facility in the C Standard. It takes the form

Subclause 6.7.10 of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] states:

> The constant expression shall be an integer constant expression. If the value of the constant expression compares unequal to 0, the declaration has no effect. Otherwise, the constraint is violated and the implementation shall produce a diagnostic message that includes the text of the string literal, except that characters not in the basic source character set are not required to appear in the message.

It means that if `constant-expression` is true, nothing will happen. However, if `constant-expression` is false, an error message containing `string-literal` will be output at compile time.

Static assertion is not available in C99.

### Noncompliant Code Example

This noncompliant code uses the `assert()` macro to assert a property concerning a memory-mapped structure that is essential for the code to behave correctly:

Although the use of the runtime assertion is better than nothing, it needs to be placed in a function and executed. This means that it is usually far away from the definition of the actual structure to which it refers. The diagnostic occurs only at runtime and only if the code path containing the assertion is executed.

### Compliant Solution

For assertions involving only constant expressions, a preprocessor conditional statement may be used, as in this compliant solution:

Using `#error` directives allows for clear diagnostic messages. Because this approach evaluates assertions at compile time, there is no runtime penalty.

### Compliant Solution

This portable compliant solution uses `static_assert` :

Static assertions allow incorrect assumptions to be diagnosed at compile time instead of resulting in a silent malfunction or runtime error. Because the assertion is performed at compile time, no runtime cost in space or time is incurred. An assertion can be used at file or block scope, and failure results in a meaningful and informative diagnostic error message.

Other uses of static assertion are shown in [FIO34-C. Distinguish between characters read from a file and EOF or WEOF](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/input-output-fio/fio34-c) .

## DCL04-C. Do not declare more than one variable per declaration

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/06.dcl04-c.md) — SHA-256 `8d4a8765a1887f0d1353936bc7330f0406ad8f7810887d7e10ee1f2775bc9701`。

Every declaration should be for a single variable, on its own line, with an explanatory comment about the role of the variable. Declaring multiple variables in a single declaration can cause confusion regarding the types of the variables and their initial values. If more than one variable is declared in a declaration, care must be taken that the type and initialized value of the variable are handled correctly.

### Noncompliant Code Example

In this noncompliant code example, a programmer or code reviewer might mistakenly believe that the two variables `src` and `c` are declared as `   char *  ` . In fact, `src` has a type of `   char *  ` , whereas `c` has a type of `char` .

### Compliant Solution

In this compliant solution, each variable is declared on a separate line:

Although this change has no effect on compilation, the programmer's intent is clearer.

### Noncompliant Code Example

In this noncompliant code example, a programmer or code reviewer might mistakenly believe that both `i` and `j` have been initialized to 1. In fact, only `j` has been initialized, and `i` remains uninitialized.

### Compliant Solution

In this compliant solution, it is readily apparent that both `i` and `j` have been initialized to 1:

### Noncompliant Code Example (C23, `auto`)

This noncompliant code example declares two variables in one declaration using `auto`.

#### Implementation Details

In GCC 11.4.0, this line of code declares both `i` and `j` as ints, setting `j` to 4!

### Compliant Solution (C23, `auto`)

In this compliant solution, each declarator is sequestered to its own declaration.

### Exceptions

<a id="DCL04C.Donotdeclaremorethanonevariableperdeclaration-DCL04-EX1"></a> **DCL04-C-EX1:** Multiple loop control variables can be declared in the same `for` statement, as shown in the following function:

The `PRECISION()` macro provides the correct precision for any integer type and is defined in [INT35-C. Use correct integer precisions](https://www.securecoding.cert.org/confluence/display/seccode/INT35-C.+Use+correct+integer+precisions) —see that rule for more information.

<a id="DCL04C.Donotdeclaremorethanonevariableperdeclaration-DCL04-EX2"></a> **DCL04-C-EX2:** Multiple, simple variable declarations can be declared on the same line given that there are no initializations. A simple variable declaration is one that is not a pointer or array.

## DCL05-C. Use typedefs of non-pointer types only

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/07.dcl05-c.md) — SHA-256 `61e54ab36d598fae19cdb913bd3dc427415bc38474914056f78fa4fefbad7676`。

Using type definitions ( `typedef` ) can often improve code readability. However, type definitions to pointer types can make it more difficult to write `const` -correct code because the `const` qualifier will be applied to the pointer type, not to the underlying declared type.

### Noncompliant Code Example

The following type definition improves readability at the expense of introducing a `const` -correctness issue. In this example, the `const` qualifier applies to the `typedef` instead of to the underlying object type. Consequently, `func` does not take a pointer to a `   const struct obj  ` but instead takes a `const` pointer to a `   struct obj  ` .

### Compliant Solution

This compliant solution makes use of type definitions but does not declare a pointer type and so cannot be used in a `const` -incorrect manner:

### Noncompliant Code Example (Windows)

The Win32 SDK headers make use of type definitions for most of the types involved in Win32 APIs, but this noncompliant code example demonstrates a `const` -correctness bug:

### Compliant Solution (Windows)

This compliant solution demonstrates a common naming convention found in the Win32 APIs, using the proper `const` type:

### Noncompliant Code Example (Windows)

Note that many structures in the Win32 API are declared with pointer type definitions but not pointer-to- `const` type definitions ( `LPPOINT` , `LPSIZE` , and others). In these cases, it is suggested that you create your own type definition from the base structure type.

### Compliant Solution (Windows)

### Noncompliant Code Example

In this noncompliant code example, the declaration of the `signal()` function is difficult to read and comprehend:

### Compliant Solution

This compliant solution makes use of type definitions to specify the same type as in the noncompliant code example:

### Exceptions

Function pointer types are an exception to this recommendation.

## DCL06-C. Use meaningful symbolic constants to represent literal values

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/08.dcl06-c.md) — SHA-256 `054950b165972feefe0d48e3d7dfa32c4ff26fa0a1301d8c2cb94d14527dcd7f`。

The C language provides several different kinds of constants: *integer* constants, such as `10` and `0x1C` ; *floating* constants, such as `1.0` and `6.022e+23` ; and *character* constants, such as `'a'` and `'\x10'` . C also provides string literals, such as `   "hello, world"  `and`   "\n"  ` . These constants can all be referred to as *literals* .

When used in program logic, literals can reduce the readability of source code. As a result, literals, in general, and integer constants, in particular, are frequently called *magic numbers* because their purpose is often obscured. Magic numbers can be constant values that represent either an arbitrary value (such as a determined appropriate buffer size) or a malleable concept (such as the age at which a person is considered an adult, which can change between geopolitical boundaries). Rather than embed literals in program logic, use appropriately named symbolic constants to clarify the intent of the code. In addition, if a specific value needs to be changed, reassigning a symbolic constant once is more efficient and less error prone than replacing every instance of the value \[ [Saks 2002](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Saks02) \].

The C programming language has several mechanisms for creating named, symbolic constants: `const` -qualified objects, enumeration constants, and [object-like macro](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-object-likemacro) definitions. Each of these mechanisms has associated advantages and disadvantages.

### `const` -Qualified Objects

Objects that are `const` -qualified have scope and can be type-checked by the compiler. Because they are named objects (unlike macro definitions), some debugging tools can show the name of the object. The object also consumes memory.

A `const` -qualified object allows you to specify the exact type of the constant. For example,

defines `buffer_size` as a constant whose type is `   unsigned int  ` .

Unfortunately, `const` -qualified objects cannot be used where compile-time integer constants are required, namely to define the

-   Size of a bit-field member of a structure.
-   Size of an array (except in the case of variable length arrays).
-   Value of an enumeration constant.
-   Value of a `case` constant.

If any of these are required, then an integer constant (which would be an [rvalue](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-rvalue) ) must be used.

`const` -qualified objects allow the programmer to take the address of the object:

`const` -qualified objects are likely to incur some runtime overhead \[ [Saks 2001b](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Saks02) \]. Most C compilers, for example, allocate memory for `const` -qualified objects. `const` -qualified objects declared inside a function body can have automatic storage duration. If so, the compiler will allocate storage for the object, and it will be on the stack. As a result, this storage will need to be allocated and initialized each time the containing function is invoked.

### Enumeration Constants

Enumeration constants can be used to represent an integer constant expression that has a value representable as an `int` . Unlike `const` -qualified objects, enumeration constants do not consume memory. No storage is allocated for the value, so it is not possible to take the address of an enumeration constant.

Enumeration constants do not allow the type of the value to be specified. An enumeration constant whose value can be represented as an `int` is always an `int` .

### Object-like Macros

A preprocessing directive of the form

`#` `define` *identifier replacement-list*

defines an [*object-like* macro](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-object-likemacro) that causes each subsequent instance of the macro name to be replaced by the replacement list of preprocessing tokens that constitute the remainder of the directive.

C programmers frequently define symbolic constants as object-like macros. For example, the code

defines `buffer_size` as a macro whose value is 256. The preprocessor substitutes macros before the compiler does any other symbol processing. Later compilation phases never see macro symbols, such as `buffer_size` ; they see only the source text after macro substitution. As a result, many compilers do not preserve macro names among the symbols they pass on to their debuggers.

Macro names do not observe the scope rules that apply to other names. Therefore, macros could substitute in unanticipated places with unexpected results.

Object-like macros do not consume memory; consequently, it is not possible to create a pointer to one. Macros do not provide for type checking because they are textually replaced by the preprocessor.

Macros can be passed as compile-time arguments.

### Summary

The following table summarizes some of the differences between `const` -qualified objects, enumeration constants, and [object-like macro](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-object-likemacro) definitions.

|                                 |              |                 |                       |               |                                  |
|---------------------------------|--------------|-----------------|-----------------------|---------------|----------------------------------|
| Method                          | Evaluated At | Consumes Memory | Viewable by Debuggers | Type Checking | Compile-Time Constant Expression |
| Enumerations                    | Compile time | No              | Yes                   | Yes           | Yes                              |
| `const` -qualified | Runtime      | Yes             | Yes                   | Yes           | No                               |
| Macros                          | Preprocessor | No              | No                    | No            | Yes                              |

### Noncompliant Code Example

The meaning of the integer literal 18 is not clear in this example:

### Compliant Solution

This compliant solution replaces the integer literal 18 with the symbolic constant `ADULT_AGE` to clarify the meaning of the code:

### Noncompliant Code Example

Integer literals are frequently used when referring to array dimensions, as shown in this noncompliant code example:

This use of integer literals can easily result in buffer overflows if, for example, the buffer size is reduced but the integer literal used in the call to `fgets()` is not.

### Compliant Solution ( `enum` )

In this compliant solution, the integer literal is replaced with an enumeration constant. (See [DCL00-C. Const-qualify immutable objects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl00-c) .)

Enumeration constants can safely be used anywhere a constant expression is required.

### Compliant Solution ( `sizeof` )

Frequently, it is possible to obtain the desired readability by using a symbolic expression composed of existing symbols rather than by defining a new symbol. For example, a `sizeof` expression can work just as well as an enumeration constant. (See [EXP09-C. Use sizeof to determine the size of a type or variable](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/expressions-exp/exp09-c) .)

Using the `sizeof` expression in this example reduces the total number of names declared in the program, which is generally a good idea \[ [Saks 2002](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Saks02) \]. The `sizeof` operator is almost always evaluated at compile time (except in the case of variable-length arrays).

When working with `sizeof()` , keep in mind [ARR01-C. Do not apply the sizeof operator to a pointer when taking the size of an array](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/arrays-arr/arr01-c) .

### Noncompliant Code Example

In this noncompliant code example, the string literal `"localhost"` and integer constant `1234` are embedded directly in program logic and are consequently difficult to change:

### Compliant Solution

In this compliant solution, the host name and port number are both defined as [object-like macros](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-object-likemacro) , so they can be passed as compile-time arguments:

### Exceptions

**DCL06-C-EX1:** Although replacing numeric constants with a symbolic constant is often a good practice, it can be taken too far. Remember that the goal is to improve readability. Exceptions can be made for constants that are themselves the abstraction you want to represent, as in this compliant solution.

Replacing numeric constants with symbolic constants in this example does nothing to improve the readability of the code and can actually make the code more difficult to read.

When implementing recommendations, it is always necessary to use sound judgment.

Note that this example does not check for invalid operations (taking the `sqrt()` of a negative number). See [FLP32-C. Prevent or detect domain and range errors in math functions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/floating-point-flp/flp32-c) for more information on detecting domain and range errors in math functions.

## DCL07-C. Include the appropriate type information in function declarators

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/09.dcl07-c.md) — SHA-256 `937ce2e5d9a58eaae8ed7496a253f0258e85d5dcd85f2055d50c5f27d1e4ae41`。

Function declarators must be declared with the appropriate type information, including a return type and parameter list. If type information is not properly specified in a function declarator, the compiler cannot properly check function type information. When using standard library calls, the easiest (and preferred) way to obtain function declarators with appropriate type information is to include the appropriate header file.

Attempting to compile a program with a function declarator that does not include the appropriate type information typically generates a warning but does not prevent program compilation. These warnings should be resolved. (See [MSC00-C. Compile cleanly at high warning levels](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc00-c) .)

### Noncompliant Code Example (Non-Prototype-Format Declarators)

This noncompliant code example uses the *identifier-list* form for parameter declarations:

Subclause 6.11.7 of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] states that "the use of function definitions with separate parameter identifier and declaration lists (not prototype-format parameter type and identifier declarators) is an obsolescent feature."

### Compliant Solution (Non-Prototype-Format Declarators)

In this compliant solution, `int` is the type specifier, `   max(int a, int b)  ` is the function declarator, and the block within the curly braces is the function body:

### Noncompliant Code Example (Function Prototypes)

Declaring a function without any prototype forces the compiler to assume that the correct number and type of parameters have been supplied to a function. This practice can result in unintended and [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

In this noncompliant code example, the definition of `func()` in `file_a.c` expects three parameters but is supplied only two:

However, because there is no prototype for `func()` in `file_b.c` , the compiler assumes that the correct number of arguments has been supplied and uses the next value on the program stack as the missing third argument:

C99 eliminated implicit function declarations from the C language. However, many compilers still allow the compilation of programs containing implicitly declared functions, although they may issue a warning message. These warnings should be resolved. (See [MSC00-C. Compile cleanly at high warning levels](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc00-c) .)

### Compliant Solution (Function Prototypes)

This compliant solution correctly includes the function prototype for `func()` in the compilation unit in which it is invoked, and the function invocation has been corrected to pass the right number of arguments:

### Noncompliant Code Example (Function Pointers)

If a function pointer refers to an incompatible function, invoking that function via the pointer may corrupt the process stack. As a result, unexpected data may be accessed by the called function.

In this noncompliant code example, the function pointer `fn_ptr` refers to the function `add()` , which accepts three integer arguments. However, `fn_ptr` is specified to accept two integer arguments. Setting `fn_ptr` to refer to `add()` results in unexpected program behavior. This example also violates [EXP37-C. Call functions with the correct number and type of arguments](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp37-c) :

### Compliant Solution (Function Pointers)

To correct this example, the declaration of `fn_ptr` is changed to accept three arguments:

## DCL08-C. Properly encode relationships in constant definitions

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/10.dcl08-c.md) — SHA-256 `36de5edaf81f890c1e7337e14a40cb905808583f02caa3a989937f33c53eb139`。

If a relation exists between constants, you should encode the relationship in the definitions. Do not give two independent definitions, because a maintainer may fail to preserve that relationship when modifying the code. As a corollary, do not encode an impermanent or false relationship between constants, because future modifications may result in an incorrect definition for the dependent constant.

### Noncompliant Code Example

In this noncompliant code example, the definition for `OUT_STR_LEN` must always be two greater than the definition of `IN_STR_LEN` . The following definitions fail to embody this relationship:

A programmer performing maintenance on this program would need to identify the relationship and modify both definitions accordingly. Although this sort of error appears relatively benign, it can easily lead to serious security [vulnerabilities](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) , such as buffer overflows.

### Compliant Solution

The declaration in this compliant solution embodies the relationship between the two definitions:

As a result, a programmer can reliably modify the program by changing the definition of `IN_STR_LEN` .

### Noncompliant Code Example

In this noncompliant code example, a relationship is established between two constants where none exists:

A programmer performing maintenance on this program may modify the definition for `ADULT_AGE` but fail to recognize that the definition for `ALCOHOL_AGE` has also been changed as a consequence.

### Compliant Solution

This compliant solution does not assume a relationship where none exists:

## DCL09-C. Declare functions that return errno with a return type of errno_t

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/11.dcl09-c.md) — SHA-256 `9fc7227dab2cc7ee2abb9a62cde672f94df48798a0795efa4eec90abb087d3c3`。

When developing new code, declare functions that return `errno` with a return type of `errno_t` .  Many existing functions that return `errno` are declared as returning a value of type `int` . It is semantically unclear by inspecting the function declaration or prototype if these functions return an error status or a value or, worse, some combination of the two. (See [ERR02-C. Avoid in-band error indicators](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/error-handling-err/err02-c) .)

C11 Annex K  introduced the new type `errno_t` that is defined to be type `int` in `errno.h` and elsewhere. Many of the functions defined in C11 Annex K return values of this type. The `errno_t` type should be used as the type of an object that may contain only values that might be found in `errno` . For example, a function that returns the value of `errno` should be declared as having the return type `errno_t` .

This recommendation depends on C11 Annex K being implemented. The following code can be added to remove this dependency:

<div class="line number3 index2 alt2">

  

  

</div>

<div class="line number3 index2 alt2">

### Noncompliant Code Example

</div>

This noncompliant code example shows a function called `opener()` that returns `errno` error codes. However, the function is declared as returning an `int` . Consequently, the meaning of the return value is not readily apparent.

This noncompliant code example nevertheless complies with [ERR30-C. Take care when reading errno](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/error-handling-err/err30-c) .

### Compliant Solution (POSIX)

In this compliant solution, the `opener()` function returns a value of type `errno_t` , providing a clear indication that this function returns an error code:

This compliant solution is categorized as a POSIX solution because it returns `   EINVAL   and       EIO   ` , which are defined by POSIX (IEEE Std 1003.1, 2013 Edition) but not by the C Standard.

## DCL10-C. Maintain the contract between the writer and caller of variadic functions

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/12.dcl10-c.md) — SHA-256 `60133154cc463639374c88ae44ac26b6f1e762578a803e1543c81e923ce30321`。

Variadic functions accept a variable number of arguments but are problematic. Variadic functions define an implicit contract between the function writer and the function user that allows the function to determine the number of arguments passed in any particular invocation. Failure to enforce this contract may result in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . See [undefined behavior 141](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_141) of Appendix J of the C Standard.

### Argument Processing

In the following code example, the variadic function `average()` calculates the average value of the positive integer arguments passed to the function \[ [Seacord 2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Seacord2013) \]. The function processes arguments until it encounters an argument with the value of `va_eol` ( `-1` ).

Note that `va_start()` must be called to initialize the argument list and that `va_end()` must be called when finished with a variable argument list.

### Noncompliant Code Example

In this noncompliant code example, the `average()` function is called as follows:

The omission of the `va_eol` terminating value means that the function will continue to process values from the stack until it encounters a `va_eol` by coincidence or an error occurs.

### Compliant Solution

This compliant solution enforces the contract by adding `va_eol` as the final argument:

### Noncompliant Code Example

Another common mistake is to use more conversion specifiers than supplied arguments, as shown in this noncompliant code example:

This code results in nonexistent arguments being processed by the function, potentially leaking information about the process.

### Compliant Solution

This compliant solution matches the number of format specifiers with the number of variable arguments:

#### Argument List Caveats

C functions that accept the variadic primitive `va_list` as an argument pose an additional risk. Calls to `vfprintf()` , `vfscanf()` , `vprintf()` , `vscanf()` , `vsnprintf()` , `vsprintf()` , and `vsscanf()` use the `va_arg()` macro, invalidating the parameterized `va_list` . Consequently, once a `va_list` is passed as an argument to any of these functions, it cannot be used again without a call to `va_end()` followed by a call to `va_start()` .

## DCL11-C. Understand the type issues associated with variadic functions

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/13.dcl11-c.md) — SHA-256 `32e0773f17751d4b76b396a4a2c1a99466f909e6c62990899f0e4bee6ca97d87`。

The variable parameters of a variadic function—that is, those that correspond with the position of the ellipsis—are interpreted by the `va_arg()` macro. The `va_arg()` macro is used to extract the next argument from an initialized argument list within the body of a variadic function implementation. The size of each parameter is determined by the specified type. If the type is inconsistent with the corresponding argument, the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) and may result in misinterpreted data or an alignment error (see [EXP36-C. Do not cast pointers into more strictly aligned pointer types](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp36-c) ).

The variable arguments to a variadic function are not checked for type by the compiler. As a result, the programmer is responsible for ensuring that they are compatible with the corresponding parameter after the default argument promotions:

-   Integer arguments of types ranked lower than `int` are promoted to `int` if `int` can hold all the values of that type; otherwise, they are promoted to `    unsigned int   ` (the *integer promotions* ).
-   Arguments of type `float` are promoted to `double` .

### Noncompliant Code Example (Type Interpretation Error)

The C `printf()` function is implemented as a variadic function. This noncompliant code example swaps its null-terminated byte string and integer parameters with respect to how they are specified in the format string. Consequently, the integer is interpreted as a pointer to a null-terminated byte string and dereferenced, which will likely cause the program to [abnormally terminate](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-abnormaltermination) . Note that the `error_message` pointer is likewise interpreted as an integer.

### Compliant Solution (Type Interpretation Error)

This compliant solution modifies the format string so that the conversion specifiers correspond to the arguments:

As shown, care must be taken to ensure that the arguments passed to a format string function match up with the supplied format string.

### Noncompliant Code Example (Type Alignment Error)

In this noncompliant code example, a type `   long long  ` integer is incorrectly parsed by the `printf()` function with a `%d` specifier. This code may result in data truncation or misrepresentation when the value is extracted from the argument list.

Because a `   long long  ` was not interpreted, if the `   long long  ` uses more bytes for storage, the subsequent format specifier `%s` is unexpectedly offset, causing unknown data to be used instead of the pointer to the message.

### Compliant Solution (Type Alignment Error)

This compliant solution adds the length modifier `ll` to the `%d` format specifier so that the variadic function parser for `printf()` extracts the correct number of bytes from the variable argument list for the `   long long  ` argument:

### Noncompliant Code Example ( `NULL` )

The C Standard allows NULL to be either an integer constant or a pointer constant. While passing NULL as an argument to a function with a fixed number of arguments will cause NULL to be cast to the appropriate pointer type, when it is passed as a variadic argument, this will not happen if `   sizeof(NULL) != sizeof(void *).  ` This is possible for several reasons:

-   Pointers and ints may have different sizes on a platform where NULL is an integer constant
-   The platform may have different pointer types with different sizes on a platform. In that case, if NULL is a void pointer, it is the same size as a pointer to char (C11 section 6.2.5, paragraph 28), which might be sized differently than the required pointer type.

On either such platform, the following code will have [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) :

On a system with 32-bit `int` and 64-bit pointers, `printf()` may interpret the `NULL` as high-order bits of the pointer and the third argument `1` as the low-order bits of the pointer. In this case, `printf()` will print a pointer with the value `0x00000001` and then attempt to read an additional argument for the `%d` conversion specifier, which was not provided.

### Compliant Solution ( `NULL` )

This compliant solution avoids sending `NULL` to `printf()` :

## DCL12-C. Implement abstract data types using opaque types

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/14.dcl12-c.md) — SHA-256 `e8f9de0be2188df4b65ba420783e96b7c2f43282508ba0055d4bcb7ffc92014a`。

Abstract data types are not restricted to object-oriented languages such as C++ and Java. They should be created and used in C language programs as well. Abstract data types are most effective when used with private (opaque) data types and information hiding.

### Noncompliant Code Example

This noncompliant code example is based on the managed string library developed by CERT \[ [Burch 2006](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Burch06) \]. In this example, the managed string type and the functions that operate on this type are defined in the `string_m.h` header file as follows:

The implementation of the `string_mx` type is fully visible to the user of the data type after including the `string_m.h` file. Programmers are consequently more likely to directly manipulate the fields within the structure, violating the software engineering principles of information hiding and data encapsulation and increasing the probability of developing incorrect or nonportable code.

### Compliant Solution

This compliant solution reimplements the `string_mx` type as a private type, hiding the implementation of the data type from the user of the managed string library. To accomplish this, the developer of the private data type creates two header files: an external `string_m.h` header file that is included by the user of the data type and an internal file that is included only in files that implement the managed string abstract data type.

In the external `string_m.h` file, the `string_mx` type is defined to be an instance of `   struct string_mx  ` , which in turn is declared as an [incomplete type](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-incompletetype) :

In the internal header file, `   struct string_mx  ` is fully defined but not visible to a user of the data abstraction:

Modules that implement the abstract data type include both the external and internal definitions, whereas users of the data abstraction include only the external `string_m.h` file. This allows the implementation of the `string_mx` data type to remain private.

## DCL13-C. Declare function parameters that are pointers to values not changed by the function as const

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/15.dcl13-c.md) — SHA-256 `6ba8004408a19dcd45efc9dd60ef65cc80a0b8c75411bcc0369714208e48e1a8`。

Declaring function parameters `const` indicates that the function promises not to change these values.

In C, function arguments are passed by value rather than by reference. Although a function may change the values passed in, these changed values are discarded once the function returns. For this reason, many programmers assume a function will not change its arguments and that declaring the function's parameters as `const` is unnecessary.

Pointers behave in a similar fashion. A function may change a pointer to reference a different object, or `NULL` , yet that change is discarded once the function exits. Consequently, declaring a pointer as `const` is unnecessary.

### Noncompliant Code Example

Unlike passed-by-value arguments and pointers, pointed-to values are a concern. A function may modify a value referenced by a pointer argument, leading to a [side effect](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sideeffect) that persists even after the function exits. Modification of the pointed-to value is not diagnosed by the compiler, which assumes this behavior was intended.

If the function parameter is `const` -qualified, any attempt to modify the pointed-to value should cause the compiler to issue a diagnostic message.

As a result, the `const` violation must be resolved before the code can be compiled without a diagnostic message being issued.

### Compliant Solution

This compliant solution addresses the `const` violation by not modifying the constant argument:

### Noncompliant Code Example

This noncompliant code example defines a fictional version of the standard `strcat()` function called `strcat_nc()` . This function differs from `strcat()` in that the second argument is not `const` -qualified.

The function behaves the same as `strcat()` , but the compiler generates warnings in incorrect locations and fails to generate them in correct locations.

In the first `strcat_nc()` call, the compiler generates a warning about attempting to cast away `const` on `c_str2` because `strcat_nc()` does not modify its second argument yet fails to declare it `const` .

In the second `strcat_nc()` call, the compiler compiles the code with no warnings, but the resulting code will attempt to modify the `"c_str1"` literal. This violates [STR05-C. Use pointers to const when referring to string literals](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/characters-and-strings-str/str05-c) and [STR30-C. Do not attempt to modify string literals](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/characters-and-strings-str/str30-c) .

In the final `strcat_nc()` call, the compiler generates a warning about attempting to cast away `const` on `c_str4` , which is a valid warning.

### Compliant Solution

This compliant solution uses the prototype for the `strcat()` from C90. Although the `restrict` type qualifier did not exist in C90, `const` did. In general, function parameters should be declared in a manner consistent with the semantics of the function. In the case of `strcat()` , the initial argument can be changed by the function, but the second argument cannot.

The `const` -qualification of the second argument, `s2` , eliminates the spurious warning in the initial invocation but maintains the valid warning on the final invocation in which a `const` -qualified object is passed as the first argument (which can change). Finally, the middle `strcat()` invocation is now valid because `c_str3` is a valid destination string and may be safely modified.

## DCL15-C. Declare file-scope objects or functions that do not need external linkage as static

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/16.dcl15-c.md) — SHA-256 `42ec8242d335461914112bb8c0165c42f536352af42e8b515292848f53d29864`。

If a file-scope object or a function does not need to be visible outside of the file, it should be hidden by being declared as `static` . This practice creates more modular code and limits pollution of the global name space.

Subclause 6.2.2 of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] states:

> If the declaration of a file scope identifier for an object or a function contains the storage-class specifier `static` , the identifier has internal linkage.

and

> If the declaration of an identifier for an object has file scope and no storage-class specifier, its linkage is external.

### Noncompliant Code Example

This noncompliant code example includes a `helper()` function that is implicitly declared to have external linkage:

### Compliant Solution

This compliant solution declares `helper()` to have internal linkage, thereby preventing external functions from using it:

## DCL16-C. Use "L," not "l," to indicate a long value

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/17.dcl16-c.md) — SHA-256 `1880e4b333b5308334183494732fde771030c4e5e44edd3c8fd021e7614ecd90`。

Lowercase letter *l* (ell) can easily be confused with the digit *1* (one). This can be particularly confusing when indicating that an integer literal constant is a long value. This recommendation is similar to [DCL02-C. Use visually distinct identifiers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl02-c) . Likewise, you should use uppercase *LL* rather than lowercase *ll* when indicating that an integer literal constant is a `   long long  ` value.

To be precise when using modifiers to indicate the type of an integer literal, the first character may not be `l` .  It may be `L` , `u` , or `U` . Subsequent characters have no strict case requirements.

### Noncompliant Code Example

This noncompliant example highlights the result of adding an integer and a long value even though it appears that two integers `1111` are being added:

### Compliant Solution

The compliant solution improvises by using an uppercase *L* instead of lowercase *l* to disambiguate the visual appearance:

## DCL17-C. Beware of miscompiled volatile-qualified variables

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/18.dcl17-c.md) — SHA-256 `806f271da77df384d0a1460b76e95eb7ff1e7607163747fc801f92020658a840`。

As described in depth in rule [DCL22-C. Use volatile for data that cannot be cached](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl22-c) , a `volatile` -qualified variable "shall be evaluated strictly according to the rules of the abstract machine" \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \]. In other words, the `volatile` qualifier is used to instruct the compiler to not make caching optimizations about a variable.

However, as demonstrated in "Volatiles Are Miscompiled, and What to Do about It" \[ [Eide and Regehr](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-EideandRegehr) \], all tested compilers generated some percentage of incorrect compiled code with regard to `volatile` accesses. Therefore, it is necessary to know how your compiler behaves when the standard `volatile` behavior is required. The authors also provide a workaround that eliminates some or all of these errors.

### Noncompliant Code Example

As demonstrated in Eide and Regehr's work, the following code example compiles incorrectly using GCC 4.3.0 for IA32 and the `-Os` optimization flag:

Because the variable `x` is `volatile` -qualified, it should be accessed 10 times in this program. However, as shown in the compiled object code, it is accessed only once due to a loop-hoisting optimization \[ [Eide and Regehr](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-EideandRegehr) \]:

Should `x` represent a hardware register or some other memory-mapped device that has [side effects](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-sideeffect) when accessed, the previous miscompiled code example may produce [unexpected behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-unexpectedbehavior) .

### Compliant Solution

Eide and Regehr tested a workaround by wrapping `volatile` accesses with function calls. They describe it with the intuition that "we can replace an action that compilers empirically get wrong by a different action—a function call—that compilers can get right" \[ [Eide and Regehr](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-EideandRegehr) \]. For example, the workaround for the noncompliant code example would be

The workarounds proposed by Eide and Regehr fix many of the `volatile` -access bugs in the tested compilers. However, compilers are always changing, so critical sections of code should be compiled as if for deployment, and the compiled object code should be inspected for the correct behavior.

## DCL18-C. Do not begin integer constants with 0 when specifying a decimal value

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/19.dcl18-c.md) — SHA-256 `a13877e66dff1efefb72d8934ede2812e4e8a3dd67ddcac72f82a35167da896c`。

The C Standard defines octal constants as a 0 followed by octal digits (0 1 2 3 4 5 6 7). Programming errors can occur when decimal values are mistakenly specified as octal constants.

### Noncompliant Code Example

In this noncompliant code example, a decimal constant is mistakenly prefaced with zeros so that all the constants are a fixed length:

Although it may appear that `i_array[2]` is assigned the decimal value 42, it is actually assigned the decimal value 34.

### Compliant Solution

To avoid using wrong values and to make the code more readable, do not preface constants with zeroes if the value is meant to be decimal:

## DCL19-C. Minimize the scope of variables and functions

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/20.dcl19-c.md) — SHA-256 `7c1f12035972f12563c53717abb7ef51cdc0aad151f2d9e4ed6e601a75297b2f`。

Variables and functions should be declared in the minimum scope from which all references to the identifier are still possible.

When a larger scope than necessary is used, code becomes less readable, harder to maintain, and more likely to reference unintended variables (see [DCL01-C. Do not reuse variable names in subscopes](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl01-c) ).

### Noncompliant Code Example

In this noncompliant code example, the function `counter()` increments the global variable `count` and then returns immediately if this variable exceeds a maximum value:

Assuming that the variable `count` is only accessed from this function, this example is noncompliant because it does not define `count` within the minimum possible scope.

### Compliant Solution

In this compliant solution, the variable `count` is declared within the scope of the `counter()` function as a static variable. The static modifier, when applied to a local variable (one inside of a function), modifies the lifetime (duration) of the variable so that it persists for as long as the program does and does not disappear between invocations of the function.

The keyword `static` also prevents reinitialization of the variable.

### Noncompliant Code Example

The counter variable `i` is declared outside of the `for` loop, which goes against this recommendation because it is not declared in the block in which it is used. If this code were reused with another index variable `j` , but there was a previously declared variable `i` , the loop could iterate over the wrong variable.

### Compliant Solution

Complying with this recommendation requires that you declare variables where they are used, which improves readability and reusability. In this example, you would declare the loop's index variable `i` within the initialization of the `for` loop. This requirement was recently relaxed in the C Standard.

### Noncompliant Code Example (Function Declaration)

In this noncompliant code example, the function `f()` is called only from within the function `g()` , which is defined in the same compilation unit. By default, function declarations are *extern* , meaning that these functions are placed in the global symbol table and are available from other compilation units.

### Compliant Solution

In this compliant solution, the function `f()` is declared with internal linkage. This practice limits the scope of the function declaration to the current compilation unit and prevents the function from being included in the external symbol table. It also limits cluttering in the global name space and prevents the function from being accidentally or intentionally invoked from another compilation unit. See [DCL15-C. Declare file-scope objects or functions that do not need external linkage as static](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl15-c) for more information.

## DCL20-C. Explicitly specify void when a function accepts no arguments

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/21.dcl20-c.md) — SHA-256 `93d12b02c3b86903820dd630194e0850d7de35e1874bd4151bd8aa6a51537352`。

According to the C Standard, subclause 6.7.6.3, paragraph 14 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \],

> An identifier list declares only the identifiers of the parameters of the function. An empty list in a function declarator that is part of a definition of that function specifies that the function has no parameters. The empty list in a function declarator that is not part of a definition of that function specifies that no information about the number or types of the parameters is supplied.

Subclause 6.11.6 states that

> The use of function declarators with empty parentheses (not prototype-format parameter type declarators) is an obsolescent feature.

Consequently, functions that accept no arguments should explicitly declare a `void` parameter in their parameter list. This holds true in both the declaration and definition sections (which should match).

Defining a function with a `void` argument list differs from declaring it with no arguments because, in the latter case, the compiler will not check whether the function is called with parameters at all \[ [TIGCC, void usage](http://tigcc.ticalc.org/) \]. Consequently, function calling with arbitrary parameters will be accepted without a warning at compile time.

Failure to declare a `void` parameter will result in

-   An ambiguous functional interface between the caller and callee.
-   Sensitive information outflow.

A similar recommendation deals with parameter type in a more general sense: [DCL07-C. Include the appropriate type information in function declarators](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl07-c) .

### Noncompliant Code Example (Ambiguous Interface)

In this noncompliant code example, the caller calls `foo()` with an argument of 3. The caller expects `foo()` to accept a single `int` argument and to output the argument as part of a longer message. Because `foo()` is declared without the `void` parameter, the compiler will not perform any caller check. It is therefore possible that the caller may not detect the error. In this example, for instance, `foo()` might output the value 3 as expected.

Because no function parameter has the same meaning as an arbitrary parameter, the caller can provide an arbitrary number of arguments to the function.

### Compliant Solution (Ambiguous Interface)

In this compliant solution, `void` is specified explicitly as a parameter in the declaration of `foo` 's prototype:

#### Implementation Details (Ambiguous Interface)

When the compliant solution is used and `foo(3)` is called, the GCC compiler issues the following diagnostic, which alerts the programmer about the misuse of the function interface:

### Noncompliant Code Example (Information Outflow)

Another possible vulnerability is the leak of privileged information. In this noncompliant code example, a user with high privileges feeds some secret input to the caller that the caller then passes to `foo()` . Because of the way `foo()` is defined, we might assume there is no way for `foo()` to retrieve information from the caller. However, because the value of `i` is really passed into a stack (before the return address of the caller), a malicious programmer can change the internal implementation and copy the value manually into a less privileged file.

### Compliant Solution (Information Outflow)

Again, the simplest solution is to explicitly specify `void` as the only parameter.

## DCL21-C. Understand the storage of compound literals

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/22.dcl21-c.md) — SHA-256 `8a05ddb21894f2f0189c769c9c2115540cfb3574c572a3cde22cac6414481a4d`。

Subclause 6.5.2.5 of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \] defines a compound literal as

> A postfix expression that consists of a parenthesized type name followed by a brace-enclosed list of initializers. . . . The value of the compound literal is that of an unnamed object initiated by the initializer list.

The storage for this object is either static (if the compound literal occurs at file scope) or automatic (if the compound literal occurs at block scope), and the storage duration is associated with its immediate enclosing block. For example, in the function

following initialization, the `int` pointer `ip` contains the address of an unnamed object of type `int[4]` , allocated on the stack. Once `func` returns, any attempts to access this object will produce [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

Note that only one object is created per compound literal—even if the compound literal appears in a loop and has dynamic initializers.

This recommendation is a specific instance of [DCL30-C. Declare objects with appropriate storage durations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl30-c) .

### Noncompliant Code Example

In this noncompliant code example, the programmer mistakenly assumes that the elements of the `ints` array of the pointer to `int_struct` are assigned the addresses of distinct `int_struct` objects, one for each integer in the range `   [0, MAX_INTS - 1]  ` :

However, only one `int_struct` object is created. At each iteration of the first loop, the `x` member of this object is set equal to the current value of the loop counter `i` . Therefore, just before the first loop terminates, the value of the `x` member is `   MAX_INTS - 1  ` .

Because the storage duration of the compound literal is associated with the `for` loop that contains it, dereferencing `ints` in the second loop results in [undefined behavior 9](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_9) (Annex J of the C Standard).

Even if the region of memory that contained the compound literal is not written to between loops, the print loop will display the value `   MAX_INTS - 1  `for`   MAX_INTS  ` lines. This is contrary to the intuitive expected result, which is that the integers `0` through `   MAX_INTS - 1  ` would be printed in order.

### Compliant Solution

This compliant solution uses an array of structures rather than an array of pointers. That way, an actual copy of each `int_struct` (rather than a pointer to the object) is stored.

## DCL22-C. Use volatile for data that cannot be cached

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/23.dcl22-c.md) — SHA-256 `b43451a1b2b33fb1520d8340050054c6b09ec9502aaa3fe7a797172cf3b2a21d`。

An object that has volatile-qualified type may be modified in ways unknown to the [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) or have other unknown side effects. Asynchronous signal handling, for example, may cause objects to be modified in a manner unknown to the compiler. Without this type qualifier, unintended optimizations may occur. These optimizations may cause race conditions because a programmer may write code that prevents a race condition, yet the compiler is not aware of the programmer's data model and may modify the code during compilation to permit race conditions.

The `volatile` keyword eliminates this confusion by imposing restrictions on access and caching. According to the C99 Rationale \[ [C99 Rationale 2003](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-C992003) \],

> No caching through this [lvalue](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-lvalue) : each operation in the abstract semantics must be performed (that is, no caching assumptions may be made, because the location is not guaranteed to contain any previous value). In the absence of this qualifier, the contents of the designated location may be assumed to be unchanged except for possible aliasing.

Type qualifying objects as volatile does not guarantee synchronization between multiple threads, protect against simultaneous memory accesses, or, unless used to declare objects of type `sig_atomic_t` , guarantee atomicity of accesses to the object. For restrictions specific to signal handlers, see [SIG31-C. Do not access shared objects in signal handlers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/signals-sig/sig31-c) . However, type qualifying objects as volatile does ensure that a conforming compiler will not elide or reorder access to the object.

### Noncompliant Code Example

In this noncompliant code example, the programmer is targeting a custom piece of hardware that controls an LED by writing values into a register bank. The register bank is memory mapped into the process such that writing to a specific memory location will actually place a value into a hardware register to be read by the LED controller. The programmer intends to turn the LED on by placing value 1 into the first register, and then turn the LED off later by placing the value 0 into the first register.

The compiler is free to optimize what it perceives as being a dead store to `bank[0]` by removing the first assignment to the variable. This would cause the LED to never be turned on in an optimized build.

### Compliant Solution

In this compliant solution, the register bank's memory is qualified with the `volatile` keyword, ensuring the compiler does not optimize access to the memory.

## DCL23-C. Guarantee that mutually visible identifiers are unique

**Recommendation / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/08.recommendations/06.declarations-and-initialization-dcl/24.dcl23-c.md) — SHA-256 `20440b88ac4ff0c54b5486574a7703e7c409e8c655c73f2c66af1c4965832842`。

According to subclause 6.2.7 of the C Standard \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \],

> All declarations that refer to the same object or function shall have compatible type; otherwise, the behavior is undefined.

(See also [undefined behavior 14](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_14) of Annex J.)

Further, according to subclause 6.4.2.1,

> Any identifiers that differ in a significant character are different identifiers. If two identifiers differ only in nonsignificant characters, the behavior is undefined.

(See also [undefined behavior 30](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_30) of Annex J.)

Identifiers in mutually visible scopes must be deemed unique by the compiler to prevent confusion about which variable or function is being referenced. [Implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) can allow additional nonunique characters to be appended to the end of identifiers, making the identifiers appear unique while actually being indistinguishable.

It is reasonable for scopes that are not visible to each other to have duplicate identifiers. For example, two functions can each have a local variable with the same name because their scopes cannot access each other. But a function's local variable names should be distinct from each other as well as from all static variables declared within the function's file (and from all included header files.)

To guarantee that identifiers are unique, the number of significant characters recognized by the most restrictive compiler used must be determined. This assumption must be documented in the code.

The standard defines the following minimum requirements:

-   63 significant initial characters in an internal identifier or a macro name. (Each universal character name or extended source character is considered a single character.)
-   31 significant initial characters in an external identifier. (Each universal character name specifying a short identifier of 0000FFFF or less is considered 6 characters; each universal character name specifying a short identifier of 00010000 or more is considered 10 characters; and each extended source character, if any exist, is considered the same number of characters as the corresponding universal character name.)

Restriction of the significance of an external name to fewer than 255 characters in the standard (considering each universal character name or extended source character as a single character) is an obsolescent feature that is a concession to existing implementations. As a result, it is not necessary to comply with this restriction as long as the identifiers are unique and the assumptions concerning the number of significant characters are documented.

### Noncompliant Code Example (Source Character Set)

On implementations that support only the minimum requirements for significant characters required by the standard, this code example is noncompliant because the first 31 characters of the external identifiers are identical:

### Compliant Solution (Source Character Set)

In a compliant solution, the significant characters in each identifier must differ:

### Noncompliant Code Example (Universal Character Names)

In this noncompliant code example, both external identifiers consist of four universal character names. Because the first three universal character names of each identifier are identical, both identify the same integer array on implementations that support only the minimum requirements for significant characters required by the standard:

### Compliant Solution (Universal Character Names)

For portability, the first three universal character name combinations used in an identifier must be unique:

## DCL30-C. Declare objects with appropriate storage durations

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/06.declarations-and-initialization-dcl/02.dcl30-c.md) — SHA-256 `c3a7df4eb23429d01acce79cdf6a4869d6e8d0f5c56d81aaa786690272c46fe2`。

Every object has a storage duration that determines its lifetime: *static* , *thread* , *automatic* , or *allocated* .

According to the C Standard, 6.2.4, paragraph 2 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \],

> The lifetime of an object is the portion of program execution during which storage is guaranteed to be reserved for it. An object exists, has a constant address, and retains its last-stored value throughout its lifetime. If an object is referred to outside of its lifetime, the behavior is undefined. If a pointer value is used in an evaluation after the object the pointer points to (or just past) reaches the end of its lifetime, the behavior is undefined.

Do not attempt to access an object outside of its lifetime. Attempting to do so is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) and can lead to an exploitable [vulnerability](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-vulnerability) . (See also [undefined behavior 9](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_9) in the C Standard, Annex J.)

### Noncompliant Code Example (Differing Storage Durations)

In this noncompliant code example, the address of the variable `c_str` with automatic storage duration is assigned to the variable `p` , which has static storage duration. The assignment itself is valid, but it is invalid for `c_str` to go out of scope while `p` holds its address, as happens at the end of `dont_do_this` `()` .

### Compliant Solution (Same Storage Durations)

In this compliant solution, `p` is declared with the same storage duration as `c_str` , preventing `p` from taking on an [indeterminate value](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) outside of `this_is_OK()` :

Alternatively, both `p` and `c_str` could be declared with static storage duration.

### Compliant Solution (Differing Storage Durations)

If it is necessary for `p` to be defined with static storage duration but `c_str` with a more limited duration, then `p` can be set to `NULL` before `c_str` is destroyed. This practice prevents `p` from taking on an [indeterminate value](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-indeterminatevalue) , although any references to `p` must check for `NULL` .

### Noncompliant Code Example (Return Values)

In this noncompliant code sample, the function `init_array` `()` returns a pointer to a character array with automatic storage duration, which is accessible to the caller:

Some compilers generate a diagnostic message when a pointer to an object with automatic storage duration is returned from a function, as in this example. Programmers should compile code at high warning levels and resolve any diagnostic messages. (See [MSC00-C. Compile cleanly at high warning levels](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/miscellaneous-msc/msc00-c) .)

### Compliant Solution (Return Values)

The solution, in this case, depends on the intent of the programmer. If the intent is to modify the value of `array` and have that modification persist outside the scope of `init_array()` , the desired behavior can be achieved by declaring `array` elsewhere and passing it as an argument to `init_array()` :

### Noncompliant Code Example (Output Parameter)

In this noncompliant code example, the function `squirrel_away()` stores a pointer to local variable `local` into a location pointed to by function parameter `ptr_param` . Upon the return of `squirrel_away()` , the pointer `ptr_param` points to a variable that has an expired lifetime.

### Compliant Solution (Output Parameter)

In this compliant solution, the variable `local` has static storage duration; consequently, `ptr` can be used to reference the `local` array within the `rodent()` function:

## DCL31-C. Declare identifiers before using them

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/06.declarations-and-initialization-dcl/03.dcl31-c.md) — SHA-256 `4773cd10ff283ba03a767b0b888a3e3c34558a8476ce5e323f2129a7e8028b36`。

The C23 Standard requires type specifiers and forbids implicit function declarations. The C90 Standard allows implicit typing of variables and functions. Consequently, some existing legacy code uses implicit typing. Some C compilers still support legacy code by allowing implicit typing, but it should not be used for new code. Such an [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) may choose to assume an implicit declaration and continue translation to support existing programs that used this feature.

### Noncompliant Code Example (Implicit `int` )

C no longer allows the absence of type specifiers in a declaration. The C Standard, 6.7.3 paragraph 2 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> Except where the type is inferred (6.7.10), at least one type specifier shall be given in the declaration specifiers in each declaration, and in the specifier-qualifier list in each member declaration and type name.

This noncompliant code example omits the type specifier:

Some C [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) do not issue a diagnostic for the violation of this constraint. These nonconforming C translators continue to treat such declarations as implying the type `int` .

### Compliant Solution (Implicit `int` )

This compliant solution explicitly includes a type specifier:

### Noncompliant Code Example (Implicit Function Declaration)

Implicit declaration of functions is not allowed; every function must be explicitly declared before it can be called. In C90, if a function is called without an explicit prototype, the compiler provides an implicit declaration.

The C90 Standard \[ [ISO/IEC 9899:1990](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-1990) \] includes this requirement:

> If the expression that precedes the parenthesized argument list in a function call consists solely of an identifier, and if no declaration is visible for this identifier, the identifier is implicitly declared exactly as if, in the innermost block containing the function call, the declaration `    extern int identifier();   ` appeared.

If a function declaration is not visible at the point at which a call to the function is made, C90-compliant platforms assume an implicit declaration of `   extern int identifier();  ` .

This declaration implies that the function may take any number and type of arguments and return an `int` . However, to conform to the current C Standard, programmers must explicitly prototype every function before invoking it. An implementation that conforms to the C Standard may or may not perform implicit function declarations, but C does require a conforming implementation to issue a diagnostic if it encounters an undeclared function being used.

In this noncompliant code example, if `malloc()` is not declared, either explicitly or by including `stdlib.h` , a compiler that conforms only to C90 may implicitly declare `malloc()` as `   int malloc()  ` . If the platform's size of `int` is 32 bits, but the size of pointers is 64 bits, the resulting pointer would likely be truncated as a result of the implicit declaration of `malloc()` , returning a 32-bit integer.

<div class="syntaxhighlighter nogutter c">

#### Implementation Details

When compiled with Microsoft Visual Studio 2013 for a 64-bit platform, this noncompliant code example will eventually cause an access violation when dereferencing `ptr` in the loop.

</div>

### Compliant Solution (Implicit Function Declaration)

This compliant solution declares `malloc()` by including the appropriate header file:

For more information on function declarations, see [DCL07-C. Include the appropriate type information in function declarators](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl07-c) .

### Noncompliant Code Example (Implicit Return Type)

Do not declare a function with an implicit return type. For example, if a function returns a meaningful integer value, declare it as returning `int` . If it returns no meaningful value, declare it as returning `void` .

Because the compiler assumes that `foo()` returns a value of type `int` for this noncompliant code example, `UINT_MAX` is incorrectly converted to `−1` .

### Compliant Solution (Implicit Return Type)

This compliant solution explicitly defines the return type of `foo()` as `   unsigned int  ` . As a result, the function correctly returns `UINT_MAX` .

## DCL36-C. Do not declare an identifier with conflicting linkage classifications

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/06.declarations-and-initialization-dcl/04.dcl36-c.md) — SHA-256 `97438b99c7af3edf63ab2f5a6b338242a8b28fd838a1375a09e070616b3a02b9`。

Linkage can make an identifier declared in different scopes or declared multiple times within the same scope refer to the same object or function. Identifiers are classified as *externally linked* , *internally linked* , or *not linked* . These three kinds of linkage have the following characteristics \[ [Kirch-Prinz 2002](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-Kirch-Prinz02) \]:

-   **External linkage:** An identifier with external linkage represents the same object or function throughout the entire program, that is, in all compilation units and libraries belonging to the program. The identifier is available to the linker. When a second declaration of the same identifier with external linkage occurs, the linker associates the identifier with the same object or function.

<!-- -->

-   **Internal linkage:** An identifier with internal linkage represents the same object or function within a given translation unit. The linker has no information about identifiers with internal linkage. Consequently, these identifiers are internal to the translation unit.

<!-- -->

-   **No linkage:** If an identifier has no linkage, then any further declaration using the identifier declares something new, such as a new variable or a new type.

According to the C Standard, 6.2.2 paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], linkage is determined as follows:

> If the declaration of a file scope identifier for:  
> - an object contains any of the storage-class specifiers `static` or `contexpr;`
>
> \- or, a function contains the storage-class specifier `static` ,
>
> then the identifier has internal linkage.
>
> For an identifier declared with the storage-class specifier `extern` in a scope in which a prior declaration of that identifier is visible, if the prior declaration specifies internal or external linkage, the linkage of the identifier at the later declaration is the same as the linkage specified at the prior declaration . If no prior declaration is visible, or if the prior declaration specifies no linkage, then the identifier has external linkage.
>
> If the declaration of an identifier for a function has no storage-class specifier, its linkage is determined exactly as if it were declared with the storage-class specifier `extern` . If the declaration of an identifier for an object has file scope and does not contain the storage-class specifier `static` or `contexpr` , its linkage is external.
>
> The following identifiers have no linkage: an identifier declared to be anything other than an object or a function; an identifier declared to be a function parameter; a block scope identifier for an object declared without the storage-class specifier `extern` .

  

Use of an identifier (within one translation unit) classified as both internally and externally linked is [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . (See also [undefined behavior 8](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_8) .) A translation unit includes the source file together with its headers and all source files included via the preprocessing directive `#include` .

The following table identifies the linkage assigned to an object that is declared twice at file scope in a single translation unit. The row designates the first declaration, and the column designates the redeclaration. *None* means the declaration has no storage-class specifier.

| First declaration | Redeclaration `static` | Redeclaration *none* | Redeclaration `extern` |
| --- | --- | --- | --- |
| `static` | Internal | Undefined | Internal |
| *None* | Undefined | External | External |
| `extern` | Undefined | External | External |

### Noncompliant Code Example

In this noncompliant code example, `i2` and `i5` are defined as having both internal and external linkage. Future use of either identifier results in [undefined behavior 8](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_8) .

#### Implementation Details

Microsoft Visual Studio 2013 issues no warnings about this code, even at the highest diagnostic levels.

GCC and Clang 14 both generate fatal diagnostics for the conflicting definitions of `i2` and `i5` .

### Compliant Solution

This compliant solution does not include conflicting definitions:

## DCL37-C. Do not declare or define a reserved identifier

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/06.declarations-and-initialization-dcl/05.dcl37-c.md) — SHA-256 `00a0caae7f05f771dd7792e64544355351394437d6e5c6755daf033ede05abb5`。

According to the C Standard, 6.4.2.1 paragraph 7 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \],

> Some identifiers are reserved.
>
> —  All identifiers that begin with a double underscore ( \_\_ ) or begin with an underscore ( \_ ) followed by an uppercase letter are reserved for any use, except those identifiers which are lexically identical to keywords.
>
> —  All identifiers that begin with an underscore are reserved for use as identifiers with file scope in both the ordinary and tag name spaces.
>
> Other identifiers may be reserved, see 7.1.3.

C Standard, 7.1.3 paragraph 1 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \],

> Each header declares or defines all identifiers listed in its associated subclause, and optionally declares or defines identifiers listed in its associated future library directions subclause and identifiers which are always reserved either for any use or for use as file scope identifiers.
>
> — All potentially reserved identifiers (including ones listed in the future library directions) that are provided by an implementation with an external definition are reserved for any use. An implementation shall not provide an external definition of a potentially reserved identifier unless that identifier is reserved for a use where it would have external linkage. All other potentially reserved identifiers that are provided by an implementation (including in the form of a macro) are reserved for any use when the associated header is included. No other potentially reserved identifiers are reserved.
>
> — Each macro name in any of the following subclauses (including the future library directions) is reserved for use as specified if any of its associated headers is included; unless explicitly stated otherwise (see 7.1.4).
>
> — All identifiers with external linkage in any of the following subclauses (including the future library directions) and `errno` are always reserved for use as identifiers with external linkage.
>
> — Each identifier with file scope listed in any of the following subclauses (including the future library directions) is reserved for use as a macro name and as an identifier with file scope in the same name space if any of its associated headers is included.

Additionally, subclause 7.33 defines many other reserved identifiers for future library directions.

No other identifiers are reserved. (The POSIX standard extends the set of identifiers reserved by the C Standard to include an open-ended set of its own. See *Portable Operating System Interface \[POSIX <sup>®</sup> \], Base Specifications, Issue 7* , [Section 2.2](http://www.opengroup.org/onlinepubs/9699919799/functions/V2_chap02.html#tag_15_02) , "The Compilation Environment" \[ [IEEE Std 1003.1-2013](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-IEEEStd1003.1-2013) \].) The behavior of a program that declares or defines an identifier in a context in which it is reserved or that defines a reserved identifier as a macro name is undefined. (See [undefined behavior 105](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_105).)

### Noncompliant Code Example (Include Guard)

A common, but noncompliant, practice is to choose a reserved name for a macro used in a preprocessor conditional guarding against multiple inclusions of a header file. (See also [PRE06-C. Enclose header files in an include guard](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/preprocessor-pre/pre06-c) .) The name may clash with reserved names defined by the implementation of the C standard library in its headers or with reserved names implicitly predefined by the compiler even when no C standard library header is included.

### Compliant Solution (Include Guard)

This compliant solution avoids using leading underscores in the macro name of the include guard:

### Noncompliant Code Example (File Scope Objects)

In this noncompliant code example, the names of the file scope objects `_max_limit` and `_limit` both begin with an underscore. Because `_max_limit` is static, this declaration might seem to be impervious to clashes with names defined by the implementation. However, because the header `<stddef.h>` is included to define `size_t` , a potential for a name clash exists. (Note, however, that a [conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conformingprogram) compiler may implicitly declare reserved names regardless of whether any C standard library header is explicitly included.)

In addition, because `_limit` has external linkage, it may clash with a symbol of the same name defined in the language runtime library even if such a symbol is not declared in any header. Consequently, it is not safe to start the name of any file scope identifier with an underscore even if its linkage limits its visibility to a single translation unit.

### Compliant Solution (File Scope Objects)

In this compliant solution, names of file scope objects do not begin with an underscore:

### Noncompliant Code Example (Reserved Macros)

In this noncompliant code example, because the C standard library header `<inttypes.h>` is specified to include `<stdint.h>` , the name `SIZE_MAX` conflicts with a standard macro of the same name, which is used to denote the upper limit of `size_t` . In addition, although the name `INTFAST16_LIMIT_MAX` is not defined by the C standard library, it is a reserved identifier because it begins with the `INT` prefix and ends with the `_MAX` suffix. (See the C Standard, 7.33.14 .)

### Compliant Solution (Reserved Macros)

This compliant solution avoids redefining reserved names or using reserved prefixes and suffixes:

### Noncompliant Code Example (Identifiers with External Linkage)

This noncompliant example provides definitions for the C standard library functions `malloc()` and `free()` . Although this practice is permitted by many traditional implementations of UNIX (for example, the [Dmalloc](https://dmalloc.com/) library), it is [undefined behavior 82](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_82) according to the C Standard. Even on systems that allow replacing `malloc()` , doing so without also replacing `aligned_alloc()` , `calloc()` , and `realloc()` is likely to cause problems.

### Compliant Solution (Identifiers with External Linkage)

The compliant, portable solution avoids redefining any C standard library identifiers with external linkage. In addition, it provides definitions for all memory allocation functions:

### Noncompliant Code Example ( `errno` )

In addition to symbols defined as functions in each C standard library header, identifiers with external linkage include `errno` and `math_errhandling` .  According to the C Standard, 7.5, paragraph 2 \[ [ISO/IEC 9899:2011](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2011) \], the behavior of a program is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) when

> A macro definition of `errno` is suppressed in order to access an actual object, or the program defines an identifier with the name `errno` .

See [undefined behavior 111](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_111).

The `errno` identifier expands to a modifiable [lvalue](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-lvalue) that has type `int` but is not necessarily the identifier of an object. It might expand to a modifiable lvalue resulting from a function call, such as `*errno()` . It is unspecified whether `errno` is a macro or an identifier declared with external linkage. If a macro definition is suppressed to access an actual object, or if a program defines an identifier with the name `errno` , the behavior is [undefined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) .

Legacy code is apt to include an incorrect declaration, such as the following:

### Compliant Solution ( `errno` )

The correct way to declare `errno` is to include the header `<errno.h>` :

[Implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) [conforming](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-conformingprogram) to C are required to declare `errno` in `<errno.h>` , although some historic implementations failed to do so.

### Exceptions

**DCL37-C-EX1:** Provided that a library function can be declared without reference to any type defined in a header, it is permissible to declare that function without including its header provided that declaration is compatible with the standard declaration.

Such code is compliant because the declaration matches what `stdlib.h` would provide and does not redefine the reserved identifier. However, it would not be acceptable to provide a definition for the `free()` function in this example.

**DCL37-C-EX2:** For compatibility with other compiler vendors or language standard modes, it is acceptable to create a macro identifier that is the same as a reserved identifier so long as the behavior is idempotent, as in this example:

**DCL37-C-EX3:** As a compiler vendor or standard library developer, it is acceptable to use identifiers reserved for your implementation. Reserved identifiers may be defined by the compiler, in standard library headers or headers included by a standard library header, as in this example declaration from the glibc standard C library implementation:

## DCL38-C. Use the correct syntax when declaring a flexible array member

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/06.declarations-and-initialization-dcl/06.dcl38-c.md) — SHA-256 `c6929ae12ee83de410bd5843cc378b3061f7a94d09a59b36609ba399cfcb09b2`。

Flexible array members are a special type of array in which the last element of a structure with more than one named member has an incomplete array type; that is, the size of the array is not specified explicitly within the structure. This "struct hack" was widely used in practice and supported by a variety of compilers. Consequently, a variety of different syntaxes have been used for declaring flexible array members. For conforming C implementations, use the syntax guaranteed to be valid by the C Standard.

Flexible array members are defined in the C Standard, 6.7.3.2, paragraph 20 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], as follows:

> As a special case, the last element of a structure with more than one named member may have an incomplete array type; this is called a *flexible array member* . In most situations, the flexible array member is ignored. In particular, the size of the structure is as if the flexible array member were omitted except that it may have more trailing padding than the omission would imply. However, when a `.` (or `->` ) operator has a left operand that is (a pointer to) a structure with a flexible array member and the right operand names that member, it behaves as if that member were replaced with the longest array (with the same element type) that would not make the structure larger than the object being accessed; the offset of the array shall remain that of the flexible array member, even if this would differ from that of the replacement array. If this array would have no elements, it behaves as if it had one element but the behavior is undefined if any attempt is made to access that element or to generate a pointer one past it.

Structures with a flexible array member can be used to produce code with defined behavior. However, some restrictions apply:

1.  The incomplete array type *must* be the last element within the structure.
2.  There cannot be an array of structures that contain a flexible array member.
3.  Structures that contain a flexible array member cannot be used as a member of another structure.
4.  The structure must contain at least one named member in addition to the flexible array member.

### Noncompliant Code Example

Before the introduction of flexible array members in the C Standard, structures with a one-element array as the final member were used to achieve similar functionality. This noncompliant code example illustrates how `   struct flexArrayStruct  ` is declared in this case.

This noncompliant code example attempts to allocate a flexible array-like member with a one-element array as the final member. When the structure is instantiated, the size computed for `malloc()` is modified to account for the actual size of the dynamic array.

This example has [undefined behavior 59](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_59) when accessing any element other than the first element of the `data` array. (See the C Standard, 6.5.7.) Consequently, the compiler can generate code that does not return the expected value when accessing the second element of data.

This approach may be the only alternative for compilers that do not yet implement the standard C syntax.

### Compliant Solution

This compliant solution uses a flexible array member to achieve a dynamically sized structure:

This compliant solution allows the structure to be treated as if its member `data[]` was declared to be `data[array_size]` in a manner that conforms to the C Standard.

## DCL39-C. Avoid information leakage when passing a structure across a trust boundary

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/06.declarations-and-initialization-dcl/07.dcl39-c.md) — SHA-256 `88b435055fa04b779f1d445e47e388f6c472fce5796f0a95bde661f7dacda458`。

The C Standard, 6.7.3.2, discusses the layout of structure fields. It specifies that non-bit-field members are aligned in an [implementation-defined](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation-definedbehavior) manner and that there may be padding within or at the end of a structure. Furthermore, initializing the members of the structure does not guarantee initialization of the padding bytes. The C Standard, 6.2.6.1, paragraph 6 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \], states

> When a value is stored in an object of structure or union type, including in a member object, the bytes of the object representation that correspond to any padding bytes take unspecified values (e.g. structure and union assignment may or may not copy any padding bits).

Additionally, the storage units in which a bit-field resides may also have padding bits. For an object with automatic storage duration, these padding bits do not take on specific values and can contribute to leaking sensitive information.

When passing a pointer to a structure across a trust boundary to a different trusted domain, the programmer must ensure that the padding bytes and bit-field storage unit padding bits of such a structure do not contain sensitive information.

### Noncompliant Code Example

This noncompliant code example runs in kernel space and copies data from `arg` to user space. However, padding bytes may be used within the structure, for example, to ensure the proper alignment of the structure members. These padding bytes may contain sensitive information, which may then be leaked when the data is copied to user space.

### Noncompliant Code Example ( `memset()` )

The padding bytes can be explicitly initialized by calling `memset()` :

However, a conforming compiler is free to implement `   arg.b = 2  ` by setting the low-order bits of a register to 2, leaving the high-order bits unchanged and containing sensitive information. Then the platform copies all register bits into memory, leaving sensitive information in the padding bits. Consequently, this implementation could leak the high-order bits from the register to a user.

### Compliant Solution

This compliant solution serializes the structure data before copying it to an untrusted context:

This code ensures that no uninitialized padding bytes are copied to unprivileged users. **Important:** The structure copied to user space is now a packed structure and the `copy_to_user()` function (or other eventual user) would need to unpack it to recreate the original padded structure.

### Compliant Solution (Padding Bytes)

Padding bytes can be explicitly declared as fields within the structure. This solution is not portable, however, because it depends on the [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) and target memory architecture. The following solution is specific to the x86-32 architecture:

The C Standard `static_assert()` macro accepts a constant expression and an [error message](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-error) . The expression is evaluated at compile time and, if false, the compilation is terminated and the error message is output. (See [DCL03-C. Use a static assertion to test the value of a constant expression](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/recommendations/declarations-and-initialization-dcl/dcl03-c) for more details.) The explicit insertion of the padding bytes into the `struct` should ensure that no additional padding bytes are added by the compiler and consequently both static assertions should be true. However, it is necessary to validate these assumptions to ensure that the solution is correct for a particular implementation.

### Compliant Solution (Structure Packing—GCC)

GCC allows specifying declaration attributes using the keyword `__attribute__((__packed__))` . When this attribute is present, the compiler will not add padding bytes for memory alignment unless an explicit alignment specifier for a structure member requires the introduction of padding bytes.

### Compliant Solution (Structure Packing—Microsoft Visual Studio)

Microsoft Visual Studio  supports `   #pragma pack()  ` to suppress padding bytes \[ [MSDN](https://msdn.microsoft.com/en-us/library/2e70t5y1(v=vs.110).aspx) \]. The compiler adds padding bytes for memory alignment, depending on the current packing mode, but still honors the alignment specified by `__declspec(align())` . In this compliant solution, the packing mode is set to 1 in an attempt to ensure all fields are given adjacent offsets:

The `pack` pragma takes effect at the first `struct` declaration after the pragma is seen.

### Noncompliant Code Example

This noncompliant code example also runs in kernel space and copies data from `   struct test  ` to user space. However, padding bits will be used within the structure due to the bit-field member lengths not adding up to the number of bits in an `unsigned` object. Further, there is an unnamed bit-field that causes no further bit-fields to be packed into the same storage unit. These padding bits may contain sensitive information, which may then be leaked when the data is copied to user space. For instance, the uninitialized bits may contain a sensitive kernel space pointer value that can be trivially reconstructed by an attacker in user space.

### Compliant Solution

Padding bits can be explicitly declared, allowing the programmer to specify the value of those bits. When explicitly declaring all of the padding bits, any unnamed bit-fields of length `0` must be removed from the structure because the explicit padding bits ensure that no further bit-fields will be packed into the same storage unit.

This solution is not portable, however, because it depends on the [implementation](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) and target memory architecture. The explicit insertion of padding bits into the `struct` should ensure that no additional padding bits are added by the compiler. However, it is still necessary to validate these assumptions to ensure that the solution is correct for a particular implementation. For instance, the DEC Alpha is an example of a 64-bit architecture with 32-bit integers that allocates 64 bits to a storage unit.

In addition, this solution assumes that there are no integer padding bits in an `   unsigned int  ` .  The portable version of the width calculation from [INT35-C. Use correct integer precisions](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/integers-int/int35-c) cannot be used because the bit-field width must be an integer constant expression.

From this situation, it can be seen that special care must be taken because no solution to the bit-field padding issue will be 100% portable.

## DCL40-C. Do not create incompatible declarations of the same function or object

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/06.declarations-and-initialization-dcl/08.dcl40-c.md) — SHA-256 `50db9c6235019b44d2d69833cf23a6feeb4ed8018bf48ae248adce3bafa07c7e`。

Two or more incompatible declarations of the same function or object must not appear in the same program because they result in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) . The C Standard, 6.2.7, mentions that two types may be distinct yet compatible and addresses precisely when two distinct types are compatible.

The C Standard identifies four situations in which [undefined behavior (UB)](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) may arise as a result of incompatible declarations of the same function or object:

| UB | Description | Code |
| --- | --- | --- |
| [14](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_14) | *Two declarations of the same object or function specify types that are not compatible (6.2.7).* | All noncompliant code in this guideline |
| [30](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_30) | *Two identifiers differ only in nonsignificant characters (6.4.2.1).* | [Excessively Long Identifiers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl40-c#DCL40C.Donotcreateincompatibledeclarationsofthesamefunctionorobject-ExcessivelyLongIdentifiers) |
| [36](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_36) | *An object has its stored value accessed other than by an lvalue of an allowable type (6.5).* | [Incompatible Object Declarations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl40-c#DCL40C.Donotcreateincompatibledeclarationsofthesamefunctionorobject-IncompatibleObjectDeclarations) <br> [Incompatible Array Declarations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl40-c#DCL40C.Donotcreateincompatibledeclarationsofthesamefunctionorobject-IncompatibleArrayDeclarations) |
| [37](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_37) | *A function is defined with a type that is not compatible with the type (of the expression) pointed to by the expression that denotes the called function (6.5.2.2).* | [Incompatible Function Declarations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl40-c#DCL40C.Donotcreateincompatibledeclarationsofthesamefunctionorobject-IncompatibleFunctionDeclarations) <br> [Excessively Long Identifiers](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/declarations-and-initialization-dcl/dcl40-c#DCL40C.Donotcreateincompatibledeclarationsofthesamefunctionorobject-ExcessivelyLongIdentifiers) |

Although the effect of two incompatible declarations simply appearing in the same program may be benign on most [implementations](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-implementation) , the effects of invoking a function through an expression whose type is incompatible with the function definition are typically catastrophic. Similarly, the effects of accessing an object using an [lvalue](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-lvalue) of a type that is incompatible with the object definition may range from unintended information exposure to memory overwrite to a hardware trap.

### <a id="DCL40C.Donotcreateincompatibledeclarationsofthesamefunctionorobject-IncompatibleObjectDeclarations"></a> Noncompliant Code Example (Incompatible Object Declarations)

In this noncompliant code example, the variable `i` is declared to have type `int` in file `a.c` but defined to be of type `short` in file `b.c` . The declarations are incompatible, resulting in [undefined behavior 14](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_14) . Furthermore, accessing the object using an [lvalue](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-lvalue) of an incompatible type, as shown in function `f()` , is [undefined behavior 36](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_36) with possible observable results ranging from unintended information exposure to memory overwrite to a hardware trap.

### Compliant Solution (Incompatible Object Declarations)

This compliant solution has compatible declarations of the variable `i` :

### <a id="DCL40C.Donotcreateincompatibledeclarationsofthesamefunctionorobject-IncompatibleArrayDeclarations"></a> Noncompliant Code Example (Incompatible Array Declarations)

In this noncompliant code example, the variable `a` is declared to have a pointer type in file `a.c` but defined to have an array type in file `b.c` . The two declarations are incompatible, resulting in [undefined behavior 14](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_13) . As before, accessing the object in function `f()` is [undefined behavior 36](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_36) with the typical effect of triggering a hardware trap.

### Compliant Solution (Incompatible Array Declarations)

This compliant solution declares `a` as an array in `a.c` and `b.c` :

### <a id="DCL40C.Donotcreateincompatibledeclarationsofthesamefunctionorobject-IncompatibleFunctionDeclarations"></a> Noncompliant Code Example (Incompatible Function Declarations)

In this noncompliant code example, the function `f()` is declared in file `a.c` with one prototype but defined in file `b.c` with another. The two prototypes are incompatible, resulting in [undefined behavior 14](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_14) . Furthermore, invoking the function is [undefined behavior 37](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_37) and typically has catastrophic consequences.

### Compliant Solution (Incompatible Function Declarations)

This compliant solution has compatible prototypes for the function `f()` :

### Noncompliant Code Example (Incompatible Variadic Function Declarations)

In this noncompliant code example, the function `buginf()` is defined to take a variable number of arguments and expects them all to be signed integers with a sentinel value of `-1` :

Although this code appears to be well defined because of the prototype-less declaration of `buginf()` , it exhibits [undefined behavior 76](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_76) in accordance with the C Standard, 6.7.7.4, paragraph 14 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \],

> For two function types to be compatible, both shall specify compatible return types. Moreover, the parameter type lists shall agree in the number of parameters and in use of the final ellipsis; corresponding parameters shall have compatible types. In the determination of type compatibility and of a composite type, each parameter declared with function or array type is taken as having the adjusted type and each parameter declared with qualified type is taken as having the unqualified version of its declared type.

### Compliant Solution (Incompatible Variadic Function Declarations)

In this compliant solution, the prototype for the function `buginf()` is included in the scope in the source file where it will be used:

### <a id="DCL40C.Donotcreateincompatibledeclarationsofthesamefunctionorobject-ExcessivelyLongIdentifiers"></a> Noncompliant Code Example (Excessively Long Identifiers)

In this noncompliant code example, the length of the identifier declaring the function pointer `bash_groupname_completion_function()` in the file `bashline.h` exceeds by 3 the minimum implementation limit of 31 significant initial characters in an external identifier. This introduces the possibility of colliding with the `bash_groupname_completion_funct` integer variable defined in file `b.c` , which is exactly 31 characters long. On an implementation that exactly meets this limit, this is [undefined behavior 30](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_30) . It results in two incompatible declarations of the same function. (See [undefined behavior 14](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_14) .) In addition, invoking the function leads to [undefined behavior 37](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_37) with typically catastrophic effects.

NOTE: The identifier `bash_groupname_completion_function` referenced here was taken from GNU [Bash](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html), version 3.2.

### Compliant Solution (Excessively Long Identifiers)

In this compliant solution, the length of the identifier declaring the function pointer `bash_groupname_completion()` in `bashline.h` is less than 32 characters. Consequently, it cannot clash with `bash_groupname_completion_funct` on any compliant platform.

## DCL41-C. Do not declare variables inside a switch statement before the first case label

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/06.declarations-and-initialization-dcl/09.dcl41-c.md) — SHA-256 `63628c247e71ef132ae4bc9e88991d922d7c3197dcb337c1d10d95520d2d23b5`。

According to the C Standard, 6.8.5.3, paragraph 4 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \],

> A switch statement causes control to jump to, into, or past the statement that is the switch body , depending on the value of a controlling expression, and on the presence of a default label and the values of any case labels on or in the switch body. A case or default label is accessible only within the closest enclosing switch statement.

If a programmer declares variables, initializes them before the first case statement, and then tries to use them inside any of the case statements, those variables will have scope inside the `switch` block but will not be initialized and will consequently contain indeterminate values.  Reading such values also violates [EXP33-C. Do not read uninitialized memory](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/expressions-exp/exp33-c) .

### Noncompliant Code Example

This noncompliant code example declares variables and contains executable statements before the first case label within the `switch` statement:

#### Implementation Details

When the preceding example is executed on GCC 4.8.1, the variable `i` is instantiated with automatic storage duration within the block, but it is not initialized. Consequently, if the controlling expression `expr` has a nonzero value, the call to `printf()` will access an indeterminate value of `i` . Similarly, the call to `f()` is not executed.

|                                     |                       |
|-------------------------------------|-----------------------|
| Value of `expr` | `Output` |
| 0                                   | 17                    |
| Nonzero                             | Indeterminate         |

### Compliant Solution

In this compliant solution, the statements before the first case label occur before the `switch` statement:

## DCL42-C. Only call functions with the unsequenced or reproducible attributes if they actually have the asserted property

**Rule / 条件付き採用**。

[固定原典](https://github.com/cmu-sei/secure-coding-standards/blob/4a2cb5ce988376f2bae03b1b269e7deb9179bf47/content/4.sei-cert-c-coding-standard/03.rules/06.declarations-and-initialization-dcl/10.dcl42-c.md) — SHA-256 `7a08f6092e73ea2169072474ed41baf4585a4b28d453c9ea69010416975968b3`。

C has function attributes `reproducible` and `unsequenced` that assert certain properties that allow compilers to optimize by changing the number of times and locations where such functions are called. The attribute `reproducible` means that a function is effectless and idempotent. The attribute `unsequenced` means that a function is stateless, effectless, idempotent and independent. The rigorous definitions of these terms can be found in the C Standard, 6.7.13.8 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \]. The paper that proposed adding these attributes to the standard \[ [JTC1/SC22/WG14 N2956](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IECJTC1-SC22-WG14N2956) \] gave these simplified versions of them:

> * stateless: function that does not define mutable static or thread-local objects (nor do functions that are called by it)
> * effectless: function that does not have observable side effects
> * idempotent: repeated evaluation gives the same result (hence may read global state)
> * independent: does not depend on other state than the arguments or constants (hence may write to globals)

It is the programmer's responsibility to ensure that functions with these attributes actually meet all of those criteria. If they do not, calling them results in [undefined behavior](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/bb-definitions#BB.Definitions-undefinedbehavior) , according to the C Standard, 6.7.13.8.1, paragraph 3 \[ [ISO/IEC 9899:2024](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/aa-bibliography#AA.Bibliography-ISO-IEC9899-2024) \]:

> if a definition that does not have the asserted property is accessed by a function declaration or a function pointer with a type that has the attribute, the behavior is undefined.

See also [undefined behavior 81](https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/back-matter/cc-undefined-behavior#CC.UndefinedBehavior-ub_81).

### Noncompliant Code Example (reproducible)

In this noncompliant code example, the `adjust()` function does not have the idempotence property that is implied by the `reproducible` attribute (repeating a call will result in changing the values by 6 and 4 instead of just by 3 and 2), so calling it results in undefined behavior.

### Compliant Solution (remove attribute)

In this compliant solution, the `reproducible` attribute has been removed.

### Compliant Solution (change behavior)

In this compliant solution, the `adjust()` operation performs a different operation that is actually idempotent (setting or clearing the same bits twice in a row won't change the value the second time).

### Noncompliant Code Example (unsequenced)

In this noncompliant code example, the `add_some()` function does not have the independence property that is implied by the `unsequenced` attribute (the `to_add` global variable has different values each time it's called), so calling it results in undefined behavior.

### Compliant Solution (replace attribute)

In this compliant solution, the `unsequenced` attribute has been removed and replaced with the weaker `reproducible` attribute, which does not assert independence.

### Compliant Solution (change behavior)

In this compliant solution, the `to_add` variable is constant, so the `add_some()` function actually is independent.
