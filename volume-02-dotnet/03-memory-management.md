# Memory Management

## Overview

Efficient memory management is one of the key characteristics of high-performance .NET applications.

Although the .NET Runtime automatically manages memory through the Garbage Collector, developers are still responsible for writing allocation-efficient code.

Understanding how memory works allows developers to:

- Reduce GC pressure
- Improve application throughput
- Lower memory usage
- Avoid memory leaks
- Diagnose production performance issues

---

# 1. Stack vs Heap

## Interview Answer

The two primary memory areas are:

- Stack
- Managed Heap

### Stack

The stack stores:

- Local variables
- Method parameters
- Value types (in most cases)
- References to heap objects

Characteristics:

- Very fast
- Automatically cleaned up
- Thread-local

Example:

```csharp
int number = 10;
```

The integer is typically stored on the stack.

---

### Heap

The managed heap stores:

- Objects
- Arrays
- Strings
- Collections

Example:

```csharp
var customer = new Customer();
```

The `Customer` object is allocated on the managed heap.

The local variable itself is only a reference stored on the stack.

---

# 2. Stack vs Heap Illustration

```text
Stack

---------------------

customer ───────────┐

count = 5           │

price = 20          │

---------------------


Heap

---------------------

Customer Object

Name

Orders

---------------------
```

The stack stores references.

The heap stores the actual object.

---

# 3. Value Types vs Reference Types

## Value Types

Examples:

- int
- long
- bool
- double
- decimal
- struct

Assignment copies the value.

```csharp
int a = 5;

int b = a;

b = 10;
```

Result:

```text
a = 5

b = 10
```

---

## Reference Types

Examples:

- class
- string
- array
- List<T>

Assignment copies the reference.

```csharp
Customer a = new();

Customer b = a;
```

Both variables point to the same object.

---

# Interview Question

### Why is string a reference type?

## Answer

Because strings are immutable objects allocated on the managed heap.

Although they behave similarly to value types in many scenarios, they are implemented as reference types.

---

# 4. Boxing

## Interview Answer

Boxing converts a value type into an object.

Example:

```csharp
int number = 5;

object value = number;
```

The runtime allocates a new object on the heap.

This creates an allocation.

---

# 5. Unboxing

```csharp
object value = 5;

int number = (int)value;
```

The object is converted back to the value type.

Invalid casts throw exceptions.

---

# 6. Why Boxing Matters

Frequent boxing:

- Allocates memory
- Increases GC pressure
- Reduces performance

Generics avoid most boxing scenarios.

Prefer:

```csharp
List<int>
```

Instead of:

```csharp
ArrayList
```

---

# 7. String Immutability

Strings cannot be modified.

Example:

```csharp
string text = "Hello";

text += " World";
```

A completely new string is created.

Large numbers of string concatenations create many temporary allocations.

Use:

```csharp
StringBuilder
```

for repeated concatenation.

---

# 8. Span<T>

## Interview Answer

`Span<T>` provides a lightweight view over contiguous memory without allocating.

Example:

```csharp
Span<int> numbers = stackalloc int[10];
```

Benefits:

- No heap allocation
- High performance
- Reduced GC pressure

Limitations:

- Stack-only type
- Cannot be stored in fields
- Cannot cross async boundaries

---

# 9. Memory<T>

`Memory<T>` is similar to `Span<T>` but can live on the heap.

Use `Memory<T>` when data must survive beyond the current stack frame or be used with asynchronous APIs.

---

# 10. Object Pooling

Instead of repeatedly allocating expensive objects:

```text
Allocate

Use

Destroy
```

Reuse them:

```text
Pool

↓

Borrow

↓

Use

↓

Return
```

Examples:

- StringBuilder
- Buffers
- Parsers
- Serializers

ASP.NET Core internally uses object pools in several components.

---

# 11. ArrayPool<T>

Large array allocations are expensive.

Instead of:

```csharp
byte[] buffer = new byte[8192];
```

Use:

```csharp
var buffer = ArrayPool<byte>.Shared.Rent(8192);

try
{
    // Use buffer
}
finally
{
    ArrayPool<byte>.Shared.Return(buffer);
}
```

Benefits:

- Fewer allocations
- Lower GC pressure
- Better throughput

Always return rented arrays.

---

# 12. Common Interview Questions

### Where are objects stored?

## Answer

Reference type objects are stored on the managed heap.

References to those objects are commonly stored on the stack.

---

### Is every value type stored on the stack?

No.

Value types may also exist:

- Inside objects
- Inside arrays
- When boxed

The common shortcut "value types are on the stack" is not always true.

---

### Why is StringBuilder faster?

Because it modifies an internal buffer instead of creating a new string for every concatenation.

---

### When should Span<T> be used?

For high-performance scenarios involving temporary memory access where heap allocations should be avoided.

---

# Performance Best Practices

- Minimize allocations in hot paths.
- Avoid unnecessary boxing.
- Prefer generic collections.
- Use StringBuilder for repeated concatenation.
- Use ArrayPool<T> for reusable buffers.
- Use Span<T> where appropriate.
- Measure before optimizing.

---

# Trade-offs

| Technique | Benefit | Trade-off |
|-----------|---------|-----------|
| Stack Allocation | Extremely fast | Limited lifetime |
| Heap Allocation | Flexible | GC overhead |
| Span<T> | No allocations | Stack-only restrictions |
| Memory<T> | Async-friendly | Slightly higher overhead |
| Object Pooling | Lower allocations | Pool management complexity |
| ArrayPool<T> | Buffer reuse | Must return rented arrays |

---

# Staff Engineer Insight

A common misconception is that heap allocations are "bad."

Allocating objects is actually very fast in .NET.

The real cost comes later:

- Garbage Collection
- Cache misses
- Memory fragmentation (especially LOH)
- Increased allocation rate

Focus on optimizing **allocation rate**, not avoiding every allocation.

Another common mistake is prematurely introducing complex pooling logic.

Object pooling is valuable only for objects that are:

- Expensive to allocate
- Frequently reused
- Proven bottlenecks through profiling

Always optimize based on measurements rather than assumptions.

---

# Summary

- The stack stores local execution data and references.
- The managed heap stores reference type objects.
- Value types and reference types have different copy semantics.
- Boxing creates heap allocations.
- Strings are immutable.
- Span<T> enables allocation-free memory access.
- Memory<T> supports asynchronous scenarios.
- ArrayPool<T> and object pooling reduce allocation pressure.
- Write allocation-efficient code, but optimize only after measuring.
