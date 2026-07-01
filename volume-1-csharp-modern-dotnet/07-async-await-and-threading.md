# Async, Await and Threading

## Overview

`async` and `await` are among the most misunderstood features in .NET. Many developers think that `async` creates a new thread or makes code faster. Neither statement is true.

A senior engineer should understand how asynchronous programming works, what the compiler generates, when threads are actually used and how to avoid common pitfalls such as deadlocks and thread pool starvation.

---

# 1. What is asynchronous programming?

## Interview Answer

Asynchronous programming allows a thread to start an operation, return to the thread pool while waiting for the result, and continue execution once the operation completes.

The goal is **not to make code execute faster**, but to avoid blocking threads while waiting for I/O operations such as database queries, HTTP requests or file access.

---

# 2. Does async create a new thread?

## Interview Answer

No.

Adding the `async` keyword does **not** create a new thread.

Example:

```csharp
public async Task<User> GetUser()
{
    return await repository.GetUserAsync();
}
```

No new thread is created.

The method simply becomes capable of suspending execution while waiting for the asynchronous operation.

---

## Common Misconception

Wrong:

```
async

↓

New Thread
```

Correct:

```
Thread starts operation

↓

Operation waits (Database, HTTP, File...)

↓

Thread returns to ThreadPool

↓

Operation completes

↓

Continuation resumes
```

---

# 3. What does await actually do?

## Interview Answer

When the compiler encounters an `await`, it checks whether the awaited task has already completed.

- If it has completed, execution continues immediately.
- If it has not completed, the current method is suspended and control is returned to the caller.

When the operation finishes, execution resumes after the `await`.

---

# 4. Compiler Transformation

The compiler rewrites an async method into a **state machine**.

Original code:

```csharp
public async Task<int> Calculate()
{
    await Task.Delay(1000);

    return 5;
}
```

Simplified conceptually:

```
State 0

↓

Start Task

↓

Pause

↓

Resume

↓

Return result
```

This transformation is completely automatic.

---

# 5. CPU-bound vs I/O-bound Work

## CPU-bound

Examples:

- Image processing
- Compression
- Encryption
- Mathematical calculations

Use:

```csharp
await Task.Run(...)
```

only if you intentionally want to move CPU work off the current thread.

---

## I/O-bound

Examples:

- SQL Server
- HTTP
- Azure Blob Storage
- Redis
- File system

Do **not** wrap these in `Task.Run()`.

Instead use the asynchronous API directly:

```csharp
await db.Users.ToListAsync();
```

---

# 6. Task vs Thread

This is a very common interview question.

A **Thread** is an operating system resource.

A **Task** is a higher-level abstraction representing asynchronous work.

Tasks usually execute on ThreadPool threads, but not always.

Think of a Task as "work", not as "a thread".

---

# 7. Task vs ValueTask

## Interview Answer

`Task` is the standard return type for asynchronous methods.

`ValueTask` reduces allocations when operations often complete synchronously.

Example:

```csharp
Task<User>
```

vs

```csharp
ValueTask<User>
```

Rule of thumb:

Use `Task` by default.

Use `ValueTask` only after measuring performance and confirming that allocations are a bottleneck.

---

# 8. ConfigureAwait(false)

## Interview Answer

`ConfigureAwait(false)` tells the awaiter not to capture the current synchronization context before resuming execution.

Example:

```csharp
await repository.GetUserAsync()
    .ConfigureAwait(false);
```

### Classic ASP.NET

SynchronizationContext exists.

`ConfigureAwait(false)` can improve performance and help avoid deadlocks.

### ASP.NET Core

There is no request SynchronizationContext.

In most ASP.NET Core applications, `ConfigureAwait(false)` has little or no behavioral effect.

---

# 9. ThreadPool

The ThreadPool is a pool of reusable worker threads managed by the CLR.

Creating threads is expensive.

Instead of creating new threads repeatedly, .NET reuses existing ones.

Almost all Tasks execute on ThreadPool threads.

---

# 10. Thread Pool Starvation

## Interview Answer

Thread pool starvation happens when all available ThreadPool threads are blocked.

Example:

```csharp
Task.Result

Task.Wait()

Thread.Sleep()
```

When enough threads are blocked, new requests must wait for threads to become available.

Symptoms:

- Slow APIs
- High response times
- Low CPU utilization
- Requests timing out

---

# 11. Why are Task.Result and Wait() dangerous?

Example:

```csharp
var user = repository.GetUserAsync().Result;
```

This blocks the current thread.

Instead:

```csharp
var user = await repository.GetUserAsync();
```

Blocking threads reduces scalability and can cause deadlocks in environments that use a synchronization context.

---

# 12. Task.WhenAll vs Task.WaitAll

Prefer:

```csharp
await Task.WhenAll(task1, task2, task3);
```

Instead of:

```csharp
Task.WaitAll(task1, task2, task3);
```

`WhenAll` is asynchronous.

`WaitAll` blocks threads.

---

# 13. Parallel.ForEach vs Task.WhenAll

`Parallel.ForEach`

- CPU-bound work
- Uses multiple cores

`Task.WhenAll`

- I/O-bound work
- Waits asynchronously

Choosing the wrong one can reduce performance instead of improving it.

---

# 14. Common Interview Questions

### Does async improve performance?

Not directly.

It improves scalability by avoiding blocked threads.

---

### Should every method be async?

No.

Only methods performing asynchronous work.

---

### Is async useful for CPU-intensive calculations?

Usually no.

CPU work is limited by processor speed, not waiting time.

---

### Does await always resume on the same thread?

Not necessarily.

It depends on the synchronization context.

---

# Senior Tips

- Never call `.Result` or `.Wait()` in ASP.NET Core request handling.
- Don't wrap naturally asynchronous APIs in `Task.Run()`.
- Use `Task.WhenAll` to execute independent asynchronous operations concurrently.
- Use `ValueTask` only after profiling.
- Remember: async improves **scalability**, not raw execution speed.

---

# Summary

- async does not create threads.
- await does not block threads.
- Task represents work, not a thread.
- ThreadPool reuses worker threads.
- Avoid `.Result` and `.Wait()`.
- Prefer `Task.WhenAll` for concurrent asynchronous operations.
- Understand the difference between CPU-bound and I/O-bound workloads.
