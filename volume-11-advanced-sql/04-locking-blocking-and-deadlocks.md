# Locking, Blocking and Deadlocks

## Overview

Relational databases must support multiple users accessing and modifying data simultaneously.

Without synchronization, concurrent transactions could corrupt data and violate consistency.

To prevent this, databases use locks.

While locking protects data integrity, it can also introduce blocking and deadlocks.

A senior engineer should understand how these mechanisms work and how to minimize their impact.

---

# 1. Why Do Databases Use Locks?

## Interview Answer

Locks ensure that concurrent transactions do not interfere with each other.

Example:

```
Transaction A

↓

Updates Order

↓

Transaction B

↓

Attempts Same Update
```

Without locking:

- Lost updates
- Dirty reads
- Corrupted data

Locks preserve consistency.

---

# 2. Shared Lock (S)

## Interview Answer

A Shared Lock allows multiple transactions to read the same data simultaneously.

Example:

```
Transaction A

↓

Read Order

↓

Shared Lock

---------------

Transaction B

↓

Read Same Order

↓

Shared Lock
```

Multiple readers can coexist.

---

# 3. Exclusive Lock (X)

An Exclusive Lock is required to modify data.

Example:

```
Transaction A

↓

Update Order

↓

Exclusive Lock
```

While the exclusive lock is held:

- Other writes wait.
- Most reads also wait (depending on the isolation level).

Only one transaction can hold an exclusive lock on the same resource.

---

# Interview Question

### Can two transactions hold an Exclusive Lock on the same row?

## Answer

No.

Only one transaction may modify a specific resource at a time.

Other transactions must wait until the lock is released.

---

# 4. Blocking

## Interview Answer

Blocking occurs when one transaction waits because another transaction holds an incompatible lock.

Example:

```
Transaction A

↓

UPDATE Orders

↓

Lock

---------------

Transaction B

↓

UPDATE Orders

↓

Waiting...
```

Blocking is normal and expected.

It becomes a problem when it lasts too long.

---

# 5. Deadlock

## Interview Answer

A deadlock occurs when two or more transactions wait indefinitely for each other.

Example:

```
Transaction A

↓

Lock Orders

↓

Wait Customers

---------------

Transaction B

↓

Lock Customers

↓

Wait Orders
```

Neither transaction can continue.

---

# 6. Deadlock Detection

Most database engines automatically detect deadlocks.

Example:

```
Transaction A

↓

Waiting

---------------

Transaction B

↓

Waiting

↓

Deadlock Detected

↓

One Transaction Rolled Back
```

The transaction selected as the **deadlock victim** receives an error and must usually be retried by the application.

---

# 7. Long Transactions

Long-running transactions hold locks longer.

Example:

```
BEGIN TRANSACTION

↓

Update

↓

Wait 30 Seconds

↓

Commit
```

Other transactions remain blocked during that time.

Keep transactions as short as possible.

---

# 8. Lock Escalation

## Interview Answer

When many row locks are acquired, the database may replace them with a table lock.

Example:

```
5000 Row Locks

↓

Table Lock
```

Benefits:

- Lower lock management overhead.

Drawback:

- Reduced concurrency because more operations are blocked.

---

# 9. Reducing Blocking

Common techniques:

- Keep transactions short.
- Commit as soon as possible.
- Access tables in a consistent order.
- Create appropriate indexes.
- Avoid user interaction inside transactions.
- Batch large updates instead of modifying millions of rows in one transaction.

---

# 10. Deadlock Prevention

Recommended practices:

- Access resources in the same order.
- Keep transactions small.
- Avoid unnecessary locks.
- Use appropriate indexes.
- Retry deadlock victims safely.

Deadlocks cannot always be eliminated, but they can often be reduced.

---

# 11. Reading Deadlock Graphs

A deadlock graph shows:

- Which sessions were involved.
- Which resources were locked.
- Which session waited.
- Which session became the victim.

Understanding deadlock graphs is an important production troubleshooting skill.

---

# 12. Common Interview Questions

### Is blocking always bad?

## Answer

No.

Blocking is a normal consequence of maintaining consistency.

The goal is to minimize unnecessary or prolonged blocking.

---

### Can deadlocks happen without UPDATE statements?

Yes.

Depending on the isolation level and query behavior, even read operations can participate in deadlocks.

---

### Why do indexes reduce blocking?

Efficient indexes help the database find rows faster.

This shortens transaction duration and reduces the time locks are held.

---

### Should deadlock errors simply be ignored?

No.

Applications should log them and retry the transaction when appropriate.

Deadlocks are expected in highly concurrent systems, but frequent deadlocks may indicate a design issue.

---

# Lock Compatibility (Simplified)

| Existing Lock | Requested Lock | Result |
|--------------|----------------|--------|
| Shared | Shared | Allowed |
| Shared | Exclusive | Wait |
| Exclusive | Shared | Wait |
| Exclusive | Exclusive | Wait |

---

# Trade-offs

| Mechanism | Benefit | Cost |
|-----------|---------|------|
| Shared Locks | Concurrent reads | Can block writers |
| Exclusive Locks | Protect updates | Block other operations |
| Long Transactions | Simpler business logic | More blocking |
| Lock Escalation | Lower lock overhead | Reduced concurrency |

---

# Staff Engineer Insight

One of the most common production issues is assuming that a slow query is caused by poor SQL.

Sometimes the query is waiting, not executing.

Before optimizing the query itself, verify:

- Is it blocked?
- Which session holds the lock?
- Is there a long-running transaction?
- Is a deadlock occurring repeatedly?

Another common mistake is performing external work inside a transaction.

For example:

```
BEGIN TRANSACTION

↓

Update Database

↓

Call External API

↓

Wait 5 Seconds

↓

Commit
```

During those five seconds, locks remain held.

A better design is often to commit first and perform external operations afterward, or coordinate them using patterns such as the Outbox Pattern when consistency requirements allow.

---

# Summary

- Locks protect data consistency.
- Shared Locks support concurrent reads.
- Exclusive Locks protect writes.
- Blocking is normal but should be minimized.
- Deadlocks occur when transactions wait for each other.
- Databases automatically detect deadlocks and choose a victim.
- Keep transactions short.
- Use indexes to reduce lock duration.
- Retry deadlock victims when appropriate.
- Diagnose locking issues before assuming the SQL itself is inefficient.
