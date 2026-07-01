# Expression Trees, Reflection and Attributes

## 1. What is an Expression Tree?

Expression tree represents code as data. Instead of executing a lambda immediately, .NET stores its structure so another library can analyze it.

```csharp
Expression<Func<User, bool>> expression = u => u.Age > 18;
```

This is very important in Entity Framework Core. EF Core reads the expression tree and translates it into SQL.

```csharp
db.Users.Where(u => u.Age > 18)
```

EF Core does not execute C# directly in the database. It analyzes the expression and generates SQL similar to:

```sql
SELECT * FROM Users WHERE Age > 18
```

## 2. Func vs Expression<Func>

`Func<T, bool>` is executable code.

```csharp
Func<User, bool> predicate = u => u.Age > 18;
```

`Expression<Func<T, bool>>` is a data structure describing code.

```csharp
Expression<Func<User, bool>> predicate = u => u.Age > 18;
```

Use `Func` when you want to execute logic in memory. Use `Expression<Func<...>>` when another system needs to inspect or translate the logic, such as EF Core.

## 3. What is Reflection?

Reflection allows code to inspect assemblies, types, properties, methods and attributes at runtime.

Example:

```csharp
var type =
