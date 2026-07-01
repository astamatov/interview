# System Design Fundamentals

## Overview

System Design is not about drawing boxes and arrows.

It is about understanding **trade-offs**.

There is no perfect architecture.

Every decision improves one aspect while making another worse.

A senior engineer should always explain:

- Why a solution was chosen.
- What problem it solves.
- What trade-offs it introduces.

Interviewers care much more about your reasoning than your final architecture.

---

# 1. What is Scalability?

## Interview Answer

Scalability is the ability of a system to handle increasing load while maintaining acceptable performance.

A scalable application should continue to serve requests as the number of users, requests or stored data grows.

---

## Example

Today:

```
100 requests/sec
```

Next year:

```
10,000 requests/sec
```

A scalable system should continue working without a complete redesign.

---

# 2. Vertical Scaling

## Answer

Vertical scaling means making one machine more powerful.

Example:

```
4 CPU

↓

32 CPU
```

or

```
16 GB RAM

↓

128 GB RAM
```

Advantages:

- Simple
- No application changes

Disadvantages:

- Expensive
- Hardware limits exist
- Single point of failure

---

# 3. Horizontal Scaling

## Answer

Horizontal scaling means adding more servers.

```
Server 1

Server 2

Server 3

Server 4
```

Traffic is distributed between them.

Advantages:

- Better availability
- Better scalability
- Fault tolerance

Disadvantages:

- More complex architecture
- Requires load balancing
- Shared state becomes difficult

---

# Interview Question

### Which scaling strategy should cloud applications prefer?

Horizontal scaling.

Cloud platforms are designed to add or remove instances automatically.

---

# 4. What is a Load Balancer?

## Interview Answer

A Load Balancer distributes incoming traffic across multiple servers.

Without it:

```
Users

↓

Server
```

One server eventually becomes overloaded.

With a Load Balancer:

```
Users

↓

Load Balancer

↓

Server A

Server B

Server C
```

Requests are spread across all instances.

Benefits:

- High availability
- Better throughput
- Fault tolerance

---

# 5. Reverse Proxy vs Load Balancer

This is a common interview question.

## Reverse Proxy

Sits in front of web servers.

Responsibilities:

- SSL termination
- Compression
- Caching
- Routing
- Security

Examples:

- Nginx
- Apache
- YARP

---

## Load Balancer

Focuses on distributing traffic.

Examples:

- Azure Load Balancer
- AWS ELB
- HAProxy

Some products perform both roles.

---

# 6. Stateless vs Stateful Applications

## Stateless

Each request is independent.

```
Request

↓

Server

↓

Response
```

No session data stored on the server.

Advantages:

- Easy horizontal scaling
- Any server can handle any request

ASP.NET Core APIs are usually stateless.

---

## Stateful

The server stores client-specific information.

Example:

```
Shopping Cart

↓

Server Memory
```

Problem:

If the next request goes to another server, the session is lost.

Solutions:

- Redis
- Distributed Cache
- Database
- Sticky Sessions (not preferred)

---

# 7. Availability

## Interview Answer

Availability measures the percentage of time a system remains operational.

Examples:

```
99%

99.9%

99.99%

99.999%
```

The more "nines", the less downtime.

---

## Common Interview Question

### How do we improve availability?

- Multiple servers
- Redundant databases
- Health checks
- Automatic failover
- Load balancing

---

# 8. Reliability

Availability means the system is running.

Reliability means the system produces correct results.

Example:

An API returning incorrect bank balances is available, but not reliable.

Both are important.

---

# 9. Fault Tolerance

A fault-tolerant system continues working even when components fail.

Example:

```
Server A

↓

Crash

↓

Traffic automatically moves to Server B
```

The user notices nothing.

---

# 10. SPOF (Single Point of Failure)

Any component whose failure stops the entire system.

Examples:

```
One Database

One Load Balancer

One Redis Instance
```

Senior engineers should always identify SPOFs during design discussions.

---

# 11. Health Checks

Health checks allow orchestration platforms to determine whether an application instance is healthy.

ASP.NET Core:

```csharp
builder.Services.AddHealthChecks();

app.MapHealthChecks("/health");
```

Example response:

```
Healthy
```

If unhealthy, Kubernetes or Azure App Service can restart the instance automatically.

---

# 12. Common Interview Questions

### Why not simply buy a bigger server?

Vertical scaling eventually reaches hardware limits and creates a single point of failure.

---

### Why are stateless applications easier to scale?

Because any request can be processed by any server instance.

---

### What happens if one server crashes?

The Load Balancer removes it from the rotation and redirects traffic to healthy instances.

---

# Staff Engineer Insight

One of the biggest differences between senior and staff engineers is that staff engineers think in terms of **failure**.

Instead of asking:

> "How does this work?"

they ask:

- What happens if Redis crashes?
- What happens if SQL Server becomes unavailable?
- What happens if one region goes offline?
- How long does recovery take?
- What data can we afford to lose?

Designing for failure is one of the defining characteristics of distributed systems.

---

# Summary

- Scalability allows systems to handle increasing load.
- Vertical scaling upgrades one machine.
- Horizontal scaling adds more machines.
- Load Balancers distribute traffic.
- Reverse Proxies provide routing and security.
- Stateless applications scale more easily.
- Availability and reliability are different concepts.
- Eliminate Single Points of Failure whenever possible.
- Health checks enable automatic recovery.
