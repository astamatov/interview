# Thread Synchronization and Locking

## Overview

When multiple threads access shared data concurrently, race conditions, deadlocks and inconsistent state can occur. Every senior .NET developer should understand synchronization primitives, when to use them and their trade-offs.

---

# 1. What is a race condition?

## Answer

A race condition occurs when two or more threads access and modify shared data simultaneously without proper synchronization.

The final result depends on the order in which the threads execute, making the behavior unpredictable.

Example:

```csharp
int counter = 0;

Parallel.For(0, 1000, i =>
{
    counter++;
});
```

Expected:

```
1000
```

Actual:

```
978
995
1000
982
```

The result changes because `counter++` is **not atomic**.

---

## Why?

`counter++` actually consists of three operations:

```
Read counter

↓

Increment value

↓

Write counter
```

Another thread can interrupt between these operations.

---

## Common Follow-up

### Is `counter++` thread-safe?

No.

It is not an atomic operation.

---

# 2. What is the lock keyword?

## Answer

The `lock` keyword ensures that only one thread can execute a critical section of code at a time.

Example:

```csharp
private readonly object _lock = new();

lock (_lock)
{
    counter++;
}
```

If another thread reaches the same lock, it waits until the first thread exits the critical section.

---

## What happens internally?

The `lock` keyword is syntactic sugar for `Monitor.Enter()` and `Monitor.Exit()`.

Equivalent code:

```csharp
Monitor.Enter(_lock);

try
{
    counter++;
}
finally
{
    Monitor.Exit(_lock);
}
```

The compiler generates this automatically.

---

## Common Follow-up

### Can I lock on a string?

No.

Never lock on:

```csharp
this

typeof(User)

"hello"
```

Always use a private object:

```csharp
private readonly object _lock = new();
```

---

# 3. What is Monitor?

## Answer

`Monitor` is the low-level synchronization primitive used by `lock`.

It offers additional functionality such as:

- TryEnter()
- Wait()
- Pulse()
- PulseAll()

Use `Monitor` when you need more control than `lock` provides.

---

# 4. What is Interlocked?

## Answer

`Interlocked` performs simple atomic operations without taking a lock.

Example:

```csharp
Interlocked.Increment(ref counter);
```

This is faster than using `lock` because it relies on CPU atomic instructions.

Use `Interlocked` for simple operations like incrementing counters or swapping references.

---

## Follow-up

### When should you use Interlocked instead of lock?

Use `Interlocked` when you only need a single atomic operation.

If multiple operations must execute together as one unit, use `lock`.

---

# 5. What is SemaphoreSlim?

## Answer

`SemaphoreSlim` limits how many threads can enter a section of code simultaneously.

Unlike `lock`, it supports asynchronous waiting with `WaitAsync()`.

Example:

```csharp
private readonly SemaphoreSlim semaphore = new(3);

await semaphore.WaitAsync();

try
{
    await ProcessOrderAsync();
}
finally
{
    semaphore.Release();
}
```

This allows up to three concurrent operations.

---

## Common Use Cases

- Rate limiting
- Limiting concurrent HTTP requests
- Protecting shared resources in async code

---

# 6. lock vs SemaphoreSlim

| lock | SemaphoreSlim |
|------|---------------|
| One thread only | Configurable number of threads |
| Synchronous | Supports async |
| Uses Monitor | Uses semaphore |
| Cannot await | Supports WaitAsync |

---

## Interview Question

### Can I use await inside a lock?

## Answer

No.

This is illegal:

```csharp
lock (_lock)
{
    await SaveAsync();
}
```

The compiler will reject it.

Use `SemaphoreSlim` instead.

---

# 7. What is ReaderWriterLockSlim?

## Answer

`ReaderWriterLockSlim` allows many readers to access shared data concurrently while ensuring that only one writer can modify it.

This improves performance when reads greatly outnumber writes.

Example:

```csharp
rwLock.EnterReadLock();

try
{
    Console.WriteLine(cache["User"]);
}
finally
{
    rwLock.ExitReadLock();
}
```

---

## When to use it?

Good:

- Caches
- Configuration
- Read-heavy applications

Bad:

- Frequently changing data

---

# 8. What is volatile?

## Answer

The `volatile` keyword ensures that reads and writes are performed directly against main memory instead of using CPU caches.

It guarantees **visibility**, but **does not guarantee atomicity**.

Example:

```csharp
private volatile bool _running;
```

Use it for simple flags that are shared between threads.

---

## Common Mistake

Many developers think `volatile` makes code thread-safe.

It does not.

It only ensures that changes made by one thread become visible to others.

---

# Senior Tips

- Prefer immutable objects whenever possible.
- Use `Interlocked` for simple atomic operations.
- Use `lock` for protecting critical sections.
- Use `SemaphoreSlim` for asynchronous synchronization.
- Use `ReaderWriterLockSlim` only when reads dominate writes.
- Never lock on public objects, strings or `this`.

---

# Summary

- Race conditions occur when shared state is modified concurrently.
- `lock` protects critical sections.
- `Monitor` is the implementation behind `lock`.
- `Interlocked` performs atomic operations efficiently.
- `SemaphoreSlim` is the preferred synchronization primitive for async code.
- `volatile` guarantees visibility, not thread safety.
