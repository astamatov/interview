# System Design Case Study - Rate Limiter

## Overview

A Rate Limiter controls how many requests a user, client, IP address or service can make within a specific time window.

It protects systems from:

- Abuse
- Brute-force attacks
- Traffic spikes
- Accidental overload
- Expensive API misuse

Rate limiting is commonly implemented at the API Gateway, reverse proxy or application level.

---

# 1. What is Rate Limiting?

## Interview Answer

Rate limiting restricts the number of requests allowed within a given period.

Example:

```
100 requests per minute per user
```

If the client exceeds the limit:

```http
HTTP 429 Too Many Requests
```

The goal is to protect the system while keeping fair access for legitimate users.

---

# 2. Fixed Window Algorithm

Example:

```
Window: 1 minute
Limit: 100 requests
```

Counter:

```
12:00:00 - 12:00:59 → 100 requests allowed
12:01:00 - 12:01:59 → counter resets
```

Advantages:

- Simple
- Fast
- Easy to implement

Disadvantage:

- Burst problem around window boundaries

Example:

```
100 requests at 12:00:59
100 requests at 12:01:00
```

That creates 200 requests in 2 seconds.

---

# 3. Sliding Window

Sliding Window smooths traffic by considering a moving time range.

Example:

```
Current time: 12:01:30

Window: last 60 seconds

Count requests between:
12:00:30 - 12:01:30
```

Advantages:

- More accurate
- Prevents boundary bursts

Disadvantages:

- More expensive to implement
- Requires storing timestamps or aggregated buckets

---

# 4. Token Bucket

## Interview Answer

Token Bucket allows controlled bursts.

Tokens are added at a fixed rate.

Each request consumes one token.

Example:

```
Bucket capacity: 100 tokens
Refill rate: 10 tokens/sec
```

If tokens exist:

```
Request allowed
```

If bucket is empty:

```
Request rejected or delayed
```

This is one of the most commonly used rate limiting algorithms.

---

# 5. Leaky Bucket

Leaky Bucket processes requests at a fixed rate.

Requests enter a queue.

They leave at a steady pace.

```
Incoming Burst

↓

Bucket

↓

Constant Output Rate
```

If the bucket overflows, requests are rejected.

Good for smoothing traffic.

---

# 6. Distributed Rate Limiting

In a single server:

```
Memory Counter
```

works.

In multiple servers:

```
Server A Counter

Server B Counter

Server C Counter
```

does not work reliably.

The user could bypass limits by hitting different servers.

Distributed systems usually use Redis.

---

# 7. Redis Implementation

Typical key:

```text
rate:user:123
```

Counter:

```text
INCR rate:user:123
EXPIRE rate:user:123 60
```

If counter exceeds limit:

```http
429 Too Many Requests
```

Redis is fast and shared across all API instances.

---

# 8. Where Should Rate Limiting Happen?

Possible layers:

- API Gateway
- Reverse Proxy
- Application Middleware
- Service Mesh

API Gateway is often preferred because it blocks abusive traffic before it reaches application services.

---

# 9. Common Interview Questions

### Why return 429?

Because `429 Too Many Requests` is the standard HTTP response for rate limit violations.

---

### Should all users have the same limit?

Not necessarily.

Examples:

- Anonymous users: lower limit
- Authenticated users: higher limit
- Premium users: higher limit
- Internal services: custom limit

---

### What should the response include?

Often:

```http
Retry-After: 30
```

This tells the client when to retry.

---

# 10. Trade-offs

| Algorithm | Advantage | Disadvantage |
|----------|-----------|--------------|
| Fixed Window | Simple | Boundary bursts |
| Sliding Window | Accurate | More expensive |
| Token Bucket | Allows bursts | More complex |
| Leaky Bucket | Smooth output | Queuing delay |

---

# Staff Engineer Insight

A common mistake is applying the same rate limit to every endpoint.

Some endpoints are cheap:

```text
GET /products
```

Some are expensive:

```text
POST /reports/generate
```

Expensive operations should often have stricter limits.

Another mistake is implementing rate limiting only inside the application.

By the time the request reaches the app, it may have already consumed load balancer, gateway and network resources.

For public APIs, rate limiting is often best applied as early as possible.

---

# Summary

- Rate limiting protects systems from overload and abuse.
- Fixed Window is simple but allows boundary bursts.
- Sliding Window is more accurate but more expensive.
- Token Bucket supports controlled bursts.
- Leaky Bucket smooths request flow.
- Distributed rate limiting usually requires Redis or gateway support.
- Return HTTP 429 when limits are exceeded.
- Different users and endpoints may require different limits.
