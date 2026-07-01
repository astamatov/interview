# Monolith vs Microservices

## Overview

One of the biggest misconceptions in software engineering is that microservices are always better than monoliths.

They are not.

Microservices solve specific organizational and scalability problems, but they introduce significant operational complexity.

A good architect chooses the simplest architecture that satisfies today's requirements while allowing room for future evolution.

---

# 1. What is a Monolith?

## Interview Answer

A monolithic application is deployed as a single unit.

Typical architecture:

```
                API

                 │

        ┌────────┼────────┐

        │        │        │

      Orders   Users   Payments

        │        │        │

        └────────┼────────┘

                 │

            SQL Database
```

Everything runs inside one application.

---

## Advantages

- Simple deployment
- Easy debugging
- One codebase
- One database
- Simple local development
- Simple transactions

---

## Disadvantages

- Entire application must be deployed together.
- Scaling affects the whole application.
- Large codebases become harder to maintain.
- Team ownership becomes more difficult.

---

# 2. What are Microservices?

## Interview Answer

Microservices split the application into independently deployable services.

Each service owns:

- Business capability
- Database
- Deployment
- Lifecycle

Example:

```
Users Service

↓

Users Database

----------------

Orders Service

↓

Orders Database

----------------

Payments Service

↓

Payments Database
```

Every service is responsible for its own data.

---

# 3. Why Do Companies Use Microservices?

Large organizations often have:

- Hundreds of developers
- Multiple teams
- Independent release schedules
- Different scalability requirements

Microservices allow each team to deploy independently.

---

# Interview Question

### What problem do microservices solve?

## Answer

Microservices primarily solve **organizational scalability**.

Technical scalability is important, but the biggest benefit is allowing many teams to work independently without blocking each other.

---

# 4. Database Per Service

One of the core principles.

Wrong:

```
Order Service

↓

Shared Database

↑

Payment Service
```

Correct:

```
Order Service

↓

Order Database

----------------

Payment Service

↓

Payment Database
```

No direct database sharing.

Communication happens through APIs or messaging.

---

# 5. Communication

Services typically communicate using:

Synchronous:

```
HTTP

gRPC
```

Asynchronous:

```
RabbitMQ

Kafka

Azure Service Bus
```

Choosing between them depends on business requirements.

---

# 6. Advantages of Microservices

- Independent deployment
- Independent scaling
- Fault isolation
- Technology flexibility
- Smaller codebases
- Team autonomy

Example:

Payment Service receives heavy traffic.

Only Payment Service is scaled.

The rest of the system remains unchanged.

---

# 7. Disadvantages

Microservices introduce significant complexity.

Examples:

- Network failures
- Authentication between services
- Service discovery
- Monitoring
- Logging
- Distributed tracing
- Eventual consistency
- Duplicate messages
- Retry policies
- Circuit breakers
- Outbox
- Sagas

All of these problems barely exist in a monolith.

---

# 8. Distributed Transactions

Monolith:

```
SQL Transaction

↓

Done
```

Microservices:

```
Order

↓

Payment

↓

Inventory
```

Now you need:

- Saga
- Outbox
- Compensation
- Messaging

Business processes become significantly more complex.

---

# 9. Deployment

Monolith:

```
Deploy Once
```

Microservices:

```
Deploy Users

Deploy Orders

Deploy Payments

Deploy Inventory

Deploy Shipping
```

Much more flexible.

Also much more operational work.

---

# 10. Scaling

Monolith:

```
One Application

↓

Scale Entire Application
```

Microservices:

```
Orders

↓

Scale Only Orders
```

More efficient when workloads differ significantly.

---

# 11. Team Organization

This is often overlooked.

Monolith:

```
20 developers

↓

One repository
```

Merge conflicts increase.

Release coordination becomes difficult.

Microservices:

```
Orders Team

Payments Team

Inventory Team
```

Each team owns its service.

This aligns well with Conway's Law: system architecture often mirrors organizational structure.

---

# 12. Common Interview Questions

### Should every application start as microservices?

## Answer

No.

Most successful systems start as a monolith.

Only split into microservices when there is a clear business or organizational need.

---

### When would you choose a monolith?

- Small team
- Startup
- Simple domain
- Limited budget
- Fast delivery required

---

### When would you choose microservices?

- Large engineering organization
- Independent deployments
- Different scalability requirements
- Multiple business domains
- Mature DevOps capabilities

---

### Is migrating to microservices easy?

No.

It requires changes in:

- Architecture
- Infrastructure
- Monitoring
- Deployment
- Security
- Team ownership

Migration should solve a real problem, not follow a trend.

---

# Example Evolution

Many successful companies evolved like this:

```
Simple Monolith

↓

Modular Monolith

↓

Extract First Service

↓

Several Services

↓

Microservice Ecosystem
```

This gradual approach reduces risk.

---

# Trade-offs

| Monolith | Microservices |
|----------|---------------|
| Simpler | More flexible |
| Easier debugging | Harder debugging |
| One deployment | Independent deployments |
| Strong consistency | Eventual consistency |
| Easy transactions | Saga / Outbox |
| Lower operational cost | Higher operational cost |

---

# Staff Engineer Insight

One of the strongest interview answers you can give is:

> "I wouldn't start with microservices unless I have evidence that the monolith is becoming a bottleneck."

Microservices are not a maturity badge.

They introduce:

- Network latency
- Distributed failures
- Operational complexity
- Higher infrastructure costs
- More difficult testing

A well-structured modular monolith can successfully support millions of users.

Architecture should evolve because the business requires it—not because microservices are fashionable.

---

# Summary

- Monoliths are simpler to build, deploy and maintain.
- Microservices enable independent deployment and scaling.
- Every microservice should own its data.
- Communication happens through APIs or messaging.
- Distributed systems introduce new failure modes.
- Start simple and evolve when necessary.
- Optimize for business needs, not architectural trends.
