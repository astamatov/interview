# dynamic and the Dynamic Language Runtime (DLR)

## Overview

C# is a statically typed language.

Normally, the compiler knows the type of every variable and validates method calls before the application runs.

The `dynamic` keyword changes this behavior.

Instead of performing type checking at compile time, member resolution is deferred until runtime.

The Dynamic Language Runtime (DLR) provides the infrastructure that makes this possible.

---

# 1. What is dynamic?

## Interview Answer

`dynamic` tells the C# compiler to postpone type checking until runtime.

Example:

```csharp
dynamic customer = new Customer();

customer.Print();
```

The compiler accepts the call without verifying that `Print()` actually exists.

The validation happens when the code executes.

---

# 2. object vs dynamic vs var

These three keywords are often confused.

---

## object

```csharp
object customer = new Customer();
```

Compile-time type:

```
object
```

Calling:

```csharp
customer.Print();
```

Produces a compile-time error unless the object is cast.

---

## var

```csharp
var customer = new Customer();
```

The compiler infers the type.

Compile-time type:

```
Customer
```

Everything remains statically typed.

---

## dynamic

```csharp
dynamic customer = new Customer();

customer.Print();
```

Compilation succeeds.

The runtime determines whether the method exists.

---

# Interview Question

### Is var dynamic?

## Answer

No.

`var` performs compile-time type inference.

After compilation, the variable has a concrete static type.

`dynamic` postpones member resolution until runtime.

---

# 3. Runtime Binding

With normal C#:

```
Compile

↓

Validate

↓

Execute
```

With `dynamic`:

```
Compile

↓

Execute

↓

Resolve Members
```

Errors move from compile time to runtime.

---

# Example

```csharp
dynamic customer = new Customer();

customer.UnknownMethod();
```

Compilation succeeds.

Runtime:

```
RuntimeBinderException
```

---

# 4. Dynamic Language Runtime (DLR)

## Interview Answer

The DLR extends the CLR with runtime binding capabilities.

Responsibilities include:

- Dynamic method invocation
- Dynamic property access
- Dynamic operators
- Call site caching

The DLR is the infrastructure behind `dynamic`.

---

# 5. ExpandoObject

`ExpandoObject` allows adding properties at runtime.

Example:

```csharp
dynamic person =
    new ExpandoObject();

person.Name = "John";

person.Age = 30;
```

The object shape is determined dynamically.

---

Typical use cases:

- Scripting
- Dynamic JSON
- Rapid prototyping

---

# 6. DynamicObject

Developers can customize runtime behavior by inheriting from `DynamicObject`.

Example:

```csharp
class MyDynamic
    : DynamicObject
{
}
```

Override methods such as:

- TryGetMember
- TrySetMember
- TryInvokeMember

This enables highly dynamic APIs.

---

# 7. Interoperability

`dynamic` is useful when interacting with technologies that lack strong static typing.

Examples:

- COM automation
- Office Interop
- Dynamic scripting languages
- Some JSON or XML scenarios

It can reduce the amount of casting required.

---

# 8. Performance

`dynamic` introduces additional runtime work.

Every dynamic call may require:

- Runtime binding
- Metadata inspection
- Member lookup

The DLR caches call sites to reduce repeated lookup costs, but dynamic dispatch is still generally slower than normal method calls.

---

# 9. When Should dynamic Be Used?

Good candidates:

- Office automation
- Plugin systems
- Dynamic scripting
- Runtime-generated objects
- Legacy COM APIs

Poor candidates:

- Business logic
- Performance-critical code
- Public APIs
- Domain models

---

# 10. Common Interview Questions

### Is dynamic faster than Reflection?

## Answer

Sometimes.

The DLR caches binding information, which can make repeated dynamic calls faster than repeatedly performing uncached reflection.

However, both are generally slower than direct method calls.

---

### Does dynamic disable IntelliSense?

In many cases, yes.

Because the compiler does not know the members at compile time, tooling support is reduced.

---

### Does dynamic eliminate compile-time errors?

No.

It postpones many type-related errors until runtime.

---

### Can generics replace dynamic?

Not always.

Generics require compile-time type information.

`dynamic` is intended for scenarios where types are unknown until runtime.

---

# Trade-offs

| Feature | Benefit | Limitation |
|----------|---------|------------|
| dynamic | Flexible runtime behavior | Runtime errors |
| var | Cleaner syntax | Still statically typed |
| object | Universal base type | Requires casting |
| ExpandoObject | Runtime properties | No compile-time safety |
| DynamicObject | Custom dynamic behavior | Increased complexity |

---

# Staff Engineer Insight

One of the biggest mistakes developers make is using `dynamic` to avoid proper modeling.

Example:

```csharp
dynamic order;
```

instead of defining:

```csharp
class Order
{
}
```

This removes many advantages of C#:

- Compile-time validation
- Refactoring support
- IntelliSense
- Static analysis

Use `dynamic` only when the type genuinely cannot be known until runtime.

If the structure is known, prefer strongly typed models.

Another common misconception is that `dynamic` is simply a more convenient `object`.

In reality, `dynamic` changes how the compiler generates code, relying on the DLR to resolve operations at runtime.

---

# Summary

- C# is statically typed by default.
- `dynamic` postpones member resolution until runtime.
- `var` performs compile-time type inference.
- `object` retains compile-time type safety but requires casting.
- The DLR enables dynamic binding and caches call sites.
- `ExpandoObject` and `DynamicObject` support runtime-defined behavior.
- Prefer strong typing for application code.
- Use `dynamic` only when runtime flexibility is truly required.
