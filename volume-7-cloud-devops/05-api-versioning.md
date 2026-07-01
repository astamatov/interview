# API Versioning

## Overview

Once an API is published, clients start depending on it.

Changing or removing endpoints without a versioning strategy can break mobile apps, frontend applications and third-party integrations.

API versioning allows an API to evolve while maintaining backward compatibility.

A senior engineer should understand **when to introduce a new version, how to version an API and how to deprecate old versions safely.**

---

# 1. What is API Versioning?

## Interview Answer

API versioning is the practice of maintaining multiple versions of an API so existing clients continue working while newer clients adopt new functionality.

Instead of changing an existing API:

```
GET /users
```

we expose versions.

```
/api/v1/users

/api/v2/users
```

Old clients continue using v1.

New clients migrate to v2.

---

# 2. Why Do We Need Versioning?

Imagine a mobile application using:

```json
{
    "id": 1,
    "name": "John"
}
```

Tomorrow you rename:

```
name

↓

fullName
```

Every existing mobile application breaks.

Versioning prevents these breaking changes.

---

# 3. URI Versioning

Example:

```
GET /api/v1/orders

GET /api/v2/orders
```

Advantages:

- Very easy to understand
- Easy to test
- Visible in URLs
- Most common approach

Disadvantages:

- Changes the resource URI

---

# 4. Header Versioning

Example:

```
GET /orders

API-Version: 2
```

Advantages:

- Cleaner URLs
- Resource URI remains stable

Disadvantages:

- Harder to test manually
- Less visible

---

# 5. Query String Versioning

Example:

```
GET /orders?version=2
```

Advantages:

- Simple implementation

Disadvantages:

- Less commonly used
- Easy to forget
- Less RESTful

---

# Interview Question

### Which versioning strategy is most common?

## Answer

URI versioning is the most common because it is explicit, easy to understand and well supported by tooling.

Header versioning is also popular in enterprise APIs.

---

# 6. Breaking vs Non-Breaking Changes

## Breaking Changes

Examples:

- Remove a property
- Rename a property
- Change property type
- Remove an endpoint
- Change endpoint behavior

Usually require a new API version.

---

## Non-Breaking Changes

Examples:

- Add an optional field
- Add a new endpoint
- Improve performance
- Add optional query parameters

These often do not require a new version.

---

# 7. Deprecation

## Interview Answer

Old API versions should not disappear immediately.

Typical lifecycle:

```
v1 Released

↓

v2 Released

↓

v1 Deprecated

↓

Migration Period

↓

v1 Removed
```

Clients need sufficient time to migrate.

---

# 8. Backward Compatibility

Whenever possible:

```
Old Client

↓

Still Works
```

This reduces deployment risk and avoids forcing immediate client upgrades.

---

# 9. ASP.NET Core Versioning

Example:

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);

    options.AssumeDefaultVersionWhenUnspecified = true;

    options.ReportApiVersions = true;
});
```

Controller:

```csharp
[ApiVersion("1.0")]

[Route("api/v{version:apiVersion}/users")]
```

Different controller:

```csharp
[ApiVersion("2.0")]
```

Each version evolves independently.

---

# 10. Versioning DTOs

Avoid changing DTOs in place.

Example:

Version 1:

```csharp
public class UserDtoV1
{
    public string Name { get; set; }
}
```

Version 2:

```csharp
public class UserDtoV2
{
    public string FirstName { get; set; }

    public string LastName { get; set; }
}
```

Each version represents its own API contract.

---

# 11. Common Interview Questions

### Should every small change create a new version?

## Answer

No.

Only breaking changes generally require a new version.

Adding optional data is usually backward compatible.

---

### Should old versions live forever?

No.

They should be deprecated and eventually removed after clients have had enough time to migrate.

---

### Can multiple API versions run simultaneously?

Yes.

Many production systems support multiple versions for months or even years.

---

### Should the database also be versioned?

Usually no.

API versioning concerns external contracts.

The internal data model can continue evolving independently.

---

# Trade-offs

| Strategy | Advantages | Disadvantages |
|----------|------------|---------------|
| URI | Simple, explicit | Changes resource path |
| Header | Clean URLs | Less visible |
| Query String | Easy implementation | Less common |

---

# Staff Engineer Insight

One of the biggest mistakes teams make is exposing Entity Framework entities directly through the API.

Example:

```
Database Entity

↓

API Response
```

This tightly couples your API contract to your database schema.

Instead:

```
Entity

↓

DTO

↓

API Response
```

Your database can evolve without forcing API changes.

Another common mistake is creating a new API version for every small enhancement.

Frequent versioning increases maintenance costs.

Reserve new versions for changes that truly break existing clients.

---

# Summary

- API versioning protects existing clients from breaking changes.
- URI versioning is the most widely used strategy.
- Breaking changes usually require a new version.
- Backward compatibility should be preserved whenever possible.
- Deprecate old versions before removing them.
- Keep API contracts separate from database entities.
- Version APIs deliberately, not automatically.
