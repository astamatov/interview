# Transactions, ACID and Concurrency

## Overview

Every enterprise application eventually faces concurrency problems. Multiple users update the same data, transactions overlap, deadlocks occur and inconsistent data appears.

A senior .NET developer should understand not only how to use transactions in EF Core, but also **how the database guarantees consistency**.

---

# 1. What is a Transaction?

## Answer

A transaction is a group of operations that are treated as a single logical unit of work.

Either **all operations succeed**, or **none of them do**.

Example:

```csharp
using var transaction = await context.Database.BeginTransactionAsync();

try
{
    context.Accounts.Add(account);

    context.Users.Add(user);

    await context.SaveChangesAsync();

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
}
```

If one operation fails, the database rolls everything back.

---

# 2. What does ACID stand for?

## Answer

ACID defines the guarantees that every reliable relational database should provide.

- **A** – Atomicity
- **C** – Consistency
- **I** – Isolation
- **D** – Durability

These properties ensure data remains correct even when failures or concurrent access occur.

---

# 3. Atomicity

## Answer

Atomicity means **all or nothing**.

Example:

Money transfer.

```
Account A

-100€

↓

Account B

+100€
```

If the application crashes after subtracting money but before adding it, the transaction must roll back.

Otherwise money disappears.

---

# Interview Question

### Why are transactions important?

Because they guarantee that partial changes are never persisted.

---

# 4. Consistency

## Answer

Consistency guarantees that every transaction leaves the database in a valid state.

Example:

Suppose a rule says:

```
Balance >= 0
```

A transaction must never commit data that violates that rule.

Consistency combines:

- Constraints
- Foreign keys
- Unique indexes
- Check constraints
- Application rules

---

# 5. Isolation

## Answer

Isolation ensures concurrent transactions do not interfere with each other.

Without isolation:

```
Transaction A

↓

Changes balance

↓

Transaction B reads half-finished data
```

Incorrect results occur.

Isolation levels control how much concurrent transactions can see.

---

# 6. Durability

## Answer

Once a transaction commits successfully, the data is permanently stored.

Even if:

- Application crashes
- Server reboots
- Power failure occurs

the committed transaction remains.

This is achieved using the database transaction log.

---

# 7. Isolation Levels

SQL Server supports several isolation levels.

---

## Read Uncommitted

Fastest.

Allows reading uncommitted changes.

Can produce:

- Dirty Reads
- Non-repeatable Reads
- Phantom Reads

Usually avoided in business applications.

---

## Read Committed (Default)

The SQL Server default.

Only committed data can be read.

Prevents dirty reads.

Still allows:

- Non-repeatable Reads
- Phantom Reads

---

## Repeatable Read

Guarantees that rows already read cannot change until the transaction completes.

Still allows phantom rows.

---

## Serializable

Highest isolation level.

Transactions execute as if they happened one after another.

Safest.

Slowest.

Can significantly reduce concurrency.

---

## Snapshot

Uses row versioning instead of locks.

Readers don't block writers.

Writers don't block readers.

Excellent for systems with many reads.

Requires database configuration.

---

# 8. Dirty Read

Example:

Transaction A:

```
Update Balance = 100
```

Not committed yet.

Transaction B:

```
Reads Balance = 100
```

Then Transaction A rolls back.

Balance was never actually 100.

Transaction B read invalid data.

---

# 9. Non-repeatable Read

Transaction A:

```
SELECT Balance
```

Result:

```
100
```

Transaction B:

```
UPDATE Balance = 200
COMMIT
```

Transaction A reads again.

Result:

```
200
```

The same query returned different values.

---

# 10. Phantom Read

Transaction A:

```sql
SELECT *
FROM Orders
WHERE Status='Pending'
```

Returns:

```
15 rows
```

Meanwhile:

Transaction B inserts another pending order.

Transaction A executes the same query again.

Now:

```
16 rows
```

A new "phantom" row appeared.

---

# 11. Optimistic Concurrency

## Answer

Optimistic concurrency assumes conflicts are rare.

No locks are held while users edit data.

Before updating, EF Core checks whether another transaction modified the row.

Usually implemented using a `rowversion` column.

Example:

```csharp
public class User
{
    public int Id { get; set; }

    [Timestamp]
    public byte[] RowVersion { get; set; } = default!;
}
```

Generated SQL:

```sql
UPDATE Users
SET Name='John'
WHERE Id=1
AND RowVersion=@RowVersion
```

If no rows are updated, EF Core throws:

```
DbUpdateConcurrencyException
```

---

# Interview Question

### Why is optimistic concurrency preferred?

Because locking rows for minutes while users edit data would destroy scalability.

Optimistic concurrency allows maximum throughput while still detecting conflicts.

---

# 12. Pessimistic Locking

Instead of detecting conflicts later, pessimistic locking prevents them.

Example:

```sql
SELECT *
FROM Users WITH (UPDLOCK)
WHERE Id = 1;
```

Other transactions must wait.

Useful when conflicts are expected.

Less scalable.

---

# 13. Deadlocks

A deadlock occurs when two transactions wait for each other forever.

Example:

Transaction A

```
Locks Users

↓

Waiting for Orders
```

Transaction B

```
Locks Orders

↓

Waiting for Users
```

SQL Server detects the deadlock and terminates one transaction.

---

# How to Reduce Deadlocks

- Keep transactions short.
- Access tables in the same order.
- Create proper indexes.
- Avoid unnecessary locks.

---

# 14. Transactions in EF Core

Default:

```csharp
await context.SaveChangesAsync();
```

EF Core automatically creates a transaction if multiple changes are being saved.

Explicit transaction:

```csharp
await using var tx =
    await context.Database.BeginTransactionAsync();

try
{
    ...

    await context.SaveChangesAsync();

    await tx.CommitAsync();
}
catch
{
    await tx.RollbackAsync();
}
```

---

# 15. Common Interview Questions

### Does SaveChanges() create a transaction?

Yes.

If multiple SQL statements must succeed together.

---

### Should every query use Serializable?

No.

Higher isolation increases locking and reduces concurrency.

Use the lowest isolation level that satisfies business requirements.

---

### Why is Snapshot Isolation popular?

Readers don't block writers.

Writers don't block readers.

Excellent scalability.

---

# Senior Tips

- Understand **why** ACID exists, not just what the letters mean.
- Default to optimistic concurrency unless business rules require pessimistic locking.
- Use `rowversion` for concurrency control in EF Core.
- Keep transactions as short as possible.
- Never perform HTTP calls inside an open database transaction.
- Learn the differences between isolation levels—they are asked frequently in senior interviews.

---

# Summary

- Transactions guarantee all-or-nothing execution.
- ACID provides reliability and consistency.
- Isolation levels control concurrent access.
- Optimistic concurrency detects conflicts using version checks.
- Pessimistic locking prevents conflicts by locking rows.
- Deadlocks occur when transactions wait on each other.
- Short transactions and proper indexing improve concurrency.
