# gRPC Fundamentals

## Overview

Most applications today communicate using REST APIs.

However, communication between internal microservices often has different requirements:

- Lower latency
- Smaller payloads
- Strong contracts
- High throughput

This is where gRPC excels.

gRPC is a high-performance Remote Procedure Call (RPC) framework developed by Google and widely used in modern distributed systems.

---

# 1. What is gRPC?

## Interview Answer

gRPC is a framework that allows applications to call methods on remote services as if they were local methods.

Instead of thinking in terms of HTTP resources:

```
GET /users/15
```

you think in terms of methods:

```csharp
GetUser(15)
```

The framework handles serialization, networking and communication.

---

# 2. How does gRPC work?

gRPC uses:

- HTTP/2
- Protocol Buffers (Protobuf)

Flow:

```
Client

↓

HTTP/2

↓

gRPC Server

↓

Method Execution

↓

Response
```

Unlike REST, which usually exchanges JSON, gRPC uses compact binary messages.

---

# 3. What is Protocol Buffers?

## Interview Answer

Protocol Buffers (Protobuf) define the contract between client and server.

Example:

```proto
syntax = "proto3";

service UserService
{
    rpc GetUser(GetUserRequest)
        returns (UserResponse);
}

message GetUserRequest
{
    int32 id = 1;
}

message UserResponse
{
    int32 id = 1;

    string name = 2;
}
```

The `.proto` file is used to generate strongly typed client and server code.

---

# 4. Why is gRPC faster than REST?

## Answer

Several reasons:

- Binary serialization (Protobuf)
- HTTP/2 multiplexing
- Smaller payloads
- Less parsing overhead

Example:

REST:

```
JSON

↓

Deserialize

↓

Object
```

gRPC:

```
Binary

↓

Deserialize

↓

Object
```

Binary messages are smaller and faster to process.

---

# Interview Question

### Is gRPC always faster?

## Answer

Usually yes for service-to-service communication.

However, the difference may be negligible for simple public APIs with low traffic.

Always measure performance instead of assuming.

---

# 5. HTTP/2 Benefits

HTTP/2 introduces several improvements.

### Multiplexing

Multiple requests share the same connection.

```
Connection

↓

Request A

Request B

Request C
```

No need to open a new TCP connection for every request.

---

### Header Compression

Headers are compressed.

Less network traffic.

---

### Persistent Connections

Connections stay open.

Lower latency.

---

# 6. Streaming

One of gRPC's biggest strengths.

gRPC supports four communication models.

---

## Unary

```
Request

↓

Response
```

Similar to REST.

---

## Server Streaming

```
Request

↓

Response 1

↓

Response 2

↓

Response 3
```

Server sends multiple responses.

---

## Client Streaming

```
Request 1

↓

Request 2

↓

Request 3

↓

Single Response
```

Useful when the client uploads a sequence of items.

---

## Bidirectional Streaming

```
Client ⇄ Server
```

Both sides send messages independently over the same connection.

Ideal for:

- Live dashboards
- Multiplayer games
- Real-time telemetry

---

# 7. gRPC vs REST

| REST | gRPC |
|------|------|
| JSON | Protobuf |
| HTTP/1.1 (commonly) | HTTP/2 |
| Human-readable | Binary |
| Great for public APIs | Great for internal services |
| Easy browser support | Browser support requires gRPC-Web for most scenarios |

---

# Interview Question

### Should every API use gRPC?

## Answer

No.

Public APIs consumed by browsers and third parties often benefit from REST because it is simpler and universally supported.

gRPC shines for internal service-to-service communication.

---

# 8. Versioning

Changing a `.proto` file requires care.

Rules:

- Never reuse field numbers.
- Add new fields instead of modifying existing ones.
- Avoid removing fields that existing clients still expect.

Example:

```proto
message User
{
    int32 id = 1;

    string name = 2;

    string email = 3;
}
```

Adding field `3` is generally backward compatible.

---

# 9. Error Handling

gRPC returns standard status codes.

Examples:

- OK
- InvalidArgument
- NotFound
- PermissionDenied
- Unavailable
- Internal

This provides consistent error handling across languages.

---

# 10. Common Interview Questions

### Can REST and gRPC coexist?

## Answer

Yes.

A common architecture is:

```
Browser

↓

REST API

↓

gRPC

↓

Internal Services
```

The public API exposes REST while internal services communicate using gRPC.

---

### Why use Protobuf instead of JSON?

Because Protobuf messages are:

- Smaller
- Faster to serialize
- Faster to deserialize
- Strongly typed

---

### Does gRPC replace messaging?

No.

gRPC is synchronous communication.

Messaging systems (Kafka, RabbitMQ, Azure Service Bus) provide asynchronous communication.

Choose based on business requirements.

---

# Trade-offs

| Advantage | Disadvantage |
|-----------|--------------|
| Very fast | Harder to inspect manually than JSON |
| Strong contracts | Requires `.proto` management |
| Streaming support | Less convenient for public browser APIs |
| Excellent for microservices | Additional learning curve |

---

# Staff Engineer Insight

A common mistake is choosing gRPC simply because it is faster.

Performance is only one factor.

Ask:

- Is this a public API?
- Will browsers call it directly?
- Do multiple teams use different programming languages?
- Is streaming required?
- Do we need human-readable payloads for debugging?

Many successful systems expose REST externally while using gRPC internally.

This combines broad compatibility with high-performance service-to-service communication.

---

# Summary

- gRPC is a high-performance RPC framework.
- It uses HTTP/2 and Protocol Buffers.
- Binary serialization reduces payload size and improves performance.
- Streaming is one of gRPC's biggest advantages.
- gRPC is ideal for internal microservice communication.
- REST remains an excellent choice for public APIs.
- Choose the protocol based on requirements, not popularity.
