# Messaging, Kafka, RabbitMQ and Azure Service Bus

## Overview

Modern distributed systems rarely communicate by calling each other directly.

Instead of synchronous HTTP calls, services often communicate asynchronously using a message broker.

Messaging improves scalability, reliability and fault tolerance while reducing coupling between services.

---

# 1. What is Messaging?

## Interview Answer

Messaging is a communication pattern where one service sends a message to a broker instead of communicating directly with another service.

Instead of:

```
Order Service

↓

HTTP

↓

Email Service
```

We have:

```
Order Service

↓

Message Broker

↓

Email Service
```

The sender doesn't need to know whether the receiver is online.

---

# 2. Why use Messaging?

## Answer

Messaging solves several problems:

- Loose coupling
- Better scalability
- Retry support
- Failure isolation
- Asynchronous processing

If the Email Service is temporarily unavailable, messages remain in the queue until it recovers.

---

# Interview Question

### Why not simply call another API?

## Answer

Synchronous HTTP creates tight coupling.

If the called service is unavailable, your request also fails.

With messaging, the sender can continue working while the receiver processes the message later.

---

# 3. What is a Message Broker?

## Answer

A Message Broker receives messages from producers and delivers them to consumers.

Examples:

- RabbitMQ
- Kafka
- Azure Service Bus
- AWS SQS
- ActiveMQ

Think of it as a highly reliable post office.

---

# 4. Producer and Consumer

Producer

```
Creates message

↓

Broker
```

Consumer

```
Reads message

↓

Processes message
```

Multiple consumers can process messages concurrently.

---

# 5. Queue

## Interview Answer

A Queue delivers each message to **one consumer only**.

```
Producer

↓

Queue

↓

Worker A
```

If multiple workers exist:

```
Worker A

Worker B

Worker C
```

Each message is processed exactly once by one worker (assuming successful processing).

Typical use cases:

- Email sending
- Invoice generation
- Image processing
- Background jobs

---

# 6. Topic (Publish / Subscribe)

A Topic broadcasts messages.

```
Producer

↓

Topic

↓

Email Service

↓

Analytics

↓

Audit Service
```

Every subscriber receives its own copy.

Use when multiple systems react to the same event.

---

# Queue vs Topic

| Queue | Topic |
|--------|-------|
| One consumer | Multiple consumers |
| Work distribution | Event broadcasting |
| Competing consumers | Publish / Subscribe |

---

# 7. RabbitMQ

## Interview Answer

RabbitMQ is a traditional message broker implementing the AMQP protocol.

Strengths:

- Easy to use
- Reliable delivery
- Flexible routing
- Dead Letter Queues
- Delayed messages

Typical use cases:

- Background jobs
- Business workflows
- Enterprise integration

---

# 8. Kafka

## Interview Answer

Kafka is a distributed event streaming platform.

Unlike RabbitMQ, Kafka stores messages in an append-only log.

Consumers track their own reading position (offset).

This allows consumers to replay old events.

---

## Kafka Example

```
Topic

↓

Event 1

↓

Event 2

↓

Event 3

↓

Event 4
```

Consumer A

```
Offset = 2
```

Consumer B

```
Offset = 4
```

Each consumer progresses independently.

---

# Interview Question

### Why choose Kafka instead of RabbitMQ?

## Answer

Kafka excels when:

- Event replay is required.
- Very high throughput is needed.
- Multiple independent consumers process the same stream.
- Event history must be retained.

RabbitMQ is often simpler for traditional work queues.

---

# 9. Azure Service Bus

## Interview Answer

Azure Service Bus is Microsoft's fully managed enterprise messaging service.

Features include:

- Queues
- Topics
- Sessions
- Dead Letter Queue
- Scheduled messages
- Duplicate detection
- Transactions

Because it is fully managed, teams don't maintain broker infrastructure.

---

# 10. Delivery Guarantees

This is a very common interview topic.

---

## At Most Once

```
Delivered

↓

Lost if processing fails
```

Fast.

No retries.

---

## At Least Once

```
Delivered

↓

Retry if failure occurs
```

A message may be delivered more than once.

Applications must handle duplicates.

This is the most common guarantee.

---

## Exactly Once

Very difficult in distributed systems.

Usually achieved through:

- Transactions
- Idempotency
- Deduplication

Interview Tip:

Be careful saying "Exactly Once."

Most systems actually implement **At Least Once + Idempotency**.

---

# 11. Idempotency

## Interview Answer

An operation is idempotent if executing it multiple times produces the same final result.

Example:

Processing this payment event twice:

```
PaymentCompleted
```

must not charge the customer twice.

A common solution is storing a unique message ID and ignoring duplicates.

---

# 12. Dead Letter Queue (DLQ)

If a message repeatedly fails processing:

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

Developers can inspect failed messages without blocking the entire queue.

---

# 13. Retry Policies

Transient failures happen.

Examples:

- SQL timeout
- HTTP 503
- Network interruption

Typical strategy:

```
Retry after 1 second

↓

Retry after 2 seconds

↓

Retry after 4 seconds
```

This is called **Exponential Backoff**.

---

# 14. Consumer Groups (Kafka)

Kafka allows multiple consumers to share work.

```
Topic

↓

Partition 1 → Consumer A

Partition 2 → Consumer B

Partition 3 → Consumer C
```

Each partition is processed by only one consumer within the same group.

This enables horizontal scaling.

---

# 15. Ordering

Interview Question:

### Are messages always processed in order?

## Answer

Not necessarily.

RabbitMQ ordering depends on queue configuration and consumer behavior.

Kafka guarantees ordering **within a partition**, not across all partitions.

If strict ordering is required, partitioning strategy becomes very important.

---

# 16. Common Interview Questions

### Should I use HTTP or Messaging?

HTTP:

- Immediate response required.
- Request/response workflows.

Messaging:

- Background work.
- Long-running operations.
- Loose coupling.
- Better resilience.

---

### What happens if the consumer crashes?

The broker redelivers the message to another consumer (depending on acknowledgment settings).

---

### Why shouldn't messages contain business objects?

Messages should contain only the data needed by consumers.

Sending entire domain models creates tight coupling between services.

---

# Staff Engineer Insight

One of the biggest mistakes teams make is treating messaging as "fire and forget."

Reliable messaging requires thinking about:

- Retries
- Duplicate delivery
- Idempotency
- Poison messages
- Dead Letter Queues
- Monitoring
- Backpressure

A successful publish does **not** guarantee successful business processing.

Your design should assume that every network call, consumer and broker can fail.

---

# Summary

- Messaging enables asynchronous communication.
- Queues distribute work to one consumer.
- Topics broadcast events to many consumers.
- RabbitMQ is ideal for work queues.
- Kafka excels at event streaming and replay.
- Azure Service Bus provides managed enterprise messaging.
- Most distributed systems provide **At Least Once** delivery.
- Design consumers to be idempotent.
- Use Dead Letter Queues for failed messages.
- Always design for failure, retries and duplicate messages.
