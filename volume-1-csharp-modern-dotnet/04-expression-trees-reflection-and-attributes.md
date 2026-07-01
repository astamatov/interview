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
var type = typeof(User);

Console.WriteLine(type.Name);

foreach (var property in type.GetProperties())
{
    Console.WriteLine(property.Name);
}
```

Output:

```text
User
Id
Name
Age
```

Reflection is widely used by:

- Dependency Injection containers
- Entity Framework Core
- ASP.NET Core Model Binding
- JSON serializers
- AutoMapper
- Plugin systems

---

## 4. Creating Objects with Reflection

Normally:

```csharp
var user = new User();
```

Reflection:

```csharp
Type type = typeof(User);

object instance = Activator.CreateInstance(type)!;
```

This allows frameworks to create objects without knowing their types at compile time.

---

## 5. Invoking Methods with Reflection

```csharp
MethodInfo method = typeof(UserService)
    .GetMethod("CreateUser")!;

method.Invoke(service, new object[]
{
    "John"
});
```

This is slower than a direct method call but provides flexibility.

---

## 6. Reading Properties Dynamically

```csharp
PropertyInfo property =
    typeof(User).GetProperty("Name")!;

var value = property.GetValue(user);

property.SetValue(user, "John");
```

This technique is frequently used in ORMs and mapping libraries.

---

## 7. Is Reflection Slow?

## Interview Answer

Yes, reflection is significantly slower than direct code execution because metadata must be inspected at runtime.

However, modern frameworks typically perform reflection once during startup and cache the results.

Example:

- ASP.NET Core scans controllers during application startup.
- The Dependency Injection container scans constructors once.
- AutoMapper builds mapping expressions once.

Because of caching, reflection is usually not a performance problem unless it is repeatedly used inside hot loops.

---

## 8. What are Attributes?

Attributes provide metadata that can be attached to classes, methods, properties or parameters.

Example:

```csharp
public class User
{
    [Required]
    public string Name { get; set; } = "";

    [Range(18, 100)]
    public int Age { get; set; }
}
```

The attributes themselves do nothing.

Frameworks inspect them using Reflection.

---

## 9. Custom Attributes

```csharp
[AttributeUsage(AttributeTargets.Class)]
public class AuditableAttribute : Attribute
{
    public string CreatedBy { get; }

    public AuditableAttribute(string createdBy)
    {
        CreatedBy = createdBy;
    }
}
```

Usage:

```csharp
[Auditable("Admin")]
public class User
{
}
```

Reading the attribute:

```csharp
var attribute = typeof(User)
    .GetCustomAttribute<AuditableAttribute>();

Console.WriteLine(attribute?.CreatedBy);
```

---

## 10. Where are Attributes Used?

Attributes are everywhere in modern .NET.

Examples:

ASP.NET Core

```csharp
[HttpGet]
[Authorize]
```

Entity Framework Core

```csharp
[Key]
[Required]
```

Validation

```csharp
[StringLength(100)]
```

Serialization

```csharp
[JsonIgnore]
```

Testing

```csharp
[Test]
[Fact]
```

---

## 11. Reflection vs Source Generators

Reflection happens at runtime.

Source Generators generate code during compilation.

Reflection

✔ Flexible

✔ Dynamic

❌ Slower

Source Generators

✔ Faster

✔ Compile-time validation

✔ No runtime reflection

Modern libraries increasingly use Source Generators for better performance.

---

## 12. Common Interview Questions

### Can private members be accessed using Reflection?

Yes.

Reflection can access private members if the appropriate binding flags are used.

---

### Why doesn't EF Core use Func?

Because EF needs to translate expressions into SQL.

Only Expression Trees contain the structure necessary for translation.

---

### Should Reflection be avoided?

Not necessarily.

It should be avoided in performance-critical loops.

For application startup, plugin loading and metadata discovery it is perfectly acceptable.

---

## Senior Tips

- Know the difference between `Func<T>` and `Expression<Func<T>>`. This is asked very frequently.
- Reflection is powerful but should be cached whenever possible.
- Attributes are passive metadata; frameworks decide what to do with them.
- Source Generators are becoming increasingly important because they eliminate much of the runtime cost associated with reflection.

---

# Summary

- Expression Trees represent code as data.
- EF Core translates Expression Trees into SQL.
- Reflection inspects assemblies, types and metadata at runtime.
- Attributes provide metadata consumed by frameworks.
- Reflection is slower than direct code but usually acceptable when cached.
- Source Generators can replace reflection in many scenarios and improve startup time and performance.
