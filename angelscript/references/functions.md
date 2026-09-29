# Functions and callbacks

Adapted from the official 2.39.0 topics: Function declarations; Parameter references; Reference return values; Function overloading; Default arguments; Function handles; Anonymous functions; Template functions; Variadic arguments.

## Declarations and calls

Define a function with its body: `int Add(int a, int b) { return a + b; }`. Ordinary global functions are visible regardless of declaration order, so forward prototypes are unnecessary. Return type alone cannot distinguish overloads, except for the special conversion operators `opConv` and `opCast`.

Named arguments use a colon: `Configure(enabled: true)`. Positional arguments cannot follow named ones. Evaluate order-sensitive argument expressions separately because calls evaluate the last argument first.

Once one parameter has a default, all later parameters must also have defaults. Default expressions can reference globally visible variables/functions, not the caller's locals. `void` can discard an output argument or supply its default.

```angelscript
void ReadPair(int input, int &out doubled, int &out tripled = void)
{
    doubled = input * 2;
    tripled = input * 3;
}

void Example()
{
    int result = 0;
    ReadPair(5, result);          // Ignore optional tripled output
    ReadPair(5, void, result);    // Ignore doubled output
}
```

## Reference direction

| Form | Documented behavior |
|---|---|
| `T value` | Pass a value; objects may be copied |
| `T &in value` | Input reference normally refers to a copy; it is not a promise to mutate the caller |
| `const T &in value` | Read-only input; can avoid copying in suitable circumstances |
| `T &out value` | Output storage is uninitialized on entry; write-back occurs after return |
| `T &inout value` or `T &value` | Refers to the actual object; ordinarily limited to reference types that can have handles |

Never read an `&out` parameter expecting the caller's previous value. Do not convert a C++ `int&` API into an ordinary script `int &inout` without checking host configuration. Choose `&out` for additional scalar outputs and handles or valid reference-type parameters for shared object access.

Returning references has separate lifetime constraints; see [handles](handles.md).

## Overload resolution

The compiler matches argument types against parameters, processing arguments from first to last, and must identify one best candidate. In increasing conversion cost, the documented order is:

1. Exact match.
2. Conversion to const.
3. Enum to integer of the same size, then enum to integer of a different size.
4. Primitive size increase, then decrease.
5. Signed to unsigned integer, then unsigned to signed.
6. Integer to floating point, then floating point to integer.
7. Reference cast.
8. Object to primitive.
9. Conversion to object.
10. Variable argument type.

For output parameters, impossible conversions back into the destination eliminate a candidate. A matching non-variadic function is preferred to a variadic one. Resolve ambiguity with explicit argument types/casts or a suitably named function, rather than assuming return context selects the overload.

## Function handles and stateful callbacks

Define the signature with `funcdef`, globally or as a class member. A function handle must match that signature. It can be null, checked with `is`, and called with ordinary function-call syntax.

```angelscript
funcdef bool Compare(int, int);

bool Ascending(int left, int right)
{
    return left < right;
}

class Ordering
{
    bool descending = false;
    bool CompareValues(int left, int right)
    {
        return descending ? left > right : left < right;
    }
}

void Example()
{
    Compare@ globalCallback = @Ascending;
    Ordering order;
    Compare@ boundCallback = Compare(order.CompareValues);
    if (boundCallback !is null)
    {
        bool before = boundCallback(1, 2);
    }
}
```

A delegate binds a method to a particular object instance using the funcdef constructor. Passing a global function through that constructor uses the function directly instead of creating a new delegate object.

## Anonymous functions

Syntax is `function(a, b) { return a < b; }`. The target function-handle signature supplies parameter and return types. Add explicit parameter types when overloads make the target ambiguous.

```angelscript
funcdef bool Predicate(int);

bool Apply(int value, Predicate@ predicate)
{
    return predicate(value);
}

bool Example()
{
    return Apply(7, function(int value) { return value > 5; });
}
```

Anonymous functions cannot capture surrounding local variables. To retain a configurable threshold or other callback state, store it on an object and bind a delegate to its method.

## Registered templates and variadic functions

For a registered template function, explicitly give its type arguments, such as `Transform<int, float>(a, b)`. The documented implementation route is host registration with the generic calling convention. Do not invent C++-style script template declarations. Template specializations are not supported by this facility.

Variadic functions are likewise registered by the application with the generic calling convention. Common declarations use `const ?&in ...`, `?&out ...`, or a fixed type such as `int ...`. The generic host API supplies the argument count. These are not grounds to invent a script implementation using C `va_list` or a rest-array parameter.
