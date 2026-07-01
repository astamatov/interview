# System Design Case Study - URL Shortener

## Problem Statement

Design a URL shortening service similar to:

```
https://tinyurl.com

https://bit.ly
```

Requirements:

- Shorten long URLs
- Redirect users quickly
- Highly available
- Scalable
- Low latency

---

# 1. Functional Requirements

The system should:

- Generate short URLs.
- Redirect short URLs.
- Prevent duplicate codes.
- Support billions of URLs.
- Track basic statistics.

---

# 2. Non-Functional Requirements

Important qualities:

- High availability
- Low latency
- Horizontal scalability
- Fault tolerance
- High read throughput

Reads greatly outnumber writes.

---

# 3. High-Level Architecture

```
            Client
               │
               ▼
         Load Balancer
               │
      ┌────────┴────────┐
      ▼                 ▼
 API Instance      API Instance
      │                 │
      └────────┬────────┘
               ▼
            Redis Cache
               │
         Cache Miss
               ▼
          SQL Database
```

---

# 4. API Design

Create Short URL

```
POST /urls
```

Request:

```json
{
    "url": "https://example.com/very/long/path"
}
```

Response:

```json
{
    "shortUrl":"https://app.ly/A7X92P"
}
```

---

Redirect

```
GET /A7X92P
```

Response:

```
HTTP 301

Location: https://example.com/very/long/path
```

---

# 5. Database Design

Table:

```
Urls
```

Columns:

```
Id

OriginalUrl

ShortCode

CreatedAt

ExpirationDate

ClickCount
```

Indexes:

```
ShortCode

UNIQUE
```

This is the primary lookup.

---

# 6. Short Code Generation

Possible strategies:

### Random

```
A7X92P
```

Advantages:

- Difficult to predict
- Simple

Problems:

- Collisions

---

### Base62 Encoding

Characters:

```
0-9

A-Z

a-z
```

62 possible characters.

Example:

```
Database Id

↓

Base62

↓

Short Code
```

Example:

```
125

↓

21

(Base62)
```

Advantages:

- No collisions
- Fast generation
- Short URLs

This is a common interview answer.

---

# 7. Caching

Redirects are read-heavy.

Instead of:

```
Database

↓

Every Request
```

Use:

```
Redis

↓

Hit

↓

Return URL
```

Cache miss:

```
Redis

↓

Miss

↓

Database

↓

Store In Cache
```

---

# 8. Scaling

Read traffic:

```
Load Balancer

↓

Multiple API Servers

↓

Redis

↓

Database
```

Reads scale horizontally.

---

# 9. Database Scaling

Initially:

```
Primary Database
```

Later:

```
Primary

↓

Read Replicas
```

If the service grows significantly:

```
Sharding
```

based on URL Id or hash.

---

# 10. Analytics

Clicks should not update the database synchronously.

Instead:

```
Redirect

↓

Publish Event

↓

Message Queue

↓

Analytics Service

↓

Database
```

This keeps redirects fast.

---

# 11. Expired URLs

Possible approaches:

- Background cleanup job.
- Scheduled deletion.
- Soft delete.

Applications should validate expiration during redirects.

---

# 12. Security

Consider:

- Rate limiting
- Spam detection
- Malicious URL scanning
- Authentication for management APIs

---

# Common Interview Questions

### Why use Redis?

## Answer

Most requests are redirects.

Caching greatly reduces database load and latency.

---

### Why Base62?

It produces short, URL-safe identifiers while supporting a large number of unique values.

---

### Why asynchronous analytics?

Updating click counts synchronously increases response time.

Publishing events keeps redirects fast.

---

### Why return HTTP 301?

A permanent redirect allows browsers and intermediaries to cache the mapping.

Some systems may choose 302 depending on business requirements.

---

# Trade-offs

| Decision | Benefit | Trade-off |
|----------|---------|-----------|
| Redis | Low latency | Cache invalidation |
| Base62 | No collisions | Sequential IDs may be predictable |
| Random Codes | Harder to guess | Collision handling |
| Async Analytics | Fast redirects | Eventual consistency |

---

# Staff Engineer Insight

One common mistake is immediately discussing databases.

Start with requirements.

A strong System Design interview usually follows this order:

1. Clarify requirements.
2. Estimate scale.
3. Design the API.
4. Model the data.
5. Design the architecture.
6. Identify bottlenecks.
7. Discuss scaling.
8. Discuss trade-offs.

Interviewers care more about your reasoning than the final architecture.

Always explain *why* you choose a component, not only *what* component you choose.

---

# Summary

- Clarify requirements before designing.
- Design APIs before infrastructure.
- Cache redirects with Redis.
- Store mappings in a relational database initially.
- Use Base62 for efficient short-code generation.
- Scale reads using multiple API instances and read replicas.
- Process analytics asynchronously.
- Discuss trade-offs throughout the interview.
