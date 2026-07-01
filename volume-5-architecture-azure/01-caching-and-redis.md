# Caching and Redis

## Overview

Caching is one of the simplest and most effective ways to improve application performance. A well-designed cache can reduce database load, improve response times and increase system scalability.

However, caching also introduces complexity such as cache invalidation, consistency and stale data.

---

# 1. What is Caching?

## Interview Answer

Caching is the process of storing frequently accessed data in a faster storage layer so future requests can retrieve it without querying the original data source.

Instead of:

```
Client

↓

API

↓

Database
```

We get:

```
Client

↓

API

↓

Cache

↓

Database (only on cache miss)
```

Benefits:

- Faster responses
- Reduced database load
- Lower latency
- Better scalability

---

# 2. Cache Hit vs Cache Miss

## Cache Hit

Requested data already exists in cache.

```
Request

↓

Cache

↓

Response
```

Database is never touched.

---

## Cache Miss

Requested data is not found.

```
Request

↓

Cache

↓

Database

↓

Cache

↓

Response
```

The application loads data from the database and stores it in cache for future requests.

---

# 3. In-Memory Cache

## Answer

ASP.NET Core provides `IMemoryCache`.

Example:

```csharp
public class ProductService
{
    private readonly IMemoryCache _cache;

    public ProductService(IMemoryCache cache)
    {
        _cache = cache;
    }

    public async Task<Product> Get(int id)
    {
        return await _cache.GetOrCreateAsync(
            $"product:{id}",
            async entry =>
            {
                entry.AbsoluteExpirationRelativeToNow =
                    TimeSpan.FromMinutes(10);

                return await repository.Get(id);
            });
    }
}
```

---

## Advantages

- Extremely fast
- Easy to use
- No external infrastructure

---

## Disadvantages

Each application instance has its own cache.

Example:

```
Server A

↓

Cache A

Server B

↓

Cache B
```

If Server A updates the cache, Server B knows nothing about it.

---

# 4. Distributed Cache

## Answer

A distributed cache is shared by all application instances.

Example:

```
App 1

↓

Redis

↑

App 2

↑

App 3
```

All servers see the same cached data.

Perfect for cloud applications.

---

# 5. What is Redis?

## Interview Answer

Redis is an in-memory key-value data store.

Although often used as a distributed cache, Redis also supports:

- Pub/Sub
- Distributed locks
- Queues
- Streams
- Counters
- Sorted Sets

Because everything is stored in memory, Redis is extremely fast.

Typical latency is below one millisecond.

---

# 6. Why Redis Instead of MemoryCache?

MemoryCache

```
Server A

↓

Own Memory
```

Redis

```
Multiple Servers

↓

Shared Cache
```

Redis scales much better in distributed environments.

---

# 7. Cache Expiration

Two common expiration strategies:

### Absolute Expiration

Item expires at a fixed time.

Example:

```
10 minutes
```

Regardless of usage.

---

### Sliding Expiration

The expiration timer resets every time the item is accessed.

Example:

```
Access

↓

Timer reset
```

Frequently used items remain in cache.

---

# 8. Cache Aside Pattern

The most common caching strategy.

```
Request

↓

Cache

↓

Hit?

↓

Yes → Return

↓

No

↓

Database

↓

Cache

↓

Return
```

Advantages:

- Easy to implement
- Works well with existing databases

Disadvantages:

- First request is always slower.

---

# 9. Write Through Cache

```
Application

↓

Cache

↓

Database
```

Data is written to cache and database simultaneously.

Benefits:

- Cache is always consistent.

Drawback:

- Slightly slower writes.

---

# 10. Write Behind Cache

```
Application

↓

Cache

↓

Immediate Response

↓

Background Write

↓

Database
```

Very fast writes.

Risk:

If Redis crashes before persisting, data may be lost.

---

# 11. Cache Invalidation

One of the hardest problems in software engineering.

When database data changes, cached data becomes stale.

Possible solutions:

- Time-based expiration
- Event-driven invalidation
- Manual cache removal

Example:

```csharp
await cache.RemoveAsync($"product:{id}");
```

---

# 12. What Should Be Cached?

Good candidates:

- Product catalogs
- Configuration
- Country lists
- Exchange rates
- Frequently accessed read-only data

Avoid caching:

- Highly volatile data
- Sensitive information
- Long-running transactions

---

# 13. Redis Data Structures

Redis supports much more than strings.

Examples:

- Strings
- Lists
- Sets
- Hashes
- Sorted Sets
- Streams
- Bitmaps

Interviewers often ask whether Redis is "just a cache."

Correct answer:

No.

Redis is an in-memory data store with multiple data structures.

---

# 14. Common Interview Questions

### Why is Redis so fast?

Because data is primarily stored in RAM and operations are highly optimized.

---

### Can Redis persist data?

Yes.

Redis supports persistence using:

- RDB snapshots
- AOF (Append Only File)

However, it is generally not used as the primary database in enterprise systems.

---

### Should every query be cached?

No.

Caching consumes memory and increases complexity.

Only cache expensive or frequently requested data.

---

### What happens if Redis goes down?

The application should gracefully fall back to the database.

Never assume the cache is always available.

---

# Staff Engineer Insight

Caching improves performance but also introduces consistency problems.

A common mistake is caching everything.

Instead, ask:

- Is the query expensive?
- Is the data frequently requested?
- How stale can the data become?
- Can users tolerate slightly outdated information?

Good caching strategies optimize for both performance and correctness.

---

# Summary

- Cache reduces database load.
- Cache Hit avoids database access.
- Cache Miss loads data from the database.
- IMemoryCache is process-local.
- Redis is distributed and shared.
- Cache Aside is the most common pattern.
- Choose appropriate expiration policies.
- Cache invalidation is one of the hardest design problems.
- Redis is much more than a simple cache.
