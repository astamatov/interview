# System Design Case Study - Notification System

## Problem Statement

Design a notification system capable of sending:

- Email
- SMS
- Push Notifications
- In-App Notifications

The system should support millions of notifications while remaining reliable and scalable.

---

# 1. Functional Requirements

The system should:

- Send notifications
- Retry failed deliveries
- Support multiple channels
- Track delivery status
- Prevent duplicate notifications
- Handle large traffic spikes

---

# 2. Non-Functional Requirements

The system should provide:

- High availability
- Reliability
- Scalability
- Low latency
- Fault tolerance

Reliability is more important than immediate delivery.

---

# 3. High-Level Architecture

```
Application

↓

Notification API

↓

Message Broker

↓

Notification Workers

↓

Provider

↓

User
```

Notification Workers may specialize by channel:

- Email Worker
- SMS Worker
- Push Worker

---

# 4. Why Use a Queue?

## Interview Answer

Sending notifications synchronously increases request latency.

Instead:

```
API

↓

Queue

↓

Immediate Response

↓

Background Processing
```

The user receives a fast response while notifications are processed asynchronously.

---

# Interview Question

### Why not send emails directly?

## Answer

Email providers may respond slowly or become temporarily unavailable.

A queue decouples the application from external providers and absorbs traffic spikes.

---

# 5. Message Broker

Possible technologies:

- RabbitMQ
- Apache Kafka
- Azure Service Bus

Responsibilities:

- Buffer traffic
- Retry failed messages
- Decouple services
- Support horizontal scaling

---

# 6. Retry Strategy

Temporary failures happen.

Example:

```
Email Provider

↓

Timeout

↓

Retry After 1 Minute

↓

Retry After 5 Minutes

↓

Retry After 15 Minutes
```

Use exponential backoff rather than retrying continuously.

---

# 7. Dead Letter Queue (DLQ)

If retries continue to fail:

```
Queue

↓

Retry

↓

Retry

↓

Retry

↓

Dead Letter Queue
```

The notification can then be inspected manually or processed separately.

---

# 8. Idempotency

## Interview Answer

Retries must not send duplicate notifications.

Example:

```
Retry

↓

Same Notification

↓

Ignored
```

Each notification should have a unique identifier.

Workers should safely handle repeated processing.

---

# 9. Multiple Channels

Example:

```
Notification Request

↓

Email

↓

SMS

↓

Push
```

Each channel should have its own processing pipeline.

This isolates failures.

---

# 10. Rate Limiting

External providers often enforce limits.

Example:

```
100 Requests/Second
```

Workers should throttle requests to avoid:

- Rejections
- Temporary bans
- Increased failure rates

---

# 11. Delivery Status

Typical states:

```
Queued

↓

Processing

↓

Sent

↓

Delivered

↓

Failed
```

Tracking status simplifies support and troubleshooting.

---

# 12. Notification Preferences

Users may choose:

- Email only
- Push only
- SMS only
- Multiple channels

Preferences should be checked before sending notifications.

---

# 13. Scaling

```
Notification API

↓

Queue

↓

Worker 1

Worker 2

Worker 3

Worker N
```

Additional workers increase throughput without changing the API.

---

# 14. Monitoring

Monitor:

- Queue length
- Retry count
- Failure rate
- Processing time
- Provider latency
- DLQ growth

Monitoring should detect issues before users report them.

---

# 15. Common Interview Questions

### Why use separate workers?

## Answer

Different notification channels have different latency, throughput and failure characteristics.

Separating them prevents one provider from blocking others.

---

### Why use a Dead Letter Queue?

Messages that repeatedly fail should not block normal processing.

The DLQ preserves failed messages for investigation.

---

### Why is idempotency important?

Retries are expected in distributed systems.

Without idempotency, users may receive duplicate notifications.

---

### Should notification sending be synchronous?

Generally no.

Asynchronous processing improves responsiveness and scalability.

---

# Trade-offs

| Decision | Benefit | Trade-off |
|----------|---------|-----------|
| Queue | Decoupling | Additional infrastructure |
| Retry | Higher reliability | Increased delivery time |
| Dead Letter Queue | Easier troubleshooting | Operational overhead |
| Multiple Workers | Better scalability | More services to manage |
| Idempotency | Prevents duplicates | Additional storage/checks |

---

# Staff Engineer Insight

One of the most common mistakes is treating every notification as equally important.

Examples:

Password Reset:

- Immediate delivery
- High priority
- Retry aggressively

Marketing Email:

- Lower priority
- Delay acceptable
- May be throttled

Different notification types often require different queues, retry policies and service-level objectives.

Another common mistake is assuming a successful API response means the notification was delivered.

In reality, it often means only that the notification was successfully accepted for processing.

Delivery and acceptance are different concepts.

---

# Summary

- Use asynchronous processing for notifications.
- Decouple producers and consumers with a Message Broker.
- Implement retries with exponential backoff.
- Move permanently failing messages to a Dead Letter Queue.
- Ensure workers are idempotent.
- Scale by adding workers.
- Respect provider rate limits.
- Track notification status throughout the delivery lifecycle.
- Monitor queues and failures continuously.
