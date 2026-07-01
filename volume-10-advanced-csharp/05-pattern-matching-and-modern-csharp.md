# Pattern Matching and Modern C#

## Overview

Modern C# has evolved far beyond classes, interfaces and simple `if` statements.

Pattern matching, records, switch expressions and newer language features make code more expressive and reduce boilerplate.

A senior engineer should know not only these features, but also when they improve readability and when they make code harder to understand.

---

# 1. What is Pattern Matching?

## Interview Answer

Pattern matching allows code to test whether a value has a specific shape, type or property structure.

Instead of writing many nested `if` statements, pattern matching expresses conditions more clearly.

Example:

```csharp
if (shape is Circle circle)
{
    Console.WriteLine(circle.Radius);
}
```

This checks the type and safely casts it in one expression.

---

# 2. Type Pattern

```csharp
object value = "hello";

if (value is string text)
{
    Console.WriteLine(text.Length);
}
```

The compiler checks whether `value` is a string and assigns it to `text`.

---

# 3. Property Pattern

```csharp
if (user is { Age: >= 18 })
{
    Console.WriteLine("Adult");
}
```

This checks a property directly inside the pattern.

More complex example:

```csharp
if (order is { Status: OrderStatus.Paid, Total: > 1000 })
{
    ApplyVipHandling(order);
}
```

This is cleaner than multiple `&&` checks.

---

# 4. Switch Expression

Traditional switch:

```csharp
switch (status)
{
    case OrderStatus.Pending:
        return "Waiting";
    case OrderStatus.Paid:
        return "Completed";
    default:
        return "Unknown";
}
```

Modern switch expression:

```csharp
var label = status switch
{
    OrderStatus.Pending => "Waiting",
    OrderStatus.Paid => "Completed",
    _ => "Unknown"
};
```

This is concise and works well for mapping values.

---

# 5. Relational Patterns

```csharp
var category = age switch
{
    < 18 => "Minor",
    >= 18 and < 65 => "Adult",
    >= 65 => "Senior"
};
```

Relational patterns make numeric comparisons cleaner.

---

# 6. List Patterns

```csharp
if (numbers is [1, 2, ..])
{
    Console.WriteLine("Starts with 1 and 2");
}
```

This checks that a collection starts with specific values.

Example:

```csharp
var result = commandParts switch
{
    ["create", var name] => Create(name),
    ["delete", var id] => Delete(id),
    _ => Invalid()
};
```

Useful for command parsing and small structured inputs.

---

# 7. Records

## Interview Answer

Records are reference types designed for immutable data and value-based equality.

Example:

```csharp
public record UserDto(int Id, string Name);
```

Two records with the same values are considered equal.

```csharp
var a = new UserDto(1, "John");
var b = new UserDto(1, "John");

Console.WriteLine(a == b); // True
```

With normal classes, this would usually be `False` unless equality is overridden.

---

# 8. init Properties

```csharp
public class User
{
    public string Name { get; init; }
}
```

`init` allows setting a property during object initialization but not later.

Example:

```csharp
var user = new User
{
    Name = "John"
};

user.Name = "Mike"; // compile error
```

This supports immutable object models.

---

# 9. required Members

```csharp
public class User
{
    public required string Name { get; init; }
}
```

Now this fails:

```csharp
var user = new User();
```

The caller must provide `Name`.

This helps avoid partially initialized objects.

---

# 10. Primary Constructors

```csharp
public class UserService(IUserRepository repository)
{
    public Task<User?> Get(int id)
    {
        return repository.GetById(id);
    }
}
```

Primary constructors reduce boilerplate when injecting dependencies.

They are especially useful for simple services.

---

# 11. Collection Expressions

Old syntax:

```csharp
var numbers = new List<int> { 1, 2, 3 };
```

New syntax:

```csharp
List<int> numbers = [1, 2, 3];
```

Works with arrays, lists and other supported collection types.

---

# 12. Raw String Literals

Old syntax:

```csharp
var json = "{\"name\":\"John\"}";
```

New syntax:

```csharp
var json = """
{
    "name": "John"
}
""";
```

Very useful for:

- JSON
- SQL
- HTML
- Test data

---

# 13. Common Interview Questions

### Are records always immutable?

## Answer

No.

Records encourage immutability, especially with positional records and `init` properties, but they can still contain mutable properties if you define them that way.

---

### Should switch expressions replace every if statement?

## Answer

No.

Use switch expressions when mapping values or matching shapes.

For complex procedural logic, normal `if` statements may remain clearer.

---

### Are primary constructors always better?

## Answer

No.

They reduce boilerplate, but for complex constructors with validation or multiple initialization steps, a traditional constructor may be clearer.

---

### Why use required properties?

## Answer

They make required object state explicit and catch missing initialization at compile time.

---

# Trade-offs

| Feature | Benefit | Limitation |
|----------|---------|------------|
| Pattern Matching | Expressive conditions | Can become too clever |
| Records | Value equality | Not always suitable for entities |
| init | Supports immutability | Less flexible mutation |
| required | Safer initialization | More verbose object creation |
| Primary Constructors | Less boilerplate | Can reduce clarity in complex classes |
| Raw Strings | Cleaner literals | Requires newer C# version |

---

# Staff Engineer Insight

Modern C# features should improve readability, not demonstrate cleverness.

A common mistake is replacing simple, obvious code with overly compressed pattern matching.

Readable code is not always the shortest code.

Another common mistake is using records for domain entities. Records provide value equality, while domain entities usually have identity-based equality.

Good candidates for records:

- DTOs
- Commands
- Queries
- Value Objects
- API responses

Poor candidates:

- EF Core tracked entities
- Mutable domain aggregates
- Long-lived identity-based objects

---

# Summary

- Pattern matching makes type and property checks more expressive.
- Switch expressions are great for value mapping.
- Records provide value-based equality.
- `init` supports immutable object initialization.
- `required` prevents missing required state.
- Primary constructors reduce boilerplate.
- Raw string literals simplify JSON, SQL and HTML.
- Modern C# features should be used to improve clarity, not to make code look clever.
