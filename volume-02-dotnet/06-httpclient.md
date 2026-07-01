# HttpClient

## Overview

`HttpClient` is the primary API for making HTTP requests in .NET.

Although it appears simple, incorrect usage can lead to serious production issues such as:

- Socket exhaustion
- DNS resolution problems
- Poor performance
- Resource leaks

Modern .NET applications should use `IHttpClientFactory` to manage `HttpClient` instances efficiently.

A senior developer should understand not only how to use `HttpClient`, but why the recommended patterns exist.

---

# 1. What is HttpClient?

## Interview Answer

`HttpClient` is a high-level API used to send HTTP requests and receive HTTP responses.

Example:

```csharp
var response =
await httpClient.GetAsync("/users");
```

It supports:

- GET
- POST
- PUT
- DELETE
- PATCH

as well as custom HTTP methods.

---

# 2. Why is this Wrong?

```csharp
public async Task GetUser()
{
    using var client =
        new HttpClient();

    await client.GetAsync(...);
}
```

Many developers assume this is correct.

It is not.

Creating a new `HttpClient` for every request can lead to socket exhaustion.

---

# Interview Question

### Why shouldn't HttpClient be created per request?

## Answer

Because each `HttpClient` creates an underlying connection handler.

Disposing the client does **not** immediately close TCP connections.

Large numbers of short-lived clients can exhaust available sockets.

---

# 3. Socket Exhaustion

Bad pattern:

```text
Request

↓

New HttpClient

↓

New TCP Connection

↓

Dispose

↓

Connection stays in TIME_WAIT
```

Repeat this thousands of times:

```text
No Available Sockets
```

The application begins failing outbound requests.

---

# 4. IHttpClientFactory

## Interview Answer

`IHttpClientFactory` manages `HttpClient` lifetimes correctly.

Example:

```csharp
builder.Services
.AddHttpClient();
```

Usage:

```csharp
public UserService(
    IHttpClientFactory factory)
{
    _client =
        factory.CreateClient();
}
```

Benefits:

- Connection pooling
- DNS refresh
- Central configuration
- Polly integration
- Logging support

---

# 5. Named Clients

Example:

```csharp
builder.Services
.AddHttpClient(
    "Payments",
    client =>
{
    client.BaseAddress =
        new Uri(...);
});
```

Usage:

```csharp
var client =
factory.CreateClient("Payments");
```

Useful when communicating with multiple external APIs.

---

# 6. Typed Clients

Example:

```csharp
builder.Services
.AddHttpClient<
PaymentClient>();
```

Usage:

```csharp
public class PaymentClient
{
    private readonly HttpClient _client;

    public PaymentClient(
        HttpClient client)
    {
        _client = client;
    }
}
```

Benefits:

- Strong typing
- Better encapsulation
- Easier testing
- Cleaner dependency injection

---

# 7. Generated Clients

Tools such as:

- Refit
- NSwag
- Kiota

can generate strongly typed API clients automatically.

This reduces boilerplate code.

---

# 8. DNS Refresh Problem

Long-lived `HttpClient` instances may continue using stale IP addresses if DNS changes.

Example:

```text
Service

↓

DNS Changes

↓

Old HttpClient

↓

Old IP
```

`IHttpClientFactory` periodically rotates handlers, allowing DNS changes to be respected.

---

# 9. HttpMessageHandler

Architecture:

```text
HttpClient

↓

Delegating Handlers

↓

SocketsHttpHandler

↓

TCP
```

The handler performs the actual network communication.

`HttpClient` itself is relatively lightweight.

---

# 10. Delegating Handlers

Handlers allow request/response processing.

Example:

```text
Request

↓

Logging Handler

↓

Authentication Handler

↓

Retry Handler

↓

Network
```

Typical use cases:

- Authentication
- Logging
- Metrics
- Correlation IDs
- Retry

---

# 11. Timeouts

Always configure appropriate timeouts.

Example:

```csharp
client.Timeout =
TimeSpan.FromSeconds(30);
```

Infinite waits should be avoided.

Long-running operations should use `CancellationToken`.

---

# 12. Cancellation

Example:

```csharp
await client.GetAsync(
    url,
    cancellationToken);
```

Cancellation:

- Releases resources earlier
- Prevents unnecessary work
- Improves scalability

Always propagate cancellation tokens when possible.

---

# 13. Common Interview Questions

### Should HttpClient be Singleton?

## Answer

Not directly.

Instead, use `IHttpClientFactory`, which manages underlying handlers efficiently.

---

### Why is IHttpClientFactory recommended?

Because it solves:

- Socket exhaustion
- DNS refresh
- Central configuration
- Resilience integration

---

### What is Socket Exhaustion?

Too many TCP connections remain in `TIME_WAIT`, preventing new outbound connections.

---

### When should Typed Clients be used?

When interacting with a specific external service.

They improve organization, encapsulation and testability.

---

# Best Practices

- Use `IHttpClientFactory`.
- Configure timeouts.
- Use typed clients for external services.
- Propagate cancellation tokens.
- Avoid creating `HttpClient` per request.
- Centralize configuration.

---

# Trade-offs

| Approach | Benefit | Trade-off |
|----------|---------|-----------|
| New HttpClient | Simple | Socket exhaustion |
| Singleton HttpClient | Reuses connections | DNS refresh issues |
| IHttpClientFactory | Recommended approach | Slightly more setup |
| Typed Client | Better organization | Additional abstraction |

---

# Staff Engineer Insight

One of the biggest misconceptions is:

> "HttpClient is expensive."

It is not.

The expensive part is the underlying **connection management**.

The real value of `IHttpClientFactory` is not object reuse.

It is:

- Connection pooling
- Handler lifetime management
- DNS refresh
- Resilience integration
- Central configuration

Another common mistake is placing business logic inside API clients.

A typed client should primarily be responsible for:

- Building requests
- Sending requests
- Mapping responses
- Handling transport-level concerns

Business rules belong in the application layer, not the transport layer.

---

# Summary

- `HttpClient` is the standard HTTP client in .NET.
- Never create a new `HttpClient` for every request.
- Use `IHttpClientFactory` in modern applications.
- Socket exhaustion results from poor connection management.
- Typed clients improve organization and testability.
- Delegating handlers enable logging, authentication and retries.
- Configure timeouts and propagate cancellation tokens.
- Focus on managing connections rather than reusing `HttpClient` objects.
