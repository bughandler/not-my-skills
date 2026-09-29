# Script/host integration and diagnostics

Adapted from the official 2.39.0 topics: Your first script; Compiling scripts; Calling a script function; Good practices; Script builder; Script modules; Template functions; Variadic arguments; Custom options. This is a troubleshooting guide, not a complete native API specification.

## Identify the boundary

AngelScript is embedded. The application registers the callable functions, properties, and types; loads source; compiles modules; selects entry points; and executes contexts. A syntactically valid script can still fail because its host lacks the required interface.

For an unknown symbol, first determine whether it is:

- A missing script declaration or wrong namespace.
- A library type/helper the host did not register.
- A host-specific API with a different declaration.
- A feature unavailable in the host's engine version/options.
- A declaration in another module that requires import binding or shared entities.

Do not add guessed native registrations or unrelated SDK add-ons merely to silence a script error. Inspect the intended host interface and change the responsible layer within the requested scope.

## Host options that change language behavior

The language references describe ordinary documented behavior. When the project behaves differently, inspect its `SetEngineProperty` calls before rewriting code. Relevant options include:

| Engine property | Effect to check |
|---|---|
| `asEP_ALLOW_UNSAFE_REFERENCES` | Allows primitive/value-type inout references beyond the normal restrictions; do not enable it just to make a signature compile |
| `asEP_DISALLOW_VALUE_ASSIGN_FOR_REF_TYPE` | Rejects reference-type value assignments that would otherwise be valid |
| `asEP_PROPERTY_ACCESSOR_MODE` | Can disable accessors, restrict them to registered functions, or retain legacy automatic get_/set_ recognition |
| `asEP_REQUIRE_ENUM_SCOPE` | Requires enum values to be qualified by the enum type |
| `asEP_DISALLOW_GLOBAL_VARS` | Disables script global-variable declarations |
| `asEP_USE_CHARACTER_LITERALS`, `asEP_STRING_ENCODING`, `asEP_HEREDOC_TRIM_MODE` | Change literal interpretation, encoding, or heredoc trimming |
| `asEP_ALLOW_UNICODE_IDENTIFIERS` | Allows identifiers beyond the ordinary ASCII spelling |
| `asEP_DISABLE_INTEGER_DIVISION` | Makes integer `/` and `/=` use floating-point division |
| `asEP_FOREACH_SUPPORT` | Can disable foreach support for compatibility |
| `asEP_MEMBER_INIT_MODE` | Selects current member initialization or pre-2.38.0 behavior |
| `asEP_ALWAYS_IMPL_DEFAULT_CONSTRUCT`, `asEP_ALWAYS_IMPL_DEFAULT_COPY_CONSTRUCT`, `asEP_ALWAYS_IMPL_DEFAULT_COPY` | Control generation of default construction, copy construction, and assignment |
| `asEP_INIT_GLOBAL_VARS_AFTER_BUILD` | Controls automatic global initialization after building a module |

Do not conflate a documented engine option with a script-level keyword or an option that the application actually enabled.

## Build pipeline

The documented native sequence is:

1. Create an engine with `asCreateScriptEngine()`.
2. Set a message callback with `SetMessageCallback` early, so registration and compile errors have readable diagnostics.
3. Register the application's interface and required add-ons; check registration return codes. Negative codes indicate errors. Failed configuration can make later builds return `asINVALID_CONFIGURATION`.
4. Obtain a module, add source sections using `AddScriptSection`, then call `Build`; or use `CScriptBuilder` to load/preprocess and build.
5. Resolve the exact entry point with `GetFunctionByDecl`, verifying that it exists before execution.

The engine does not load source files itself. Multiple sections compile together, resolving declarations independent of section order.

## Preprocessing belongs to the host/build helper

`CScriptBuilder` supports include directives, `#if`/`#endif`, pragmas, and metadata. These should not be presented as an unconditional C/C++ preprocessor built into the language.

- Default include resolution is relative to the including file. A custom include callback can load sections differently.
- Conditional words come from the application's `DefineWord` calls. Do not infer arbitrary C-preprocessor macro expansion from this support.
- Pragmas require a registered pragma callback; otherwise they are build errors.
- The builder removes `#!` lines to support interpreter directives.
- Bracketed metadata is stripped before compilation and retained for the host's interpretation. It does not have universal attribute semantics.

The builder workflow is `StartNewModule`, `AddSectionFromFile` or `AddSectionFromMemory`, then `BuildModule`, checking results at each step.

## Execution and lifetime

A context call follows `Prepare(function)`, argument setters, `Execute()`, then return-value retrieval only on `asEXECUTION_FINISHED`.

- Argument indices start at zero. Choose setters for the declared types: `SetArgDWord`, `SetArgQWord`, `SetArgFloat`, `SetArgDouble`, `SetArgByte`, or `SetArgWord` for appropriate primitives.
- `SetArgAddress` supplies primitive references; `SetArgObject` supplies registered objects. Passing an object by value makes a copy according to the function declaration.
- A return object obtained with `GetReturnObject` is still owned/referenced by the context. Copy it or add the appropriate reference before releasing or reusing the context.
- `asEXECUTION_SUSPENDED` means execution can resume by calling `Execute` again. It is not a completed call with a valid return value.
- `asEXECUTION_EXCEPTION` requires exception diagnostics. Use `GetExceptionString`, `GetExceptionFunction`, and `GetExceptionLineNumber`; stack and variable inspection are also available through the context APIs.
- Release contexts when finished and shut down the engine with `ShutDownAndRelease` when the application is done.

## Diagnose and validate

| Symptom | First checks |
|---|---|
| Build reports invalid configuration | Earlier registration return codes and message callback output |
| No matching function/ambiguous overload | Exact parameter types, reference directions, constness, default/named arguments |
| Null-handle exception | Construction, explicit handle assignment, failed casts, weak-reference results, initialization order |
| Import call fails | Whether the application bound the imported function |
| Callback cannot access a local | Anonymous functions have no captures; bind state through a delegate |
| Value changes unexpectedly through `auto` | Inferred handle aliasing versus an intended object copy |
| Property increment fails | Accessor restrictions; expand into separate read/write operations |
| UTF-8 slicing breaks text | SDK string operations are byte-based |
| New syntax fails on an older host | Engine version and configuration; this skill targets 2.39.0 |

Use an existing project harness to compile a minimal reproducer with the same registrations/options, then exercise the affected path. Do not claim a successful compile or runtime result from reading the reference alone. Native ABI, registration ownership, and engine APIs beyond this guide need evidence from the project; if that evidence is absent, report the coverage gap.
