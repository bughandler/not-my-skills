# Optional SDK library types

Adapted from the official 2.39.0 topics: Standard library; string; array; dictionary; ref; weakref. Every type and helper here requires the corresponding host registration. A host may provide a different implementation or expose only a subset.

## String

The SDK string methods generally operate on bytes, not decoded Unicode characters. `length()`, indexing, substring positions, and search positions must not be treated as character or grapheme counts for UTF-8 text. Comparisons use byte values rather than locale-aware collation.

Common operations:

| Operation | API |
|---|---|
| Size | `uint length() const`, `bool isEmpty() const`, `void resize(uint)` |
| Substring | `string substr(uint start = 0, int count = -1) const` |
| Edit | `void insert(uint pos, const string &in other)`, `void erase(uint pos, int count = -1)` |
| Search | `int findFirst(const string &in str, uint start = 0) const`, `int findLast(const string &in str, int start = -1) const` |
| Character-set search | `findFirstOf`, `findFirstNotOf`, `findLastOf`, `findLastNotOf` |
| Split | `array<string>@ split(const string &in delimiter) const` |
| Join (global) | `string join(const array<string> &in arr, const string &in delimiter)` |

Search returns a negative value when absent. Indexing accesses a byte. Assignment copies content; `+` and `+=` concatenate. Primitive values can be converted by string assignment/concatenation.

Global `parseInt` / `parseUInt` support base 10 or 16 and an optional output byte count; they return `int64` / `uint64`. `parseFloat` returns `double`, also with an optional output byte count. `formatInt`, `formatUInt`, and `formatFloat` accept formatting options and width; the floating-point form also accepts precision. Options include `l` (left justify), `0` (zero padding), `+`, a space for positive values, integer hex `h`/`H`, and floating-point exponent `e`/`E`.

The documented newer helpers include `string format(const string &in fmt, const ?&in ...)`, replacing `{}` markers with primitive/string values, and `uint scan(const string &in str, ?&out ...)`, returning how many values were parsed. Verify they were registered by the host.

`regexFind` uses ECMAScript-style regular expressions and byte-based matching, with an optional output match length. Detailed regex syntax is outside this bundled reference; do not assume Unicode character classification.

## Array

```angelscript
array<int> empty;
array<int> sized(3);
array<int> repeated(3, 7);
array<int> values = {3, 1, 2};
array<array<int>> matrix = {{1, 2}, {3, 4}};
```

Use `array<T>` unless the host's alternate array syntax is known. The array itself is a reference type even for primitive elements. A handle such as `array<int>@` can avoid copying the array. Assignment is a shallow copy of contents; handles within a copied array still refer to their objects.

Indices range from zero through `length() - 1`. Out-of-range indexing raises an exception. For handle elements, rebind explicitly: `@items[index] = object`.

| Need | Methods |
|---|---|
| Size | `uint length() const`, `void resize(uint)` |
| Append/insert | `insertLast(value)`, `insertAt(index, value)`, `insertAt(index, anotherArray)` |
| Remove | `removeLast()`, `removeAt(index)`, `removeRange(start, count)` |
| Order | `reverse()`, `sortAsc()`, `sortDesc()`; sort variants also accept start/count |
| Search value | `find(value)` or `find(start, value)` |
| Search identity | `findByRef(value)` or `findByRef(start, value)` |

Search returns an `int` index, negative when absent. `find` uses object `opEquals`/`opCmp` and skips null entries for handle arrays. `findByRef` matches the address. Object sorting needs `opCmp`, or a custom comparator passed to `sort`.

```angelscript
// Requires SDK array registration.
array<int> values = {3, 1, 2};
values.sort(function(a, b) { return a < b; });
```

The comparator returns true when its first argument belongs before its second. An explicitly declared integer comparator uses `bool Less(const int &in a, const int &in b)`. For object handles the documented parameter form is `const T@ const &in`.

`foreach(auto value, auto index : values)` yields a value and optional index. Modify entries by index; do not resize the array during iteration.

## Dictionary

A dictionary maps string keys to values of arbitrary types. It is a reference type. Assignment shallow-copies its content.

```angelscript
// Requires SDK string and dictionary support.
dictionary data = {{"count", 3}};
int count = 0;
if (data.get("count", count))
    data.set("count", count + 1);
```

- `set(key, value)` adds or replaces a value. `get(key, output)` returns success only when the stored value can be retrieved as the requested type. Check its boolean result.
- `exists(key)`, `delete(key)`, `deleteAll()`, `isEmpty()`, and `getSize()` support membership and removal.
- `getKeys()` returns `array<string>@` with unspecified order.
- `data[key]` returns a `dictionaryValue` reference. A missing key is inserted with a null value, so indexing is not a non-mutating existence check.
- `int(data[key])` performs value conversion. If no conversion exists, the documented result is uninitialized; prefer checked `get` when compatibility is uncertain.
- `cast<T>(data[key])` retrieves a compatible stored handle, otherwise null. Use `@data[key] = object` to store a handle and ordinary assignment to store a value.
- `dictionaryValue` itself is a value type and cannot be held by an object handle.
- `foreach(auto value, auto key : data)` yields a value and optional key; update existing values via their keys. Do not insert/delete keys during iteration.

## Generic and weak references

`ref` is an optional generic object-handle wrapper for unrelated reference types. Use `ref@`, handle assignment, identity checks, and `cast<T>`; an incompatible cast returns null. It is not a universal primitive-value variant.

`weakref<T>` observes an object without keeping it alive. `const_weakref<T>` observes a read-only object. Their `get()` returns a strong handle or null if the object is already dead; check and retain that returned handle while using the object.

```angelscript
// Requires the SDK weakref registration.
class Item {}
void Example()
{
    Item@ strong = Item();
    weakref<Item> observed(strong);
    @strong = null;
    Item@ available = observed.get(); // null after the last strong reference is gone
}
```

Weak-reference value assignment copies the weak reference; handle assignment changes the object it observes. These semantics differ from an ordinary strong object handle.
