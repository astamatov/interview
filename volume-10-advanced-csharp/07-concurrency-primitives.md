# Concurrency Primitives in .NET

## Overview

Modern applications execute multiple operations simultaneously.

Without proper synchronization, multiple threads can corrupt shared state, producing race conditions, inconsistent data and difficult-to-reproduce bugs.

.NET provides several synchronization primitives.

Choosing the correct one depends on:

- Number of threads
- Read/write ratio
- Process boundaries
- Performance requirements
- Async compatibility

---

# 1. Race Condition

## Interview Answer

A race condition occurs when multiple threads access shared data simultaneously and the final result depends on execution timing.

Example:

```csharp
count++;
```

Looks harmless.

Actually executes roughly as:

```
Read

↓

Increment

↓

Write
```

Two threads can overwrite each other's changes.

---

# 2. lock

## Interview Answer

`lock` is the simplest synchronization primitive in C#.

Example:

```csharp
private readonly object _sync = new();

lock (_sync)
{
    count++;
}
```

Only one thread can execute the protected block at a time.

---

### Under the hood

The compiler translates:

```csharp
lock(obj)
{
}
```

into approximately:

```csharp
Monitor.Enter(obj);

try
{
}
finally
{
    Monitor.Exit(obj);
}
```

---

# Interview Question

### Should you lock on `this`?

## Answer

No.

External code could also lock the same object.

Always lock on a private object.

---

# 3. Monitor

`Monitor` provides more control than `lock`.

Example:

```csharp
Monitor.Enter(_sync);

try
{
}
finally
{
    Monitor.Exit(_sync);
}
```

Supports additional operations such as:

- TryEnter
- Wait
- Pulse
- PulseAll

---

# 4. Mutex

## Interview Answer

A Mutex synchronizes threads across **multiple processes**.

Unlike `lock`, it is not limited to a single process.

Typical use case:

```
Application A

↓

Mutex

↑

Application B
```

Use it when only one process should access a shared resource.

---

# 5. SemaphoreSlim

## Interview Answer

A Semaphore limits how many threads may enter simultaneously.

Example:

```csharp
var semaphore =
    new SemaphoreSlim(3);
```

```
10 Threads

↓

Only 3 Run

↓

Others Wait
```

Common use cases:

- API throttling
- Database connection limits
- Parallel downloads

---

# Interview Question

### Why use SemaphoreSlim instead of lock?

## Answer

`lock` allows only one thread.

`SemaphoreSlim` allows a configurable number of concurrent threads and provides asynchronous APIs such as `WaitAsync()`.

---

# 6. ReaderWriterLockSlim

Optimized for read-heavy workloads.

```
Many Readers

↓

Allowed

---------------

Writer

↓

Exclusive Access
```

Multiple readers execute concurrently.

Writers obtain exclusive access.

Useful when reads vastly outnumber writes.

---

# 7. Interlocked

## Interview Answer

`Interlocked` performs atomic operations without locking.

Example:

```csharp
Interlocked.Increment(ref count);
```

This is faster than:

```csharp
lock(...)
{
    count++;
}
```

for simple counters.

---

# 8. Volatile

`volatile` tells the compiler and CPU not to cache a field aggressively.

Example:

```csharp
private volatile bool _running;
```

This improves visibility between threads.

Important:

`volatile` **does not** make compound operations thread-safe.

---

# 9. Concurrent Collections

Instead of:

```csharp
List<T>
```

Use:

- ConcurrentDictionary
- ConcurrentQueue
- ConcurrentBag
- ConcurrentStack

These collections are designed for concurrent access.

---

# 10. Deadlocks

Deadlocks occur when two or more threads wait indefinitely for each other.

Example:

```
Thread A

↓

Lock X

↓

Wait Lock Y

---------------

Thread B

↓

Lock Y

↓

Wait Lock X
```

Neither thread can continue.

---

## Preventing Deadlocks

- Always acquire locks in a consistent order.
- Keep lock duration short.
- Avoid nested locks when possible.
- Prefer higher-level concurrent collections where appropriate.

---

# 11. Common Interview Questions

### Is lock async-compatible?

## Answer

No.

You cannot safely `await` inside a `lock` block.

For asynchronous synchronization, use `SemaphoreSlim`.

---

### Is Interlocked always better?

No.

It supports only a limited set of atomic operations.

Complex critical sections still require locks or other synchronization mechanisms.

---

### When should ReaderWriterLockSlim be used?

When reads greatly outnumber writes and profiling shows lock contention.

For many applications, a simple `lock` is sufficient and easier to maintain.

---

### Does volatile replace lock?

No.

`volatile` improves visibility.

It does not provide mutual exclusion or atomicity.

---

# Choosing the Right Primitive

| Scenario | Recommended Primitive |
|----------|-----------------------|
| Simple critical section | lock |
| Async synchronization | SemaphoreSlim |
| Atomic counter | Interlocked |
| Read-heavy cache | ReaderWriterLockSlim |
| Cross-process synchronization | Mutex |
| Thread-safe queue | ConcurrentQueue |

---

# Trade-offs

| Primitive | Advantage | Limitation |
|-----------|-----------|------------|
| lock | Simple | Blocks one thread at a time |
| Monitor | More control | More verbose |
| Mutex | Cross-process | Slower than lock |
| SemaphoreSlim | Async support | Requires careful release |
| ReaderWriterLockSlim | Excellent for read-heavy workloads | More complex |
| Interlocked | Extremely fast | Limited operations |

---

# Staff Engineer Insight

One of the biggest mistakes developers make is reaching for synchronization too quickly.

The first question should be:

> **"Can I eliminate shared mutable state?"**

Immutable objects, message passing and actor-style architectures often remove the need for locking entirely.

Another common mistake is replacing every `lock` with `ReaderWriterLockSlim` because it seems more advanced.

In many applications, the additional complexity outweighs the performance benefit.

Profile first.

Choose the simplest synchronization primitive that satisfies the requirements.

---

# Summary

- Race conditions occur when multiple threads modify shared state without synchronization.
- `lock` is the simplest synchronization primitive.
- `Monitor` powers `lock` and offers additional capabilities.
- `Mutex` synchronizes across processes.
- `SemaphoreSlim` supports asynchronous throttling.
- `ReaderWriterLockSlim` benefits read-heavy workloads.
- `Interlocked` performs atomic operations efficiently.
- `volatile` improves visibility but not atomicity.
- Prefer immutable designs when possible.
- Measure contention before introducing more complex synchronization primitives.
