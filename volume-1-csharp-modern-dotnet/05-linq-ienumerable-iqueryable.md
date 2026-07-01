# LINQ, IEnumerable and IQueryable

## Overview

LINQ (Language Integrated Query) is one of the most frequently discussed topics in senior .NET interviews. Most developers know how to write LINQ queries, but senior engineers understand **when queries execute, where they execute, how deferred execution works, and how Entity Framework translates LINQ into SQL.**

---

# 1. What is LINQ?

## Interview Answer

LINQ (Language Integrated Query) is a feature in C# that provides a unified syntax for querying different data sources such as collections, databases, XML and APIs.

Instead of writing loops manually, LINQ allows developers to express queries declaratively.

Example:

```csharp
var adults = users
    .Where(u => u.Age >= 18)
    .OrderBy(u => u.Name)
    .Select(u => u.Name);
```

LINQ makes code more readable, composable and reusable.

---

# 2. Deferred Execution

## Interview Answer

Most LINQ operators use **deferred execution**.

This means the query is **not executed immediately** when it is created.

Instead, execution happens only when the results are enumerated.

Example:

```csharp
var query = users.Where(u => u.Age > 18);

// No query executed yet

foreach (var user in query)
{
    Console.WriteLine(user.Name);
}
```

The filtering happens during enumeration.

---

## Why is this useful?

Imagine:

```csharp
var query = users.Where(u => u.IsActive);
```

Later:

```csharp
users.Add(new User { IsActive = true });
```

When enumerated:

```csharp
query.ToList();
```

The new user is included because the query is evaluated at enumeration time.

---

# 3. Immediate Execution

Some LINQ methods execute immediately.

Examples:

```csharp
ToList()

ToArray()

Count()

Any()

First()

Single()

Max()

Min()

Average()
```

These methods force query execution.

Example:

```csharp
var list = users
    .Where(u => u.IsActive)
    .ToList();
```

Now the query has already been executed.

---

# 4. IEnumerable

## Interview Answer

`IEnumerable<T>` represents a sequence of objects that can be enumerated in memory.

It exposes a single method:

```csharp
GetEnumerator()
```

LINQ operators on `IEnumerable` execute using C# code.

Example:

```csharp
IEnumerable<User> users = repository.GetUsers();

var active = users.Where(u => u.IsActive);
```

Filtering happens **inside the application**.

---

# 5. IQueryable

## Interview Answer

`IQueryable<T>` represents a query that can be translated into another query language.

In Entity Framework Core, LINQ expressions are translated into SQL.

Example:

```csharp
IQueryable<User> users = db.Users;

var active = users
    .Where(u => u.IsActive);
```

This does **not** filter in memory.

Instead, EF Core generates SQL similar to:

```sql
SELECT *
FROM Users
WHERE IsActive = 1
```

The database performs the filtering.

---

# 6. IEnumerable vs IQueryable

| IEnumerable | IQueryable |
|-------------|------------|
| Executes in memory | Executes in database |
| Uses delegates | Uses expression trees |
| Suitable for collections | Suitable for EF Core |
| Cannot generate SQL | Can generate SQL |

---

# 7. Common Mistake

Bad:

```csharp
var users = db.Users
    .ToList()
    .Where(u => u.Age > 18);
```

What happens?

```
Database
↓

Load ALL users

↓

Memory

↓

Filter users
```

Good:

```csharp
var users = db.Users
    .Where(u => u.Age > 18)
    .ToList();
```

Now SQL becomes:

```sql
SELECT *
FROM Users
WHERE Age > 18
```

Only matching rows are returned.

---

# 8. Select Projection

Avoid loading unnecessary columns.

Bad:

```csharp
var users = db.Users.ToList();
```

Good:

```csharp
var names = db.Users
    .Select(u => u.Name)
    .ToList();
```

Generated SQL:

```sql
SELECT Name
FROM Users
```

This reduces:

- network traffic
- memory usage
- query execution time

---

# 9. Any vs Count

Bad:

```csharp
if(users.Count() > 0)
```

This counts every matching row.

Better:

```csharp
if(users.Any())
```

Generated SQL usually becomes:

```sql
SELECT TOP(1)
```

or

```sql
EXISTS(...)
```

Much faster.

---

# 10. First vs Single

`First()`

Returns the first matching element.

Throws if none exists.

---

`FirstOrDefault()`

Returns null/default if nothing is found.

---

`Single()`

Exactly one element must exist.

Throws if:

- none found
- more than one found

---

`SingleOrDefault()`

Zero or one element is allowed.

Throws if more than one exists.

---

## Interview Tip

Use `Single()` only when the business rule guarantees uniqueness.

Examples:

- Email
- National ID
- Username

---

# 11. yield return

## Interview Answer

`yield return` allows lazy generation of sequences.

Instead of creating an entire collection, items are produced one at a time.

Example:

```csharp
public IEnumerable<int> Numbers()
{
    yield return 1;
    yield return 2;
    yield return 3;
}
```

Useful for:

- streaming
- large datasets
- reducing memory allocations

---

# 12. Common Interview Questions

### Does LINQ always execute immediately?

No.

Most operators use deferred execution.

---

### Does IQueryable execute SQL immediately?

No.

SQL is generated only when the query is materialized.

---

### Which methods materialize a query?

- ToList
- ToArray
- First
- Single
- Count
- Any

---

### Why is IQueryable faster with databases?

Because filtering, sorting and projection happen on the database server instead of in application memory.

---

# Senior Tips

- Keep queries as `IQueryable` for as long as possible.
- Materialize (`ToList`) only when necessary.
- Always project only the columns you need.
- Prefer `Any()` over `Count() > 0`.
- Avoid calling `ToList()` too early.
- Remember: `IQueryable` is built on **Expression Trees**, not delegates.

---

# Summary

- LINQ provides a unified query syntax.
- Deferred execution delays query execution.
- `IEnumerable` works in memory.
- `IQueryable` translates expressions into SQL.
- Materialize queries only when needed.
- Avoid premature `ToList()`.
- Use projections to improve performance.
