# CQRS and MediatR

## Overview

CQRS (Command Query Responsibility Segregation) is an architectural pattern that separates operations that modify data from operations that read data.

Many developers think CQRS means using two databases or microservices. That is incorrect.

CQRS is about separating **responsibilities**, not necessarily infrastructure.

---

# 1. What is CQRS?

## Interview Answer

CQRS separates operations into two categories:

- Commands → modify data
- Queries → read data

Instead of having one service responsible for everything:

```
UserService

Create()

Update()

Delete()

Get()

GetAll()
```

We split responsibilities.

```
Commands

↓

CreateUser

UpdateUser

DeleteUser

-----------------

Queries

↓

GetUser

GetUsers
```

This separation makes the application easier to maintain and evolve.

---

# 2. Why use CQRS?

## Answer

Reading and writing often have different requirements.

Example:

Reading:

- Fast
- Optimized projections
- Cached

Writing:

- Validation
- Transactions
- Business rules

Separating them allows each side to evolve independently.

---

# Interview Question

### Does CQRS require two databases?

## Answer

No.

CQRS can work perfectly with a single SQL database.

Many systems start with one database and only introduce separate read models if performance requires it.

---

# 3. Command

A command changes system state.

Examples:

```
CreateOrder

UpdateCustomer

DeleteProduct

ApproveInvoice
```

Commands usually:

- Return nothing
- Return an ID
- Return success/failure

Example:

```csharp
public record CreateUserCommand(
    string Name,
    string Email);
```

---

# 4. Query

Queries never modify data.

Example:

```csharp
public record GetUserQuery(
    int Id);
```

Queries should not have side effects.

They simply return data.

---

# 5. What is MediatR?

## Interview Answer

MediatR is a library that implements the Mediator pattern.

Instead of controllers depending directly on services, controllers send commands and queries to a mediator.

```
Controller

↓

Mediator

↓

Handler
```

The controller does not know who handles the request.

This reduces coupling.

---

# 6. Without MediatR

```csharp
public class UserController
{
    private readonly UserService _service;

    public UserController(UserService service)
    {
        _service = service;
    }

    public async Task Create(CreateUserDto dto)
    {
        await _service.Create(dto);
    }
}
```

Controller depends directly on UserService.

---

# 7. With MediatR

```csharp
public class UserController
{
    private readonly IMediator _mediator;

    public UserController(IMediator mediator)
    {
        _mediator = mediator;
    }

    public async Task Create(CreateUserCommand command)
    {
        await _mediator.Send(command);
    }
}
```

The controller only knows about the mediator.

---

# 8. Command Handler

Example:

```csharp
public class CreateUserHandler
    : IRequestHandler<CreateUserCommand>
{
    public async Task Handle(
        CreateUserCommand request,
        CancellationToken cancellationToken)
    {
        ...
    }
}
```

Each command has exactly one handler.

---

# 9. Query Handler

Example:

```csharp
public class GetUserHandler
    : IRequestHandler<GetUserQuery, UserDto>
{
    public async Task<UserDto> Handle(
        GetUserQuery request,
        CancellationToken cancellationToken)
    {
        ...
    }
}
```

Again:

One query.

One handler.

---

# 10. Pipeline Behaviors

One of MediatR's most powerful features.

Instead of writing logging, validation and authorization inside every handler, MediatR allows pipeline behaviors.

```
Validation

↓

Logging

↓

Performance

↓

Handler
```

Example:

```csharp
public class ValidationBehavior<TRequest, TResponse>
```

Now every command automatically goes through validation.

---

# 11. Common Use Cases

Pipeline Behaviors are commonly used for:

- Validation
- Logging
- Performance monitoring
- Authorization
- Transactions
- Retry policies

This keeps handlers focused only on business logic.

---

# 12. Advantages of CQRS

- Clear separation of responsibilities.
- Easier testing.
- Smaller classes.
- Better scalability.
- Independent optimization of reads and writes.
- Simpler maintenance.

---

# 13. Disadvantages

- More files.
- More classes.
- Higher complexity.
- Can be overkill for small CRUD applications.

---

# Interview Question

### Should every project use CQRS?

## Answer

No.

For simple CRUD applications, CQRS often introduces unnecessary complexity.

It becomes valuable when:

- Business rules are complex.
- The application is large.
- Read and write workloads differ significantly.
- Multiple developers work on the same domain.

---

# 14. Common Interview Questions

### Is MediatR required for CQRS?

No.

CQRS is an architectural pattern.

MediatR is simply a convenient implementation.

---

### Can one handler call another handler?

Generally avoid it.

Extract shared business logic into a domain service instead.

This keeps handlers independent.

---

### Where should validation happen?

Prefer pipeline behaviors or FluentValidation instead of putting validation inside controllers.

Controllers should remain thin.

---

# Staff Engineer Insight

One mistake I often see is teams introducing MediatR because "everyone uses it."

Ask yourself:

- Does this application actually benefit from CQRS?
- Are commands and queries becoming difficult to maintain?
- Is business logic growing?

Patterns should solve problems.

They should not exist simply because they are popular.

A clean CRUD application without CQRS is often a better solution than an over-engineered architecture.

---

# Summary

- CQRS separates reads from writes.
- Commands modify state.
- Queries return data.
- MediatR implements the Mediator pattern.
- Pipeline Behaviors centralize cross-cutting concerns.
- CQRS does not require multiple databases.
- Introduce CQRS when complexity justifies it, not by default.
