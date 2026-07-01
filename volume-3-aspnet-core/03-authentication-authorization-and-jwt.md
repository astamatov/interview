# Authentication, Authorization and JWT

## Overview

Authentication and Authorization are two completely different concepts, yet many developers confuse them during interviews.

A senior engineer should understand how identity is established, how permissions are evaluated and how JWT-based authentication works in modern distributed systems.

---

# 1. Authentication vs Authorization

## Interview Answer

Authentication answers the question:

> **Who are you?**

Authorization answers the question:

> **What are you allowed to do?**

Authentication always happens first.

Without a valid identity there is nothing to authorize.

---

## Example

A user logs into an application.

Authentication:

```
Username

+

Password

↓

User verified
```

Authorization:

```
Role = Admin

↓

Can access /admin
```

---

## Common Interview Question

### Can Authorization happen before Authentication?

## Answer

No.

The system must first know who the user is before it can decide what the user is allowed to do.

---

# 2. What is JWT?

## Interview Answer

JWT (JSON Web Token) is a compact, digitally signed token used to securely transmit user identity and claims between systems.

A JWT is **not encrypted by default**.

Anyone can read its contents.

Its security comes from the digital signature, which prevents tampering.

---

## JWT Structure

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

Contains information about the token.

Example:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

---

## Payload

Contains claims.

Example:

```json
{
  "sub": "123",
  "name": "John",
  "role": "Admin",
  "exp": 1760000000
}
```

Anyone can decode this payload.

Never store:

- passwords
- API keys
- secrets

inside JWTs.

---

## Signature

The server signs the token using a secret key.

```
Header

+

Payload

↓

Hash

↓

Secret Key

↓

Signature
```

If someone modifies the payload, the signature no longer matches and the token becomes invalid.

---

# 3. What are Claims?

## Answer

Claims are pieces of information about the authenticated user.

Examples:

```
UserId

Email

Role

Department

Country
```

Example:

```csharp
User.FindFirst(ClaimTypes.NameIdentifier);

User.FindFirst(ClaimTypes.Role);
```

---

# 4. Roles vs Claims

Roles are coarse-grained permissions.

```
Admin

Manager

Customer
```

Claims are individual pieces of information.

```
Department = Finance

Subscription = Premium

Age = 21
```

Claims provide much more flexibility.

Modern applications often prefer claims over roles.

---

# 5. Authentication Flow

```
User

↓

Login

↓

Server validates credentials

↓

JWT created

↓

JWT returned

↓

Client stores token

↓

Authorization header

↓

Server validates token

↓

User authenticated
```

Every request carries the JWT.

The server does not need to store session state.

---

# 6. Why JWT?

Benefits:

- Stateless
- Scales well
- Ideal for APIs
- Works across multiple services
- No server session required

Perfect for:

- Microservices
- Mobile apps
- SPAs
- Public APIs

---

# 7. Cookie Authentication vs JWT

| Cookie | JWT |
|---------|-----|
| Server-managed session | Stateless |
| Browser-oriented | API-oriented |
| Automatic browser handling | Sent manually in Authorization header |
| Great for MVC apps | Great for REST APIs |

---

## Interview Question

### Which one should I choose?

MVC web application?

→ Cookies.

REST API?

→ JWT.

---

# 8. Where should JWT be stored?

## Interview Answer

Prefer an **HttpOnly Secure Cookie** whenever possible.

Avoid storing JWTs in Local Storage for browser applications because JavaScript can access Local Storage, making tokens vulnerable to XSS attacks.

---

## Common Mistake

```
LocalStorage

↓

XSS

↓

Token stolen
```

Using an HttpOnly cookie prevents JavaScript from reading the token.

---

# 9. Access Token vs Refresh Token

## Access Token

- Short lifetime
- Sent with every request
- Used for authorization

Example:

```
15 minutes
```

---

## Refresh Token

- Long lifetime
- Stored securely
- Used only to obtain a new access token

Example:

```
30 days
```

If the access token expires:

```
Refresh Token

↓

New Access Token
```

The user does not need to log in again.

---

# 10. What happens when JWT expires?

The server rejects the request.

Response:

```
401 Unauthorized
```

The client uses the refresh token to obtain a new access token.

If the refresh token is also invalid, the user must log in again.

---

# 11. OAuth 2.0 vs OpenID Connect

## OAuth 2.0

Authorization framework.

Allows applications to access resources on behalf of the user.

Example:

```
Login with GitHub

↓

Application receives access token
```

---

## OpenID Connect (OIDC)

Built on top of OAuth2.

Adds user authentication and identity.

Example:

```
Login with Google
```

OIDC returns an **ID Token** describing the authenticated user.

---

# 12. Authentication Middleware

Typical configuration:

```csharp
builder.Services
.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
.AddJwtBearer(options =>
{
    options.TokenValidationParameters = ...;
});

app.UseAuthentication();

app.UseAuthorization();
```

Authentication must execute before authorization.

---

# 13. Common Interview Questions

### Is JWT encrypted?

No.

It is signed.

---

### Can anyone read the payload?

Yes.

Never store secrets inside JWTs.

---

### Why use Refresh Tokens?

Because access tokens should remain short-lived.

If stolen, they become useless quickly.

---

### Can JWT be revoked?

Not easily.

Since JWTs are stateless, the server usually does not track them.

Revocation is typically implemented using:

- Token blacklist
- Short expiration times
- Refresh token rotation

---

# Staff Engineer Insight

JWT is not always the correct choice.

Many teams use JWT simply because it is popular.

For traditional server-rendered web applications, secure cookie authentication is often simpler, safer and easier to revoke.

JWT shines in distributed architectures where multiple services need to validate the same identity without sharing server-side session state.

Choose the authentication mechanism based on the architecture, not on trends.

---

# Summary

- Authentication identifies the user.
- Authorization determines permissions.
- JWT is signed, not encrypted.
- Never store sensitive information in JWT payloads.
- Claims provide flexible authorization.
- Use short-lived access tokens.
- Use refresh tokens for long-lived sessions.
- Prefer HttpOnly cookies for browser-based applications.
- OAuth2 handles authorization.
- OpenID Connect adds authentication.
