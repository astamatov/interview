# System Design Interview - Design a URL Shortener

## Problem Statement

Design a service similar to TinyURL or Bitly.

Requirements:

- Users submit long URLs.
- System returns short URLs.
- Visiting the short URL redirects to the original URL.
- The system should handle millions of requests.
- High availability is required.

Example:

```
Input

https://www.myverylongwebsite.com/products/12345/details

↓

Output

https://sho.rt/aB12X
```

---

# 1. Questions You Should Ask First

## Interview Answer

Before designing anything, clarify the requirements.

Ask questions like:

- How many URLs are created per day?
- How many redirects per second?
- Should links expire?
- Can users choose custom aliases?
- Should analytics be collected?
- Is high availability required?
- Is global deployment required?

Interviewers like candidates who clarify requirements instead of jumping into architecture.

---

# 2. Functional Requirements

- Create short URLs.
- Redirect to original URL.
- Optional custom aliases.
- Link expiration.
- Click statistics.

---

# 3. Non-Functional Requirements

- High availability
- Low latency
- Horizontal scalability
- Fault tolerance
- Data durability

---

# 4. High-Level Architecture

```
               Users
                  │
                  ▼
          Load Balancer
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
    API Instance       API Instance
        │                   │
        └─────────┬─────────┘
                  ▼
              Redis Cache
                  │
            Cache Miss
                  ▼
             SQL Database
```

---

# 5. Database Design

Table:

```
ShortUrls

Id

ShortCode

OriginalUrl

CreatedAt

ExpiresAt

ClickCount
```

Example:

| ShortCode | OriginalUrl |
|------------|-------------|
| aB12X | https://google.com |

Create an index on `ShortCode`.

---

## Interview Question

### Why index ShortCode?

Because every redirect searches by ShortCode.

Without an index:

```
Table Scan
```

With an index:

```
Index Seek
```

Huge performance difference.

---

# 6. Redirect Flow

```
GET /aB12X

↓

Redis

↓

Hit?

↓

Yes

↓

302 Redirect

↓

Browser loads original URL
```

If Redis misses:

```
Redis

↓

SQL

↓

Redis

↓

302 Redirect
```

---

# 7. Why Redis?

Popular URLs are requested thousands of times.

Without Redis:

```
Every request

↓

SQL
```

Database becomes the bottleneck.

With Redis:

```
Most requests

↓

Memory

↓

Response
```

Latency drops from milliseconds to microseconds.

---

# 8. Generating Short Codes

Option 1:

Auto Increment ID

```
1

↓

Base62

↓

b
```

Example:

```
125

↓

cb
```

Simple and deterministic.

---

Option 2:

Random string

```
A8Dj2
```

Need collision detection.

---

Option 3:

Hash

```
SHA256(URL)

↓

Take first characters
```

Risk of collisions if truncated.

---

## Interview Answer

For most systems, Base62 encoding of an auto-increment ID is simple, efficient and guarantees uniqueness.

---

# 9. Why Base62?

Characters:

```
a-z

A-Z

0-9
```

Total:

```
62 characters
```

Much shorter than decimal numbers.

Example:

```
12567891

↓

Ghd82
```

---

# 10. What Happens if Redis Crashes?

## Interview Answer

The application falls back to SQL Server.

Performance decreases, but functionality remains.

This is called **graceful degradation**.

A cache should improve performance, not become a single point of failure.

---

# 11. Scaling the Database

Single SQL Server eventually becomes a bottleneck.

Possible improvements:

Read Replicas

```
Writes

↓

Primary

Reads

↓

Replica 1

Replica 2
```

Later:

- Sharding
- Partitioning
- Distributed databases

---

# 12. Analytics

Every redirect increments ClickCount.

Naive approach:

```
Redirect

↓

UPDATE ClickCount
```

This creates contention.

Better:

```
Redirect

↓

Kafka / Azure Service Bus

↓

Background Worker

↓

Batch Update SQL
```

The redirect stays fast while analytics are processed asynchronously.

---

# 13. Common Interview Questions

### Why not store everything in Redis?

Redis is primarily an in-memory data store.

Memory is expensive and data persistence characteristics differ from a relational database.

Use Redis for speed, SQL for durable storage.

---

### Should redirect return 200 OK?

No.

Use:

```
301

Permanent Redirect
```

or

```
302

Temporary Redirect
```

depending on the business requirement.

---

### What becomes the bottleneck first?

Usually:

- Database
- Network
- Cache capacity

Not CPU.

---

# Trade-offs

| Decision | Advantage | Disadvantage |
|----------|-----------|--------------|
| Redis | Very fast | Extra infrastructure |
| SQL | Reliable | Slower than memory |
| Base62 | Simple | Sequential IDs may be predictable |
| Random IDs | Harder to guess | Collision handling required |
| Analytics Queue | Fast redirects | Eventual consistency |

---

# Staff Engineer Insight

The biggest mistake candidates make is trying to build a "perfect" solution immediately.

A strong system design interview starts with a simple architecture and evolves it as new requirements are introduced.

Example progression:

1. Single API + SQL database.
2. Add an index on `ShortCode`.
3. Add Redis to reduce database load.
4. Add a Load Balancer and multiple API instances.
5. Introduce read replicas.
6. Move analytics to asynchronous processing.
7. Consider sharding if the dataset becomes massive.

Interviewers want to see **how your design evolves**, not whether you immediately draw a hyperscale architecture.

---

# Summary

- Clarify requirements before designing.
- Start simple and evolve the architecture.
- Use indexes for fast lookups.
- Cache popular URLs in Redis.
- Use Base62 for compact unique identifiers.
- Scale horizontally with multiple API instances.
- Offload analytics to background processing.
- Always explain the trade-offs behind architectural decisions.
