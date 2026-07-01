# Outbox Pattern

## Overview

One of the biggest challenges in distributed systems is ensuring that **database changes and published events stay consistent**.

Imagine placing an order:

1. Save the order in SQL Server.
2. Publish an `OrderCreated` event to Kafka or Azure Service Bus.

What happens if the database transaction succeeds but publishing the message fails?

The order exists.

The event does not.

Other services never learn that the order was created.

This is exactly the problem the Outbox Pattern solves.

---

# 1. The Problem

Suppose we write:

```csharp
await context.Orders.AddAsync(order);

await context.SaveChangesAsync();

await serviceBus.PublishAsync(
    new OrderCreated(order.Id));
```

Looks fine.

But imagine this sequence:

```
SaveChanges()

↓

SUCCESS

↓

Publish Message

↓

FAIL
```

Now:

- Order exists in database.
- Email is never sent.
- Inventory is never updated.
- Billing never starts.

The system becomes inconsistent.

---

# Interview Question

### Why can't we simply wrap both operations in one transaction?

## Answer

Because SQL Server and Kafka (or Azure Service Bus, RabbitMQ, etc.) do **not** participate in the same database transaction.

They are two completely different systems.

A database transaction cannot roll back a message that was never published.

---

# 2. The Solution

Instead of publishing immediately:

```
Save Order

+

Save Event

↓

Same SQL Transaction
```

Both succeed together.

Later:

```
Background Worker

↓

Read Outbox

↓

Publish Event

↓

Mark as Published
```

Now the database is the source of truth.

---

# 3. Outbox Table

Example:

```
OutboxMessages

----------------------------

Id

EventType

Payload

CreatedAt

PublishedAt

Status
```

Example row:

```
OrderCreated

{

OrderId : 15

CustomerId : 20

}

Pending
```

---

# 4. Writing Data

Transaction:

```
BEGIN TRANSACTION

↓

Insert Order

↓

Insert OutboxMessage

↓

COMMIT
```

If the transaction fails:

Nothing is saved.

If it succeeds:

Both the order and the event are stored.

Consistency guaranteed.

---

# 5. Publishing

A background worker runs every few seconds.

```
Read Pending Messages

↓

Publish

↓

Success?

↓

Yes

↓

Mark Published
```

If publishing fails:

```
Leave as Pending
```

The worker retries later.

---

# 6. Why is this Reliable?

Suppose Service Bus is down.

```
Order Saved

↓

Outbox Saved

↓

Publish Failed
```

No problem.

The event remains in the Outbox.

Once Service Bus is available again:

```
Worker

↓

Reads Pending

↓

Publishes

↓

Done
```

No data is lost.

---

# 7. Background Worker Example

ASP.NET Core:

```csharp
public class OutboxPublisher
    : BackgroundService
{
    protected override async Task ExecuteAsync(
        CancellationToken stoppingToken)
    {
        while(!stoppingToken.IsCancellationRequested)
        {
            // Read pending messages

            // Publish

            // Mark published
        }
    }
}
```

Many production systems implement the Outbox publisher as an `IHostedService` or `BackgroundService`.

---

# 8. Duplicate Messages

Suppose:

```
Publish

↓

SUCCESS

↓

Application crashes

↓

Before PublishedAt updated
```

The next worker execution publishes the same message again.

This is expected.

Consumers must therefore be **idempotent**.

---

# Interview Question

### Does Outbox guarantee Exactly Once delivery?

## Answer

No.

Outbox guarantees **reliable delivery**, not exactly-once delivery.

Messages may still be published more than once.

Consumers must detect duplicates.

This is why idempotency is so important.

---

# 9. Typical Flow

```
Client

↓

Order API

↓

SQL Transaction

↓

Orders

+

Outbox

↓

Commit

↓

Background Worker

↓

Azure Service Bus

↓

Inventory

↓

Email

↓

Billing
```

Notice:

The HTTP request finishes immediately after the transaction.

Publishing happens asynchronously.

---

# 10. Advantages

- No lost events.
- Strong consistency between database and messages.
- Automatic retries.
- Works with any message broker.
- No distributed transactions.

---

# 11. Disadvantages

- More tables.
- Background worker required.
- Messages may be delayed slightly.
- Duplicate delivery must be handled.

---

# 12. Common Interview Questions

### Can Outbox be used with RabbitMQ?

## Answer

Yes.

Outbox is independent of the message broker.

It works with:

- RabbitMQ
- Kafka
- Azure Service Bus
- AWS SQS
- Google Pub/Sub

---

### Why not publish directly?

Because failures between `SaveChanges()` and `Publish()` create inconsistent systems.

Outbox eliminates that gap.

---

### Should Outbox messages be deleted?

Usually not immediately.

Many systems:

- mark them as Published,
- archive them later,
- or delete them after a retention period.

Keeping them temporarily helps with troubleshooting and auditing.

---

# Staff Engineer Insight

One of the biggest misconceptions is believing that messaging systems solve consistency problems.

They don't.

The broker only guarantees message delivery **after the message reaches it**.

The real challenge is getting the message from your database to the broker without losing it.

That's exactly what the Outbox Pattern solves.

In many enterprise systems, Outbox is considered a standard building block whenever business data and integration events must stay synchronized.

---

# Summary

- The Outbox Pattern solves the "database updated but event not published" problem.
- Store business data and the event in the **same database transaction**.
- Publish events asynchronously from the Outbox table.
- Expect duplicate message delivery.
- Build idempotent consumers.
- Outbox provides reliability without distributed transactions.
- It works with any message broker and is widely used in enterprise systems.
