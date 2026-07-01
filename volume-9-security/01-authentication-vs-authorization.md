# Authentication vs Authorization

## Overview

Security is one of the most important aspects of modern applications.

Every API should answer two questions:

1. Who is the user?
2. What is the user allowed to do?

Authentication answers the first question.

Authorization answers the second.

Although these concepts are closely related, they solve different problems.

---

# 1. Authentication

## Interview Answer

Authentication is the process of verifying a user's identity.

The application answers:

> "Who are you?"

Common authentication methods:

- Username and password
- Multi-Factor Authentication (MFA)
- OAuth 2.0
- OpenID Connect
- Windows Authentication
- Certificate Authentication

Example:

```
Username

↓

Password

↓

Identity Provider

↓

Authenticated User
```

Authentication proves identity.

---

# 2. Authorization

## Interview Answer

Authorization determines what an authenticated user is allowed to do.

The application answers:

> "What are you allowed to access?"

Example:

```
Authenticated User

↓

Can View Orders

Can Edit Orders

Cannot Delete Orders
```

Authorization happens after authentication.

---

# Interview Question

### Can authorization happen without authentication?

## Answer

Normally no.

The system must know who the user is before deciding what they are allowed to do.

Anonymous endpoints are the main exception.

---

# 3. JWT (JSON Web Token)

## Interview Answer

JWT is a compact, signed token used to transmit user information between systems.

Typical flow:

```
User Logs In

↓

Identity Provider

↓

JWT Returned

↓

Client Stores Token

↓

Client Sends Token

↓

API Validates Token
```

The API does not need to store session state.

This makes JWT ideal for stateless APIs.

---

# 4. JWT Structure

A JWT consists of three parts.

```
Header

.

Payload

.

Signature
```

Example:

```
xxxxx.yyyyy.zzzzz
```

---

## Header

Contains:

- Token type
- Signing algorithm

---

## Payload

Contains claims.

Examples:

```
UserId

Email

Role

Permissions
```

---

## Signature

Protects the token from tampering.

If the payload changes, the signature becomes invalid.

---

# 5. Claims

## Interview Answer

Claims are pieces of information about the authenticated user.

Examples:

```
UserId

Email

Department

Role

Country
```

Claims are used during authorization.

---

# 6. Roles vs Policies

## Roles

Simple authorization.

Example:

```
Admin

Manager

User
```

Controller:

```csharp
[Authorize(Roles = "Admin")]
```

---

## Policies

More flexible.

Example:

```
Minimum Age

Department

Country

Subscription Level
```

Example:

```csharp
[Authorize(Policy = "FinanceOnly")]
```

Policies are preferred for complex business rules.

---

# 7. OAuth 2.0

## Interview Answer

OAuth 2.0 is an authorization framework.

It allows one application to access resources on behalf of a user without exposing the user's password.

Example:

```
Application

↓

Microsoft Login

↓

Consent

↓

Access Token
```

Notice:

OAuth does **not** authenticate the user.

It delegates authorization.

---

# Interview Question

### Does OAuth authenticate users?

## Answer

Not by itself.

OAuth is primarily an authorization framework.

Authentication is commonly provided by OpenID Connect.

---

# 8. OpenID Connect (OIDC)

OpenID Connect builds on OAuth 2.0.

It adds identity information.

Result:

```
OAuth

↓

Authorization

+

OpenID Connect

↓

Authentication
```

This is why Microsoft Entra ID, Auth0 and many identity providers use OpenID Connect.

---

# 9. Access Token vs Refresh Token

## Access Token

- Short lifetime
- Sent with API requests
- Used for authorization

---

## Refresh Token

- Longer lifetime
- Used to obtain a new access token
- Should be protected carefully

Typical flow:

```
Access Token Expires

↓

Client Sends Refresh Token

↓

Identity Provider

↓

New Access Token
```

Users remain signed in without entering credentials again.

---

# 10. ASP.NET Core Authentication

Example:

```csharp
builder.Services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(...);
```

Protect endpoint:

```csharp
[Authorize]
```

Anonymous endpoint:

```csharp
[AllowAnonymous]
```

---

# 11. Common Security Mistakes

Examples:

- Storing passwords in plain text.
- Logging JWT tokens.
- Hardcoding secrets.
- Long-lived access tokens.
- Missing HTTPS.
- Missing token validation.
- Weak signing keys.

---

# 12. Common Interview Questions

### Should JWT contain sensitive information?

## Answer

No.

JWT payloads are encoded, not encrypted.

Anyone possessing the token can read its payload.

Store only information that is safe to expose.

---

### Should passwords be stored using encryption?

No.

Passwords should be hashed using a strong password hashing algorithm.

Hashing is one-way.

Encryption is reversible.

---

### Why are JWTs popular?

Because APIs remain stateless.

Servers do not maintain session state for each client.

---

### Should roles contain all authorization logic?

No.

Roles are useful for coarse-grained authorization.

Policies provide greater flexibility for complex business rules.

---

# Trade-offs

| Approach | Advantage | Limitation |
|----------|-----------|------------|
| JWT | Stateless | Harder token revocation |
| Cookie Authentication | Simple for web apps | Less suitable for APIs |
| Roles | Easy | Limited flexibility |
| Policies | Powerful | More configuration |

---

# Staff Engineer Insight

One of the biggest mistakes developers make is putting too much information inside JWTs.

Example:

```
User

↓

Permissions

↓

Address

↓

Phone

↓

Preferences

↓

Entire Profile
```

JWTs should remain small.

Every request carries the token.

Larger tokens increase network overhead.

Another common mistake is giving access tokens very long expiration times.

Short-lived access tokens combined with refresh tokens provide a much better security model.

Always assume that tokens can be leaked.

Design the system to minimize the impact.

---

# Summary

- Authentication verifies identity.
- Authorization determines permissions.
- JWT enables stateless authentication.
- Claims describe the authenticated user.
- Roles provide simple authorization.
- Policies support more complex authorization rules.
- OAuth 2.0 is an authorization framework.
- OpenID Connect adds authentication.
- Access tokens should be short-lived.
- Never store passwords in plain text.
- Prefer hashed passwords and HTTPS for secure applications.
