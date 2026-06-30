# Stack, Heap and Garbage Collection

## Overview

Memory management is one of the most common senior .NET interview topics. Most developers know that "value types go on the stack" and "reference types go on the heap", but that explanation is incomplete and often incorrect.

A senior engineer should understand **how memory is allocated, when objects are collected, how the Garbage Collector works and how memory decisions affect performance**.

---

# 1. What is the difference between Stack and Heap?

## Interview Answer

The stack is used to store method call frames, local variables and execution state. It is extremely fast because memory allocation and deallocation only require moving the stack pointer.

The managed heap stores objects that have a lifetime independent of the current method execution. Memory on the heap is automatically managed by the .NET Garbage Collector.

A common misconception is that all value types live on the stack and all reference types live on the heap. In reality, **where a value type lives depends on where it is declared**.

---

## Stack Characteristics

- Very fast allocation
- Very fast deallocation
- Thread-local
- Automatically cleaned when the method exits
- Stores method frames and local variables

Example:

```csharp
int age = 30;
double salary = 1500;
bool active = true;
```

These variables are stored inside the current stack frame.

---

## Heap Characteristics

- Used for objects with longer lifetimes
- Managed by the Garbage Collector
- Shared between threads
- Allocation is slightly slower than the stack
- Can become fragmented

Example:

```csharp
var customer = new Customer();
```

The variable `customer` is a reference.

The actual `Customer` object lives on the managed heap.

---

# 2. Common Interview Trick

Many candidates answer:

> Value types live on the stack.

This is **not always true**.

Example:

```csharp
public class Order
{
    public int Id { get; set; }
}
```

`Id` is an `int` (value type).

Does it live on the stack?

No.

It is stored **inside the Order object**, which lives on the heap.

Therefore, value types can exist on the heap whenever they are fields of reference types.

---

# 3. How does the Garbage Collector work?

## Interview Answer

The .NET Garbage Collector automatically releases memory occupied by objects that are no longer reachable by the application.

Instead of freeing memory immediately after an object is no longer used, the GC periodically scans object references, determines which objects are still reachable and reclaims memory occupied by unreachable objects.

This removes the need for manual memory management while reducing memory leaks.

---

# 4. Why does .NET use Generations?

Most objects die very quickly.

For example:

```csharp
var dto = new UserDto();
```

This object may exist for only a few milliseconds.

Scanning the entire heap every time would be inefficient.

Instead, the GC separates objects into generations.

---

## Generation 0

All new objects start here.

Most objects are collected here.

Generation 0 collections are extremely fast.

---

## Generation 1

Objects that survive a Gen0 collection are promoted to Generation 1.

Think of Gen1 as a buffer between short-lived and long-lived objects.

---

## Generation 2

Objects that survive multiple collections eventually reach Generation 2.

These are usually long-lived objects.

Examples:

- Application caches
- Singleton services
- Long-running collections
- Configuration objects

Gen2 collections are much more expensive than Gen0 collections.

---

# 5. Large Object Heap (LOH)

Objects larger than approximately **85 KB** are allocated on the Large Object Heap.

Examples:

- Large byte arrays
- Images
- Large strings
- Buffers

Example:

```csharp
byte[] buffer = new byte[100000];
```

This allocation goes directly to the LOH.

---

## Why is LOH important?

Large objects are expensive to allocate.

Frequent allocations increase memory pressure and may trigger expensive Gen2 collections.

For performance-sensitive applications, consider using `ArrayPool<T>` instead of repeatedly allocating large arrays.

---

# 6. What is an Object Graph?

The GC doesn't simply check whether a variable exists.

Instead, it starts from **GC Roots**.

Examples of GC Roots:

- Static fields
- Local variables
- CPU registers
- Active threads

If an object can be reached from a GC Root, it remains alive.

Otherwise, it becomes eligible for collection.

---

# 7. IDisposable vs Garbage Collector

Many developers believe:

> Garbage Collection closes files.

Wrong.

The Garbage Collector only manages **managed memory**.

Resources such as:

- Database connections
- File handles
- Network sockets
- OS handles

must be released explicitly.

Example:

```csharp
using var stream = File.OpenRead(path);
```

The `using` statement guarantees that `Dispose()` is called, even if an exception occurs.

---

# 8. Finalizers

Finalizers act as a safety net.

They are executed by the Garbage Collector before reclaiming an object that owns unmanaged resources.

Because finalizers delay memory reclamation and introduce additional GC work, they should be used only when a type directly owns unmanaged resources.

---

# 9. Boxing and Unboxing

Boxing converts a value type into an object.

Example:

```csharp
int number = 10;

object obj = number;
```

This allocates a new object on the heap.

Unboxing:

```csharp
int value = (int)obj;
```

Frequent boxing can negatively affect performance due to additional heap allocations.

---

# 10. Common Interview Questions

### Why is the stack faster than the heap?

Because allocating stack memory only moves the stack pointer.

No Garbage Collection is required.

---

### Does calling Dispose() free managed memory?

No.

Dispose releases unmanaged resources.

Managed memory is reclaimed later by the Garbage Collector.

---

### Why are Gen2 collections expensive?

Because the GC must inspect many long-lived objects.

---

### Why should large arrays be reused?

Repeated allocations increase LOH pressure and trigger more expensive collections.

---

# Senior Tips

- Prefer short-lived objects over unnecessarily long-lived ones.
- Avoid allocating large temporary arrays repeatedly.
- Use `ArrayPool<T>` for reusable buffers.
- Don't call `GC.Collect()` unless you have a very specific reason.
- Use `using` or `await using` for disposable resources.
- Remember that value types can live on the heap when they are fields of reference types.

---

# Summary

- The stack stores execution state and local variables.
- The heap stores managed objects.
- New objects start in Generation 0.
- Long-lived objects end up in Generation 2.
- Objects larger than ~85 KB go to the Large Object Heap.
- The Garbage Collector manages managed memory, not unmanaged resources.
- `IDisposable` is for deterministic cleanup of unmanaged resources.
- Boxing creates heap allocations and should be minimized in performance-critical code.
