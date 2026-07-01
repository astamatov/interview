# Common Senior C# Interview Questions

## Overview

This chapter contains questions commonly asked during Senior and Lead .NET interviews.

The goal is not to memorize answers.

Instead, understand the underlying concepts and trade-offs.

---

# 1. Why is Dictionary<TKey, TValue> lookup O(1)?

## Interview Answer

A Dictionary uses a hash table.

The key is hashed into an integer.

The hash determines which bucket should contain the value.

```
Key

↓

GetHashCode()

↓

Bucket

↓

Find Entry
```

Average lookup complexity is O(1).

Worst-case complexity becomes O(n) when many keys collide into the same bucket.

Good hash functions and resizing keep collisions low.

---

# 2. Why is string immutable?

## Interview Answer

Strings are immutable to improve:

- Thread safety
- Hashing
- Interning
- Security

If strings could change:

```
Dictionary Key

↓

Modified

↓

Hash Changes

↓

Dictionary Breaks
```

Immutability guarantees that a string's hash code never changes.

---

# 3. Why is Task not a thread?

## Interview Answer

A Task represents work.

A thread executes work.

A Task may:

- Run on a ThreadPool thread
- Resume on another thread
- Complete without occupying a thread while waiting for I/O

Task is an abstraction, not an operating system thread.

---

# 4. Why can't Span<T> cross await?

## Interview Answer

Span<T> is a ref struct.

It lives on the stack.

After an await, the original stack frame may no longer exist.

Allowing Span<T> to survive would risk referencing invalid memory.

Use Memory<T> instead for asynchronous scenarios.

---

# 5. Explain Boxing

## Interview Answer

Boxing converts a value type into an object.

Example:

```csharp
int x = 10;

object value = x;
```

This allocates an object on the heap.

Unboxing converts the object back to a value type.

Frequent boxing increases allocations and GC pressure.

---

# 6. IEnumerable vs IQueryable

## IEnumerable

Execution happens in memory.

```
Database

↓

All Data

↓

Filter
```

---

## IQueryable

The LINQ expression becomes an Expression Tree.

```
LINQ

↓

Expression Tree

↓

SQL

↓

Database
```

Filtering occurs on the database server.

---

# 7. Why is ConfigureAwait(false) useful?

## Interview Answer

It prevents continuation on a captured SynchronizationContext.

Useful primarily for reusable libraries and UI applications.

In ASP.NET Core it usually provides little benefit because there is no UI SynchronizationContext.

---

# 8. What happens when you call new?

## Interview Answer

Conceptually:

```
Allocate Memory

↓

Zero Initialize

↓

Run Constructor

↓

Return Reference
```

Allocation occurs on the managed heap for reference types.

---

# 9. Why shouldn't GC.Collect() be called?

The Garbage Collector already determines the optimal collection time.

Manual collections often:

- Increase pause times
- Reduce throughput
- Collect objects unnecessarily

Use it only in very specific scenarios where measurement justifies it.

---

# 10. Difference between record, class and struct

## class

Reference type.

Identity-based equality by default.

---

## record

Reference type.

Value-based equality by default.

Ideal for DTOs and immutable data.

---

## struct

Value type.

Copied by value.

Best for small immutable data structures.

---

# 11. Why is Dictionary not thread-safe?

Multiple threads modifying buckets simultaneously may corrupt the internal structure.

For concurrent scenarios use:

```csharp
ConcurrentDictionary<TKey, TValue>
```

---

# 12. ref vs out vs in

## ref

Pass by reference.

Variable must already be initialized.

---

## out

Pass by reference for output.

Method must assign a value.

---

## in

Pass by readonly reference.

Avoids copying large value types while preventing modification.

---

# 13. Why are Expression Trees important?

Because frameworks such as Entity Framework Core inspect the expression tree and translate it into SQL instead of executing the lambda directly.

---

# 14. Why are records not always suitable for EF Core entities?

Records compare values.

Entities usually represent identity.

Two entities with the same values are not necessarily the same business object.

---

# 15. What causes ThreadPool starvation?

Common causes:

- .Result
- .Wait()
- Blocking I/O
- Long-running synchronous work
- Too many blocked ThreadPool threads

The solution is usually to avoid blocking and use asynchronous APIs.

---

# 16. Explain Gen 0, Gen 1 and Gen 2

## Gen 0

Short-lived objects.

Fast collection.

---

## Gen 1

Intermediate generation.

---

## Gen 2

Long-lived objects.

Collections are more expensive.

---

# 17. Why is async more scalable?

Because threads are returned to the ThreadPool while waiting for I/O.

The application can process more concurrent requests with the same number of threads.

---

# 18. Why use Dependency Injection?

Benefits:

- Loose coupling
- Easier testing
- Better separation of concerns
- Centralized object lifetime management

---

# 19. Singleton vs Scoped vs Transient

Singleton:

One instance for the application lifetime.

Scoped:

One instance per request.

Transient:

New instance every resolution.

Choosing the wrong lifetime can introduce subtle bugs or memory issues.

---

# 20. One Interview Tip

When answering architecture or language questions:

Don't stop at **what**.

Continue with:

- Why?
- Trade-offs?
- When would you choose something else?

Example:

Instead of saying:

> "Dictionary is O(1)."

Say:

> "Dictionary provides average O(1) lookup because it uses a hash table. However, poor hash distribution can increase collisions and degrade performance toward O(n)."

That additional explanation demonstrates understanding rather than memorization.

---

# Final Advice

A Senior engineer is not expected to know every CLR implementation detail.

They are expected to:

- Understand core runtime concepts.
- Explain trade-offs.
- Recognize common pitfalls.
- Make informed engineering decisions.
- Communicate clearly.

The strongest interview answers are usually the ones that explain **why**, not just **what**.
