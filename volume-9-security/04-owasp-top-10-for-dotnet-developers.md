# OWASP Top 10 for .NET Developers

## Overview

The OWASP Top 10 is a list of the most critical web application security risks.

It is not a checklist of every possible vulnerability.

Instead, it highlights the attacks developers are most likely to encounter in real-world applications.

A senior engineer should understand:

- What each category means.
- How vulnerabilities occur.
- How to reduce the risk during development.

---

# 1. Broken Access Control

## Interview Answer

Broken Access Control occurs when users can access resources they should not be allowed to access.

Example:

```
GET /api/orders/100
```

A user changes:

```
100

↓

101
```

If Order 101 belongs to another customer and the API returns it, access control is broken.

---

## Protection

Always verify ownership.

Example:

```csharp
var order = await repository
    .GetById(orderId);

if(order.CustomerId != currentUserId)
    return Forbid();
```

Never rely on IDs being difficult to guess.

---

# 2. Cryptographic Failures

## Interview Answer

Sensitive information must be protected both in transit and at rest.

Examples:

- Passwords
- Credit card numbers
- API keys
- Personal information

---

## Protection

- HTTPS everywhere
- Strong password hashing
- Encryption where required
- Secure key management
- Azure Key Vault for secrets

Never invent your own cryptographic algorithms.

---

# 3. Injection

Injection attacks occur when untrusted input becomes part of a command.

Examples:

- SQL Injection
- LDAP Injection
- OS Command Injection

---

## Protection

- Parameterized SQL
- ORM frameworks
- Input validation
- Least privilege database accounts

Never concatenate user input into commands.

---

# 4. Insecure Design

## Interview Answer

Some vulnerabilities exist because of architectural decisions rather than coding mistakes.

Examples:

- Missing rate limiting
- Weak password policies
- No MFA
- Unlimited login attempts

Secure design starts before implementation.

---

# 5. Security Misconfiguration

Examples:

- Debug mode enabled in production
- Default passwords
- Open storage accounts
- Excessive permissions
- Missing security headers

---

## Protection

- Secure defaults
- Infrastructure reviews
- Environment-specific configuration
- Regular security audits

---

# 6. Vulnerable Components

Applications often depend on external libraries.

Example:

```
NuGet Packages

↓

Known Vulnerability
```

---

## Protection

- Keep dependencies updated.
- Monitor security advisories.
- Remove unused packages.
- Scan dependencies regularly.

---

# 7. Authentication Failures

Examples:

- Weak passwords
- Missing MFA
- Long-lived sessions
- Poor token management

---

## Protection

- Strong password hashing
- MFA
- Short-lived access tokens
- Refresh token rotation
- Secure session management

---

# 8. Software and Data Integrity Failures

Examples:

- Downloading unsigned software
- Running unverified scripts
- Insecure CI/CD pipelines

---

## Protection

- Verify package integrity.
- Use signed artifacts where appropriate.
- Protect build pipelines.
- Review deployment permissions.

---

# 9. Logging and Monitoring Failures

Without logging:

```
Attack

↓

No Detection

↓

Months Pass

↓

Data Loss
```

Applications should log:

- Authentication failures
- Authorization failures
- Administrative actions
- Unexpected exceptions

Avoid logging sensitive information.

---

# 10. Server-Side Request Forgery (SSRF)

## Interview Answer

SSRF tricks the server into making requests to unintended destinations.

Example:

```
User

↓

Provides URL

↓

Server Downloads Resource

↓

Internal Network Exposed
```

---

## Protection

- Allow-list external hosts.
- Validate URLs.
- Block internal IP ranges.
- Restrict outbound network access.

---

# Common Interview Questions

### Should passwords be encrypted?

## Answer

No.

Passwords should be hashed using a strong password hashing algorithm such as Argon2, bcrypt or PBKDF2.

Hashing is one-way.

Encryption is reversible.

---

### Can HTTPS replace input validation?

No.

HTTPS protects communication.

Input validation protects the application.

They solve different problems.

---

### Is authentication enough to secure an API?

No.

Authentication identifies the user.

Authorization determines what the user is allowed to access.

Input validation, logging, rate limiting and secure configuration are also essential.

---

# Secure Development Checklist

Before releasing an API, verify:

- HTTPS is enforced.
- Authentication is enabled.
- Authorization is implemented.
- SQL queries are parameterized.
- Sensitive data is not logged.
- Secrets are stored securely.
- Security headers are configured.
- Rate limiting is enabled where appropriate.
- Dependencies are up to date.
- Monitoring and alerting are configured.

---

# Trade-offs

| Practice | Benefit | Cost |
|----------|---------|------|
| MFA | Stronger security | Slightly more user friction |
| Short-lived Tokens | Reduced attack window | More frequent token refresh |
| Parameterized Queries | Prevents injection | None in most applications |
| Dependency Updates | Reduced vulnerability risk | Ongoing maintenance |

---

# Staff Engineer Insight

Many teams treat security as a final testing phase.

High-performing teams build security into every stage of development.

Examples:

- Security-focused code reviews.
- Dependency scanning in CI/CD.
- Secret scanning before commits.
- Automated security tests.
- Threat modeling for new features.

Security should be continuous, not a one-time checklist.

---

# Summary

- Broken Access Control is one of the most common and severe vulnerabilities.
- Use strong
