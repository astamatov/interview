# System Design Case Study - Distributed Cache

## Overview

A distributed cache is a shared caching layer used by multiple application instances.

It improves performance by reducing database load and serving frequently requested data from memory.

Common technologies:

- Redis
- Memcached
- Azure Cache for Redis

Distributed caching is especially important in horizontally scaled systems.

---

# 1. What Problem Does Distributed Cache Solve?

## Interview Answer

In a single-server application, in-memory cache may be enough.

But in a distributed system:

```text
Server A → Memory Cache A
Server B → Memory Cache B
Server C → Memory Cache C
```

each server has its own cache.

This creates inconsistency.

A distributed cache provides one shared cache:

```text
Server A
Server B  → Redis
Server C
```

All instances read and write to the same cache.

---

# 2. Cache Aside Pattern

The most common caching strategy.

Flow:

```text
Request
↓
Check Cache
↓
Cache Hit?
↓
Yes → Return Data
↓
No
↓
Read Database
↓
Store In Cache
↓
Return Data
```

Example:

```csharp
var cached = await cache.GetStringAsync(key);

if (cached is not null)
{
    return JsonSerializer.Deserialize<ProductDto>(cached);
}

var product = await repository.GetProductAsync(id);

await cache.SetStringAsync(
    key,
    JsonSerializer.Serialize(product),
    options);

return product;
```

---

# 3. Cache Hit vs Cache Miss

## Cache Hit

Data exists in cache.

```text
Request → Redis → Response
```

Fast.

Database is not touched.

---

## Cache Miss

Data does not exist in cache.

```text
Request → Redis Miss → Database → Redis → Response
```

Slower, but future requests become faster.

---

# 4. What Should Be Cached?

Good candidates:

- Product catalog
- User permissions
- Configuration
- Exchange rates
- Reference data
- Expensive read queries

Poor candidates:

- Highly volatile data
- Sensitive data
- Data requiring strong consistency
- Rarely accessed data

---

# 5. Expiration Strategies

## Absolute Expiration

Data expires after a fixed time.

Example:

```text
Expire after 10 minutes
```

Good for data that should never stay too long.

---

## Sliding Expiration

Expiration extends whenever the item is accessed.

Example:

```text
Expire if unused for 10 minutes
```

Good for frequently used data.

---

# 6. Cache Invalidation

## Interview Answer

Cache invalidation means removing or updating cached data when the source of truth changes.

Example:

```text
Update Product
↓
Remove product:123 from cache
```

If invalidation is not handled correctly, users may see stale data.

---

# 7. Cache Stampede

A cache stampede happens when many requests miss the cache at the same time.

Example:

```text
Popular item expires
↓
10,000 requests miss cache
↓
10,000 database queries
```

Solutions:

- Distributed lock
- Early refresh
- Randomized expiration
- Background refresh
- Request coalescing

---

# 8. Distributed Locking

Sometimes only one instance should rebuild a cache entry.

Example:

```text
Server A gets lock
↓
Loads from DB
↓
Updates cache
↓
Releases lock
```

Other servers wait or return stale data.

Redis can be used for distributed locks, but it must be implemented carefully.

---

# 9. Consistency Trade-off

Cache improves performance but weakens consistency.

Example:

```text
Database updated
↓
Cache still old
↓
User sees stale value
```

This is often acceptable for read-heavy data where slight staleness is tolerable.

For financial transactions, caching must be handled much more carefully.

---

# 10. Common Interview Questions

### Why not cache everything?

Because cache consumes memory and adds invalidation complexity.

Only cache data that is frequently accessed, expensive to compute, or expensive to retrieve.

---

### Is Redis the source of truth?

Usually no.

The database remains the source of truth.

Redis is a performance optimization layer.

---

### What happens if Redis goes down?

The application should degrade gracefully and fall back to the database.

Cache failure should not bring down the entire system unless Redis is intentionally used as critical infrastructure.

---

### How do you avoid stale cache?

Use:

- Short TTL
- Cache invalidation on writes
- Event-driven invalidation
- Versioned cache keys

---

# Trade-offs

| Decision | Benefit | Trade-off |
|----------|---------|-----------|
| Distributed Cache | Shared state across instances | Extra infrastructure |
| Short TTL | Less stale data | More cache misses |
| Long TTL | Better performance | Staler data |
| Cache Aside | Simple and common | First request is slower |
| Distributed Lock | Prevents stampede | More complexity |

---

# Staff Engineer Insight

The hardest part of caching is not storing data.

The hard part is deciding:

- What can be stale?
- For how long?
- Who invalidates it?
- What happens when cache is unavailable?
- What happens during traffic spikes?

A good caching strategy starts with business requirements, not Redis.

For example, stale product descriptions may be acceptable for minutes.

Stale account balances may be unacceptable even for seconds.

---

# Summary

- Distributed cache is shared across application instances.
- Redis is commonly used for distributed caching.
- Cache Aside is the most common pattern.
- Cache hits reduce database load.
- Cache invalidation is one of the hardest problems.
- Cache stampede can overload the database.
- Do not cache everything.
- The database usually remains the source of truth.
- Design caching around business consistency requirements.
