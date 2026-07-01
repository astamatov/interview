# Indexes Deep Dive

## Overview

Indexes are one of the most important database optimization techniques.

A well-designed index can reduce query execution time from minutes to milliseconds.

A poorly designed index can slow down writes, consume unnecessary storage and even reduce overall database performance.

A senior engineer should understand not only what indexes are, but also how they work internally and when they should be used.

---

# 1. What is an Index?

## Interview Answer

An index is a data structure that allows the database engine to locate rows efficiently without scanning the entire table.

Instead of:

```
Table

↓

Read Every Row

↓

Find Match
```

The database uses:

```
Index

↓

Locate Key

↓

Retrieve Row
```

Most relational databases implement indexes using B-Tree structures.

---

# 2. Why Are Indexes Fast?

Imagine a phone book.

Without an index:

```
Read Every Page
```

With an index:

```
Open Near Letter M

↓

Find Name

↓

Done
```

The same principle applies to databases.

---

# 3. Clustered Index

## Interview Answer

A clustered index determines the physical order of rows in the table.

```
Table

↓

Sorted By Primary Key
```

Only one clustered index can exist because the data can only be physically ordered one way.

---

# Example

```sql
CREATE CLUSTERED INDEX
IX_Orders_Id
ON Orders(Id);
```

---

# 4. Nonclustered Index

A nonclustered index stores:

```
Indexed Column

↓

Pointer

↓

Actual Row
```

Multiple nonclustered indexes may exist on the same table.

Example:

```sql
CREATE INDEX
IX_Orders_CustomerId
ON Orders(CustomerId);
```

---

# Interview Question

### Why can a table have only one clustered index?

## Answer

Because a table's rows can only have one physical order.

Nonclustered indexes are separate structures that reference those rows.

---

# 5. Index Seek vs Index Scan

## Index Seek

```
Index

↓

Jump Directly

↓

Result
```

Very efficient.

---

## Index Scan

```
Index

↓

Read Large Portion

↓

Filter
```

Better than a table scan, but still more expensive than a seek.

---

## Table Scan

```
Entire Table

↓

Read Everything
```

Usually the most expensive option for selective queries.

---

# 6. Composite Indexes

Example:

```sql
(CustomerId, OrderDate)
```

Useful query:

```sql
WHERE CustomerId = 5
```

Also useful:

```sql
WHERE CustomerId = 5
AND OrderDate > '2024-01-01'
```

Not ideal:

```sql
WHERE OrderDate > ...
```

The order of columns matters.

This is known as the **leftmost prefix rule**.

---

# 7. Covering Index

A covering index contains all columns required by a query.

Example:

```sql
CREATE INDEX IX_Orders
ON Orders(CustomerId)
INCLUDE (Total, Status);
```

Now the database can satisfy:

```sql
SELECT Total, Status
FROM Orders
WHERE CustomerId = 1;
```

without reading the base table.

This reduces I/O significantly.

---

# 8. Too Many Indexes

Indexes speed up reads but slow down writes.

Every:

```sql
INSERT

UPDATE

DELETE
```

must also update affected indexes.

More indexes mean:

- More storage
- More maintenance
- Slower write operations

Indexes should support actual query patterns.

---

# 9. Fragmentation

Over time:

- Inserts
- Updates
- Deletes

can fragment index pages.

Consequences:

- More page reads
- Poor cache utilization
- Slower queries

Databases provide maintenance operations such as rebuild and reorganize.

---

# 10. Reading Execution Plans

Execution plans reveal:

- Index Seek
- Index Scan
- Table Scan
- Key Lookup
- Join types
- Estimated cost

Rather than guessing why a query is slow, inspect its execution plan.

---

# 11. Common Interview Questions

### Should every column have an index?

## Answer

No.

Indexes consume storage and increase write costs.

Only index columns that are frequently filtered, joined or sorted.

---

### Why are indexes bad for INSERT performance?

Because every affected index must also be updated.

Additional index maintenance increases write latency.

---

### Is Index Scan always bad?

No.

If most rows are needed, scanning can be cheaper than performing many seeks.

The optimizer chooses the strategy with the lowest estimated cost.

---

### Why does column order matter in composite indexes?

Because B-Tree indexes are organized by the leading column(s).

Queries that do not filter on the leading column often cannot use the index efficiently.

---

# Trade-offs

| Feature | Benefit | Cost |
|----------|---------|------|
| Clustered Index | Fast ordered access | Only one per table |
| Nonclustered Index | Flexible lookups | Additional storage |
| Composite Index | Optimizes multi-column filters | Column order is critical |
| Covering Index | Eliminates key lookups | Larger index size |

---

# Staff Engineer Insight

A common mistake is adding indexes whenever a query becomes slow.

The better approach is:

1. Inspect the execution plan.
2. Identify whether the bottleneck is I/O, CPU or joins.
3. Design an index that supports the actual query pattern.
4. Measure the improvement.

Another frequent mistake is creating several overlapping indexes.

For example:

```
(CustomerId)

(CustomerId, OrderDate)

(CustomerId, OrderDate, Status)
```

These may duplicate much of the same data.

Review indexes periodically and remove redundant ones to reduce maintenance overhead.

---

# Summary

- Indexes reduce the amount of data the database must read.
- Clustered indexes define the physical order of table rows.
- Nonclustered indexes store keys with references to rows.
- Prefer Index Seek over Index Scan when possible.
- Composite indexes depend on column order.
- Covering indexes can eliminate additional table lookups.
- Too many indexes hurt write performance.
- Always analyze execution plans before adding new indexes.
- Design indexes based on real query workloads, not assumptions.
