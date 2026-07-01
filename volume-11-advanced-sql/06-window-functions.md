# SQL Window Functions

## Overview

Window Functions perform calculations across a set of related rows while preserving every row in the result.

Unlike `GROUP BY`, which collapses multiple rows into a single result, Window Functions return one result for every row.

They are among the most powerful SQL features for reporting, analytics and interview questions.

A senior engineer should understand both how they work and when to use them.

---

# 1. What is a Window Function?

## Interview Answer

A Window Function performs calculations over a logical "window" of rows.

General syntax:

```sql
Function(...)
OVER(...)
```

The `OVER` clause defines which rows belong to the window.

Unlike `GROUP BY`, the original rows remain in the result.

---

# GROUP BY vs Window Function

GROUP BY

```
Orders

↓

Grouped

↓

One Row Per Group
```

Window Function

```
Orders

↓

Calculate

↓

All Rows Remain
```

---

# 2. ROW_NUMBER()

## Interview Answer

Assigns a unique sequential number within a partition.

Example:

```sql
SELECT
    CustomerId,
    OrderDate,
    ROW_NUMBER()
        OVER(
            PARTITION BY CustomerId
            ORDER BY OrderDate DESC
        ) AS RowNum
FROM Orders;
```

Latest order:

```
RowNum = 1
```

---

# Interview Question

### Find the latest order for every customer.

## Answer

```sql
WITH RankedOrders AS
(
    SELECT *,
           ROW_NUMBER()
           OVER
           (
               PARTITION BY CustomerId
               ORDER BY OrderDate DESC
           ) AS rn
    FROM Orders
)

SELECT *

FROM RankedOrders

WHERE rn = 1;
```

This is one of the most common SQL interview questions.

---

# 3. RANK()

Example:

Scores:

```
100

90

90

80
```

Ranks:

```
1

2

2

4
```

Notice the gap after ties.

---

# 4. DENSE_RANK()

Same scores:

```
100

90

90

80
```

Ranks:

```
1

2

2

3
```

No gaps.

---

# Interview Question

### Difference between RANK() and DENSE_RANK()?

## Answer

`RANK()` leaves gaps after ties.

`DENSE_RANK()` does not.

---

# 5. LAG()

Returns a value from a previous row.

Example:

```sql
SELECT
    Amount,
    LAG(Amount)
    OVER(ORDER BY OrderDate)
FROM Orders;
```

Useful for:

- Comparing previous values
- Detecting changes
- Trend analysis

---

# 6. LEAD()

Returns the next row.

Example:

```sql
SELECT
    Amount,
    LEAD(Amount)
    OVER(ORDER BY OrderDate)
FROM Orders;
```

Useful for forecasting and comparisons.

---

# 7. Running Total

Example:

```sql
SELECT

Amount,

SUM(Amount)

OVER(

ORDER BY OrderDate

)

AS RunningTotal

FROM Orders;
```

Produces:

```
100

300

450

700
```

instead of a single aggregate.

---

# 8. Moving Average

Example:

```sql
AVG(Amount)

OVER(

ORDER BY OrderDate

ROWS BETWEEN 2 PRECEDING
AND CURRENT ROW
)
```

Useful for:

- Sales trends
- Stock prices
- Monitoring metrics

---

# 9. FIRST_VALUE()

Returns the first value within the window.

Example:

```sql
FIRST_VALUE(Price)

OVER(

PARTITION BY ProductId

ORDER BY PriceDate
)
```

---

# 10. LAST_VALUE()

Returns the last value in the window.

Be careful.

The default window frame may produce unexpected results.

Specify the frame explicitly when needed.

---

# 11. PARTITION BY

Without:

```
All Rows

↓

One Window
```

With:

```sql
PARTITION BY CustomerId
```

```
Customer 1

↓

Own Window

---------------

Customer 2

↓

Own Window
```

Calculations restart for every partition.

---

# 12. Common Interview Questions

### Do Window Functions reduce rows?

## Answer

No.

Unlike GROUP BY, they preserve every row.

---

### Can Window Functions replace GROUP BY?

Sometimes.

If every original row must remain visible, Window Functions are often a better choice.

---

### Why are Window Functions faster than self-joins?

Many analytical queries can be expressed without multiple joins or correlated subqueries.

The optimizer can often execute them more efficiently.

Always verify with the execution plan.

---

### What is the most common use of ROW_NUMBER()?

Selecting the latest record per group.

---

# Trade-offs

| Function | Common Use | Notes |
|----------|------------|-------|
| ROW_NUMBER() | Latest record | Always unique |
| RANK() | Competition ranking | Gaps after ties |
| DENSE_RANK() | Ranking | No gaps |
| LAG() | Previous row | Trend analysis |
| LEAD() | Next row | Forecasting |
| SUM() OVER | Running totals | Preserves rows |
| AVG() OVER | Moving averages | Analytical reports |

---

# Staff Engineer Insight

Many developers solve analytical SQL problems using:

- Nested queries
- Self joins
- Temporary tables

Often, Window Functions produce:

- Simpler SQL
- Better readability
- Better optimizer choices

Another common mistake is using `ROW_NUMBER()` without deterministic ordering.

If two rows have the same `ORDER BY` value, the returned row may vary between executions.

Always include a stable ordering, such as:

```sql
ORDER BY OrderDate DESC,
         OrderId DESC
```

This guarantees deterministic results.

---

# Summary

- Window Functions perform calculations without collapsing rows.
- `ROW_NUMBER()` is commonly used to retrieve the latest record per group.
- `RANK()` and `DENSE_RANK()` differ in how they handle ties.
- `LAG()` and `LEAD()` compare neighboring rows.
- Running totals and moving averages are natural use cases.
- `PARTITION BY` creates independent calculation windows.
- Window Functions often replace more complex joins and subqueries.
- Always use deterministic ordering when ranking rows.
