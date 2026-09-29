# Classes, properties, and mixins

Adapted from the official 2.39.0 topics: Script classes; Class constructors; Class destructors; Class methods; Inheritance and polymorphism; Protected and private class members; Initialization of class members; Property accessors; Mixin classes.

## Members and construction

Script classes are reference types with automatic memory management. Members are public by default. Prefix each restricted declaration with `private` or `protected`; do not substitute C++ access-label blocks. Derived classes can access protected members but not private ones. `this.member` disambiguates a member hidden by a local.

```angelscript
class Counter
{
    private int current = 0;

    Counter() {}
    Counter(int initial) { current = initial; }
    int Read() const { return current; }
    void Add(int amount) { current += amount; }
}
```

Constructors have the class name and no return type. A single-argument constructor permits implicit conversion unless its signature ends with `explicit`. One constructor cannot call another constructor; factor common logic into a method instead.

The default constructor is generated only if no constructor is explicitly declared. The compiler generates a copy constructor if none is explicitly declared and its members can be copied. Failed generation of that copy constructor is suppressed rather than reported as an error in isolation. A generated copy uses member copy construction, or default construction followed by assignment when necessary.

The documented copy-constructor signature is `T(const T &inout other)`. Unwanted generated operations can be excluded:

```angelscript
class Restricted
{
    Restricted(int value) {}
    Restricted() delete;
    Restricted(const Restricted &inout) delete;
    Restricted &opAssign(const Restricted &inout) delete;
}
```

A destructor is `~T() { ... }`. It runs only once, even if it resurrects the object by adding a reference. It cannot be called directly. Use a public cleanup method when the caller needs explicit timing.

## Initialization order

Member initialization order deserves explicit review when members depend on each other:

- Within a simple class, members without explicit initializers are initialized in declaration order, followed by explicitly initialized members in declaration order.
- With inheritance, the documented sequence is derived members without explicit initializers, base members, then derived members with explicit initializers.
- Explicit `super(...)` calls and member initializations in a constructor body can defer the affected initialization until those statements. A member initialized conditionally must be initialized in both branches.
- A base constructor can call a virtual method overridden by the derived class; that override may access derived members before initialization.

The host can alter member-initialization behavior through engine options, including behavior retained for compatibility with versions before 2.38.0. When diagnosing a constructor failure, inspect the actual option rather than treating one order as universal across all hosts.

## Inheritance and interfaces

Only one base class is allowed, but a class can implement multiple interfaces. All script class methods are virtual automatically. Use `Base::Method()` to call a base implementation and `super(arguments)` for a base constructor. If no base constructor is explicitly called, the compiler inserts a default-constructor call. The base destructor follows the derived destructor automatically.

```angelscript
interface Readable
{
    int Read() const;
}

class Base
{
    protected int current = 0;
    Base(int value) { current = value; }
    int Read() const { return current; }
}

class Derived : Base, Readable
{
    Derived() { super(5); }
    int Read() const override { return Base::Read() + 1; }
}
```

`override` asks the compiler to verify that a matching base method exists. `final` on a class prevents derivation; `final` on a method prevents overriding it. An `abstract class` cannot be instantiated, but its methods still need implementations: abstract methods are not supported by the documented language.

Read-only object access permits only const methods. Mutable object access can call either const or non-const overloads, preferring the non-const overload when otherwise matching.

## Virtual properties

Property accessors require host support. Block syntax and explicit accessor methods are equivalent:

```angelscript
class Settings
{
    private int stored = 1;
    int level
    {
        get const { return stored; }
        set { stored = value; }
    }
}

// Alternative member declarations inside a class:
// int get_level() const property { return stored; }
// void set_level(int value) property { stored = value; }
```

The setter receives an implicit `value` parameter. Explicit `get_`/`set_` functions need the `property` decorator, and getter return/setter parameter types must agree. Omitting a setter makes the property read-only; omitting a getter makes it write-only. Interfaces can use `int level { get const; set; }`. Global accessors follow the same rules without object-method const qualification.

`++` and `--` are not supported on virtual properties. A compound assignment is supported for a global property or a property on a reference-type owner, but not on a value-type owner. Indexed properties do not support compound assignment; expand the read and write into separate steps.

An indexed accessor takes the index first, then the new value for a setter. For example, `get_item(int index) property` and `set_item(int index, int value) property` expose `item[index]`. See [operators](operators.md) for `get_opIndex` / `set_opIndex` on the object itself.

## Mixins

`mixin class` declares reusable members, not an instantiable type. Include it in a class's base/interface list. Its methods compile in the including class's context and may use members supplied by that class.

```angelscript
mixin class Counted
{
    int count = 0;
    void Increment() { count++; }
}
class Worker : Counted {}
```

Explicitly declared members in the including class take precedence. Mixin methods override inherited base methods, but mixin properties do not replace already inherited properties. A mixin can require interfaces, with methods supplied by the mixin or including class. A mixin cannot inherit another class.
