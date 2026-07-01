# Common Table Expressions (CTE) and Recursive Queries

## Overview

A Common Table Expression (CTE) is a temporary named result set that exists only for the duration of a single SQL statement.

CTEs improve readability and simplify complex queries.

Recursive CTEs extend this concept by allowing a query to reference itself, making them ideal for hierarchical data.

---

# 1. What is a CTE?

## Interview Answer

A CTE is a temporary result set defined using the `WITH` keyword.

Example:

```sql
WITH HighValueOrders AS
(
    SELECT *
    FROM Orders
    WHERE Total > 1000
)

SELECT *
FROM HighValueOrders;
```

The CTE exists only while this statement executes.

It is **not** a permanent database object.

---

# 2. Why Use CTEs?

Instead of deeply nested queries:

```sql
SELECT ...
FROM
(
    SELECT ...
    FROM
    (
        SELECT ...
    )
)
```

Use:

```sql
WITH FilteredOrders AS (...)
```

Benefits:

- Better readability
- Easier maintenance
- Logical separation of steps

---

# Interview Question

### Is a CTE stored on disk?

## Answer

No.

A CTE is a query expression.

The optimizer decides how to execute it.

It is not automatically materialized into a temporary table.

---

# 3. Multiple CTEs

Example:

```sql
WITH Customers AS
(
    ...
),
Orders AS
(
    ...
)

SELECT ...
```

CTEs can build upon previous CTEs, making complex logic easier to follow.

---

# 4. CTE vs Subquery

Subquery:

```sql
SELECT ...
FROM
(
    ...
)
```

CTE:

```sql
WITH Data AS (...)
SELECT ...
```

Both can produce the same results.

CTEs generally improve readability, especially when the same logical result is referenced multiple times within the statement.

Remember that the optimizer may still inline the CTE.

---

# 5. Recursive CTE

## Interview Answer

A recursive CTE references itself.

It consists of:

1. Anchor member
2. Recursive member

General structure:

```sql
WITH Hierarchy AS
(
    Anchor Query

    UNION ALL

    Recursive Query
)
```

Execution continues until no new rows are produced.

---

# 6. Employee Hierarchy Example

Table:

```
Employee

ManagerId
```

Example:

```
CEO

↓

Manager

↓

Developer

↓

Intern
```

Recursive CTE:

```sql
WITH EmployeeTree AS
(
    SELECT EmployeeId,
           ManagerId,
           Name
    FROM Employees
    WHERE ManagerId IS NULL

    UNION ALL

    SELECT e.EmployeeId,
           e.ManagerId,
           e.Name
    FROM Employees e
    JOIN EmployeeTree t
        ON e.ManagerId = t.EmployeeId
)

SELECT *
FROM EmployeeTree;
```

This traverses the hierarchy level by level.

---

# 7. Recursive Execution

Conceptually:

```
CEO

↓

Managers

↓

Developers

↓

Interns
```

The recursive member continues until no additional children are found.

---

# 8. Common Recursive Use Cases

Examples:

- Organization charts
- Category trees
- Folder structures
- Bill of Materials
- Parent-child relationships

---

# 9. MAXRECURSION

To prevent infinite recursion:

```sql
OPTION (MAXRECURSION 100);
```

Useful when cycles or unexpected data may exist.

---

# 10. CTE vs Temporary Table

## CTE

- Readability
- Query organization
- Single statement scope

---

## Temporary Table

- Stores intermediate results
- Can be indexed
- Can be reused across multiple statements

Use a temporary table when intermediate results are large or need to be referenced multiple times.

---

# 11. Common Interview Questions

### Can a CTE improve performance?

## Answer

Not necessarily.

A CTE primarily improves readability.

Performance depends on the execution plan, not the use of the `WITH` keyword itself.

---

### Can multiple CTEs reference each other?

Yes.

Later CTEs can reference earlier ones within the same `WITH` block.

---

### When should a temporary table be preferred?

When:

- Intermediate results are reused.
- Indexes are beneficial.
- Large datasets would otherwise be recalculated repeatedly.

---

### Can recursive CTEs loop forever?

Yes.

If the recursive condition never terminates, recursion can continue until the engine reaches its recursion limit or fails.

Use safeguards such as `MAXRECURSION` and ensure the data model cannot produce infinite cycles.

---

# Trade-offs

| Technique | Benefit | Limitation |
|-----------|---------|------------|
| CTE | Readable SQL | Usually not reusable outside the statement |
| Recursive CTE | Elegant hierarchy traversal | Can be expensive on deep hierarchies |
| Temporary Table | Reusable and indexable | Additional storage and management |

---

# Staff Engineer Insight

A common misconception is that CTEs are always faster than subqueries.

In reality, they are primarily a readability feature.

Always compare execution plans before assuming a performance improvement.

Another common mistake is using recursive CTEs for very deep hierarchies without considering performance.

For extremely large hierarchies, alternative models such as closure tables, nested sets or materialized paths may scale better, depending on the workload.

---

# Summary

- CTEs create temporary named result sets within a single statement.
- They improve readability and simplify complex queries.
- Recursive CTEs are ideal for hierarchical data.
- Every recursive CTE has an anchor member and a recursive member.
- Use `MAXRECURSION` to guard against runaway recursion.
- Temporary tables are better when intermediate data must be reused or indexed.
- Choose CTEs for clarity, not because they are automatically faster.
