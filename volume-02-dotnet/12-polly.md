# Polly & Resilience

## Overview

Modern applications rarely operate in isolation.

They communicate with:

- REST APIs
- Databases
- Message Brokers
- Caches
- Third-party services

These dependencies can fail due to:

- Network issues
- Timeouts
- Temporary outages
- Rate limiting
- High latency

Polly is the resilience library for .NET that helps applications handle these failures gracefully.

In .NET 8+, Polly integrates with the new **Microsoft.Extensions.Resilience** APIs and **Resilience Pipelines**.

---

# 1. What is Resilience?

## Interview Answer

Resilience is the ability of a system to continue operating despite temporary failures.

Examples:

- Retry transient failures
- Prevent cascading failures
- Recover automatically
- Fail fast when appropriate

The goal is to improve reliability without hiding real problems.

---

# 2. What is Polly?

## Interview Answer

Polly is a .NET resilience library that provides policies for handling transient faults.

Common resilience strategies include:

- Retry
- Circuit Breaker
- Timeout
- Fallback
- Bulkhead Isolation
- Hedging
- Rate Limiting

These strategies can be composed into a resilience pipeline.

---

# 3. Retry

Retry handles temporary failures.

Example:

```text
Request

↓

Timeout

↓

Retry

↓

Success
```

Retries should only be used for **transient** failures.

Typical examples:

- Temporary network issues
- HTTP 503
- HTTP 504

Retries should generally **not** be used for:

- Validation failures
- Authentication errors
- Business rule violations

---

# Interview Question

### Should every failed request be retried?

## Answer

No.

Only failures that are likely to succeed on a later attempt should be retried.

Retrying permanent failures increases load without improving success.

---

# 4. Exponential Backoff

Instead of:

```text
Retry

Retry

Retry
```

Use increasing delays:

```text
1 second

↓

2 seconds

↓

4 seconds

↓

8 seconds
```

This reduces pressure on recovering systems.

Adding random jitter helps avoid synchronized retry storms.

---

# 5. Circuit Breaker

## Interview Answer

A Circuit Breaker prevents repeated requests to an unhealthy dependency.

States:

```text
Closed

↓

Failures

↓

Open

↓

Wait

↓

Half Open

↓

Healthy?

↓

Closed
```

When Open:

Requests fail immediately without contacting the dependency.

---

# Why Use a Circuit Breaker?

Without one:

```text
Service Down

↓

Thousands of Retries

↓

Dependency Overloaded
```

With a Circuit Breaker:

```text
Service Down

↓

Fail Fast

↓

Recover Later
```

---

# 6. Timeout

Timeout prevents requests from hanging indefinitely.

Example:

```text
Request

↓

30 Seconds

↓

Cancel
```

Timeouts should be shorter than the client's overall request timeout.

Always combine timeouts with `CancellationToken`.

---

# 7. Fallback

Fallback provides an alternative response.

Example:

```text
Recommendation API Down

↓

Fallback

↓

Return Popular Products
```

Fallback should be meaningful rather than hiding failures.

---

# 8. Bulkhead Isolation

Bulkhead Isolation prevents one failing dependency from consuming all resources.

Example:

```text
Payments

↓

Dedicated Thread Pool

---------------

Inventory

↓

Dedicated Thread Pool
```

A failure in one area does not block unrelated work.

The name comes from ship bulkheads, which prevent flooding from spreading.

---

# 9. Hedging

Introduced in newer Polly versions.

Instead of waiting too long:

```text
Request A

↓

Slow
```

Issue a second request:

```text
Request A

↓

Request B

↓

First Successful Response Wins
```

Useful for highly available read operations.

Avoid hedging non-idempotent operations.

---

# 10. Resilience Pipeline

Modern Polly encourages composing strategies.

Example:

```text
Request

↓

Timeout

↓

Retry

↓

Circuit Breaker

↓

Fallback

↓

Response
```

This creates reusable resilience behavior.

---

# 11. HttpClient Integration

Example:

```csharp
builder.Services
    .AddHttpClient<PaymentClient>()
    .AddResilienceHandler();
```

Typical strategies:

- Retry
- Timeout
- Circuit Breaker

External API clients are the most common place to apply resilience.

---

# 12. Idempotency

Retries are only safe when the operation is idempotent.

Safe examples:

```http
GET /products
```

Potentially unsafe examples:

```http
POST /payments
```

Payment APIs typically require an **Idempotency-Key** before retries are allowed.

---

# 13. Common Interview Questions

### Why use exponential backoff?

## Answer

Immediate retries increase pressure on already struggling systems.

Backoff gives the dependency time to recover.

---

### Why not retry forever?

Unlimited retries:

- Increase latency
- Waste resources
- Amplify outages

Retries should have limits.

---

### Should Circuit Breakers be used everywhere?

No.

They are most valuable for unstable external dependencies.

Internal method calls usually do not require them.

---

### When is Hedging appropriate?

For idempotent read operations where reducing latency is more important than avoiding duplicate requests.

---

# Best Practices

- Retry only transient failures.
- Use exponential backoff with jitter.
- Combine retries with timeouts.
- Protect external dependencies using Circuit Breakers.
- Apply resilience at service boundaries.
- Keep retry counts small.
- Ensure retried operations are idempotent.

---

# Trade-offs

| Strategy | Benefit | Trade-off |
|----------|---------|-----------|
| Retry | Handles transient failures | Increased latency |
| Circuit Breaker | Prevents cascading failures | Temporary request rejection |
| Timeout | Prevents hanging requests | Timeout tuning required |
| Fallback | Better user experience | May return stale or partial data |
| Bulkhead | Fault isolation | Resource partitioning |
| Hedging | Lower latency | Duplicate requests |

---

# Staff Engineer Insight

A common misconception is:

> "More retries make systems more reliable."

The opposite is often true.

Imagine a dependency under heavy load.

Every failed request immediately retries three more times.

Instead of reducing traffic, the application quadruples it.

This can turn a small outage into a cascading failure.

Resilience is about making intelligent decisions:

- Retry only when success is likely.
- Fail fast when recovery is unlikely.
- Isolate failures.
- Respect timeouts.
- Design APIs to be idempotent.

Finally, resilience policies should not be identical for every dependency.

A payment provider, a product catalog and an internal cache each require different resilience strategies based on business impact and failure characteristics.

---

# Summary

- Polly provides resilience strategies for transient failures.
- Retry only operations likely to succeed on another attempt.
- Use exponential backoff with jitter.
- Circuit Breakers prevent cascading failures.
- Timeouts avoid hanging operations.
- Fallback provides alternative behavior.
- Bulkhead Isolation limits failure impact.
- Hedging reduces latency for suitable read operations.
- Apply resilience at service boundaries and ensure retried operations are idempotent.
