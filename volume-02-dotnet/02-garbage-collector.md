# Garbage Collector (GC)

## Overview

The Garbage Collector (GC) is one of the most important components of the .NET Runtime.

Its job is to automatically reclaim memory occupied by objects that are no longer reachable.

Unlike languages such as C++, .NET developers do not manually free memory.

Instead, the runtime periodically determines which objects are still in use and removes those that are no longer referenced.

A senior developer should understand how the GC works, what triggers collections and how application design affects GC performance.

---

# 1. What is Garbage Collection?

## Interview Answer

Garbage Collection is the automatic process of reclaiming memory occupied by unreachable objects.

Example:

```csharp
void Example()
{
    var customer = new Customer();
}
```

After `Example()` finishes, if no references to `customer` remain, the object becomes eligible for collection.

The GC will eventually reclaim its memory.

---

# 2. Reachable vs Unreachable Objects

The GC starts from **GC Roots**.

Examples of GC Roots:

- Local variables
- Static fields
- Active threads
- CPU registers

Example:

```text
GC Root
   │
   ▼
Customer
   │
   ▼
Order
   │
   ▼
Address
```

All reachable objects remain alive.

If no path exists from a GC Root:

```text
Customer

Order

Address
```

they become garbage.

---

# 3. Managed Heap

All reference types are allocated on the managed heap.

```text
Managed Heap

--------------------------------

Customer

Order

Product

Invoice

--------------------------------
```

Value types usually live on the stack unless boxed or embedded inside reference types.

---

# 4. Generational GC

## Interview Answer

The .NET GC is generational.

It assumes:

> Most objects die young.

There are three generations:

```text
Gen 0

↓

Gen 1

↓

Gen 2
```

---

## Generation 0

New objects are allocated here.

Most objects are collected before leaving Gen 0.

Examples:

- Temporary DTOs
- LINQ objects
- JSON objects

---

## Generation 1

Objects that survive Gen 0 move here.

Acts as a buffer between short-lived and long-lived objects.

---

## Generation 2

Long-lived objects.

Examples:

- Application caches
- Singletons
- Static collections

Collections are less frequent because scanning Gen 2 is expensive.

---

# Interview Question

### Why are generations useful?

## Answer

Most objects have short lifetimes.

Collecting only Gen 0 is much faster than scanning the entire heap.

---

# 5. Large Object Heap (LOH)

Objects larger than approximately **85 KB** are allocated in the Large Object Heap.

Examples:

```csharp
byte[] buffer = new byte[200000];
```

LOH characteristics:

- Separate heap
- Collected with Gen 2
- More expensive collections

Large allocations should be minimized when possible.

---

# 6. What Triggers GC?

Garbage Collection may occur when:

- Gen 0 becomes full
- Memory pressure increases
- The runtime decides memory should be reclaimed
- `GC.Collect()` is called (rarely appropriate)

Developers should almost never call:

```csharp
GC.Collect();
```

---

# 7. GC Phases

Simplified process:

```text
Mark

↓

Find Reachable Objects

↓

Sweep

↓

Remove Garbage

↓

Compact

↓

Move Remaining Objects
```

Compaction reduces fragmentation.

---

# 8. Stop-The-World

During parts of collection, application threads may pause.

```text
Application Running

↓

GC Starts

↓

Threads Paused

↓

GC Completes

↓

Resume
```

Modern .NET minimizes pause times through background and concurrent GC.

---

# 9. Server GC vs Workstation GC

## Workstation GC

Optimized for:

- Desktop applications
- Lower latency
- Fewer cores

---

## Server GC

Optimized for:

- ASP.NET Core
- Services
- High throughput
- Multi-core machines

Each CPU core receives its own allocation heap.

This improves scalability.

---

# 10. IDisposable vs GC

## Interview Answer

Garbage Collection manages memory.

It does **not** manage unmanaged resources.

Examples:

- File handles
- Database connections
- Network sockets

These should implement:

```csharp
IDisposable
```

Example:

```csharp
using var stream =
    File.OpenRead("file.txt");
```

The `using` statement ensures deterministic cleanup.

---

# 11. Common Interview Questions

### Does GC immediately destroy objects?

## Answer

No.

Objects become eligible for collection when unreachable.

The runtime decides when to reclaim memory.

---

### Can memory leaks still happen?

Yes.

Example:

```csharp
static List<Customer> cache = new();
```

If objects remain referenced forever, the GC cannot collect them.

A managed memory leak is usually caused by retaining references longer than necessary.

---

### Should developers call GC.Collect()?

Almost never.

The runtime has far more information than the application about when collection is appropriate.

---

### Why is allocating many temporary objects expensive?

Because allocation increases GC pressure.

More allocations lead to more frequent collections.

---

# Performance Best Practices

- Avoid unnecessary allocations.
- Reuse expensive objects when appropriate.
- Avoid large temporary arrays.
- Dispose unmanaged resources promptly.
- Be careful with static collections.
- Profile before optimizing.

---

# Trade-offs

| Feature | Benefit | Trade-off |
|----------|---------|-----------|
| Automatic GC | Simplifies memory management | GC pauses |
| Generational GC | Fast collections | More runtime complexity |
| LOH | Efficient large allocations | Expensive collections |
| Server GC | High throughput | Higher memory usage |
| Workstation GC | Lower latency | Lower throughput |

---

# Staff Engineer Insight

One of the biggest misconceptions is:

> "The Garbage Collector causes performance problems."

Usually it does not.

The real issue is often:

- Excessive allocations
- Poor object lifetime management
- Large object allocations
- Boxing
- Repeated LINQ allocations
- Unnecessary string creation

The GC is optimized for well-designed applications.

If GC dominates CPU time, investigate **why so many objects are being allocated**, rather than blaming the collector itself.

A useful performance mindset is:

> **Reduce allocations first, optimize GC second.**

---

# Summary

- The Garbage Collector automatically reclaims unreachable objects.
- The managed heap stores reference types.
- The GC uses three generations to optimize performance.
- Large objects are allocated on the LOH.
- The GC determines when to collect memory.
- `IDisposable` is for unmanaged resources, not memory.
- Avoid unnecessary allocations to reduce GC pressure.
- Understand object lifetime before attempting GC optimizations.
