# Reflection and Source Generators

## Overview

Reflection allows a .NET application to inspect types, assemblies and metadata at runtime.

It enables highly flexible frameworks but comes with performance costs.

Modern .NET increasingly favors compile-time code generation through Source Generators to achieve similar flexibility with better performance.

A senior engineer should understand when Reflection is appropriate, when it becomes a bottleneck and why Source Generators were introduced.

---

# 1. What is Reflection?

## Interview Answer

Reflection is the ability of a .NET application to inspect its own metadata at runtime.

Using Reflection, an application can discover:

- Types
- Properties
- Methods
- Constructors
- Attributes
- Interfaces
- Generic arguments

without knowing them at compile time.

---

# Example

```csharp
Type type = typeof(Customer);

Console.WriteLine(type.Name);
```

Output:

```
Customer
```

The application is reading metadata, not executing business logic.

---

# 2. Assemblies

Every compiled .NET application is packaged into assemblies.

Examples:

```
MyApi.dll

Domain.dll

Infrastructure.dll
```

Reflection allows applications to inspect assemblies dynamically.

Example:

```csharp
Assembly.Load(...)
```

---

# 3. Discovering Types

Example:

```csharp
var types =
    assembly.GetTypes();
```

Typical use cases:

- Plugin systems
- Dependency Injection
- Test discovery
- Serialization

---

# 4. Reading Properties

Example:

```csharp
PropertyInfo property =
    typeof(Customer)
        .GetProperty("Name");
```

Read value:

```csharp
var value =
    property.GetValue(customer);
```

Reflection allows accessing members dynamically.

---

# 5. Invoking Methods

Example:

```csharp
MethodInfo method =
    typeof(Customer)
        .GetMethod("Print");
```

Invoke:

```csharp
method.Invoke(customer, null);
```

The method name is discovered at runtime.

---

# Interview Question

### Why is Reflection slower?

## Answer

Reflection performs runtime metadata inspection.

The CLR cannot optimize these calls as effectively as direct method calls.

Reflection also involves additional allocations and security checks.

---

# 6. Attributes

Attributes attach metadata to code.

Example:

```csharp
[Required]

public string Name
{
    get;
    set;
}
```

Reflection can discover attributes.

Example:

```csharp
property.GetCustomAttributes();
```

Many frameworks rely heavily on attributes.

---

# 7. Real-world Uses

Reflection powers many familiar frameworks.

Examples:

ASP.NET Core

```
Controllers

↓

Attributes

↓

Routing
```

Dependency Injection

```
Discover Constructors

↓

Create Instances
```

Entity Framework

```
Properties

↓

Mappings
```

JSON Serialization

```
Properties

↓

JSON
```

---

# 8. Performance Costs

Reflection is powerful but relatively expensive.

Costs include:

- Metadata lookup
- Method invocation overhead
- Boxing in some scenarios
- Allocations

Reflection should generally not be used repeatedly inside performance-critical loops.

---

# 9. Caching Reflection

Bad:

```csharp
GetProperty(...)
```

called millions of times.

Better:

```
Reflection

↓

Cache PropertyInfo

↓

Reuse
```

Many frameworks cache reflection results to reduce overhead.

---

# 10. Source Generators

## Interview Answer

Source Generators generate C# code during compilation.

Instead of inspecting metadata at runtime:

```
Reflection

↓

Runtime Work
```

the compiler generates code:

```
Compile Time

↓

Generated Code

↓

Fast Execution
```

This reduces startup time and runtime overhead.

---

# 11. Why Source Generators?

Advantages:

- No runtime reflection
- Better startup performance
- Fewer allocations
- Better Native AOT compatibility
- Compile-time diagnostics

Many modern .NET libraries are moving toward code generation.

---

# 12. Native AOT

Reflection and Native AOT do not always work well together.

Native AOT removes unused metadata during compilation.

Applications relying heavily on Reflection may require additional configuration or alternative approaches.

Source Generators are generally more compatible with Native AOT.

---

# 13. Common Interview Questions

### Should Reflection be avoided?

## Answer

No.

Reflection is an excellent tool when flexibility is required.

However, it should not be used unnecessarily in performance-critical paths.

---

### Is Reflection compile-time or runtime?

Runtime.

The application discovers metadata while executing.

---

### Why are Source Generators faster?

Because the work happens during compilation rather than at runtime.

The generated code behaves like handwritten C#.

---

### Do Source Generators replace Reflection?

Not entirely.

Reflection is still required for some dynamic scenarios.

Source Generators replace many common reflection-heavy patterns where compile-time knowledge is sufficient.

---

# Trade-offs

| Technique | Advantage | Limitation |
|-----------|-----------|------------|
| Reflection | Very flexible | Runtime overhead |
| Cached Reflection | Better performance | Additional complexity |
| Source Generators | Fast runtime | Compile-time only |
| Native AOT | Excellent startup and memory | Reflection limitations |

---

# Staff Engineer Insight

One of the biggest misconceptions is that Reflection is "slow" and should never be used.

The better question is:

**How often is it used?**

Using Reflection once during application startup to discover services is usually insignificant.

Using Reflection for every object in a loop processing millions of records can become a bottleneck.

Another trend in modern .NET is shifting work from runtime to compile time.

Examples include:

- Source Generators
- Native AOT
- Compile-time serializers

These approaches reduce startup time, memory usage and runtime allocations while preserving developer productivity.

---

# Summary

- Reflection allows runtime inspection of assemblies and types.
- It enables many framework features, including routing, dependency injection and serialization.
- Reflection is flexible but introduces runtime overhead.
- Cache reflection results when repeated access is required.
- Source Generators move work from runtime to compile time.
- Native AOT encourages reducing runtime reflection.
- Choose Reflection for dynamic scenarios and Source Generators when compile-time generation is possible.
