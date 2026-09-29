# AngelScript in the PotPlayer host

This guide incorporates the language rules from an AngelScript 2.39.0 reference and reconciles them with the inspected PotPlayer scripts. It contains the relevant guidance directly; no separate language skill is required. Language-version facts do not establish PotPlayer's engine version.

## Syntax and data

Use explicit types, braces, semicolons, ordinary `for` loops, and ordinary string concatenation for broad compatibility. Script function declarations may follow their callers. Do not invent C++ headers, pointers, macros, or a mandatory `main` function. Include preprocessing is host-provided, not intrinsic to the language; keep a script self-contained unless the target already demonstrates multi-file loading.

```angelscript
class StreamChoice
{
    string url;
    string label;
    double fps = 0.0;
}

array<dictionary> MakeEmptyList()
{
    array<dictionary> result;
    return result;
}
```

`string`, `array`, `dictionary`, and numeric formatting helpers are registered library facilities. Their use is demonstrated in PotPlayer, but the entire SDK library is not thereby guaranteed. `JsonReader` and `JsonValue` are PotPlayer bindings, not AngelScript keywords. Do not substitute a JavaScript object literal for a dictionary or JSON parser.

## Handles, copying, references

- `T@` is an object handle; an uninitialized handle is null. `@destination = source` rebinds a handle; ordinary assignment can assign the referenced object's value. Use `is` and `!is` for handle identity/null checks.
- `auto` selects handles for types supporting them. Use explicit types to avoid accidental aliasing.
- `const T &in` is read-only input. `T &out` does not preserve the input value. `T &inout` or `T &name` refers to the caller's object, subject to host/reference-type constraints. Do not change the host callback signature to a by-value dictionary or array.
- Arrays and dictionaries have shallow-copy behavior: nested handles may still share objects. Construct a fresh dictionary for each independent result row.
- Do not return references to local objects. Do not rely on destructor timing to close host HTTP/file handles; they are numeric host resources, not managed AngelScript object handles.

PotPlayer examples check reference outputs with `@MetaData !is null`, despite declaring them as `dictionary &MetaData`. Preserve this host idiom; do not infer that all ordinary AngelScript references can be null.

## Containers

```angelscript
array<dictionary> rows;
dictionary row;
row["title"] = "Example item";
row["duration"] = int64(12000);
rows.insertLast(row);

string title;
if (rows.length() > 0 && rows[0].get("title", title))
    HostPrintUTF8(title);
```

Check array length before indexing, even after `isArray()` on JSON. Native array bounds are zero through `length() - 1`. Do not subtract one from an empty unsigned length or assume an unsuccessful `find` returned a valid index.

Dictionary keys are case-sensitive. `get(key, output)` reports whether retrieval succeeded; use it before acting on values. Dictionary indexing can insert a missing key and is not a safe existence test. Preserve the callback contract's numeric, boolean, string, and nested array value types.

## Strings and regular expressions

The inspected scripts use `length()`, `size()`, `empty()`, `find()`, `rfind()`, `substr()`, `split()`, `erase()`, and `replace()`. The generic SDK reference instead describes names such as `isEmpty()` and `findFirst()`. Use methods demonstrated by the target host rather than replacing PotPlayer spellings with newer SDK names.

String search returns a negative position when absent. Check it before substring operations. Treat UTF-8 indices and lengths as bytes, not Unicode character counts. Encode query components with `HostUrlEncode`; do not encode a complete assembled URL or damage its separators.

The API text lists custom `MakeLower`, `TrimRight`, and `replace` declarations with unusual const/return annotations, while examples also call them for mutation. Mutation versus returned-copy behavior needs a target-host check if correctness depends on it; do not silently infer C++ or SDK behavior. `replace` is documented to return `int`, not a replacement string.

Prefer ordinary double-quoted strings and explicit escapes. The Twitch example avoids multiline literals for host compatibility. Embedded JSON requires JSON escaping as well as AngelScript escaping; a URL encoder does not escape JSON. Keep generated JavaScript and its containing AngelScript string as separate syntax layers.

The one-string-result `HostRegExpParse` overload succeeds only with exactly one capture group according to the supplied implementation sketch. Use noncapturing groups for grouping that should not be returned. Double backslashes in ordinary AngelScript strings where a regex needs one literal backslash.

## Callbacks and version limits

Anonymous functions cannot capture surrounding local variables. A delegate bound to a script object can carry callback state when the host supports the relevant registration. Thread callback signatures are given in the bundled API; do not redeclare built-in callback types without evidence that they are missing.

`foreach`, registered variadic functions, and registered template functions arrived in AngelScript 2.38.0. Avoid assuming them in an unidentified PotPlayer build. Nor should `print`, `throw`, sockets, file add-ons, or C++ engine registration functions be assumed available in a PotPlayer script. Use the documented `Host*` interface for host services.

Separate syntax/type errors, unavailable registered symbols, reference/null errors, and service-response failures during diagnosis. Reading these rules is not a compilation test.
