# System Design Interview - Design a Chat System

## Problem Statement

Design a chat application similar to WhatsApp, Messenger or Slack.

Requirements:

- One-to-one messaging
- Group chats
- Real-time message delivery
- Message history
- Online/offline status
- Read receipts
- High availability
- Millions of concurrent users

---

# 1. Questions to Ask

## Interview Answer

Before drawing the architecture, clarify the requirements.

Ask:

- Is the chat one-to-one or group?
- How many concurrent users?
- How large can groups become?
- Should messages be encrypted?
- Are attachments supported?
- How long should messages be stored?
- Is message ordering important?
- What happens if a user is offline?

Senior engineers always clarify requirements before proposing a solution.

---

# 2. High-Level Architecture

```
             Client

                │

                ▼

         Load Balancer

                │

      ┌─────────┴─────────┐

      ▼                   ▼

 Chat Server A      Chat Server B

      │                   │

      └─────────┬─────────┘

                ▼

         Redis Pub/Sub

                │

                ▼

        SQL / NoSQL Database
```

---

# 3. Why HTTP is Not Enough

## Answer

HTTP follows the request-response model.

```
Client

↓

Request

↓

Server

↓

Response
```

The server cannot initiate communication.

For real-time chat, the server must be able to push messages to clients instantly.

---

# 4. Why WebSockets?

## Interview Answer

WebSockets provide a persistent, full-duplex connection between the client and server.

Once established, both sides can send messages at any time without opening a new connection.

```
Client

⇄

Server
```

Benefits:

- Low latency
- Fewer HTTP requests
- Real-time communication

---

# 5. What is SignalR?

## Answer

SignalR is Microsoft's real-time communication library for ASP.NET Core.

It abstracts transport selection.

SignalR automatically chooses the best available transport:

- WebSockets (preferred)
- Server-Sent Events
- Long Polling

Developers don't need to manage transport selection manually.

---

# Interview Question

### Why use SignalR instead of raw WebSockets?

## Answer

SignalR provides:

- Automatic reconnection
- Connection management
- Groups
- User mapping
- Fallback transports
- Simpler programming model

For most business applications, SignalR significantly reduces development effort.

---

# 6. What Happens When a Message is Sent?

Example:

Alice sends:

```
Hello Bob
```

Flow:

```
Alice

↓

Chat Server

↓

Store in Database

↓

Publish to Redis

↓

Bob's Chat Server

↓

Bob receives message
```

The database guarantees durability.

Redis distributes the message to all server instances.

---

# 7. Why Redis Pub/Sub?

Suppose:

Alice is connected to:

```
Server A
```

Bob is connected to:

```
Server B
```

Without Redis:

Server A has no idea where Bob is connected.

With Redis Pub/Sub:

```
Server A

↓

Publish

↓

Redis

↓

Server B

↓

Bob
```

All chat servers stay synchronized.

---

# 8. Message Ordering

## Interview Answer

Users expect messages to appear in the correct order.

Example:

```
Hello

↓

How are you?

↓

See you later
```

Ordering can be guaranteed using:

- Database sequence IDs
- Timestamps
- Per-conversation sequence numbers

Never rely on the client clock for ordering.

---

# 9. What if the User is Offline?

If Bob is offline:

```
Message

↓

Database

↓

Pending
```

When Bob reconnects:

```
Read pending messages

↓

Deliver

↓

Mark delivered
```

Messages should never be lost simply because the recipient was disconnected.

---

# 10. Read Receipts

Typical message states:

```
Sent

↓

Delivered

↓

Read
```

When Bob opens the chat:

```
Client

↓

Read event

↓

Server

↓

Alice receives "Seen"
```

---

# 11. Online Presence

The server tracks active connections.

Example:

```
User Connected

↓

Redis

↓

Online
```

Disconnect:

```
Connection Closed

↓

Offline
```

Distributed systems often store presence information in Redis because it is shared across all chat servers.

---

# 12. Scaling

As users grow:

```
Load Balancer

↓

Server A

Server B

Server C

Server D
```

Every server handles a subset of WebSocket connections.

Redis keeps them synchronized.

---

# 13. Database Choice

Messages are mostly:

- Inserts
- Reads by conversation

Both SQL and NoSQL can work.

Example:

SQL

Advantages:

- Transactions
- Strong consistency

NoSQL

Advantages:

- Massive horizontal scalability
- Flexible schema

The correct answer depends on the business requirements.

---

# 14. Common Interview Questions

### Why not store active WebSocket connections in SQL Server?

SQL is not designed for highly dynamic connection state.

Redis provides much lower latency.

---

### What happens if a Chat Server crashes?

Clients reconnect.

Load Balancer routes them to another server.

Pending messages remain safely stored in the database.

---

### Why store messages before publishing?

If Redis crashes after publishing but before persistence, messages could be lost.

Persist first.

Publish second.

---

### Can SignalR scale horizontally?

Yes.

Using:

- Redis Backplane
- Azure SignalR Service

---

# Trade-offs

| Decision | Advantage | Disadvantage |
|----------|-----------|--------------|
| SignalR | Easy development | Slight abstraction overhead |
| Raw WebSockets | Maximum control | More complex implementation |
| Redis | Fast message distribution | Additional infrastructure |
| SQL | Reliable persistence | Can become bottleneck |
| NoSQL | Easy scaling | Weaker consistency depending on database |

---

# Staff Engineer Insight

One of the biggest mistakes candidates make is assuming that sending a message means it has been delivered.

In reality there are multiple stages:

1. Accepted by the API.
2. Persisted to durable storage.
3. Published to connected servers.
4. Delivered to the recipient.
5. Read by the recipient.

Each stage can fail independently.

Designing for retries, acknowledgements and idempotency is what separates a distributed messaging system from a simple demo application.

---

# Summary

- Use WebSockets for real-time communication.
- SignalR simplifies real-time applications in ASP.NET Core.
- Redis Pub/Sub synchronizes multiple chat servers.
- Persist messages before publishing them.
- Store presence information in Redis.
- Handle offline users by storing pending messages.
- Design for retries and failures, not only the happy path.
