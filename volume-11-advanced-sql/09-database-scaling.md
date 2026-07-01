# Database Scaling

## Overview

As applications grow, the database often becomes the bottleneck.

Scaling a database is not simply a matter of buying a larger server.

A senior engineer should understand the different scaling strategies, when to use them and the trade-offs involved.

The most common approaches are:

- Vertical Scaling
- Read Replicas
- Partitioning
- Sharding
- Caching

---

# 1. Vertical Scaling (Scale Up)

## Interview Answer

Vertical scaling increases the resources of a single database server.

Examples:

- More CPU
- More RAM
- Faster SSD storage

```
Database

↓

Bigger Server
```

Advantages:

- Simple
- No application changes
- Immediate improvement

Limitations:

- Hardware limits
- Higher cost
- Single point of failure remains

---

# 2. Horizontal Scaling (Scale Out)

Instead of making one server larger:

```
Database

↓

Database A

Database B

Database C
```

Data or workload is distributed across multiple servers.

Benefits:

- Greater scalability
- Better fault tolerance

Trade-offs:

- Increased operational complexity
- More complex data access patterns

---

# Interview Question

### Which is easier?

## Answer

Vertical scaling.

Horizontal scaling provides greater long-term scalability but requires architectural changes.

---

# 3. Read Replicas

## Interview Answer

A primary database handles writes.

One or more replicas handle read traffic.

```
Application

↓

Primary

↓

Replicas
```

Typical flow:

Writes:

```
Application

↓

Primary
```

Reads:

```
Application

↓

Replica
```

This reduces load on the primary server.

---

# 4. Replication Lag

Replication is often asynchronous.

Example:

```
Write

↓

Primary

↓

2 Seconds

↓

Replica
```

A user may write data and immediately read stale data from a replica.

Applications must account for eventual consistency where appropriate.

---

# 5. Partitioning vs Sharding

Partitioning:

```
One Database

↓

Multiple Partitions
```

Still a single logical database.

---

Sharding:

```
Shard A

Shard B

Shard C
```

Data is distributed across independent databases.

Each shard owns a subset of the data.

---

# 6. Sharding Strategies

### Range Sharding

```
Customers

1-1000

↓

Shard A

1001-2000

↓

Shard B
```

Simple but can create uneven load.

---

### Hash Sharding

```
CustomerId

↓

Hash

↓

Shard
```

Provides more even distribution.

Range queries become more difficult.

---

### Geographic Sharding

```
Europe

↓

Shard EU

America

↓

Shard US

Asia

↓

Shard APAC
```

Useful for globally distributed systems.

---

# 7. Choosing a Shard Key

A good shard key:

- Distributes traffic evenly.
- Matches access patterns.
- Minimizes cross-shard queries.
- Remains stable over time.

A poor shard key can create hotspots.

---

# 8. Cross-Shard Queries

Example:

```
SELECT *

FROM ALL SHARDS
```

The application or middleware may need to:

- Query every shard.
- Merge the results.
- Sort them.

Cross-shard operations are expensive and should be minimized.

---

# 9. Database Caching

Not every request should reach the database.

Typical flow:

```
Request

↓

Redis

↓

Hit?

↓

Yes

↓

Return

---------------

No

↓

Database

↓

Store In Cache
```

Caching often provides larger performance gains than adding hardware.

---

# 10. CQRS and Scaling

Command Query Responsibility Segregation (CQRS) separates reads from writes.

```
Write Model

↓

Primary Database

---------------

Read Model

↓

Read Replicas
```

This allows each side to scale independently.

---

# 11. Common Interview Questions

### When should sharding be introduced?

## Answer

Only when simpler approaches are no longer sufficient.

Typical order:

1. Optimize queries.
2. Add indexes.
3. Introduce caching.
4. Scale vertically.
5. Add read replicas.
6. Consider sharding.

Sharding is one of the most complex scaling strategies.

---

### Are read replicas eventually consistent?

Usually yes.

Because replication is often asynchronous, recent writes may not appear immediately.

---

### Can partitioning replace sharding?

No.

Partitioning operates within a single database.

Sharding distributes data across multiple databases.

---

### What is the biggest challenge with sharding?

Maintaining consistency and minimizing cross-shard operations.

---

# Scaling Strategy Comparison

| Strategy | Complexity | Scalability | Typical Use |
|----------|------------|-------------|-------------|
| Vertical Scaling | Low | Moderate | Early growth |
| Read Replicas | Low-Medium | High for reads | Read-heavy systems |
| Partitioning | Medium | Moderate | Large tables |
| Sharding | High | Very High | Massive datasets |
| Caching | Low-Medium | Very High | Frequently read data |

---

# Trade-offs

| Technique | Benefit | Cost |
|-----------|---------|------|
| Vertical Scaling | Simple | Hardware limits |
| Read Replicas | Scales reads | Replication lag |
| Partitioning | Easier maintenance | Doesn't distribute write load |
| Sharding | Near-unlimited growth | Significant architectural complexity |
| Caching | Excellent performance | Cache invalidation complexity |

---

# Staff Engineer Insight

One of the most common mistakes is choosing sharding too early.

Many systems can support millions of users using:

- Proper indexes
- Optimized queries
- Redis caching
- Read replicas
- Vertical scaling

Sharding should usually be the final step, not the first.

Another common mistake is optimizing only the database.

Performance is often constrained by:

- Network latency
- Application design
- N+1 queries
- Excessive serialization
- Missing caching

Always identify the true bottleneck before changing the architecture.

---

# Summary

- Start with optimization before scaling.
- Vertical scaling is the simplest approach.
- Read replicas increase read throughput.
- Replication introduces eventual consistency.
- Partitioning organizes large tables.
- Sharding distributes data across databases.
- Caching often delivers the greatest performance improvement.
- CQRS can separate read and write workloads.
- Choose the simplest scaling strategy that satisfies current requirements.
