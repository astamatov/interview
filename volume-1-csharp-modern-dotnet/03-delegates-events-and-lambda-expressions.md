# Delegates, Events and Lambda Expressions

## Overview

Delegates are one of the core language features in C#. They enable methods to be treated as first-class citizens by allowing them to be passed, stored and invoked dynamically. Events build on delegates to implement the publish/subscribe pattern, while lambda expressions provide a concise syntax for creating delegates.

Understanding how these features work is essential because they are heavily used throughout .NET, including LINQ, ASP.NET Core, Entity Framework Core and asynchronous programming.

---

# 1. What is a Delegate?

## Interview Answer

A delegate is a type-safe function pointer. It can reference one or more methods that share the same signature. Delegates allow behavior to be passed as data, making code more flexible and reusable.

Unlike function pointers in C++, delegates are fully type-safe and integrate with the .NET type system.

---

## Example

```csharp
public delegate int MathOperation(int a, int b);

public static int Add(int x, int y)
{
    return x + y;
}

MathOperation operation = Add;

int result = operation(5, 3);
```

Output:

```
8
```

---

# 2. Why use Delegates?

Without delegates:

```csharp
public void Process()
{
    SaveToDatabase();
}
```

The behavior is fixed.

With delegates:

```csharp
public void Process(Action action)
{
    action();
}
```

Now the caller decides what should happen.

Example:

```csharp
Process(() => Console.WriteLine("Hello"));
```

Delegates enable inversion of control and are the foundation of callback-based programming.

---

# 3. Multicast Delegates

A delegate can reference multiple methods.

```csharp
Action action = FirstMethod;
action += SecondMethod;
action += ThirdMethod;

action();
```

Execution order:

```
FirstMethod()
SecondMethod()
ThirdMethod()
```

This is called a multicast delegate.

---

# 4. Func, Action and Predicate

These built-in delegates eliminate the need to define custom delegate types.

### Action

Returns nothing.

```csharp
Action<string> log = Console.WriteLine;

log("Hello");
```

---

### Func

Returns a value.

```csharp
Func<int, int, int> add = (a, b) => a + b;

Console.WriteLine(add(2, 3));
```

Output:

```
5
```

The last generic parameter is always the return type.

---

### Predicate

Always returns bool.

```csharp
Predicate<int> isEven = x => x % 2 == 0;

Console.WriteLine(isEven(10));
```

Output:

```
True
```

Predicate is commonly used for filtering.

---

# 5. Lambda Expressions

Lambdas are anonymous functions.

Instead of:

```csharp
public bool IsAdult(Person p)
{
    return p.Age >= 18;
}
```

We can write:

```csharp
person => person.Age >= 18
```

This syntax is shorter, easier to read and works naturally with delegates and LINQ.

---

# 6. Delegates vs Interfaces

This is a common interview question.

### Delegate

Use when you need to pass **a single piece of behavior**.

Example:

```csharp
Action sendEmail
```

---

### Interface

Use when an object exposes multiple related behaviors or maintains state.

Example:

```csharp
public interface IEmailService
{
    void Send();
    void Schedule();
    void Cancel();
}
```

Rule of thumb:

- Delegate → behavior
- Interface → capability

---

# 7. What are Events?

## Interview Answer

Events implement the publish/subscribe pattern.

They allow one object to notify many subscribers when something happens while preventing external code from raising the event.

Events are built on top of delegates.

---

## Example

```csharp
public class OrderService
{
    public event EventHandler? OrderCreated;

    public void Create()
    {
        OrderCreated?.Invoke(this, EventArgs.Empty);
    }
}
```

Subscriber:

```csharp
service.OrderCreated += (_, _) =>
{
    Console.WriteLine("Order created.");
};
```

---

# 8. Why use Events instead of Delegates?

If you expose a delegate publicly:

```csharp
public Action OnSave;
```

Anyone can call:

```csharp
OnSave();
```

That breaks encapsulation.

With events:

```csharp
public event Action OnSave;
```

External code can only subscribe or unsubscribe.

Only the publisher can invoke the event.

---

# 9. Anonymous Methods vs Lambdas

Anonymous method:

```csharp
delegate(int x)
{
    return x * 2;
}
```

Lambda:

```csharp
x => x * 2
```

Lambdas are the preferred modern syntax.

---

# 10. Common Interview Questions

### Are delegates reference types?

Yes.

They inherit from `System.MulticastDelegate`.

---

### Can delegates return values?

Yes.

Only `Action` returns void.

---

### Can an event have multiple subscribers?

Yes.

That's one of its primary purposes.

---

### Can events return values?

No.

Events are notifications.

---

### Are delegates thread-safe?

The delegate object itself is immutable.

However, adding and removing subscribers should be done carefully in multithreaded scenarios.

---

# Senior Tips

- Prefer `Func` and `Action` over custom delegates unless a custom name improves readability.
- Use events only for notifications, not for request/response workflows.
- Delegates are heavily used throughout LINQ, async programming and dependency injection frameworks.
- Remember that events are simply a restricted wrapper around multicast delegates.

---

# Summary

- Delegates are type-safe function pointers.
- Events are built on delegates.
- `Action` returns void.
- `Func` returns a value.
- `Predicate<T>` returns bool.
- Lambda expressions are concise anonymous functions.
- Delegates represent behavior; interfaces represent contracts.
- Events implement the publish/subscribe pattern.
