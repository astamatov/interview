# Resiliency Patterns: Retry, Circuit Breaker, Timeout and Fallback

## Overview

In distributed systems, failures are normal.

Services crash.

Networks become slow.

Databases restart.

External APIs become unavailable.

A resilient system assumes failures will happen and is designed to recover automatically whenever possible.

---

# 1. What is Resiliency?

## Interview Answer

Resiliency is the ability of a system to continue functioning despite failures.

The goal is not to prevent failures.

The goal is to minimize their impact.

Examples:

- Retry temporary failures.
- Stop calling unhealthy services.
- Return cached data.
- Fail gracefully.

---

# 2. Retry Pattern

## Interview Answer

A Retry attempts the same operation again after a temporary failure.

Useful for transient errors such as:

- Network timeout
- HTTP 503
- SQL timeout
- Temporary broker outage

Example:

```csharp
await retryPolicy.ExecuteAsync(async () =>
{
    await paymentClient.ProcessAsync();
});
```

---

# Why Retry?

Imagine:

```
API Call

↓

503 Service Unavailable
```

One second later the service is healthy again.

Retry succeeds.

Without retry the entire request would fail unnecessarily.

---

# 3. Should Every Error Be Retried?

## Answer

No.

Retry only transient failures.

Retry:

- Timeout
- HTTP 503
- Connection reset
- Deadlock

Do NOT retry:

- HTTP 400
- Validation errors
- Authentication failures
- Business rule violations

Retrying these only increases load.

---

# 4. Exponential Backoff

Never retry immediately.

Bad:

```
Retry

Retry

Retry

Retry
```

Good:

```
1 second

↓

2 seconds

↓

4 seconds

↓

8 seconds
```

This gives the failing service time to recover.

---

# 5. Circuit Breaker

## Interview Answer

A Circuit Breaker stops calling a service that is already failing.

Without Circuit Breaker:

```
API

↓

Payment

↓

Timeout

↓

Retry

↓

Timeout

↓

Retry

↓

Timeout
```

The failing service becomes even more overloaded.

---

With Circuit Breaker:

```
Too many failures

↓

Circuit Opens

↓

Requests fail immediately

↓

Recovery period

↓

Try again
```

This protects both systems.

---

# 6. Circuit Breaker States

## Closed

Everything works normally.

```
Requests

↓

Service
```

---

## Open

Failures exceed the configured threshold.

```
Requests

↓

Rejected Immediately
```

No calls reach the service.

---

## Half-Open

After a delay:

```
Allow one request

↓

Success?

↓

Yes

↓

Close Circuit

No

↓

Open Again
```

This tests whether the dependency has recovered.

---

# Interview Question

### Why is Circuit Breaker better than endless retries?

## Answer

Retries increase pressure on an already failing service.

Circuit Breaker gives the service time to recover instead of overwhelming it.

---

# 7. Timeout

## Interview Answer

Every remote call should have a timeout.

Bad:

```
Wait Forever
```

Good:

```
Wait 5 seconds

↓

Timeout

↓

Fallback
```

Without timeouts, blocked threads accumulate and the application becomes unresponsive.

---

# 8. Fallback

Fallback provides an alternative response when the primary operation fails.

Examples:

- Return cached data.
- Return default configuration.
- Display "Service temporarily unavailable."
- Queue the request for later processing.

Example:

```
Redis Down

↓

Read SQL

↓

Continue
```

---

# 9. Bulkhead Pattern

## Interview Answer

The Bulkhead Pattern isolates resources so one failing component cannot consume everything.

Example:

```
Thread Pool

↓

Payments

20 Threads

Inventory

20 Threads

Notifications

20 Threads
```

If Notifications become overloaded, Payments continue working.

Without isolation:

```
Notifications consume all threads

↓

Payments blocked

↓

Entire API slows down
```

---

# 10. Hedging

Hedging sends multiple requests to different replicas.

Example:

```
Replica A

↓

Slow

Replica B

↓

Fast

↓

Return first successful response
```

Useful for latency-sensitive systems.

Downside:

Higher infrastructure cost.

---

# 11. Polly

## Interview Answer

Polly is the most popular resiliency library in .NET.

It supports:

- Retry
- Circuit Breaker
- Timeout
- Fallback
- Bulkhead
- Hedging

Example:

```csharp
services.AddHttpClient<PaymentClient>()
    .AddPolicyHandler(retryPolicy)
    .AddPolicyHandler(circuitBreakerPolicy);
```

---

# 12. Common Interview Questions

### Should retries be infinite?

## Answer

No.

Infinite retries can overload downstream systems and delay failure detection.

Always define:

- Maximum retry count.
- Backoff strategy.
- Timeout.

---

### Should every API call use Circuit Breaker?

Not necessarily.

Use it for remote dependencies where failures can cascade.

---

### Can Retry and Circuit Breaker work together?

Yes.

Typical order:

```
Timeout

↓

Retry

↓

Circuit Breaker
```

Timeout detects slow calls.

Retry handles transient failures.

Circuit Breaker protects the dependency.

---

### Why are timeouts important?

Without timeouts, requests may wait indefinitely, consuming threads and reducing application throughput.

---

# Trade-offs

| Pattern | Solves | Drawback |
|----------|--------|----------|
| Retry | Temporary failures | Can increase load |
| Circuit Breaker | Cascading failures | Adds complexity |
| Timeout | Hanging requests | Requires tuning |
| Fallback | Better user experience | May return stale/default data |
| Bulkhead | Resource isolation | More resource management |
| Hedging | Lower latency | Higher infrastructure cost |

---

# Staff Engineer Insight

One of the most common mistakes is configuring retries without considering the total request duration.

Example:

```
Timeout = 30 seconds

Retries = 5
```

Worst case:

```
30 × 5 = 150 seconds
```

One user request can occupy resources for over two minutes.

Always calculate:

- Request timeout.
- Retry delay.
- Total execution time.

Another mistake is retrying non-idempotent operations.

If a payment endpoint charges a credit card, blindly retrying may charge the customer twice unless the operation is idempotent.

---

# Summary

- Failures are expected in distributed systems.
- Retry only transient failures.
- Use Exponential Backoff instead of immediate retries.
- Circuit Breaker prevents cascading failures.
- Every remote call should have a timeout.
- Fallback improves user experience during outages.
- Bulkhead isolates resources.
- Polly is the standard resiliency library in .NET.
- Design retries carefully to avoid duplicate operations and excessive latency.
