# Cancellation Tokens

## Overview

Modern .NET applications should avoid performing work that is no longer needed.

A `CancellationToken` allows an operation to stop cooperatively when cancellation is requested.

Typical scenarios include:

- Client disconnects
- Request timeout
- Application shutdown
- Background service stopping
- User cancels an operation

Using cancellation correctly improves scalability by avoiding wasted CPU, memory and network resources.

---

# 1. What is a CancellationToken?

## Interview Answer

A `CancellationToken` is a lightweight structure used to signal that an operation should stop.

It does **not** forcibly terminate execution.

Instead, it allows code to cooperate by periodically checking whether cancellation has been requested.

---

# 2. Cancellation Flow

```text
Request Starts

↓

CancellationToken Created

↓

Operation Running

↓

Cancellation Requested

↓

Operation Stops Gracefully
```

Cancellation is cooperative, not preemptive.

---

# 3. CancellationTokenSource

Cancellation begins with:

```csharp
var cts =
    new CancellationTokenSource();

CancellationToken token =
    cts.Token;
```

Request cancellation:

```csharp
cts.Cancel();
```

Every operation using that token is notified.

---

# 4. Checking for Cancellation

Typical loop:

```csharp
while (!token.IsCancellationRequested)
{
    // Work
}
```

Or explicitly throw:

```csharp
token.ThrowIfCancellationRequested();
```

This throws an `OperationCanceledException`.

---

# Interview Question

### Does CancellationToken stop a thread?

## Answer

No.

It simply signals that cancellation has been requested.

The running code decides how and when to stop.

---

# 5. ASP.NET Core

Every HTTP request has its own cancellation token.

Example:

```csharp
public async Task<IActionResult> Get(
    CancellationToken cancellationToken)
{
}
```

If the client disconnects:

```text
Browser Closed

↓

RequestAborted

↓

Cancellation Requested
```

The application should stop unnecessary work.

---

# 6. HttpClient

Always pass the token.

Example:

```csharp
await httpClient.GetAsync(
    url,
    cancellationToken);
```

If the request is cancelled:

- Network operations stop.
- Resources are released earlier.
- Threads become available sooner.

---

# 7. Entity Framework Core

EF Core supports cancellation.

Example:

```csharp
await context.Users
    .ToListAsync(cancellationToken);
```

Long-running database queries can be cancelled when the client no longer needs the result.

---

# 8. Background Services

`BackgroundService` provides:

```csharp
ExecuteAsync(
    CancellationToken stoppingToken)
```

Typical pattern:

```csharp
while (!stoppingToken
    .IsCancellationRequested)
{
    ...
}
```

This allows graceful shutdown.

---

# 9. Chaining Cancellation

Always propagate tokens.

Example:

```text
Controller

↓

Service

↓

Repository

↓

HttpClient

↓

Database
```

Every layer should pass the same token downstream.

Do not create new tokens unless there is a specific reason.

---

# 10. Timeouts

Timeouts often create cancellation automatically.

Example:

```csharp
using var cts =
    new CancellationTokenSource(
        TimeSpan.FromSeconds(30));
```

After 30 seconds:

```text
Cancellation Requested
```

Timeouts and user cancellation use the same mechanism.

---

# 11. Linked Tokens

Sometimes multiple cancellation sources should be combined.

Example:

```text
User Cancels

OR

Application Stops

↓

Cancel Operation
```

Use:

```csharp
CancellationTokenSource
.CreateLinkedTokenSource(...)
```

This creates a token cancelled when **any** source is cancelled.

---

# 12. Common Interview Questions

### Should every async method accept a CancellationToken?

## Answer

Public asynchronous methods that perform meaningful work should generally accept and propagate a `CancellationToken`.

---

### Why propagate tokens?

Because cancellation is only effective if every layer cooperates.

If one layer ignores the token, unnecessary work continues.

---

### Should OperationCanceledException be treated as an error?

Usually no.

Cancellation is expected behavior.

Applications often log it at Debug or Information level rather than Error.

---

### What happens if cancellation is ignored?

The operation continues consuming resources even though the result is no longer needed.

---

# Best Practices

- Accept `CancellationToken` in async APIs.
- Propagate tokens to downstream operations.
- Pass tokens to EF Core and HttpClient.
- Respect cancellation in loops.
- Avoid swallowing `OperationCanceledException`.
- Do not create unnecessary `CancellationTokenSource` instances.

---

# Trade-offs

| Approach | Benefit | Trade-off |
|----------|---------|-----------|
| CancellationToken | Better scalability | Requires propagation |
| Timeout-based Cancellation | Prevents hanging operations | Timeout selection matters |
| Linked Tokens | Flexible cancellation | Slightly more complexity |
| Ignoring Cancellation | Simpler code | Wasted resources |

---

# Staff Engineer Insight

A common misconception is that cancellation exists only for long-running operations.

In reality, cancellation is important for **resource efficiency**.

Example:

A client disconnects after 100 ms.

Without cancellation:

- Database query continues.
- External API call continues.
- JSON serialization continues.
- CPU continues working.

All of that work is wasted.

Another common mistake is creating a new `CancellationTokenSource` in every layer.

The request already provides a cancellation token.

Most application layers should simply propagate it rather than replacing it.

Well-designed systems treat cancellation as part of normal control flow rather than exceptional behavior.

---

# Summary

- `CancellationToken` provides cooperative cancellation.
- It does not forcibly stop threads.
- ASP.NET Core automatically supplies a request cancellation token.
- Propagate tokens through every application layer.
- Pass tokens to `HttpClient`, EF Core and background workers.
- Use linked tokens when multiple cancellation sources exist.
- Treat cancellation as expected behavior, not necessarily an error.
- Proper cancellation improves scalability and resource utilization.
