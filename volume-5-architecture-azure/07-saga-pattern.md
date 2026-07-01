# Saga Pattern

## Overview

In a monolithic application, a database transaction guarantees that all operations either succeed or fail together.

In a microservices architecture, each service owns its own database.

A single SQL transaction cannot span multiple independent databases.

The Saga Pattern provides a way to coordinate long-running business processes without using distributed transactions.

---

# 1. The Problem

Imagine placing an order.

The business process involves multiple services.

```
Order Service

↓

Payment Service

↓

Inventory Service

↓

Shipping Service
```

Each service has its own database.

Question:

How do we ensure consistency if one service fails?

---

# 2. Why Not Use One Transaction?

## Interview Answer

A SQL transaction only works within a single database.

Example:

```
Order Database

Payment Database

Inventory Database
```

These databases cannot participate in one local SQL transaction.

Distributed transactions (Two-Phase Commit) exist but are rarely used in modern cloud applications because they:

- Reduce scalability.
- Increase latency.
- Create tight coupling.
- Are not supported by many cloud-native systems.

---

# 3. What is a Saga?

## Interview Answer

A Saga is a sequence of local transactions.

Each service completes its own transaction.

If one step fails, previously completed steps are compensated instead of rolled back.

Think of it as:

```
Do

↓

Do

↓

Do

↓

Fail

↓

Undo

↓

Undo
```

---

# 4. Example Flow

Customer places an order.

```
Create Order

↓

Reserve Inventory

↓

Charge Payment

↓

Create Shipment
```

Everything succeeds.

Done.

---

Now imagine payment fails.

```
Create Order

↓

Reserve Inventory

↓

Payment Failed
```

We cannot roll back SQL transactions across services.

Instead we execute compensation actions.

```
Release Inventory

↓

Cancel Order
```

---

# 5. Compensation

Compensation is **not** a database rollback.

It is another business action.

Example:

Instead of:

```
Rollback Payment
```

We perform:

```
Issue Refund
```

Instead of:

```
Rollback Inventory
```

We perform:

```
Release Reserved Items
```

Compensation should leave the business in a consistent state.

---

# 6. Choreography vs Orchestration

There are two common Saga styles.

---

## Choreography

Services communicate only through events.

```
Order Created

↓

Inventory Reserved

↓

Payment Completed

↓

Shipment Created
```

Every service listens for events and decides what to do.

Advantages:

- Loose coupling
- No central coordinator

Disadvantages:

- Difficult to follow
- Harder debugging
- Complex event chains

---

## Orchestration

A dedicated Saga Orchestrator coordinates the workflow.

```
Saga

↓

Order

↓

Inventory

↓

Payment

↓

Shipping
```

The orchestrator decides which service executes next.

Advantages:

- Easier monitoring
- Centralized logic
- Simpler debugging

Disadvantages:

- Additional component
- Orchestrator becomes critical infrastructure

---

# Interview Question

### Which approach should I choose?

## Answer

Small workflows:

Choreography is often sufficient.

Complex business processes:

Orchestration is usually easier to maintain.

---

# 7. Example

Customer buys a laptop.

```
Order Created

↓

Reserve Laptop

↓

Charge Credit Card

↓

Arrange Delivery
```

Suppose shipping fails.

The Saga performs:

```
Refund Payment

↓

Release Laptop

↓

Cancel Order
```

Notice:

Nothing was rolled back.

Business actions compensated previous work.

---

# 8. Why Idempotency Matters

Imagine:

```
Refund

↓

Network Failure

↓

Retry

↓

Refund Again
```

Without idempotency:

Customer receives two refunds.

Every Saga participant should be idempotent.

---

# 9. Timeouts

Suppose Payment Service never responds.

Should the Saga wait forever?

No.

Typical strategy:

```
Wait 30 seconds

↓

Timeout

↓

Start Compensation
```

Timeouts are an essential part of long-running workflows.

---

# 10. Failures

Every distributed system must expect failures.

Examples:

- Network interruption
- Service unavailable
- Database timeout
- Broker unavailable
- Duplicate events

A Saga is designed around these failures.

---

# 11. Common Interview Questions

### Is Saga eventually consistent?

## Answer

Yes.

Unlike database transactions, consistency is achieved over time through successful execution or compensation.

For a short period, different services may observe different states.

---

### Can Saga replace SQL transactions?

## Answer

No.

Inside a single service, continue using normal database transactions.

Saga coordinates **multiple local transactions**, not individual SQL statements.

---

### Is Saga synchronous?

Usually not.

Most Saga implementations rely on asynchronous messaging.

---

### Does every microservice need a Saga?

No.

Only workflows spanning multiple independent services.

---

# 12. Trade-offs

| Advantage | Disadvantage |
|-----------|--------------|
| No distributed transactions | More complex logic |
| Better scalability | Compensation required |
| Cloud friendly | Eventual consistency |
| Works across services | Harder debugging |

---

# Example Timeline

```
Order Created

↓

Inventory Reserved

↓

Payment Charged

↓

Shipping Failed

↓

Refund Payment

↓

Release Inventory

↓

Cancel Order
```

Final state:

No shipment.

No payment kept.

No inventory reserved.

No active order.

Business consistency restored.

---

# Staff Engineer Insight

One of the biggest mistakes candidates make is saying:

> "We'll just rollback."

In distributed systems there is no global rollback.

The payment may already have been processed.

The email may already have been sent.

The warehouse may already have packed the item.

Instead of rollback, think in terms of **compensation**.

Another common mistake is assuming every operation can be compensated.

Some actions cannot be undone.

For example:

- Sending an email
- Sending an SMS
- Calling an external banking API

In these cases, your business process must be designed to tolerate irreversible actions.

---

# Summary

- Saga coordinates long-running business processes.
- Each service performs a local transaction.
- Failures are handled using compensation.
- Compensation is a business action, not a database rollback.
- Choreography uses events.
- Orchestration uses a central coordinator.
- Sagas are eventually consistent.
- Idempotency is essential.
- Timeouts and retries are first-class design concerns.
