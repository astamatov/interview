# ASP.NET Core Middleware Pipeline

## Overview

The Middleware Pipeline is the core request processing mechanism in ASP.NET Core.

Every HTTP request passes through a sequence of middleware components before reaching the application endpoint.

Likewise, every HTTP response travels back through the same middleware in reverse order.

Understanding the middleware pipeline is essential for building secure, performant and maintainable ASP.NET Core applications.

---

# 1. What is Middleware?

## Interview Answer

Middleware is a software component that processes HTTP requests and responses.

Each middleware can:

- Execute logic before the next middleware
- Call the next middleware
- Execute logic after the next middleware
- Short-circuit the request completely

Simplified flow:

```text
HTTP Request

↓

Middleware A

↓

Middleware B

↓

Middleware C

↓

Endpoint

↑

Middleware C

↑

Middleware B

↑

Middleware A

↓

HTTP Response
```

---

# 2. Request and Response Pipeline

Every middleware executes twice:

First:

```text
Before Next()
```

Then:

```text
After Next()
```

Example:

```text
Request

↓

Logging

↓

Authentication

↓

Authorization

↓

Controller

↑

Authorization

↑

Authentication

↑

Logging

↓

Response
```

This behavior is often compared to nested function calls.

---

# 3. The RequestDelegate

Each middleware receives a `RequestDelegate`.

Example:

```csharp
public async Task InvokeAsync(
    HttpContext context,
    RequestDelegate next)
{
    await next(context);
}
```

Calling:

```csharp
await next(context);
```

passes execution to the next middleware.

---

# 4. Short-Circuiting

Not every middleware calls `next()`.

Example:

```text
Request

↓

Static Files

↓

File Exists?

↓

Yes

↓

Return File
```

The remaining middleware never executes.

Examples:

- Static Files
- Authentication failures
- Custom error middleware
- Rate limiting

---

# Interview Question

### What happens if middleware doesn't call next()?

## Answer

The request pipeline stops.

The request never reaches later middleware or endpoints.

This is called **short-circuiting**.

---

# 5. Middleware Order

Order matters.

Correct example:

```text
Exception Handling

↓

HTTPS Redirection

↓

Static Files

↓

Routing

↓

Authentication

↓

Authorization

↓

Endpoints
```

Authentication must execute before Authorization.

Otherwise, Authorization has no authenticated user to evaluate.

---

# 6. Built-in Middleware

Common middleware:

- Exception Handler
- HTTPS Redirection
- Static Files
- Routing
- CORS
- Authentication
- Authorization
- Response Compression
- Rate Limiting
- Output Caching

Each solves one responsibility.

---

# 7. Routing Middleware

Routing identifies which endpoint should handle the request.

```text
GET /users/10

↓

Routing

↓

UsersController.Get()
```

Routing itself does not execute the endpoint.

It only selects it.

---

# 8. Authentication vs Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
Are you allowed?
```

Correct order:

```text
Authentication

↓

Authorization
```

Incorrect order causes authorization failures.

---

# 9. Custom Middleware

Example:

```csharp
public class TimingMiddleware
{
    public async Task InvokeAsync(
        HttpContext context,
        RequestDelegate next)
    {
        var sw = Stopwatch.StartNew();

        await next(context);

        sw.Stop();
    }
}
```

Typical custom middleware:

- Logging
- Correlation IDs
- Request timing
- Multi-tenancy
- Audit logging

---

# 10. Exception Handling Middleware

Unhandled exceptions should be processed centrally.

Instead of:

```text
Controller

↓

try/catch
```

Use:

```text
Exception Middleware

↓

Log

↓

Return ProblemDetails
```

Centralized exception handling produces consistent API responses.

---

# 11. Endpoint Execution

The endpoint is the final destination.

Examples:

- Controller Action
- Minimal API
- Razor Page
- SignalR Hub

Once the endpoint finishes, execution returns through the middleware stack.

---

# 12. Common Interview Questions

### Why is middleware order important?

## Answer

Middleware depends on previous middleware.

Authentication must run before Authorization.

Routing must occur before endpoint execution.

Incorrect ordering changes application behavior.

---

### Does every request execute every middleware?

No.

Requests may short-circuit.

Static files, authentication failures or custom middleware may return responses early.

---

### Can middleware modify the response?

Yes.

Middleware can inspect or modify both the request and the response.

For example:

- Add headers
- Compress responses
- Log execution time
- Transform content

---

### Can middleware call services from Dependency Injection?

Yes.

Middleware supports constructor injection and method parameter injection.

---

# Common Pipeline

```text
Request

↓

Exception Handler

↓

HTTPS

↓

Static Files

↓

Routing

↓

CORS

↓

Authentication

↓

Authorization

↓

Endpoints

↓

Response
```

---

# Trade-offs

| Middleware | Benefit | Trade-off |
|------------|---------|-----------|
| Exception Handler | Centralized error handling | Must execute early |
| Authentication | User identity | Authentication cost |
| Authorization | Security | Depends on Authentication |
| Response Compression | Lower bandwidth | CPU overhead |
| Output Caching | Faster responses | Cache invalidation |

---

# Staff Engineer Insight

A common mistake is treating middleware as business logic.

Middleware should handle **cross-cutting concerns**, not domain behavior.

Good middleware examples:

- Logging
- Authentication
- Correlation IDs
- Metrics
- Exception handling
- Security headers

Poor middleware examples:

- Customer validation
- Order processing
- Pricing logic
- Business rules

Those belong in the application layer.

Another common production issue is incorrect middleware ordering.

Many authentication and authorization bugs are caused by configuration order rather than incorrect code.

Always understand the execution flow before debugging middleware behavior.

---

# Summary

- Every HTTP request passes through the middleware pipeline.
- Middleware executes before and after the next component.
- Calling `next()` continues the pipeline.
- Middleware can short-circuit requests.
- Order is critical.
- Authentication must run before Authorization.
- Middleware is ideal for cross-cutting concerns.
- Endpoints execute after the middleware pipeline selects the correct route.
