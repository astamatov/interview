# Query Optimization

## Overview

Query optimization is the process of improving SQL performance while preserving correctness.

A slow query is rarely caused by a single issue.

Instead, performance is usually influenced by:

- Index design
- Query structure
- Statistics
- Data volume
- Join strategy
- Hardware resources

A senior engineer should approach optimization methodically rather than making random changes.

---

# 1. Performance Investigation Process

## Interview Answer

When investigating a slow query:

```
Identify Slow Query

↓

Collect Execution Plan

↓

Find Expensive Operator

↓

Understand Root Cause

↓

Optimize

↓

Measure Again
```

Never optimize blindly.

---

# 2. Avoid SELECT *

Bad:

```sql
SELECT *
FROM Orders;
```

Problems:

- Reads unnecessary columns
- More network traffic
- More memory usage
- May prevent covering indexes

Preferred:

```sql
SELECT Id,
       CustomerId,
       Total
FROM Orders;
```

Retrieve only what is required.

---

# 3. Filter Early

Bad:

```sql
SELECT *
FROM Orders
JOIN Customers
```

then filter later.

Better:

```
Filter

↓

Join Smaller Dataset

↓

Return Result
```

Reducing rows early often reduces the total amount of work.

---

# 4. SARGable Queries

## Interview Answer

A query is **SARGable** (Search ARGument Able) when the optimizer can efficiently use an index.

Good:

```sql
WHERE OrderDate >= '2025-01-01'
```

Bad:

```sql
WHERE YEAR(OrderDate) = 2025
```

The function prevents efficient index usage.

Rewrite as:

```sql
WHERE OrderDate >= '2025-01-01'
AND OrderDate < '2026-01-01'
```

---

# Interview Question

### Why is YEAR(OrderDate) slower?

## Answer

Because the function must be evaluated for every row before filtering.

The optimizer usually cannot perform an efficient Index Seek on the original column.

---

# 5. Reduce Returned Rows

Example:

Bad:

```sql
SELECT *
FROM Orders;
```

Better:

```sql
SELECT TOP 100 *
FROM Orders
ORDER BY OrderDate DESC;
```

Returning fewer rows reduces:

- Network traffic
- Memory usage
- Client-side processing

---

# 6. EXISTS vs IN

For existence checks:

Preferred:

```sql
WHERE EXISTS (...)
```

instead of:

```sql
WHERE Id IN (...)
```

The optimizer may stop searching once a matching row is found with `EXISTS`.

The best choice depends on the database engine and query shape, so verify with the execution plan.

---

# 7. Avoid Repeated Queries

Instead of:

```
Application

↓

100 SQL Queries
```

Prefer:

```
One Optimized Query
```

This avoids the classic **N+1 Query Problem**.

Common solutions:

- Proper joins
- Batch loading
- Eager loading (when appropriate)

---

# 8. Reduce Sorting

Sorting requires CPU and memory.

Example:

```sql
ORDER BY Name
```

Supporting indexes can sometimes avoid explicit sort operators.

Sort only when the business requirement needs ordered results.

---

# 9. Pagination

Bad:

```sql
SELECT *
FROM Orders;
```

Better:

```sql
OFFSET 100 ROWS

FETCH NEXT 50 ROWS ONLY;
```

Large datasets should be paginated.

For very large offsets, consider **keyset pagination**, which often performs better.

---

# 10. Avoid Functions in WHERE

Bad:

```sql
WHERE LOWER(Name) = 'john'
```

Better:

Normalize the data where possible or use a case-insensitive collation if appropriate.

Avoid applying functions to indexed columns in predicates.

---

# 11. Index Strategy

Create indexes for:

- Frequently filtered columns
- Join columns
- ORDER BY columns
- GROUP BY columns

Avoid indexing:

- Very small tables
- Columns with extremely low selectivity (when not useful)
- Rarely queried columns

---

# 12. Parameter Sniffing

A cached execution plan may not suit all parameter values.

Possible solutions:

- Update statistics
- Rewrite the query
- Use `OPTION (RECOMPILE)` selectively
- Consider `OPTIMIZE FOR` when justified

Avoid using hints without understanding the trade-offs.

---

# 13. Common Interview Questions

### Why shouldn't every query use indexes?

## Answer

Indexes improve reads but increase the cost of inserts, updates and deletes.

Every additional index has a maintenance cost.

---

### Is SELECT * always bad?

Not always.

It can be acceptable for small administrative queries or debugging.

For production APIs and high-throughput systems, selecting only required columns is usually preferable.

---

### Is ORDER BY expensive?

Often yes.

Sorting large result sets consumes CPU and memory.

Indexes may eliminate the need for explicit sorting in some cases.

---

### What is the N+1 Query Problem?

One query retrieves a list of entities.

Then an additional query executes for each entity.

Example:

```
1 Query

↓

100 Rows

↓

100 Additional Queries
```

This pattern can dramatically increase latency.

---

# Performance Optimization Checklist

When reviewing a slow query, ask:

- Is there an appropriate index?
- Is the predicate SARGable?
- Are unnecessary columns selected?
- Can fewer rows be returned?
- Is there a costly sort?
- Are joins efficient?
- Is the execution plan using scans unexpectedly?
- Are statistics current?
- Is parameter sniffing involved?
- Is the application causing an N+1 problem?

---

# Trade-offs

| Optimization | Benefit | Trade-off |
|-------------|---------|-----------|
| Indexes | Faster reads | Slower writes |
| SELECT specific columns | Less I/O | More maintenance if schema changes |
| Pagination | Better scalability | More query complexity |
| EXISTS | Efficient existence checks | Depends on query shape |
| OPTION (RECOMPILE) | Better plan for current parameters | Additional compilation overhead |

---

# Staff Engineer Insight

One of the biggest mistakes is optimizing the SQL query while ignoring the application.

Example:

```
API

↓

100 Requests

↓

Each Request Executes 20 Queries
```

Even if every individual query is fast, the overall design may still perform poorly.

Performance tuning should consider:

- Database
- Application
- Network
- Caching
- Concurrency

Another common mistake is stopping after the query becomes "fast enough."

Always compare before and after using measurable metrics such as execution time, logical reads and CPU usage.

---

# Summary

- Start with the execution plan, not assumptions.
- Write SARGable predicates.
- Avoid `SELECT *` in production workloads.
- Return only the rows and columns you need.
- Design indexes for real query patterns.
- Watch for the N+1 Query Problem.
- Keep statistics up to date.
- Validate every optimization with measurements.
- Optimize the entire system, not just individual SQL statements.
