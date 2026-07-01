# Web Security Attacks and Protection

## Overview

Security is not only about authentication.

Even perfectly authenticated applications can be vulnerable to attacks if input validation, browser security and communication are not designed correctly.

A senior engineer should understand the most common web attacks, how they work and how to prevent them.

The goal is not to memorize attack names but to understand the underlying principles.

---

# 1. SQL Injection

## Interview Answer

SQL Injection occurs when untrusted user input becomes part of a SQL query.

Unsafe example:

```csharp
var sql =
    $"SELECT * FROM Users WHERE Name = '{name}'";
```

If the user enters:

```
' OR 1=1 --
```

the query may return all users.

---

## Protection

Always use parameterized queries.

Correct:

```csharp
var users = await context.Users
    .Where(x => x.Name == name)
    .ToListAsync();
```

or

```csharp
command.Parameters.AddWithValue("@name", name);
```

Never concatenate SQL strings with user input.

---

# Interview Question

### Does Entity Framework prevent SQL Injection?

## Answer

Yes, when using LINQ or properly parameterized raw SQL.

If developers manually concatenate SQL strings, SQL Injection is still possible.

---

# 2. Cross-Site Scripting (XSS)

## Interview Answer

XSS occurs when an attacker injects malicious JavaScript that is later executed in another user's browser.

Example:

```
Comment

↓

<script>alert("Hacked")</script>
```

If rendered without encoding:

```
Browser

↓

JavaScript Executes
```

---

## Types

### Stored XSS

The malicious script is stored in the database.

Every visitor executes it.

---

### Reflected XSS

The script is immediately reflected in the response.

Usually through query parameters.

---

### DOM-based XSS

The vulnerability exists entirely in client-side JavaScript.

---

## Protection

- HTML encode output.
- Validate input.
- Use Content Security Policy (CSP).
- Avoid rendering raw HTML.
- Sanitize user-generated content when HTML is allowed.

---

# 3. Cross-Site Request Forgery (CSRF)

## Interview Answer

CSRF tricks a logged-in user's browser into sending an unwanted request.

Example:

```
User Logged In

↓

Visits Malicious Website

↓

Hidden POST Request

↓

Bank Transfer Executed
```

The browser automatically includes authentication cookies.

---

## Protection

- Anti-forgery tokens
- SameSite cookies
- Validate Origin/Referer where appropriate

ASP.NET Core provides built-in anti-forgery support for cookie-based applications.

---

# Interview Question

### Does JWT authentication eliminate CSRF?

## Answer

Usually yes, if the JWT is sent in the `Authorization` header.

If JWTs are stored in cookies and sent automatically by the browser, CSRF protections may still be required.

---

# 4. Cross-Origin Resource Sharing (CORS)

## Interview Answer

CORS is a browser security mechanism controlling whether one origin can call another.

Example:

```
Frontend

https://app.company.com

↓

API

https://api.company.com
```

The API must explicitly allow the frontend origin.

---

## Protection

Configure only trusted origins.

Bad:

```csharp
AllowAnyOrigin()
```

Good:

```csharp
WithOrigins(
    "https://app.company.com")
```

Never combine unrestricted origins with credentials.

---

# 5. Server-Side Request Forgery (SSRF)

## Interview Answer

SSRF occurs when the server makes requests to attacker-controlled URLs.

Example:

```
User Provides URL

↓

Server Downloads URL
```

An attacker may access internal systems through the server.

---

## Protection

- Allow-list trusted hosts.
- Block internal IP ranges.
- Validate URLs.
- Restrict outbound network access.

---

# 6. Clickjacking

## Interview Answer

Clickjacking tricks users into clicking hidden UI elements.

Example:

A malicious page places your application inside an invisible iframe.

The user thinks they are clicking one button but actually triggers another action.

---

## Protection

Set headers such as:

```
X-Frame-Options: DENY
```

or use an appropriate Content Security Policy with `frame-ancestors`.

---

# 7. HTTPS

## Interview Answer

HTTPS encrypts communication between the client and server using TLS.

Without HTTPS:

```
Username

↓

Password

↓

Readable by attackers
```

With HTTPS:

```
Encrypted Traffic
```

All authentication and sensitive APIs should require HTTPS.

---

# 8. Content Security Policy (CSP)

## Interview Answer

CSP tells browsers which sources of scripts, styles and images are trusted.

Example:

```
Only allow scripts from:

company.com
```

This significantly reduces the impact of many XSS attacks.

---

# 9. Security Headers

Common security headers include:

- Content-Security-Policy
- X-Content-Type-Options
- Referrer-Policy
- Permissions-Policy
- Strict-Transport-Security (HSTS)

These headers provide additional browser-side protection.

---

# 10. Rate Limiting

## Interview Answer

Rate Limiting restricts how many requests a client can make within a period.

Example:

```
100 Requests

↓

1 Minute
```

Requests beyond the limit receive:

```
429 Too Many Requests
```

Rate limiting helps mitigate abuse, brute-force attacks and accidental overload.

---

# 11. Input Validation

Never trust client input.

Validate:

- Length
- Format
- Allowed values
- Numeric ranges

Client-side validation improves user experience.

Server-side validation enforces security.

Always perform server-side validation.

---

# 12. Common Interview Questions

### Is input validation enough to prevent SQL Injection?

## Answer

No.

Validation helps, but parameterized queries are the primary defense.

Use both.

---

### Can HTTPS prevent XSS?

No.

HTTPS protects data in transit.

XSS executes malicious code in the browser after the page is delivered.

---

### Is CORS a security boundary?

Not by itself.

CORS is enforced by browsers.

It should not replace authentication or authorization.

---

### Why shouldn't user input be trusted?

Because every client can be modified.

Only server-side validation can be considered authoritative.

---

# Trade-offs

| Mechanism | Protects Against | Does Not Protect Against |
|-----------|------------------|--------------------------|
| HTTPS | Eavesdropping | XSS |
| CSP | Many XSS attacks | SQL Injection |
| Anti-forgery Tokens | CSRF | XSS |
| Parameterized Queries | SQL Injection | CSRF |
| CORS | Unauthorized browser origins | Authentication bypass |

---

# Staff Engineer Insight

One of the biggest mistakes teams make is assuming security is a single feature.

Security is a collection of layers.

A secure application typically includes:

- HTTPS
- Authentication
- Authorization
- Input validation
- Output encoding
- Parameterized SQL
- Security headers
- Rate limiting
- Logging and monitoring

No single mechanism is sufficient.

Another common mistake is trusting frontend validation.

Frontend validation improves usability.

Backend validation protects the system.

Always assume requests can be crafted manually.

---

# Summary

- Use parameterized queries to prevent SQL Injection.
- Encode output and use CSP to reduce XSS risk.
- Protect cookie-based applications from CSRF.
- Configure CORS carefully.
- Validate all server-side input.
- Use HTTPS everywhere.
- Enable security headers.
- Apply rate limiting where appropriate.
- Build multiple layers of defense instead of relying on a single mechanism.
