# Advanced Generics

## Overview

Generics are one of the most powerful features of C#.

Most developers use generics every day:

```csharp
List<Customer>

Dictionary<int, string>

Task<Order>

IEnumerable<Product>
```

However, senior engineers are expected to understand not only how to use generics, but also why they exist, how they improve performance and type safety, and what advanced capabilities they provide.

---

# 1. Why Generics?

## Interview Answer

Generics allow us to write reusable, type-safe code without sacrificing performance.

Without generics:

```csharp
ArrayList list = new();

list.Add(10);

int value = (int)list[0];
```

Problems:

- Runtime casting
- Boxing
- InvalidCastException
- No compile-time type safety

With generics:

```csharp
List<int> numbers = new();

numbers.Add(10);

int value = numbers[0];
```

Benefits:

- Compile-time type checking
- Better performance
- No casting
- Cleaner code

---

# Interview Question

### Why are generics faster than object?

## Answer

Because value types remain value types.

Using `object` causes boxing and unboxing.

Generics avoid these allocations.

---

# 2. Generic Methods

Example:

```csharp
public T Max<T>(T a, T b)
```

The method works with any type.

Usage:

```csharp
Max(5, 10);

Max("A", "B");
```

---

# 3. Generic Classes

Example:

```csharp
public class Repository<T>
{
}
```

Instead of:

```
CustomerRepository

OrderRepository

InvoiceRepository
```

One generic implementation supports multiple entity types.

---

# 4. Generic Constraints

Sometimes not every type should be allowed.

Example:

```csharp
where T : class
```

Only reference types.

---

Common constraints:

```csharp
where T : class

where T : struct

where T : new()

where T : BaseEntity

where T : IDisposable
```

Constraints allow generic code to use members guaranteed by the constraint.

---

# 5. Multiple Constraints

Example:

```csharp
where T : class,
          IEntity,
          new()
```

Meaning:

- Reference type
- Implements IEntity
- Has parameterless constructor

---

# 6. Generic Interfaces

Example:

```csharp
public interface IRepository<T>
{
}
```

This enables dependency injection:

```csharp
IRepository<Customer>

IRepository<Order>
```

without duplicating interfaces.

---

# 7. Covariance

Covariance allows assignment from a more derived type to a less derived type.

Example:

```csharp
IEnumerable<string>

↓

IEnumerable<object>
```

This works because `IEnumerable<out T>` is covariant.

Covariance applies only to output positions.

---

# 8. Contravariance

Contravariance allows assignment in the opposite direction.

Example:

```csharp
IComparer<object>

↓

IComparer<string>
```

This works because the interface consumes values.

Contravariance applies to input positions.

---

# Interview Question

### Why isn't List<T> covariant?

## Answer

Because `List<T>` allows adding items.

If a `List<string>` could become a `List<object>`, someone could insert an `int`, breaking type safety.

Read-only interfaces such as `IEnumerable<T>` can safely support covariance.

---

# 9. Open Generics

Dependency Injection supports open generic registrations.

Example:

```csharp
services.AddScoped(
    typeof(IRepository<>),
    typeof(Repository<>));
```

The container automatically creates:

```
Repository<Customer>

Repository<Order>

Repository<Product>
```

No manual registration required.

---

# 10. Generic Performance

Generics reduce:

- Boxing
- Casting
- Runtime errors

They also allow the JIT compiler to generate optimized code for many value-type scenarios.

---

# 11. Common Interview Questions

### Are generics resolved at compile time?

## Answer

Type checking happens at compile time.

The CLR also maintains generic type information at runtime, unlike Java's type erasure.

---

### Can static members exist in generic classes?

Yes.

Each closed generic type has its own static fields.

Example:

```
Cache<int>

↓

Own static fields

Cache<string>

↓

Different static fields
```

---

### Can generic constraints improve performance?

Indirectly.

Constraints allow more specialized implementations and eliminate unnecessary runtime checks.

Their primary purpose is correctness and expressiveness.

---

### Why are generics important for Dependency Injection?

They allow reusable services to work with many types while preserving compile-time type safety.

---

# Trade-offs

| Feature | Benefit | Limitation |
|----------|---------|------------|
| Generic Classes | Reuse | Additional abstraction |
| Constraints | Safer APIs | Reduced flexibility |
| Covariance | Flexible read-only APIs | Output only |
| Contravariance | Flexible consumers | Input only |
| Open Generics | Cleaner DI registration | Slightly harder to understand initially |

---

# Staff Engineer Insight

One common mistake is creating a generic abstraction simply because it seems reusable.

Ask first:

- Does this abstraction model a real concept?
- Will multiple implementations actually use it?
- Does it simplify or complicate the code?

Another common mistake is introducing a generic repository over Entity Framework without a clear need.

EF Core already implements many repository and unit-of-work behaviors.

A generic repository should solve a real problem, not duplicate the framework.

Good abstractions remove duplication while preserving clarity.

Poor abstractions hide useful behavior behind unnecessary generic layers.

---

# Summary

- Generics provide type safety without sacrificing performance.
- They eliminate most boxing and runtime casting.
- Constraints define which types are allowed.
- Covariance and contravariance improve API flexibility.
- Open generics integrate naturally with Dependency Injection.
- Generic abstractions should solve real problems, not exist for their own sake.
