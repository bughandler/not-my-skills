# Modules and global declarations

Adapted from the official 2.39.0 topics: Script modules; Global variables; Namespaces; Enums; Typedefs; Interfaces; Imports; Shared script entities.

## Module boundaries

A module is an independent scope of functions, globals, and types. All contexts executing that module share its globals. Compiling identical source in two modules normally produces distinct functions, types, and global storage. Shared entities are an explicit exception.

Several source sections can form one module; section order does not determine function visibility. The application decides which sections to load and when to build them. A source filename is not itself a module import mechanism.

Global values persist between function calls. Primitive globals initialize before non-primitive globals. Avoid initialization expressions or constructors that call code depending on other non-primitive globals: their order is not guaranteed, and uninitialized access can cause unpredictable behavior or a null exception.

## Namespaces and type declarations

Use `namespace Name { ... }` and qualified names such as `Outer::Inner::value`. `::value` explicitly selects global scope. Parent namespace names remain visible unless shadowed. `using namespace Name;` can apply to the enclosing namespace, or to subsequent statements within the containing statement block.

```angelscript
namespace Config
{
    enum Mode { Idle, Active = 4, Paused }
    typedef double Scalar;
    int selected = 0;
}

void Reset()
{
    Config::selected = 0;
}
```

An enum starts at zero unless explicitly assigned; subsequent implicit values increment the preceding value. Expressions can define values. An underlying primitive type can be specified, for example `enum Flags : uint8 { None = 0, Enabled = 1 }`. An enum variable can hold a value not listed in its declaration, so handle unexpected values when branching.

`typedef` aliases primitive types only in this documented version. Use `funcdef` for function signatures; do not invent typedef aliases for classes or containers.

Interfaces declare a contract, e.g. `interface Task { void Run(); }`. A class can implement several interfaces. Use interface handles for polymorphic access; a concrete implementation must provide their methods.

## Imports

```angelscript
import int Compute(int input) from "worker";
```

An import permits compilation before the implementation is available. The application must bind it to a function, and can later unbind it. Calling an unbound import raises a script exception. The string names a module according to the host's binding scheme; it does not cause a file load by itself.

For include directives and build preprocessing, see [host integration](host-integration.md).

## Shared and external entities

`shared` gives entities the same identity across modules and reduces duplicate implementation storage. Classes, interfaces, functions, enums, and funcdefs can be shared. Global variables cannot.

```angelscript
shared class Message
{
    int code = 0;
}
shared int Normalize(int value) { return value < 0 ? 0 : value; }
```

A shared entity cannot access non-shared script entities belonging exclusively to a module. Every definition of the same shared entity must be consistent; subsequent conflicting definitions fail compilation.

If a shared entity has already been compiled in another module, reference it without repeating its body:

```angelscript
external shared class Message;
external shared int Normalize(int value);
```

An external shared declaration fails if the entity has not previously been compiled. Host-managed modules and binding remain necessary; `shared` does not automatically provide a general module loader.
