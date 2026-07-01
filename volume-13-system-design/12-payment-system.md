# System Design Case Study - Payment System (Stripe / Adyen / PayPal)

## Problem Statement

Design a payment processing system capable of handling online payments securely and reliably.

The system should support:

- Payment authorization
- Payment capture
- Refunds
- Payment status tracking
- Idempotency
- Webhooks
- High availability

In payment systems, **correctness is more important than speed**.

---

# 1. Functional Requirements

The system should support:

- Create payment
- Authorize payment
- Capture payment
- Cancel payment
- Refund payment
- Query payment status
- Retry failed operations safely

---

# 2. Non-Functional Requirements

The system should provide:

- High reliability
- High availability
- Strong consistency for critical operations
- Auditability
- Fault tolerance

A payment must never be processed twice.

---

# 3. High-Level Architecture

```text
                 Client
                    │
                    ▼
              API Gateway
                    │
                    ▼
            Payment Service
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
 Payment Database       Message Broker
        │                       │
        ▼                       ▼
 Payment Provider       Notification Service
        │
        ▼
      Webhooks
```

---

# 4. Payment Lifecycle

A typical payment progresses through these states:

```text
Created
   ↓
Authorized
   ↓
Captured
   ↓
Completed
```

Possible alternative paths:

```text
Created
   ↓
Cancelled
```

or

```text
Completed
   ↓
Refunded
```

Every transition should be validated.

---

# 5. Authorization vs Capture

## Interview Answer

Authorization reserves funds.

Capture transfers funds.

Example:

```text
Hotel Booking

↓

Authorize Card

↓

Guest Checks Out

↓

Capture Payment
```

Many payment providers separate these operations.

---

# 6. Idempotency

## Interview Answer

Payment APIs must support idempotency.

Example:

```text
POST /payments

Idempotency-Key:

abc123
```

If the client retries the request because of a timeout:

```text
Retry

↓

Same Key

↓

Return Existing Payment
```

The payment is **not processed twice**.

---

# Interview Question

### Why is idempotency critical?

## Answer

Network failures are inevitable.

Clients will retry requests.

Without idempotency, duplicate charges can occur.

---

# 7. Payment Provider Communication

Typical flow:

```text
Application

↓

Payment Provider

↓

Authorization

↓

Response
```

Never assume the response always arrives.

The provider may complete the payment even if the client times out.

---

# 8. Webhooks

Providers notify applications asynchronously.

Example:

```text
Payment Completed

↓

Webhook

↓

Update Database
```

Applications must verify webhook signatures and treat webhooks as idempotent.

---

# 9. Outbox Pattern

Updating the database and publishing events should happen reliably.

Example:

```text
Save Payment

↓

Save Outbox Event

↓

Commit

↓

Background Worker

↓

Publish Event
```

This avoids losing events if the application crashes after committing the database transaction.

---

# 10. Refunds

Refund flow:

```text
Refund Request

↓

Provider

↓

Refund Completed

↓

Webhook

↓

Update Status
```

Refunds should also be idempotent.

---

# 11. Failure Scenarios

Example:

```text
Application

↓

Timeout

↓

Did provider charge card?
```

Never guess.

Instead:

- Query payment status.
- Wait for webhook.
- Retry safely using the same idempotency key.

---

# 12. Security

Payment systems should:

- Encrypt sensitive data
- Authenticate every request
- Validate webhook signatures
- Protect secrets
- Audit every critical action

Cardholder data should generally never be stored unless the organization is designed and certified to do so.

---

# 13. PCI DSS

## Interview Answer

PCI DSS is the security standard governing payment card data.

Most systems integrate with providers such as:

- Stripe
- Adyen
- PayPal

so that sensitive card information is entered directly into the provider's secure components.

This significantly reduces PCI scope.

---

# 14. Common Interview Questions

### Why not call the payment provider synchronously and assume success?

## Answer

Because network failures create uncertainty.

A timeout does not necessarily mean the payment failed.

Applications must verify the final payment status before retrying.

---

### Why use webhooks?

Payment processing is often asynchronous.

Webhooks notify the application when the provider reaches a final state.

---

### Why use the Outbox Pattern?

To ensure database changes and published events remain consistent.

Without it, events may be lost after successful database commits.

---

### Why is payment state important?

Payments are state machines.

Only valid transitions should be allowed.

Example:

```text
Refunded

↓

Captured ❌
```

---

# Trade-offs

| Decision | Benefit | Trade-off |
|----------|---------|-----------|
| Idempotency | Prevents duplicate charges | Requires key management |
| Webhooks | Reliable asynchronous updates | Eventual consistency |
| Outbox Pattern | Reliable event publishing | Additional infrastructure |
| Separate Authorization/Capture | Business flexibility | More complex workflow |

---

# Staff Engineer Insight

One of the biggest mistakes in payment systems is assuming every API response is definitive.

In distributed systems there are three possibilities:

1. The provider completed the payment.
2. The provider rejected the payment.
3. The application **does not know yet** because communication failed.

Design for the third case.

Another common mistake is trying to wrap everything in one distributed transaction.

Payment providers cannot participate in your database transaction.

Instead, build workflows that are:

- Idempotent
- Retryable
- Event-driven
- Observable

Correctness always takes priority over raw performance in financial systems.

---

# Summary

- Model payments as a state machine.
- Separate authorization from capture when required.
- Use idempotency keys to prevent duplicate charges.
- Never assume a timeout means failure.
- Process provider callbacks through verified webhooks.
- Use the Outbox Pattern for reliable event publishing.
- Implement refunds as independent idempotent operations.
- Follow PCI DSS principles by minimizing card data exposure.
- Design payment workflows for retries, failures and eventual consistency.
