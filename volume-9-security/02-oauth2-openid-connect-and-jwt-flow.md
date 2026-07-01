# OAuth 2.0, OpenID Connect and JWT Flow

## Overview

Modern applications rarely authenticate users directly.

Instead, they rely on Identity Providers such as:

- Microsoft Entra ID
- Auth0
- Okta
- Keycloak
- Google Identity
- Azure AD B2C

These providers implement industry standards like OAuth 2.0 and OpenID Connect.

Understanding these protocols is essential for designing secure enterprise applications.

---

# 1. OAuth 2.0

## Interview Answer

OAuth 2.0 is an authorization framework that allows an application to obtain limited access to resources on behalf of a user.

The application never needs to know the user's password.

Instead, the Identity Provider authenticates the user and issues tokens.

---

# Example

```
User

↓

Application

↓

Identity Provider

↓

Consent

↓

Access Token

↓

API
```

The application receives permission, not the user's credentials.

---

# 2. OAuth Roles

OAuth defines several participants.

---

## Resource Owner

Usually the user.

Owns the protected data.

---

## Client

The application requesting access.

Examples:

- Mobile App
- Web App
- SPA
- Backend Service

---

## Authorization Server

Authenticates users and issues tokens.

Examples:

- Microsoft Entra ID
- Auth0
- Keycloak

---

## Resource Server

The protected API.

It validates access tokens before returning data.

---

# 3. Access Token

## Interview Answer

An Access Token proves that the client has permission to call an API.

Typical lifetime:

```
5–60 minutes
```

The API validates:

- Signature
- Expiration
- Audience
- Issuer

If validation succeeds:

```
Request Accepted
```

---

# 4. Refresh Token

A Refresh Token is used to obtain a new Access Token without asking the user to log in again.

Flow:

```
Access Token Expires

↓

Refresh Token

↓

Identity Provider

↓

New Access Token
```

Refresh Tokens typically have a much longer lifetime.

---

# Interview Question

### Should APIs accept Refresh Tokens?

## Answer

No.

Refresh Tokens are exchanged only with the Authorization Server.

APIs should accept Access Tokens only.

---

# 5. OpenID Connect

## Interview Answer

OpenID Connect (OIDC) is an identity layer built on top of OAuth 2.0.

OAuth answers:

> "Can this application access the resource?"

OIDC answers:

> "Who is the user?"

---

# ID Token

OIDC introduces the **ID Token**.

Unlike an Access Token, the ID Token contains identity information.

Typical claims:

```
User ID

Email

Name

Preferred Username
```

Applications use the ID Token to identify the authenticated user.

---

# 6. OAuth vs OpenID Connect

| OAuth 2.0 | OpenID Connect |
|-----------|----------------|
| Authorization | Authentication + Authorization |
| Access Token | Access Token + ID Token |
| API access | User identity |
| Resource access | Login |

---

# 7. Authorization Code Flow

## Interview Answer

The Authorization Code Flow is the recommended flow for:

- Web Applications
- Server-side Applications
- Mobile Applications (with PKCE)

Flow:

```
User

↓

Application

↓

Authorization Server

↓

Authorization Code

↓

Application

↓

Access Token

↓

API
```

The Access Token is never exposed in the browser URL.

---

# 8. PKCE

## Interview Answer

PKCE (Proof Key for Code Exchange) protects the Authorization Code Flow against interception attacks.

It is strongly recommended for:

- Mobile Apps
- Single Page Applications
- Public Clients

Even if an attacker steals the authorization code, it cannot be exchanged without the original code verifier.

---

# 9. Token Validation

Before accepting a JWT, an API should validate:

- Signature
- Issuer
- Audience
- Expiration
- Signing Key

Failure to validate any of these increases security risks.

---

# 10. Token Expiration

Access Tokens should be short-lived.

Example:

```
15 minutes
```

If compromised, the attack window remains limited.

Refresh Tokens allow obtaining new Access Tokens without forcing frequent logins.

---

# 11. Common Interview Questions

### Should JWTs be encrypted?

## Answer

Usually not.

JWTs are typically signed, not encrypted.

Signing guarantees integrity.

Encryption is only needed when the payload itself contains sensitive information.

---

### Can a JWT be revoked?

Yes, but not easily.

Since JWTs are stateless, common approaches include:

- Short expiration times
- Token revocation lists
- Refresh token rotation

---

### Why are short-lived Access Tokens recommended?

They reduce the impact of token theft.

A compromised token becomes unusable after a short period.

---

### Should APIs trust any JWT?

No.

Every JWT must be validated.

Never trust a token simply because it "looks valid."

---

# 12. Common Security Mistakes

Examples:

- Accepting expired tokens.
- Ignoring issuer validation.
- Ignoring audience validation.
- Logging JWTs.
- Using HTTP instead of HTTPS.
- Long-lived access tokens.
- Weak signing keys.

---

# Trade-offs

| Concept | Advantage | Limitation |
|----------|-----------|------------|
| Access Token | Fast API authorization | Expires quickly |
| Refresh Token | Better user experience | Must be stored securely |
| OAuth | Standard authorization | Does not identify users |
| OpenID Connect | Standard authentication | Slightly more complex |

---

# Staff Engineer Insight

A common misconception is treating JWTs as session storage.

JWTs should contain only the information required by the API.

Avoid placing:

- Large permission lists
- User preferences
- Shopping carts
- Personal data

inside tokens.

Another common mistake is implementing custom authentication protocols.

Whenever possible, rely on established standards such as OAuth 2.0 and OpenID Connect.

They have been extensively reviewed, tested and supported by mature libraries.

---

# Summary

- OAuth 2.0 is an authorization framework.
- OpenID Connect adds authentication.
- Access Tokens authorize API access.
- Refresh Tokens obtain new Access Tokens.
- ID Tokens identify authenticated users.
- Validate every JWT before accepting it.
- Use Authorization Code Flow for server-side applications.
- Use PKCE for public clients.
- Keep Access Tokens short-lived.
- Prefer industry standards over custom authentication solutions.
