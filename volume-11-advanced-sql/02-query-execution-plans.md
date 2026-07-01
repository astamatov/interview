# Query Execution Plans

## Overview

When a SQL query is executed, the database engine does not simply process it from top to bottom.

Instead, the Query Optimizer analyzes the query, available indexes, table statistics and estimated costs to determine the most efficient execution strategy.

The result is called the Execution Plan.

Understanding execution plans is one of the most valuable skills for SQL performance tuning.

---

# 1. What is an Execution Plan?

## Interview Answer

An Execution Plan is the sequence of operations the database engine performs to execute a query.

It shows:

- Which indexes are used
- Join algorithms
- Estimated row counts
- Sort operations
- Scans and seeks
- Relative operator costs

Instead of guessing why a query is slow, inspect its execution plan.

---

# 2. Query Optimizer

## Interview Answer

The Query Optimizer chooses what it estimates to be the lowest-cost execution strategy.

It considers:

- Available indexes
- Statistics
- Table size
- Estimated cardinality
- Join order
- Operator cost

The optimizer is cost-based rather than rule-based.

---

# Example

Query:

```sql
SELECT *
FROM Orders
WHERE CustomerId = 15;
```

Possible strategies:

```
Table Scan

↓

Read Everything
```

or

```
Index Seek

↓

Read Matching Rows
```

The optimizer chooses the lower estimated cost.

---

# 3. Estimated vs Actual Execution Plan

## Estimated Plan

Generated before execution.

Uses statistics and estimates.

Useful for understanding optimizer decisions.

---

## Actual Plan

Generated after execution.

Includes:

- Actual row counts
- Actual execution time
- Actual operator behavior

Actual plans are generally more useful when investigating performance issues.

---

# 4. Reading an Execution Plan

Typical flow:

```
Index Seek

↓

Nested Loop

↓

Key Lookup

↓

SELECT
```

Execution plans should be read from right to left because data flows toward the final result.

---

# 5. Table Scan

Example:

```
Orders

↓

Read Every Row

↓

Apply Filter
```

A table scan is not inherently bad.

If most rows are required, scanning may be the cheapest option.

However, for highly selective queries, a table scan often indicates a missing or unusable index.

---

# 6. Index Seek

Example:

```
CustomerId = 15

↓

Seek

↓

Return Matching Rows
```

This is typically the most efficient access method for selective queries.

---

# 7. Key Lookup

## Interview Answer

A Key Lookup occurs when the index does not contain all required columns.

Example:

```
Index

↓

Find Key

↓

Lookup Remaining Columns

↓

Return Row
```

Repeated lookups can become expensive.

A covering index may eliminate them.

---

# Interview Question

### Is a Key Lookup always bad?

## Answer

No.

For a small number of rows, it is often inexpensive.

If thousands of lookups occur, they may dominate query cost and indicate that a covering index should be considered.

---

# 8. Join Operators

The optimizer chooses different algorithms depending on data size and available indexes.

---

## Nested Loop Join

Best for:

- Small result sets
- Indexed lookups

---

## Hash Join

Best for:

- Large tables
- No suitable indexes

Requires building a hash table in memory.

---

## Merge Join

Best for:

- Already sorted inputs
- Indexed data

Very efficient when its prerequisites are met.

---

# 9. Sort Operator

Sorting may be required for:

```sql
ORDER BY
GROUP BY
DISTINCT
```

Sorting is CPU and memory intensive.

Supporting indexes can sometimes eliminate explicit sort operations.

---

# 10. Missing Index Suggestions

SQL Server may recommend:

```
Missing Index
```

Treat these as hints rather than automatic solutions.

Always evaluate:

- Existing indexes
- Write overhead
- Workload patterns

Blindly creating every suggested index often leads to redundancy.

---

# 11. Statistics

## Interview Answer

Statistics describe the distribution of data values.

The optimizer uses them to estimate how many rows a query will return.

Outdated statistics may cause poor execution plans.

Regular maintenance helps keep estimates accurate.

---

# 12. Parameter Sniffing

## Interview Answer

When a parameterized query executes for the first time, SQL Server may optimize the plan using the initial parameter values.

Example:

```
CustomerId = 1

↓

Plan Cached

↓

CustomerId = 900000

↓

Same Plan Reused
```

If data distribution is uneven, the cached plan may be inefficient for later executions.

---

# Common mitigations

- Update statistics.
- Rewrite the query.
- Use `OPTION (RECOMPILE)` selectively.
- Consider `OPTIMIZE FOR` hints when appropriate.

---

# 13. Common Interview Questions

### Why can the same query become slow over time?

## Answer

Possible reasons include:

- Data growth
- Outdated statistics
- Fragmented indexes
- Parameter sniffing
- Changed workload

---

### Should the highest-cost operator always be optimized first?

Usually.

However, always understand why it is expensive before changing indexes or rewriting queries.

---

### Can the optimizer make mistakes?

Yes.

The optimizer relies on estimates.

Poor statistics or skewed data can lead to suboptimal plans.

---

### Why are statistics important?

Because they help estimate row counts.

Accurate estimates usually lead to better execution plans.

---

# Performance Investigation Workflow

Recommended process:

```
Slow Query

↓

Actual Execution Plan

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

# Trade-offs

| Operator | Strength | Weakness |
|----------|----------|----------|
| Index Seek | Very fast for selective queries | Requires suitable index |
| Index Scan | Efficient for many rows | Reads more pages |
| Table Scan | Simple for full-table access | Poor for selective queries |
| Nested Loop | Excellent for small inputs | Slow with large datasets |
| Hash Join | Handles large datasets well | Memory intensive |
| Merge Join | Extremely efficient on sorted data | Requires ordered inputs |

---

# Staff Engineer Insight

A common anti-pattern is adding indexes without first examining the execution plan.

The execution plan often reveals that the real bottleneck is:

- An inefficient join order
- Excessive key lookups
- Stale statistics
- A non-sargable predicate
- Parameter sniffing

Another common mistake is focusing only on operator cost percentages.

A node showing 80% cost is not automatically the root cause.

Understand the entire execution plan and validate improvements with measurements.

---

# Summary

- The Query Optimizer selects an execution strategy based on estimated cost.
- Execution Plans explain how a query is processed.
- Actual plans provide real runtime information.
- Index Seeks are generally preferable for selective queries.
- Key Lookups are acceptable in small numbers but may require covering indexes at scale.
- Join algorithms are chosen based on data characteristics.
- Statistics strongly influence optimizer decisions.
- Parameter sniffing can produce inefficient cached plans.
- Use execution plans to guide optimization rather than relying on intuition.
