# ASP.NET Core Request Pipeline & Middleware

## Overview

Every HTTP request in ASP.NET Core passes through a pipeline of middleware components before reaching your controller or Minimal API endpoint.

Understanding the request pipeline is essential because almost every ASP.NET Core feature is implemented as middleware.

---

# 1. What is the Request Pipeline?

## Answer

The ASP.NET Core request pipeline is a sequence of middleware components executed for every HTTP request.

Each middleware can:

- Handle the request
- Modify the request
- Call the next middleware
- Short-circuit the pipeline
- Modify the response

Think of it as a chain of responsibility.

```
HTTP Request

↓

Exception Middleware

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

Endpoint

↓

Controller

↓

Response
```

---

## Interview Tip

Most middleware executes twice.

Once on the request.

Once again while the response travels back.

---

# 2. What is Middleware?

## Answer

Middleware is a component that participates in request processing.

Every middleware receives:

- HttpContext
- RequestDelegate next

Example:

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;

    public LoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        Console.WriteLine("Request started");

        await _next(context);

        Console.WriteLine("Response finished");
    }
}
```

Register:

```csharp
app.UseMiddleware<LoggingMiddleware>();
```

---

# 3. Why is Middleware Order Important?

## Answer

Middleware executes **in the order it is registered**.

Example:

```csharp
app.UseAuthentication();

app.UseAuthorization();
```

Correct.

Wrong:

```csharp
app.UseAuthorization();

app.UseAuthentication();
```

Authorization runs before the user is authenticated.

Every request fails authorization.

---

## Common Interview Question

### Why must Routing execute before Authorization?

Because Authorization needs to know which endpoint is being executed.

Routing determines that endpoint.

---

# 4. app.Use vs app.Run vs app.Map

## app.Use()

Continues the pipeline.

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Before");

    await next();

    Console.WriteLine("After");
});
```

---

## app.Run()

Terminates the pipeline.

```csharp
app.Run(async context =>
{
    await context.Response.WriteAsync("Hello");
});
```

Nothing after `Run()` executes.

---

## app.Map()

Creates a branch.

```csharp
app.Map("/health", builder =>
{
    builder.Run(async context =>
    {
        await context.Response.WriteAsync("Healthy");
    });
});
```

Only `/health` requests enter that branch.

---

# 5. HttpContext

## Answer

`HttpContext` contains everything related to the current HTTP request.

Examples:

```csharp
context.Request

context.Response

context.User

context.Items

context.RequestServices
```

Think of it as the container for request-specific information.

---

# 6. Dependency Injection in Middleware

Never inject scoped services through the middleware constructor.

Wrong:

```csharp
public LoggingMiddleware(
    UserService service)
```

Middleware is created once.

Scoped services are created per request.

Correct:

```csharp
public async Task Invoke(
    HttpContext context,
    UserService service)
{
}
```

ASP.NET Core resolves scoped services per request.

---

# 7. Short-Circuiting the Pipeline

A middleware can stop execution.

Example:

```csharp
if (!context.User.Identity.IsAuthenticated)
{
    context.Response.StatusCode = 401;
    return;
}
```

No later middleware executes.

---

# 8. Built-in Middleware

Common middleware:

```csharp
UseExceptionHandler()

UseHttpsRedirection()

UseRouting()

UseCors()

UseAuthentication()

UseAuthorization()

UseResponseCompression()

UseRateLimiter()

UseStaticFiles()
```

A senior engineer should know what each one does.

---

# 9. Exception Middleware

Instead of:

```csharp
try
{
}
catch
{
}
```

inside every controller,

ASP.NET Core typically uses one global exception middleware.

Benefits:

- Centralized logging
- Consistent error responses
- Cleaner controllers

---

# 10. Custom Middleware vs Filters

## Middleware

Runs for every request.

Good for:

- Logging
- Authentication
- Correlation IDs
- Exception handling

---

## Filters

Run only for MVC actions.

Good for:

- Validation
- Action logging
- Authorization
- Result transformation

---

# 11. Common Interview Questions

### Can middleware modify the response?

Yes.

Both before and after calling `next()`.

---

### Can middleware skip the controller?

Yes.

It can generate the response directly.

---

### Can middleware access Dependency Injection?

Yes.

Through constructor injection (Singleton services) or `Invoke()` parameters (Scoped services).

---

### Why is middleware preferred over code duplication?

Because cross-cutting concerns should be implemented once instead of repeating them in every controller.

---

# 12. Senior Tips

- Middleware should do one thing well.
- Keep middleware lightweight.
- Never perform long-running work synchronously.
- Use middleware for cross-cutting concerns.
- Learn the default ASP.NET Core middleware order by heart.

---

# Real Interview Questions

### Q:
Why must `UseAuthentication()` come before `UseAuthorization()`?

### A:
Authorization checks the current user and their claims. If authentication hasn't run yet, no user has been established, so authorization will always fail.

---

### Q:
When would you create custom middleware instead of an action filter?

### A:
Use middleware when the logic applies to **all HTTP requests**, regardless of whether they hit MVC, Minimal APIs or static files. Use filters when the logic is specific to MVC actions.

---

### Q:
Can multiple middleware modify the same response?

### A:
Yes. Since the response travels back through the pipeline, each middleware has an opportunity to inspect or modify it after `await next()` completes.

---

# Summary

- Every request flows through the middleware pipeline.
- Middleware executes in registration order.
- `Use()` continues the pipeline.
- `Run()` terminates the pipeline.
- `Map()` creates branches.
- `HttpContext` contains all request information.
- Middleware is the preferred place for cross-cutting concerns.
- Middleware order is critical for correct application behavior.
