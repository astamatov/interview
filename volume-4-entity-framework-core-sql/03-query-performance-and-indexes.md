# Query Performance and Indexes

## Overview

Writing correct SQL is not enough.

Senior engineers should understand **why one query executes in 5 milliseconds while another takes 5 seconds**.

Knowing how indexes, execution plans and Entity Framework interact is one of the biggest differences between a mid-level and a senior developer.

---

# 1. What is an Index?

## Answer

An index is a data structure that allows SQL Server to locate rows efficiently without scanning the entire table.

Think of an index like the index at the end of a book.

Without an index:

```
Search every page

↓

Find result
```

With an index:

```
Find keyword

↓

Jump directly to page
```

Indexes dramatically reduce the amount of data SQL Server must read.

---

# 2. Clustered Index

## Answer

A clustered index determines the physical order of rows on disk.

A table can have **only one** clustered index because rows can only be stored in one order.

Example:

```sql
CREATE CLUSTERED INDEX IX_Users_Id
ON Users(Id);
```

When searching by `Id`, SQL Server can quickly locate the required rows.

---

## Common Interview Question

### Why only one clustered index?

Because the table itself is stored in that order.

You cannot physically sort one table in two different ways.

---

# 3. Nonclustered Index

## Answer

A nonclustered index is a separate structure that stores indexed columns together with a pointer to the actual data row.

A table may contain many nonclustered indexes.

Example:

```sql
CREATE INDEX IX_Users_Email
ON Users(Email);
```

Searching by email becomes significantly faster.

---

# 4. Index Seek vs Index Scan

## Answer

**Index Seek**

SQL Server jumps directly to the required rows.

```
Index

↓

Desired Row
```

Very fast.

---

**Index Scan**

SQL Server scans large portions of the index.

```
Index

↓

↓

↓

↓

Find Row
```

Slower.

---

## Interview Tip

When analyzing execution plans, **Seek is generally preferred over Scan** because it reads far fewer pages.

---

# 5. Table Scan

Worst-case scenario.

No useful index exists.

SQL Server must read the entire table.

```
Row 1

Row 2

Row 3

...

Row 2,000,000
```

Large table scans are often the first thing to investigate during performance tuning.

---

# 6. Composite Index

Indexes may contain multiple columns.

Example:

```sql
CREATE INDEX IX_User_Email_Status
ON Users(Email, Status);
```

This index efficiently supports:

```sql
WHERE Email = ...
```

and

```sql
WHERE Email = ...
AND Status = ...
```

However, it is **not** efficient for:

```sql
WHERE Status = ...
```

because SQL Server uses the **leftmost prefix rule**.

---

## Interview Question

### Why column order matters?

Because SQL Server can only efficiently use the leading columns of a composite index.

Poor ordering may prevent index usage altogether.

---

# 7. Covering Index

## Answer

A covering index contains all columns required by a query.

Example:

```sql
SELECT Name
FROM Users
WHERE Email = @Email
```

Instead of returning to the clustered index to fetch `Name`, we can include it:

```sql
CREATE INDEX IX_Email
ON Users(Email)
INCLUDE(Name);
```

The query can now be satisfied entirely from the index.

Benefits:

- Fewer page reads
- Better performance
- Reduced I/O

---

# 8. Execution Plan

## Answer

An execution plan describes how SQL Server executes a query.

It shows:

- Which indexes are used
- Join strategy
- Estimated cost
- Scans vs Seeks
- Sort operations
- Hash matches
- Parallelism

Execution plans are the first place to investigate when diagnosing slow queries.

---

# 9. N+1 Problem

## Answer

The N+1 problem occurs when one query loads parent entities and additional queries are executed for each child entity.

Example:

```csharp
var orders = await context.Orders.ToListAsync();

foreach (var order in orders)
{
    Console.WriteLine(order.Customer.Name);
}
```

If lazy loading is enabled:

```
1 query for Orders

+

100 queries for Customers
```

Total:

101 queries.

---

## Solution

Use eager loading.

```csharp
var orders = await context.Orders
    .Include(o => o.Customer)
    .ToListAsync();
```

Now EF Core executes a single query.

---

# 10. Include vs SplitQuery

Large Includes may generate huge SQL joins.

Example:

```csharp
.Include(x => x.Orders)
.Include(x => x.Payments)
.Include(x => x.Addresses)
```

This may produce a Cartesian explosion.

Alternative:

```csharp
.AsSplitQuery()
```

EF Core executes multiple simpler queries instead of one huge join.

---

## Interview Question

### Should SplitQuery always be used?

No.

It reduces duplication but increases the number of round trips to the database.

Choose based on the size and shape of the data.

---

# 11. Compiled Queries

EF Core normally translates LINQ into SQL every time the query executes.

Frequently executed queries can be compiled once.

Example:

```csharp
private static readonly Func<AppDbContext, int, User?>
GetUserById =
    EF.CompileQuery(
        (AppDbContext db, int id) =>
            db.Users.FirstOrDefault(x => x.Id == id));
```

Compiled queries reduce translation overhead for hot paths.

---

# 12. Select Only What You Need

Bad:

```csharp
var users = await context.Users.ToListAsync();
```

Loads every column.

Better:

```csharp
var users = await context.Users
    .Select(x => new UserDto
    {
        Id = x.Id,
        Name = x.Name
    })
    .ToListAsync();
```

Benefits:

- Less network traffic
- Lower memory usage
- Faster materialization

---

# 13. Common Interview Questions

### Why are indexes not added to every column?

Indexes speed up reads but slow down inserts, updates and deletes because every index must also be maintained.

---

### Why is SELECT * discouraged?

Because it transfers unnecessary data, increases network traffic and prevents efficient index usage.

---

### When should you use AsNoTracking()?

For read-only queries where entities will not be modified.

---

### Why can too many indexes be harmful?

Every INSERT, UPDATE and DELETE must update all affected indexes.

More indexes = slower writes.

---

# Senior Tips

- Learn to read execution plans.
- Prefer Index Seek over Scan whenever possible.
- Use projections instead of loading entire entities.
- Watch for the N+1 problem.
- Use `AsNoTracking()` for read-heavy APIs.
- Don't create indexes blindly—measure first.
- Always ask: **"How much data am I actually reading?"**

---

# Summary

- Indexes improve lookup performance.
- Clustered indexes define physical row order.
- Nonclustered indexes are separate lookup structures.
- Covering indexes reduce extra reads.
- Composite index column order matters.
- Execution plans reveal query bottlenecks.
- Avoid the N+1 problem.
- Retrieve only the columns you need.
- Balance read performance against write performance when designing indexes.
