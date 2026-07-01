# Domain-Driven Design (DDD)

## Overview

Domain-Driven Design (DDD), introduced by Eric Evans, is an approach to software design that focuses on modeling software around the business domain.

The primary goal is **to make the code reflect the business**, rather than the database or framework.

DDD is not a framework and it is not a project structure. It is a way of thinking about software.

---

# 1. What is a Domain?

## Interview Answer

A domain is the business area that the application solves.

Examples:

- Banking
- Healthcare
- E-commerce
- Asset Finance
- Insurance

DDD focuses on understanding business rules first and implementing technology second.

---

# 2. Entity

## Interview Answer

An Entity is an object that has a unique identity and lives throughout the application's lifecycle.

Even if its properties change, it is still considered the same entity.

Example:

```csharp
public class Customer
{
    public Guid Id { get; private set; }

    public string Name { get; private set; }
}
```

The `Name` may change.

The `Id` identifies the entity.

---

## Examples

Entities:

- Customer
- Order
- Invoice
- Product

Not entities:

- Money
- Address
- DateRange

Those are usually Value Objects.

---

# 3. Value Object

## Interview Answer

A Value Object has **no identity**.

It is defined entirely by its values.

Example:

```csharp
public record Money(
    decimal Amount,
    string Currency);
```

Two Money objects containing:

```
100 EUR
```

are considered equal.

---

## Why use Value Objects?

They:

- Are immutable.
- Are easier to test.
- Prevent invalid states.
- Better model business concepts.

---

## Example

Instead of:

```csharp
decimal amount;
string currency;
```

Use:

```csharp
Money price;
```

Now invalid combinations become harder to create.

---

# 4. Entity vs Value Object

| Entity | Value Object |
|---------|--------------|
| Has identity | No identity |
| Mutable | Usually immutable |
| Compared by Id | Compared by value |
| Lives long | Often short-lived |

---

# 5. Aggregate

## Interview Answer

An Aggregate is a group of related entities that should always remain consistent.

Example:

```
Order

↓

OrderLines

↓

Payments
```

The Order controls access to its children.

External code should not modify OrderLines directly.

---

# 6. Aggregate Root

The Aggregate Root is the only object that outside code may reference.

Example:

```
Order

↓

OrderLine

↓

Discount
```

External code interacts only with `Order`.

Example:

```csharp
order.AddItem(product);
```

Not:

```csharp
order.OrderLines.Add(...)
```

The Aggregate Root protects business rules.

---

# Interview Question

### Why use an Aggregate Root?

## Answer

To ensure business invariants are enforced in one place.

Without it, different parts of the application could modify child entities independently and leave the aggregate in an invalid state.

---

# 7. Domain Service

Not every business operation belongs inside an Entity.

Example:

```
Currency Conversion

Interest Calculation

Fraud Detection
```

These operations often involve multiple entities.

They belong in Domain Services.

---

# 8. Domain Events

## Interview Answer

A Domain Event represents something important that already happened in the business.

Examples:

```
OrderPlaced

InvoicePaid

CustomerRegistered
```

Events describe the past.

Notice the naming.

Good:

```
OrderApproved
```

Bad:

```
ApproveOrder
```

Commands describe intent.

Events describe facts.

---

## Example

```
Order Approved

↓

Raise Domain Event

↓

Send Email

↓

Generate Invoice

↓

Notify Warehouse
```

The Order doesn't know who reacts.

It only publishes the event.

---

# 9. Ubiquitous Language

One of the most important DDD concepts.

Developers and business experts should use the same terminology.

Bad:

```
Customer

↓

Client

↓

Account

↓

User
```

Four different names for the same concept.

Good:

Everyone calls it:

```
Customer
```

This reduces misunderstandings.

---

# 10. Bounded Context

Large companies often use the same word to mean different things.

Example:

```
Customer
```

Sales:

Potential buyer.

Billing:

Invoice recipient.

Support:

Person opening tickets.

DDD separates these into different bounded contexts.

Each context has its own model.

---

# 11. Rich Domain Model vs Anemic Domain Model

## Anemic Model

```csharp
public class Order
{
    public decimal Total;
}
```

Business logic lives elsewhere.

Usually inside services.

---

## Rich Domain Model

```csharp
public class Order
{
    public void Approve()
    {
        if(Status == Approved)
            throw new BusinessException();

        Status = Approved;
    }
}
```

Business behavior lives inside the entity.

DDD generally prefers rich domain models.

---

# 12. Common Interview Questions

### Should every project use DDD?

## Answer

No.

DDD introduces complexity.

Use it when business rules are complex.

Simple CRUD applications rarely benefit.

---

### Can Value Objects change?

Usually no.

Create a new instance instead.

Immutability simplifies reasoning and reduces bugs.

---

### Can Aggregates reference other Aggregates?

Prefer referencing them by ID rather than holding direct object references.

This keeps aggregates independent and reduces coupling.

---

### Are Domain Events the same as Integration Events?

No.

Domain Events occur inside the application.

Integration Events are published to other systems, often through a message broker.

---

# Staff Engineer Insight

One of the biggest mistakes teams make is introducing DDD terminology without following DDD principles.

You may see folders named:

```
Entities

ValueObjects

Aggregates
```

while every business rule still lives inside services.

That's not DDD.

DDD is about placing business behavior where it belongs.

Another common mistake is making every entity an aggregate root.

Aggregate boundaries should be chosen carefully because they directly affect transactions, concurrency and scalability.

---

# Summary

- DDD models software around business concepts.
- Entities have identity.
- Value Objects are defined by their values.
- Aggregates maintain consistency.
- Aggregate Roots protect business invariants.
- Domain Services handle logic that doesn't belong to a single entity.
- Domain Events represent business facts.
- Ubiquitous Language aligns developers and domain experts.
- Use DDD when the business domain is complex, not for every CRUD application.
