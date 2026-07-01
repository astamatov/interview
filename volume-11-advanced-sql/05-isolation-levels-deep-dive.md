# Isolation Levels Deep Dive

## Overview

Databases often execute thousands of transactions concurrently.

Without isolation, transactions could interfere with each other, producing inconsistent or incorrect results.

Isolation Levels define how much one transaction can see the work of another transaction.

Higher isolation increases consistency but usually reduces concurrency.

A senior engineer should understand the trade-offs and choose the appropriate level for each workload.

---

# 1. What is Transaction Isolation?

## Interview Answer

Transaction isolation determines how visible one transaction's changes are to other concurrent transactions.

It answers questions such as:

- Can another transaction read my uncommitted changes?
- Can the same query return different results?
- Can new rows appear while I am reading?

Isolation is one of the ACID properties.

---

# 2. Dirty Read

## Interview Answer

A Dirty Read occurs when one transaction reads data that another transaction has modified but not yet committed.

Example:

```
Transaction A

↓

Update Balance = 500

↓

Not Committed

---------------

Transaction B

↓

Reads Balance = 500

---------------

Transaction A

↓

Rollback
```

Transaction B observed data that never actually existed.

---

# 3. Non-Repeatable Read

Example:

```
Transaction A

↓

Read Balance = 100

---------------

Transaction B

↓

Update Balance = 200

↓

Commit

---------------

Transaction A

↓

Read Again

↓

Balance = 200
```

The same query returns different values within a single transaction.

---

# 4. Phantom Read

Example:

```
Transaction A

↓

SELECT Orders
WHERE Status = 'Pending'

↓

10 Rows

---------------

Transaction B

↓

Insert Pending Order

↓

Commit

---------------

Transaction A

↓

Run Same Query

↓

11 Rows
```

New rows appear during the transaction.

---

# 5. Read Uncommitted

## Interview Answer

Lowest isolation level.

Allows:

- Dirty Reads
- Non-Repeatable Reads
- Phantom Reads

Advantages:

- Minimal locking
- Maximum concurrency

Disadvantages:

- Lowest consistency

Generally unsuitable for business-critical data.

---

# 6. Read Committed

## Interview Answer

Default isolation level in SQL Server.

Prevents Dirty Reads.

Still allows:

- Non-Repeatable Reads
- Phantom Reads

Balances consistency and concurrency well for many applications.

---

# 7. Repeatable Read

Guarantees that rows already read cannot change during the transaction.

Prevents:

- Dirty Reads
- Non-Repeatable Reads

Still allows:

- Phantom Reads

Reads become more stable, but locking increases.

---

# 8. Serializable

## Interview Answer

Highest traditional isolation level.

Prevents:

- Dirty Reads
- Non-Repeatable Reads
- Phantom Reads

The database behaves as though transactions execute one after another.

This provides maximum consistency but also the highest locking overhead.

---

# 9. Snapshot Isolation

## Interview Answer

Snapshot Isolation uses row versioning instead of locking for reads.

Readers see a consistent snapshot of the data taken when the transaction started.

Advantages:

- Readers do not block writers.
- Writers do not block readers (for reads).

Trade-offs:

- Additional storage for row versions.
- Update conflicts must still be handled.

Snapshot Isolation is widely used in high-concurrency systems.

---

# Interview Question

### Why is Snapshot Isolation often preferred?

## Answer

It significantly reduces blocking while still providing consistent reads.

It achieves this by reading historical row versions instead of waiting for locks to be released.

---

# 10. Isolation Level Comparison

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-----------------|------------|---------------------|--------------|
| Read Uncommitted | Yes | Yes | Yes |
| Read Committed | No | Yes | Yes |
| Repeatable Read | No | No | Yes |
| Serializable | No | No | No |
| Snapshot | No | No | No (within the transaction snapshot) |

---

# 11. Choosing the Right Isolation Level

Typical guidance:

Read-heavy APIs:

```
Read Committed

or

Snapshot
```

Financial transfers:

```
Serializable

or

Snapshot + conflict handling
```

Reporting:

```
Snapshot
```

Background jobs:

Depends on consistency requirements and workload.

Always choose the weakest isolation level that still satisfies business correctness.

---

# 12. Common Interview Questions

### Should every transaction use Serializable?

## Answer

No.

Serializable provides the strongest guarantees but reduces concurrency and may increase blocking.

Use it only when the business requirements demand it.

---

### Is Read Uncommitted safe?

Usually not.

It can return data that is later rolled back, leading to incorrect business decisions.

It may be acceptable for very specific reporting scenarios where approximate results are acceptable.

---

### Can Snapshot Isolation eliminate deadlocks?

No.

It reduces many reader/writer blocking scenarios but write-write conflicts and some deadlocks are still possible.

---

### Why does higher isolation reduce performance?

Because stronger consistency often requires:

- More locking
- Longer lock duration
- Reduced concurrency

---

# Trade-offs

| Isolation Level | Consistency | Concurrency |
|-----------------|------------|-------------|
| Read Uncommitted | Very Low | Very High |
| Read Committed | Good | Good |
| Repeatable Read | High | Moderate |
| Serializable | Very High | Low |
| Snapshot | High | High (with version storage overhead) |

---

# Staff Engineer Insight

One of the biggest mistakes is assuming that stronger isolation is always better.

In reality, the goal is not maximum isolation.

The goal is **correctness with acceptable throughput**.

For example:

- A banking transfer may justify Serializable or Snapshot with conflict detection.
- A product catalog search usually does not.

Choosing the wrong isolation level can either:

- Reduce performance unnecessarily, or
- Introduce subtle data consistency bugs.

Always let business requirements drive the decision.

---

# Summary

- Isolation Levels define how transactions interact.
- Dirty Reads occur when uncommitted data is visible.
- Non-Repeatable Reads occur when previously read rows change.
- Phantom Reads occur when new matching rows appear.
- Read Committed is the default in SQL Server.
- Serializable offers the strongest guarantees but reduces concurrency.
- Snapshot Isolation uses row versioning to reduce blocking.
- Stronger isolation generally means lower concurrency.
- Select the isolation level based on business requirements rather than always choosing the strongest option.
