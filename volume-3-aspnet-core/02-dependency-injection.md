# Dependency Injection (DI)

## Overview

Dependency Injection (DI) is one of the core architectural concepts in modern .NET applications. ASP.NET Core has a built-in IoC (Inversion of Control) container that manages object creation, dependency resolution and service lifetimes.

A senior engineer should understand **how the container resolves dependencies, what service lifetimes mean, when problems occur and how to design applications around DI.**

---

# 1. What is Dependency Injection?

## Answer

Dependency Injection is a design pattern where an object's dependencies are provided from the outside instead of being created inside the object.

Without DI:

```csharp
public class OrderService
{
    private readonly EmailService _email =
        new EmailService();
}
```

Problems:

- Tight coupling
- Hard to test
- Hard to replace implementations

With DI:

```csharp
public class OrderService
{
    private readonly IEmailService _email;

    public OrderService(IEmailService email)
    {
        _email = email;
    }
}
```

Now the service depends on an abstraction instead of a concrete implementation.

---

# 2. What is IoC?

## Answer

IoC (Inversion of Control) means that object creation is no longer the responsibility of your classes.

Instead of this:

```csharp
var service = new OrderService(
    new EmailService());
```

The container creates everything.

```csharp
builder.Services.AddScoped<IEmailService, EmailService>();

builder.Services.AddScoped<OrderService>();
```

Later:

```csharp
var service =
    provider.GetRequiredService<OrderService>();
```

---

# 3. How does the DI container resolve dependencies?

Suppose:

```csharp
OrderController

↓

OrderService

↓

EmailService

↓

ILogger
```

The container builds an object graph.

```
OrderController

↓

OrderService

↓

EmailService

↓

ILogger
```

It recursively resolves constructor dependencies until the graph is complete.

---

# 4. Constructor Injection

## Answer

Constructor injection is the preferred way to inject dependencies.

Example:

```csharp
public class UserService
{
    private readonly IUserRepository _repository;

    public UserService(
        IUserRepository repository)
    {
        _repository = repository;
    }
}
```

Advantages:

- Required dependencies are explicit.
- Object is always valid after construction.
- Easier testing.
- Immutable dependencies.

---

# 5. Service Lifetimes

ASP.NET Core supports three lifetimes.

---

## Transient

```csharp
builder.Services.AddTransient<IEmailService, EmailService>();
```

A new instance is created every time the service is requested.

Use for:

- Lightweight, stateless services.

---

## Scoped

```csharp
builder.Services.AddScoped<IUserRepository, UserRepository>();
```

One instance per HTTP request.

Every component within the same request receives the same instance.

This is the recommended lifetime for:

- DbContext
- Repositories
- Business services

---

## Singleton

```csharp
builder.Services.AddSingleton<ICacheService, CacheService>();
```

One instance for the lifetime of the application.

Shared by every request and every thread.

Use for:

- Configuration
- In-memory cache
- Stateless utilities

---

# 6. Which lifetime should DbContext use?

## Answer

Scoped.

Each HTTP request should have its own DbContext.

Never register DbContext as Singleton.

Never share DbContext across requests.

---

# Interview Question

### Why not Singleton?

Because DbContext is **not thread-safe**.

Sharing one instance between requests causes:

- Race conditions
- Tracking issues
- Memory growth
- Random exceptions

---

# 7. Captive Dependency

One of the most common senior interview questions.

Example:

```
Singleton

↓

Scoped
```

```csharp
public class CacheService
{
    public CacheService(UserRepository repository)
    {
    }
}
```

CacheService is Singleton.

UserRepository is Scoped.

Problem:

The singleton keeps a reference to the scoped object forever.

This usually results in runtime exceptions.

---

## Rule

A longer-lived service must never depend on a shorter-lived service.

```
Singleton

↓

Scoped

❌
```

```
Scoped

↓

Singleton

✔
```

---

# 8. Circular Dependencies

Example:

```
OrderService

↓

PaymentService

↓

OrderService
```

The container cannot build the object graph.

Runtime exception:

```
A circular dependency was detected...
```

Solution:

Redesign responsibilities.

Avoid service cycles.

---

# 9. IServiceProvider

The DI container itself.

Example:

```csharp
var logger =
    provider.GetRequiredService<ILogger<User>>();
```

Normally avoid injecting IServiceProvider everywhere.

Prefer constructor injection.

---

# 10. Factory Pattern

Sometimes objects require runtime parameters.

Example:

```csharp
public interface IPaymentProcessorFactory
{
    IPaymentProcessor Create(string provider);
}
```

Factories work well with DI.

---

# 11. Open Generic Registration

```csharp
builder.Services.AddScoped(
    typeof(IRepository<>),
    typeof(Repository<>));
```

Now:

```
IRepository<User>

↓

Repository<User>

IRepository<Order>

↓

Repository<Order>
```

No extra registrations needed.

---

# 12. Common Interview Questions

### Is constructor injection always preferred?

Yes.

Avoid property injection unless absolutely necessary.

---

### Can Singleton depend on Singleton?

Yes.

---

### Can Scoped depend on Singleton?

Yes.

---

### Can Singleton depend on Scoped?

No.

---

### Why?

Because the Singleton would hold a reference to an object that should exist only during a single request.

---

### Why avoid Service Locator?

Example:

```csharp
public class OrderService
{
    public OrderService(IServiceProvider provider)
    {
    }
}
```

Problems:

- Hidden dependencies
- Harder testing
- Harder maintenance

Constructor injection makes dependencies explicit.

---

# Staff Engineer Insight

A common mistake in large systems is putting too much business logic into Singleton services because they seem "more efficient."

In reality, most business services should be **Scoped**. They often depend on request-specific information (current user, DbContext, transactions) and keeping them Singleton introduces subtle bugs and lifetime issues.

Another common optimization mistake is creating services manually with `new` to "avoid DI overhead." The built-in container is already highly optimized. The loss of testability and maintainability is almost never worth the tiny performance gain.

---

# Summary

- Dependency Injection reduces coupling.
- Constructor injection is the preferred approach.
- The built-in container builds the dependency graph automatically.
- Service lifetimes:
  - Transient → new instance every resolution.
  - Scoped → one instance per request.
  - Singleton → one instance for the application.
- DbContext should be Scoped.
- Avoid captive dependencies.
- Avoid circular dependencies.
- Prefer constructor injection over Service Locator.
