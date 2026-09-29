# Language basics

Adapted from the official 2.39.0 topics: Primitive types; Strings; Auto declarations; Reserved keywords and tokens; Statements; Expressions. Examples here are newly written.

## Types and initialization

| Type | Meaning |
|---|---|
| `void` | No return value; not a variable type |
| `bool` | `true` or `false` |
| `int8`, `int16`, `int`, `int64` | Signed 8-, 16-, 32-, 64-bit integers |
| `uint8`, `uint16`, `uint`, `uint64` | Unsigned equivalents |
| `int32`, `uint32` | Aliases of `int`, `uint` |
| `float`, `double` | Floating-point values; approximately 6 and 15 significant digits in the documented IEEE 754 representation |

Prefer 32-bit integer locals unless the application interface or required range calls for another width. Numeric conversion may lose range or precision.

Local primitive variables without an initializer have an indeterminate value. Initialize them before reading. Handles default to `null`; object variables use their default constructor. A `const` variable cannot change after initialization.

```angelscript
int count = 0;
const float scale = 0.5f;
auto total = 18;               // int
auto fraction = 18 + 5.f;      // float
```

`auto` requires assignment-style initialization. For handle-capable objects it selects a handle, even without `@`; `auto@` may make that intention explicit. Do not use `auto` when an object value is required. Auto handles cannot declare class members because their resolution depends on the constructor.

Script classes are reference types. Application-registered object types may be value types or reference types; not all allow handles. See [handles](handles.md).

## Tokens and literals

Identifiers use letters/underscores followed by letters, digits, or underscores. Comments use `//` or `/* ... */`. Examples of numeric literals include `123`, `123.5`, `1.25e3`, `1.25f`, `0xFF`, `0d123`, `0o77`, and `0b1010`.

Reserved words:

```text
and auto bool break case cast catch class const continue default
do double else enum false float for foreach funcdef if import
in inout int interface int8 int16 int32 int64 is mixin namespace
not null or out private protected return switch true try typedef
uint uint8 uint16 uint32 uint64 using void while xor
```

Context-sensitive words include `abstract`, `delete`, `explicit`, `external`, `final`, `from`, `function`, `get`, `override`, `property`, `set`, `shared`, `super`, and `this`. The host may reserve additional names.

Strings require host registration. Both single and double quotation marks delimit ordinary strings; do not automatically interpret a single-quoted literal as a C++ character. Escapes include `\0`, `\\`, `\'`, `\"`, `\n`, `\r`, `\t`, `\xFFFF` (one to four hex digits), `\uFFFF`, and `\UFFFFFFFF`. For 8-bit strings, a hex byte escape must fit in 255. Unicode escapes encode to UTF-8 or UTF-16 according to the host's settings; surrogate code points and values above U+10FFFF are invalid.

Adjacent string literals separated by whitespace or comments concatenate. Triple-double-quoted heredoc strings span lines and do not process escapes. Whitespace-only text before the first line break and after the final line break is removed at the respective edges.

```angelscript
// Requires a registered string type.
string message = "first\n"
                 "second\n";
string raw = """
Path text with literal backslashes: a\b\c
""";
```

## Statements and scope

Declarations precede use within their block. Nested blocks can see outer variables and can shadow their names. Expression statements end with `;`. Functions with a non-void return type must return a compatible value; `return;` exits a void function.

Conditions for `if`, `while`, `do-while`, and `for` evaluate to `bool`. `switch` uses signed or unsigned integer expressions and compile-time constant case values. Cases fall through unless terminated. `break` exits the smallest enclosing loop or switch; `continue` advances the smallest enclosing loop.

```angelscript
int SumBelow(int limit)
{
    int result = 0;
    for (int i = 0; i < limit; i++)
        result += i;
    return result;
}
```

`foreach` requires a container that supplies the iteration operators; it is not a universal operation on arbitrary objects. The container determines the number and types of loop values. In the SDK array, the optional second value is the index; in the dictionary it is the key.

```angelscript
// Requires the SDK array registration and a host supporting foreach.
array<int> values = {2, 4, 6};
foreach (auto value, auto index : values)
    values[index] = value * 2;
```

Adding or removing elements during `foreach` has undefined behavior. Updating an existing element through its index/key is shown in the documented array/dictionary usage; assigning a loop variable is not the documented way to update the container.

Exception syntax is `try { ... } catch { ... }`, without a typed catch parameter. Null handle access, division by zero, and host functions can raise exceptions. Explicit `throw` and exception-message retrieval are optional library functions; see [services](library-services.md).

## Expressions that differ from common assumptions

- Assignment evaluates its right-hand expression before its left-hand expression.
- Function argument expressions evaluate in reverse order: the last argument first. Split side effects into statements when order matters.
- `and`/`&&` and `or`/`||` short-circuit. Logical exclusive-or is `xor`/`^^`; bitwise exclusive-or is `^`.
- Exponentiation is `**`, with `**=` for compound assignment.
- Objects and handles use `.` for member access.
- `cast<T>(object)` performs a reference cast and returns `null` if the actual object is incompatible. `T(value)` performs value conversion/construction.
- The conditional expression `condition ? a : b` needs compatible branches; conversion uses the same cost principles as overload resolution. Equal-cost ambiguity is an error. It can be an lvalue when both branches are lvalues of the same type.
- Anonymous initialization lists can omit the type only when the expected type is unambiguous. Use `array<int> = {1, 2}` or `dictionary = {{'key', 1}}` to disambiguate a call argument.

See [operators](operators.md) for precedence and [functions](functions.md) for calls and overloads.
