---
name: angelscript
description: Guidelines for AngelScript script language. AngelScript syntax, handles, classes, callbacks, optional SDK libraries, and script/host integration issues.
---

# AngelScript

Use the bundled references as the authority for AngelScript language behavior. They are a curated adaptation of the official AngelScript 2.39.0 documentation, with newly written examples. They require no external documentation or network access. They are not an exhaustive C++ API reference.

## Workflow

1. Identify whether the task concerns script code, an optional SDK add-on, or the embedding application. Inspect available project code for its engine version, registered types/functions, engine options, and script loading/build path. Treat this as evidence of the host environment, not as a replacement for documented language semantics.
2. Read only the relevant references below. Do not fetch web material or assume C++, C#, or Java semantics where these references specify something different. If a required detail is not covered, state the gap rather than invent an API or undocumented behavior.
3. Implement the requested script or focused fix. Make required host registrations explicit, especially for strings, arrays, dictionaries, printing, exceptions, and file access. Follow the project's actual entry point; `main` is an example convention, not an automatically invoked language feature.
4. Validate with the project's existing host/build/run path when available. Check compiler diagnostics and execution results separately. Without a runnable host, describe the work as reference-checked, not compiled or runtime-tested.

## Choose a reference

| Task | Read |
|---|---|
| Declarations, primitive types, literals, control flow, evaluation order | [Language basics](references/language-basics.md) |
| Aliasing, null errors, value assignment, casts, const, ownership | [Handles and lifetimes](references/handles.md) |
| Parameters, defaults, overloads, callbacks, delegates, lambdas | [Functions](references/functions.md) |
| Constructors, inheritance, access control, properties, mixins | [Classes](references/classes.md) |
| Operator precedence, overload names, conversions, custom iteration | [Operators](references/operators.md) |
| Namespaces, enums, typedefs, imports, shared entities, initialization | [Modules and declarations](references/modules.md) |
| Strings, arrays, dictionaries, generic and weak references | [Library types](references/library-types.md) |
| File/socket APIs, dates, math, exceptions, coroutines, system functions | [Optional services](references/library-services.md) |
| Missing symbols, build failures, preprocessing, execution diagnostics | [Host integration](references/host-integration.md) |

## High-impact checks

- `@target = source` rebinds a handle; `target = source` can assign the referenced object's value. Use `is` / `!is` for ordinary handle identity and null checks.
- `auto` produces a handle for types that support handles. Use an explicit object type when value semantics are required.
- Parameter direction matters: `&in`, `&out`, and `&inout` have different copy/lifetime behavior. Do not translate C++ references mechanically.
- Anonymous functions use `function(...) { ... }` and cannot capture surrounding local variables. Use a delegate bound to an object when callback state is required.
- All script class methods are virtual; only one base class is allowed. Interfaces and mixins serve different purposes.
- SDK library names are not universally built in. Neither `print`, `throw`, nor `#include` should be assumed available without host support.
- This reference targets 2.39.0. In particular, `foreach`, registered variadic functions, and registered template functions arrived in 2.38.0; do not assume older hosts accept them.

When debugging, distinguish syntax/type errors, missing registrations, null/lifetime errors, and host execution failures before changing the code.
