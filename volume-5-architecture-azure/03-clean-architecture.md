# Clean Architecture

## Overview

Clean Architecture, proposed by Robert C. Martin (Uncle Bob), is an architectural style that organizes code around the business domain instead of frameworks or technologies.

The main idea is simple:

> **Business rules should not depend on infrastructure. Infrastructure should depend on business rules.**

This makes applications easier to test, maintain and evolve.

---

# 1. What is Clean Architecture?

## Interview Answer

Clean Architecture separates the application into layers with clear responsibilities.

The dependencies always point **towards the center**, where the business rules live.

```
Presentation

↓

Application

↓

Domain

↑

Infrastructure
```

Notice that Infrastructure depends on Domain, not the other way around.

---

# 2. The Four Layers

## Domain

The core of the application.

Contains:

- Entities
- Value Objects
- Domain Services
- Business Rules

The Domain should have **zero knowledge** of:

- SQL Server
- Entity Framework
- ASP.NET Core
- Azure
- Redis

Example:

```csharp
public class Order
{
    public void Approve()
    {
        if(Status == OrderStatus.Approved)
            throw new InvalidOperationException();

        Status = OrderStatus.Approved;
    }
}
```

This logic belongs to the business, not the database.

---

## Application

Coordinates use cases.

Contains:

- Commands
- Queries
- DTOs
- Interfaces
- Validation
- Business workflows

Example:

```
ApproveOrderCommand

↓

ApproveOrderHandler

↓

IOrderRepository

↓

Order
```

Application knows **what** should happen.

Not **how** data is stored.

---

## Infrastructure

Contains technical implementations.

Examples:

- Entity Framework Core
- SQL Server
- Redis
- Azure Blob Storage
- Email Service
- External APIs

Example:

```csharp
public class OrderRepository
    : IOrderRepository
{
}
```

Infrastructure implements interfaces defined by the Application layer.

---

## Presentation

Everything related to HTTP.

Examples:

- Controllers
- Minimal APIs
- SignalR Hubs
- gRPC Endpoints

Controllers should remain thin.

Bad:

```
Controller

↓

Validation

↓

Business Logic

↓

SQL
```

Good:

```
Controller

↓

Mediator

↓

Handler

↓

Repository
```

---

# 3. Dependency Rule

## Interview Answer

Dependencies always point inward.

Correct:

```
Infrastructure

↓

Application

↓

Domain
```

Incorrect:

```
Domain

↓

Entity Framework

↓

SQL Server
```

The Domain should never reference Entity Framework.

---

# Interview Question

### Why shouldn't the Domain reference EF Core?

## Answer

Because the business rules should not depend on a specific ORM.

Tomorrow you might replace EF Core with Dapper or MongoDB.

The business logic should remain unchanged.

---

# 4. Repository Pattern

Application defines the contract.

```csharp
public interface IUserRepository
{
    Task<User?> GetById(int id);
}
```

Infrastructure implements it.

```csharp
public class UserRepository
    : IUserRepository
{
}
```

The Domain never knows which database is used.

---

# 5. Dependency Injection

At startup:

```csharp
builder.Services.AddScoped<
    IUserRepository,
    UserRepository>();
```

The Application depends only on `IUserRepository`.

It doesn't know the concrete implementation.

---

# 6. Benefits

- Easier testing
- Better separation of concerns
- Replace infrastructure easily
- Independent business logic
- Better maintainability

---

# 7. Common Mistakes

## Fat Controllers

Wrong:

```csharp
public IActionResult Create(...)
{
    ...

    db.Users.Add(...);

    ...

    SendEmail();

    ...

    return Ok();
}
```

Controllers become difficult to maintain.

---

## Business Logic in Repository

Repositories should retrieve and persist data.

Not contain business rules.

Wrong:

```csharp
public void ApproveOrder(...)
{
    ...
}
```

Repositories should not decide business behavior.

---

## Domain Depends on Infrastructure

Wrong:

```csharp
public class User
{
    private readonly DbContext _context;
}
```

Entities should never depend on EF Core.

---

# 8. Is Clean Architecture Always the Right Choice?

## Interview Answer

No.

For a simple CRUD application with five endpoints, Clean Architecture may introduce unnecessary complexity.

Use it when:

- Business logic is complex.
- Multiple developers work on the project.
- The application is expected to evolve over years.
- Different infrastructure components may change.

---

# 9. Common Interview Questions

### Can the Application reference Infrastructure?

No.

Only through interfaces.

---

### Can Infrastructure reference Application?

Yes.

Infrastructure implements Application interfaces.

---

### Where should validation live?

Application layer.

Often using FluentValidation.

---

### Where should EF Core DbContext live?

Infrastructure.

It is an implementation detail.

---

### Should DTOs exist in the Domain?

No.

DTOs are transport objects.

The Domain should expose business concepts, not API contracts.

---

# Trade-offs

| Advantage | Disadvantage |
|-----------|--------------|
| Easier testing | More projects/files |
| Clear separation | Higher learning curve |
| Framework independence | Can feel verbose |
| Better maintainability | Overkill for small apps |

---

# Staff Engineer Insight

One of the biggest misconceptions is believing that Clean Architecture is about folders.

It is not.

You can have:

```
Domain/

Application/

Infrastructure/
```

and still completely violate Clean Architecture by letting your Domain depend on EF Core.

The architecture is defined by **dependency direction**, not directory structure.

Another common mistake is introducing every possible pattern from day one.

A good architect starts with the simplest design that solves today's problem and evolves it as the system grows.

---

# Summary

- Clean Architecture keeps business rules independent of frameworks.
- The Domain is the center of the application.
- Infrastructure implements interfaces defined by the Application layer.
- Controllers should remain thin.
- Repositories handle persistence, not business logic.
- Dependencies always point inward.
- Choose Clean Architecture when complexity justifies it.
