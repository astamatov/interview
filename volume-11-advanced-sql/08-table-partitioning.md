# Table Partitioning

## Overview

As databases grow, some tables can reach hundreds of millions or even billions of rows.

At that scale, simply adding more indexes is often not enough.

Table Partitioning divides a large table into smaller logical partitions while presenting it as a single table to applications.

Proper partitioning can improve maintenance, scalability and query performance for suitable workloads.

---

# 1. What is Table Partitioning?

## Interview Answer

Table Partitioning splits a large table into multiple partitions based on a partition key.

Applications still query one logical table.

Example:

```
Orders

↓

2023

2024

2025

2026
```

Each year is stored in a different partition.

The application still executes:

```sql
SELECT *
FROM Orders;
```

without knowing how the data is physically partitioned.

---

# 2. Why Partition Data?

Benefits:

- Smaller indexes per partition
- Faster maintenance
- Easier archival
- Improved backup strategies
- Potentially less data scanned

Partitioning is primarily a data management technique, with performance benefits in appropriate scenarios.

---

# Interview Question

### Does partitioning always make queries faster?

## Answer

No.

Partitioning helps only when queries can eliminate unnecessary partitions.

If every query scans every partition, little or no performance improvement will be achieved.

---

# 3. Partition Elimination

Example:

Partitioned by:

```
OrderDate
```

Query:

```sql
WHERE OrderDate >= '2025-01-01'
AND OrderDate < '2026-01-01'
```

The optimizer may access only the 2025 partition.

Conceptually:

```
2023 ❌

2024 ❌

2025 ✅

2026 ❌
```

This is called **partition elimination** (or partition pruning in some databases).

---

# 4. Choosing a Partition Key

Good candidates:

- OrderDate
- TransactionDate
- CreatedAt
- TenantId (for some multi-tenant systems)

Poor candidates:

- Random GUID
- FirstName
- Email

The partition key should align with common query patterns.

---

# 5. Range Partitioning

Most common strategy.

Example:

```
2023

↓

Partition 1

---------------

2024

↓

Partition 2

---------------

2025

↓

Partition 3
```

Ideal for time-series data.

---

# 6. Hash Partitioning

Rows are distributed using a hash function.

Example:

```
CustomerId

↓

Hash

↓

Partition
```

Provides a more even distribution but does not naturally support range-based queries.

---

# 7. List Partitioning

Rows are partitioned by discrete values.

Example:

```
Europe

↓

Partition A

---------------

Asia

↓

Partition B

---------------

America

↓

Partition C
```

Useful when data naturally falls into well-defined categories.

---

# 8. Sliding Window Pattern

A common enterprise strategy.

```
2022

↓

Archive

---------------

2023

↓

Historical

---------------

2024

↓

Active

---------------

2025

↓

Current
```

Old partitions can often be archived or removed with minimal impact on current data.

---

# 9. Partition Maintenance

Instead of deleting millions of rows:

```sql
DELETE ...
```

A database may allow dropping or switching an entire partition.

Benefits:

- Faster operations
- Less transaction log usage
- Reduced locking

Exact capabilities depend on the database engine.

---

# 10. Partitioned Indexes

Indexes can also be partitioned.

This allows:

- Faster index rebuilds
- Per-partition maintenance
- Better scalability for very large datasets

---

# 11. Common Interview Questions

### Should every large table be partitioned?

## Answer

No.

Partitioning introduces operational complexity.

It should be justified by data volume, maintenance needs and query patterns.

---

### Can partitioning replace indexes?

No.

Partitioning and indexing solve different problems.

Partitioning reduces the amount of data considered.

Indexes speed up locating rows within that data.

---

### Why is partition elimination important?

Because it allows the optimizer to skip entire partitions, reducing I/O and improving query performance.

---

### Is partitioning transparent to applications?

Usually yes.

Applications continue querying a single logical table while the database manages the underlying partitions.

---

# Trade-offs

| Feature | Benefit | Limitation |
|----------|---------|------------|
| Range Partitioning | Excellent for time-series data | Requires boundary management |
| Hash Partitioning | Even data distribution | Less efficient for range queries |
| List Partitioning | Simple category separation | Limited flexibility |
| Sliding Window | Easy archival | Additional operational processes |

---

# Staff Engineer Insight

One of the biggest misconceptions is believing partitioning is a performance feature.

Its greatest value is often **manageability**.

For example:

A table containing 2 billion audit records may require:

- Monthly archival
- Faster backups
- Partition-level maintenance
- Simplified retention policies

Partitioning makes these operational tasks significantly easier.

Another common mistake is partitioning on a column rarely used in filtering.

Without partition elimination, the database still examines every partition.

Choose the partition key based on real access patterns, not intuition.

---

# Summary

- Partitioning divides a large table into smaller logical partitions.
- Applications continue querying a single logical table.
- Partition elimination is the primary source of query performance improvements.
- Range partitioning is common for time-based data.
- Hash partitioning distributes data evenly.
- List partitioning separates discrete categories.
- Partitioning complements indexes rather than replacing them.
- It is often adopted for manageability as much as for performance.
- Select the partition key based on actual query workloads.
