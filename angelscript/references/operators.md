# Operators

Adapted from the official 2.39.0 topics: Operator precedence; Expressions; Operator overloads.

## Precedence

Unary operators bind more tightly than binary/ternary operators. Among unary operators, proximity to the value matters; postfix operators bind more tightly than prefix operators. Use parentheses when combining unary negation and powers instead of importing mathematical precedence assumptions.

Binary/ternary groups, highest first:

| Group | Operators |
|---|---|
| Power | `**` |
| Multiplicative | `* / %` |
| Additive | `+ -` |
| Shifts | `<< >> >>>` |
| Bitwise and | `&` |
| Bitwise xor | `^` |
| Bitwise or | `\|` |
| Relational | `<= < >= >` |
| Equality, identity, logical xor | `== != is !is xor ^^` |
| Logical and | `and &&` |
| Logical or | `or \|\|` |
| Conditional | `?:` |
| Assignment | `=` and compound assignments |

The documentation calls `>>` right shift and `>>>` arithmetic right shift. Do not substitute another language's interpretation based on the spelling alone. Bitwise operands convert to integers while retaining the original sign; the result type follows the left operand.

## Overload mapping

Implement specially named methods; do not write C++ `operator+` declarations.

| Operator | Method |
|---|---|
| Unary `-`, `~` | `opNeg`, `opCom` |
| Prefix `++`, `--` | `opPreInc`, `opPreDec` |
| Postfix `++`, `--` | `opPostInc`, `opPostDec` |
| `==`, `!=` | `opEquals` returning `bool` |
| `<`, `<=`, `>`, `>=` | `opCmp` returning `int` |
| `=` | `opAssign` |
| `+ - * / % **` | `opAdd`, `opSub`, `opMul`, `opDiv`, `opMod`, `opPow` |
| `& \| ^ << >> >>>` | `opAnd`, `opOr`, `opXor`, `opShl`, `opShr`, `opUShr` |
| `[]` | `opIndex` |
| `()` on an object | `opCall` |

Binary operations consider `left.opAdd(right)` and `right.opAdd_r(left)` (with the corresponding method name for each operator) and select the best match. Compound assignments use the binary method name plus `Assign`, for example `opAddAssign`, `opPowAssign`, `opUShrAssign`; ordinary assignment uses `opAssign`.

Equality considers either operand's `opEquals`; inequality negates it. Without `opEquals`, equality can fall back to `opCmp`. `opCmp(other)` returns a negative value when this object is smaller, zero for equal, and a positive value when larger.

The ordinary handle identity operators compare identity independently of object-value equality. The documented overload route for identity uses an `opEquals` accepting a handle so the addresses can be compared, relevant to specialized handle-like types. Avoid implementing ordinary value equality as handle identity.

If no single-parameter `opAssign` is explicitly declared, the compiler generates one that copies the members. Returning a reference to `this` permits chained assignments:

```angelscript
class Number
{
    int value = 0;
    Number &opAssign(const Number &inout other)
    {
        value = other.value;
        return this;
    }
    bool opEquals(const Number &inout other) const
    {
        return value == other.value;
    }
}
```

`opIndex` can support multiple index arguments and can return a reference for writable access. The property form uses `get_opIndex(index) const property` and `set_opIndex(index, value) property`. Accessor restrictions still apply; see [classes](classes.md).

## Conversion operators

For `Target(source)`, the compiler first looks for an appropriate target constructor, then a source `opConv()` returning the target type. For implicit value conversion, it looks for a non-explicit constructor, then `opImplConv()`. These represent new values, not ordinary aliases to the same object.

For reference conversion, use `opCast()` returning the desired handle; `opImplCast()` permits an implicit reference conversion. Supply const-preserving forms where appropriate. Do not remove const through a conversion.

For a reference type, a `bool opImplConv()` is not used automatically in conditions because it would be ambiguous whether the condition tests the object or the handle. Write an explicit comparison or conversion.

## Custom foreach containers

The compiler uses these methods:

| Purpose | Method |
|---|---|
| Initial iterator | `opForBegin()` |
| End test | `opForEnd(iterator)` |
| Next iterator | `opForNext(iterator)` |
| One yielded value | `opForValue(iterator)` |
| Multiple yielded values | `opForValue0(iterator)`, `opForValue1(iterator)`, ... |

The iterator may be an integer or an object. Iteration continues while the end test is false. Handle-capable results use handle assignments; other results use value assignments. The yielded-value methods determine the types and number of loop variables. Do not add/remove elements during iteration.
