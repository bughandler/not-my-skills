# Handles and lifetimes

Adapted from the official 2.39.0 topics: Objects; Object handles; Auto declarations; Function references; Reference return values; Class destructors.

## Value access versus handle operations

`T@` holds a reference to an object. An uninitialized handle is `null`; member access through it raises an exception. Primitive types cannot have handles. Application object types permit them only when registered accordingly.

```angelscript
class Counter
{
    int value = 0;
}

void Example()
{
    Counter original;
    Counter@ first = @original;
    Counter@ second;

    @second = @first;          // Same object; no object-value copy
    second.value = 7;          // Also changes original.value
    bool same = first is second;
    @first = null;             // Releases this reference only
    if (second !is null)
        second.value++;
}
```

Ordinary assignment through an existing handle operates on the referenced object when the type supplies value assignment. `@destination = source` explicitly changes the handle. The compiler can infer handle intent in some expressions, but explicit `@` prevents confusion when rebinding is essential.

For ordinary handles, `is` / `!is` test identity, including `null`. `==` / `!=` generally test object values using `opEquals` or `opCmp`; explicitly taking both handles with `@` also permits identity comparison. Do not use value equality as a null test. Special application-provided generic handle wrappers can define their own operator behavior.

## Const placement

| Declaration | Meaning |
|---|---|
| `T@ h` | Rebindable handle to a modifiable object |
| `const T@ h` | Rebindable handle through which the object is read-only |
| `T@ const h = T()` | Non-rebindable handle to a modifiable object |
| `const T@ const h = T()` | Non-rebindable handle to a read-only object |

A read-only handle is initialized at declaration. A handle to const can refer to a mutable object but cannot be converted to a handle that permits modifying it. Only const methods are callable through read-only object access.

## Lifetimes and casts

Holding a handle can keep an object alive after the declaring block exits. Clearing one handle does not destroy an object while other strong references remain. Automatic memory management and garbage collection mean destructor timing should not be used as a precise scheduling mechanism. Implement a callable cleanup method when cleanup must occur explicitly.

An upcast to an implemented interface or base class is implicit. A downcast uses `cast<Derived>(baseHandle)`; always check the result before dereferencing. Ordinary inheritance/interface reference casts expose the same object through another type, rather than copying it.

```angelscript
class Base {}
class Derived : Base { int value = 3; }

int Inspect(Base@ item)
{
    Derived@ specific = cast<Derived>(item);
    return specific is null ? 0 : specific.value;
}
```

`auto` is a frequent source of unintended aliasing: for a type that supports handles, it selects a handle rather than making a value copy. Use an explicit type and the appropriate constructor/assignment operation when a copy is required. Copies of objects containing handles can still share their referenced objects.

## Reference parameters and returns

Parameter references are not persistent local reference variables. Their direction determines preparation and write-back; see [functions](functions.md).

A reference return uses `T &Function()`; `const T &Function()` exposes a read-only value. Global variables and their reachable members can be returned by reference. A method can return its own object's members because its caller must retain the object.

Do not return references to local variables or parameters. Expressions that need local objects after their cleanup, or that involve deferred parameter processing which could invalidate the result, are also disallowed. Primitive locals may participate in computing a return expression because they require no cleanup; that does not permit returning a reference to the local itself.

For optional weak references and type-erased handles, see [library types](library-types.md).
