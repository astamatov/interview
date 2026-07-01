# Span<T>, Memory<T> and ArrayPool<T>

## Overview

High-performance .NET applications often spend more time allocating memory than executing business logic.

Repeated allocations increase pressure on the Garbage Collector, resulting in more frequent collections and reduced throughput.

.NET provides several APIs to reduce allocations:

- Span<T>
- Memory<T>
- ReadOnlySpan<T>
- Memory<T>
- ArrayPool<T>

These APIs help improve performance by reusing memory and avoiding unnecessary allocations.

---

# 1. What is Span<T>?

## Interview Answer

`Span<T>` represents a lightweight view over a contiguous region of memory.

Instead of creating a new array or string, a Span references existing memory.

Example:

```csharp
Span<int> numbers = stackalloc int[5];
```

Or:

```csharp
int[] values = {1,2,3,4,5};

Span<int> span = values;
```

No new array is created.

---

# 2. Why Use Span<T>?

Consider:

```csharp
string text = "Hello World";

string first = text.Substring(0, 5);
```

`Substring()` creates a new string allocation.

Using `Span<T>`:

```csharp
ReadOnlySpan<char> first =
    text.AsSpan(0, 5);
```

No additional string is allocated.

This is particularly useful in parsing scenarios.

---

# Interview Question

### Why is Span<T> faster?

## Answer

Because it avoids unnecessary memory allocations.

It references existing memory instead of copying it.

Less allocation means less work for the Garbage Collector.

---

# 3. Why Can't Span<T> Be Stored?

`Span<T>` is a `ref struct`.

It exists only on the stack.

This means:

✔ Can be used inside methods.

✘ Cannot:

- Be stored in fields.
- Cross `await` boundaries.
- Be boxed.
- Implement interfaces.

These restrictions prevent unsafe memory access.

---

# 4. Memory<T>

## Interview Answer

`Memory<T>` is similar to `Span<T>` but is designed for longer lifetimes.

Unlike `Span<T>`, it can:

- Be stored in fields.
- Be returned from methods.
- Cross asynchronous boundaries.

Example:

```csharp
Memory<byte> buffer;
```

When synchronous access is needed:

```csharp
Span<byte> span =
    buffer.Span;
```

---

# 5. ReadOnlySpan<T>

Use `ReadOnlySpan<T>` when data should not be modified.

Example:

```csharp
ReadOnlySpan<char> text =
    "Hello".AsSpan();
```

This improves safety by preventing accidental changes.

---

# 6. stackalloc

## Interview Answer

`stackalloc` allocates memory directly on the stack.

Example:

```csharp
Span<int> values =
    stackalloc int[100];
```

Advantages:

- Extremely fast allocation.
- Automatically released when the method exits.
- No Garbage Collection required.

Use only for small buffers.

Allocating very large objects on the stack may cause a stack overflow.

---

# 7. ArrayPool<T>

## Interview Answer

`ArrayPool<T>` allows applications to rent and return arrays instead of allocating new ones.

Example:

```csharp
var pool = ArrayPool<byte>.Shared;

byte[] buffer = pool.Rent(4096);

try
{
    // Use buffer
}
finally
{
    pool.Return(buffer);
}
```

The array is reused instead of discarded.

This reduces GC pressure.

---

# 8. When Should ArrayPool Be Used?

Typical scenarios:

- File processing
- Image processing
- Network protocols
- Serialization
- High-throughput APIs

Avoid using it for small, infrequent allocations where the added complexity offers little benefit.

---

# 9. Memory Pooling

Without pooling:

```
Allocate

↓

GC

↓

Allocate

↓

GC
```

With pooling:

```
Rent

↓

Use

↓

Return

↓

Rent Again
```

Far fewer allocations occur.

---

# 10. Common Interview Questions

### Should every application use Span<T>?

## Answer

No.

`Span<T>` increases code complexity.

Use it only when profiling identifies allocation-related bottlenecks.

---

### Is ArrayPool<T> always faster?

Not necessarily.

Pooling introduces its own overhead.

For small arrays or infrequent operations, allocating a new array may be simpler and just as fast.

Always benchmark before optimizing.

---

### Why can't Span<T> cross an await?

Because it references stack memory.

After an `await`, the original stack frame may no longer exist.

`Memory<T>` is designed for those scenarios.

---

### Can Span<T> improve string parsing?

Yes.

It allows parsing substrings without creating additional string objects.

This is one of its most common use cases.

---

# Example

Without Span:

```
Input String

↓

Substring

↓

New Allocation

↓

Parse
```

With Span:

```
Input String

↓

Span View

↓

Parse
```

No intermediate allocation.

---

# Trade-offs

| Feature | Benefit | Limitation |
|----------|---------|------------|
| Span<T> | Zero-copy access | Stack-only lifetime |
| Memory<T> | Async-friendly | Slightly more overhead |
| stackalloc | Extremely fast | Limited stack space |
| ArrayPool<T> | Fewer allocations | Must return rented arrays |

---

# Staff Engineer Insight

One of the biggest mistakes developers make is introducing `Span<T>` everywhere because it is "faster."

Performance optimizations should be guided by measurements.

If a method executes once per minute, reducing one allocation is unlikely to matter.

If it executes millions of times per second, avoiding even a single allocation can significantly reduce GC pressure.

Another common mistake is forgetting to return arrays to `ArrayPool<T>`.

This defeats the purpose of pooling and can lead to increased memory usage.

Always use `try/finally` when renting pooled arrays.

---

# Summary

- `Span<T>` provides allocation-free access to contiguous memory.
- `ReadOnlySpan<T>` offers safe read-only access.
- `Memory<T>` supports asynchronous scenarios.
- `stackalloc` allocates temporary memory on the stack.
- `ArrayPool<T>` reuses arrays to reduce allocations.
- Use these APIs only when profiling demonstrates a performance benefit.
- Measure first, optimize second.
