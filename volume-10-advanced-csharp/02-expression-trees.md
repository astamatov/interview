# Expression Trees

## Overview

Expression Trees allow C# code to be represented as data instead of being executed immediately.

This makes it possible to inspect, modify and translate code at runtime.

They are one of the core technologies behind:

- Entity Framework Core
- LINQ Providers
- AutoMapper
- Moq
- Dynamic query builders

Understanding Expression Trees helps explain how LINQ queries become SQL and how many popular .NET libraries work internally.

---

# 1. Delegate vs Expression Tree

## Interview Answer

A delegate represents executable code.

An Expression Tree represents the structure of code.

Example:

```csharp
Func<Customer, bool> func =
    c => c.Age > 18;
```

The lambda is compiled and can be executed immediately.

Expression Tree:

```csharp
Expression<Func<Customer, bool>> expr =
    c => c.Age > 18;
```

Nothing is executed.

Instead, the lambda becomes a tree describing the operation.

---

# Interview Question

### Why can't Entity Framework use Func<T, bool>?

## Answer

Because a delegate only contains executable code.

EF Core needs to inspect the expression so it can translate it into SQL.

Only Expression Trees expose the structure of the lambda.

---

# 2. Expression Tree Structure

The expression:

```csharp
c => c.Age > 18
```

becomes something conceptually like:

```
Lambda

↓

Parameter (Customer)

↓

GreaterThan

↓

Member (Age)

↓

Constant (18)
```

Instead of executing code, the runtime stores an object graph describing it.

---

# 3. Why Expression Trees Exist

Imagine:

```csharp
customers.Where(c => c.Age > 18);
```

With LINQ-to-Objects:

```
Lambda

↓

Execute

↓

Filter Objects
```

With EF Core:

```
Lambda

↓

Expression Tree

↓

SQL Translator

↓

SELECT ...

↓

Database
```

Without Expression Trees, EF Core would have no way to generate SQL.

---

# 4. Building Expressions

Expression Trees can also be created programmatically.

Example:

```csharp
ParameterExpression parameter =
    Expression.Parameter(typeof(Customer), "c");
```

Member:

```csharp
Expression.Property(parameter, "Age");
```

Constant:

```csharp
Expression.Constant(18);
```

Comparison:

```csharp
Expression.GreaterThan(...)
```

Finally:

```csharp
Expression.Lambda(...)
```

This allows dynamic query generation.

---

# 5. Dynamic Filtering

Suppose users can filter by any property.

Instead of writing:

```csharp
if(name != null)

if(age != null)

if(city != null)
```

an Expression Tree can be built dynamically.

Result:

```
UI Filters

↓

Expression Tree

↓

LINQ Query

↓

SQL
```

This technique is widely used in enterprise applications.

---

# 6. Compiling Expressions

An Expression Tree can later become executable code.

Example:

```csharp
var func =
    expr.Compile();
```

Now it behaves like a normal delegate.

Remember:

```
Expression

↓

Compile

↓

Delegate
```

Compilation has a cost, so compiled expressions are often cached.

---

# 7. Visiting Expressions

Expression Trees can be traversed using an `ExpressionVisitor`.

Typical use cases:

- Modify queries
- Replace nodes
- Add filters
- Analyze expressions
- Build custom LINQ providers

Many frameworks rely on this pattern.

---

# 8. Expression Trees in EF Core

When writing:

```csharp
context.Users
    .Where(u => u.Age > 18)
```

EF Core:

1. Receives an Expression Tree.
2. Parses the tree.
3. Generates SQL.
4. Executes the SQL.
5. Materializes entities.

This explains why some C# methods cannot be translated into SQL.

---

# 9. Common Translation Problems

Example:

```csharp
.Where(x => MyMethod(x.Name))
```

If `MyMethod` has no SQL equivalent:

```
Expression

↓

Cannot Translate

↓

Exception
```

Only expressions EF Core knows how to translate can run on the database.

---

# 10. Common Interview Questions

### Are Expression Trees executed automatically?

## Answer

No.

They describe code.

They can later be translated or compiled.

---

### Why are Expression Trees immutable?

Immutability makes them thread-safe and easier for frameworks to analyze.

To modify an expression, create a new tree rather than changing the existing one.

---

### Can every C# feature appear in an Expression Tree?

No.

Expression Trees support a defined subset of language constructs.

Some newer C# features cannot be represented or translated.

---

### Why does EF Core sometimes evaluate on the client?

When part of a query cannot be translated into SQL, EF Core may fail the query or, in some scenarios depending on configuration and version, evaluate portions on the client.

This is usually undesirable for large datasets.

---

# Trade-offs

| Feature | Benefit | Limitation |
|----------|---------|------------|
| Expression Trees | Runtime analysis | More complex than delegates |
| Dynamic Queries | Flexible filtering | Harder debugging |
| Compile() | Reusable delegate | Compilation cost |
| ExpressionVisitor | Powerful transformations | Steeper learning curve |

---

# Staff Engineer Insight

One of the most common misunderstandings is assuming every LINQ query behaves the same.

Consider:

```csharp
IEnumerable<Customer>
```

versus

```csharp
IQueryable<Customer>
```

`IEnumerable<T>` executes in memory.

`IQueryable<T>` builds an Expression Tree that may execute on another system, such as a SQL database.

Understanding this distinction explains why some LINQ methods execute immediately while others become SQL.

Another common mistake is building highly dynamic Expression Trees when simpler code would be easier to maintain.

Use Expression Trees when flexibility is genuinely required, not as a replacement for straightforward business logic.

---

# Summary

- Delegates execute code.
- Expression Trees describe code.
- EF Core uses Expression Trees to generate SQL.
- Expression Trees can be inspected, modified and compiled.
- Dynamic filtering is a common enterprise use case.
- `ExpressionVisitor` enables advanced transformations.
- `IQueryable<T>` relies on Expression Trees.
- Use Expression Trees when runtime analysis is required, not for ordinary application logic.
