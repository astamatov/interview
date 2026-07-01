# System Design Case Study - Chat Application

## Problem Statement

Design a real-time chat application similar to:

- WhatsApp
- Slack
- Microsoft Teams

The system should support:

- One-to-one messaging
- Group chats
- Online presence
- Read receipts
- Typing indicators
- Offline message delivery

---

# 1. Functional Requirements

The system should allow users to:

- Send messages
- Receive messages in real time
- Create group conversations
- Receive notifications
- View message history
- See online/offline status

---

# 2. Non-Functional Requirements

Important qualities:

- Low latency
- High availability
- Horizontal scalability
- Reliable message delivery
- Ordered messages

---

# 3. High-Level Architecture

```
               Client

                  │

                  ▼

           Load Balancer

                  │

      ┌───────────┴───────────┐

      ▼                       ▼

 Chat Server            Chat Server

      │                       │

      └───────────┬───────────┘

                  ▼

           Message Broker

                  ▼

           Message Storage

                  ▼

              Redis Cache
```

---

# 4. Communication Protocol

HTTP is suitable for:

- Login
- User profile
- Chat history

Real-time messaging should use:

```
WebSockets
```

In .NET applications this is commonly implemented using:

- SignalR
- Native WebSockets

---

# Interview Question

### Why not use HTTP polling?

## Answer

Polling generates unnecessary requests.

Example:

```
Client

↓

Every 2 Seconds

↓

"Any Messages?"
```

Most requests return nothing.

WebSockets maintain a persistent connection and allow the server to push messages immediately.

---

# 5. Sending a Message

Flow:

```
Sender

↓

Chat Server

↓

Message Broker

↓

Recipient Chat Server

↓

Recipient
```

The sender should receive an acknowledgement only after the message has been accepted for processing.

---

# 6. Offline Users

If the recipient is offline:

```
Message

↓

Database

↓

Notification

↓

User Logs In

↓

Deliver Pending Messages
```

Messages should not be lost.

---

# 7. Message Ordering

Messages should appear in the correct order.

Possible approaches:

- Timestamp
- Sequence Number
- Conversation-specific ordering

For distributed systems, relying only on timestamps may be insufficient due to clock differences.

---

# 8. Read Receipts

States:

```
Sent

↓

Delivered

↓

Read
```

Each state is updated independently.

These updates can also be propagated through WebSockets.

---

# 9. Typing Indicators

Typing indicators should not be stored permanently.

Instead:

```
Typing

↓

WebSocket Event

↓

Recipient
```

Transient events should avoid unnecessary database writes.

---

# 10. Presence

Online status is usually stored in Redis.

Example:

```
User Connected

↓

Redis

↓

Online

---------------

Disconnected

↓

Offline
```

Redis provides fast access and supports expiration if heartbeat messages stop.

---

# 11. Database Design

Messages table:

```
MessageId

ConversationId

SenderId

Content

CreatedAt

Status
```

Indexes:

```
ConversationId

CreatedAt
```

Optimized for retrieving conversation history.

---

# 12. Scaling

```
Load Balancer

↓

Multiple Chat Servers

↓

Redis

↓

Message Broker

↓

Database
```

Sticky sessions or a distributed connection registry may be required so the system knows which server currently owns each WebSocket connection.

---

# 13. Message Broker

Examples:

- RabbitMQ
- Apache Kafka
- Azure Service Bus

Responsibilities:

- Decouple services
- Buffer bursts
- Reliable delivery
- Retry failed processing

---

# 14. Common Interview Questions

### Why use WebSockets?

## Answer

They provide low-latency bidirectional communication.

The server can push messages immediately without polling.

---

### Why use Redis?

Redis stores fast-changing state such as:

- Online users
- Active connections
- Temporary session data

Persistent messages still belong in the primary database.

---

### Why use a Message Broker?

It prevents chat servers from communicating directly and improves scalability and reliability.

---

### Should messages be written directly to the database?

Usually yes for persistence, but message delivery should be decoupled using a broker so slow consumers do not block senders.

---

# Trade-offs

| Decision | Benefit | Trade-off |
|----------|---------|-----------|
| WebSockets | Real-time communication | Persistent connections |
| Redis | Fast presence lookup | Volatile storage |
| Message Broker | Reliability | Additional infrastructure |
| Database Persistence | Durable messages | Higher write latency |

---

# Staff Engineer Insight

A common mistake is assuming a chat application is primarily a database problem.

It is usually a **connection management problem**.

Large chat systems must manage:

- Millions of simultaneous WebSocket connections
- User presence
- Reliable message delivery
- Ordering guarantees
- Reconnection after network interruptions

Another common mistake is trying to make every operation synchronous.

Typing indicators, read receipts and notifications can often be processed asynchronously because eventual consistency is acceptable for those features.

---

# Summary

- Use HTTP for standard APIs and WebSockets for real-time messaging.
- Persist chat messages in a database.
- Store transient state such as online presence in Redis.
- Use a Message Broker to decouple message delivery.
- Support offline delivery through persistent storage.
- Ensure message ordering within conversations.
- Scale horizontally using multiple chat servers behind a load balancer.
- Separate transient events from persistent business data.
