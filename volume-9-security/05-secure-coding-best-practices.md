# Secure Coding Best Practices

## Overview

Secure software is not created by adding security at the end of a project.

It is created by making secure decisions throughout the entire development lifecycle.

A senior engineer should write code that is secure by default rather than relying on penetration testing to discover vulnerabilities later.

---

# 1. Never Trust User Input

## Interview Answer

Every piece of data coming from outside the application should be considered untrusted.

Sources include:

- HTTP requests
- Query parameters
- Headers
- Cookies
- Uploaded files
- Message queues
- External APIs

Always validate input on the server.

Client-side validation improves user experience but does not provide security.

---

# 2. Validate Input

Validation should verify:

- Required fields
- Length
- Numeric ranges
- Allowed characters
- Allowed values
- File size
- File type

Example:

```csharp
public class CreateUserValidator
{
    RuleFor(x => x.Email)
        .EmailAddress();

    RuleFor(x => x.Name)
        .MaximumLength(100);
}
```

Validation belongs close to the application boundary.

---

# 3. Encode Output

User input rendered in HTML should be encoded.

Example:

Bad:

```html
<div>@Html.Raw(comment)</div>
```

Good:

```html
<div>@comment</div>
```

Output encoding is one of the primary defenses against XSS.

---

# 4. Principle of Least Privilege

Applications should operate with the minimum permissions required.

Examples:

Database:

- Read only when writes are unnecessary.

Azure:

- Grant access only to required resources.

File System:

- Avoid administrator permissions.

Compromised applications should have limited impact.

---

# 5. Secure Secret Management

Never store secrets in:

- Source code
- Git repositories
- Docker images

Prefer:

- Azure Key Vault
- Environment variables
- Managed Identity

Rotate secrets regularly.

---

# 6. Password Storage

Passwords should never be stored in plain text.

Use strong password hashing algorithms.

Recommended:

- Argon2
- bcrypt
- PBKDF2

Each password should use a unique random salt.

---

# 7. HTTPS Everywhere

All authentication and sensitive communication should use HTTPS.

Redirect HTTP traffic to HTTPS.

Enable HSTS in production.

Avoid transmitting credentials over unencrypted connections.

---

# 8. Logging

Logs should contain enough information for troubleshooting without exposing sensitive data.

Good:

```
OrderId

UserId

TraceId

ErrorCode
```

Avoid logging:

- Passwords
- JWT tokens
- Credit card numbers
- Connection strings
- API keys

Treat logs as sensitive data.

---

# 9. File Upload Security

Never trust uploaded files.

Validate:

- Extension
- MIME type
- Size
- Virus scan (where applicable)

Store uploads outside the web root or in object storage such as Azure Blob Storage.

Generate random file names instead of using user-supplied names.

---

# 10. Dependency Management

Third-party packages are part of your attack surface.

Best practices:

- Remove unused packages.
- Update dependencies regularly.
- Monitor security advisories.
- Review package sources.

Security updates should be part of normal maintenance.

---

# 11. Secure Error Handling

Users should receive friendly error messages.

Developers should receive detailed logs.

Bad:

```
SQL Exception

Connection String

Stack Trace
```

Good:

```
Unexpected error occurred.

Reference ID: 5A72B9
```

Detailed diagnostics belong in logs, not API responses.

---

# 12. Secure Defaults

Applications should start in the safest configuration.

Examples:

- HTTPS enabled
- Authentication required
- Debug disabled
- Security headers enabled
- Least privilege access

Avoid relying on developers to remember every configuration manually.

---

# 13. Defense in Depth

Security should use multiple independent layers.

Example:

```
HTTPS

↓

Authentication

↓

Authorization

↓

Input Validation

↓

Parameterized SQL

↓

Output Encoding

↓

Logging

↓

Monitoring
```

If one control fails, others continue protecting the system.

---

# 14. Security During Code Reviews

Reviewers should verify:

- Authentication
- Authorization
- Input validation
- Secret handling
- Logging
- SQL queries
- Error handling
- Dependency usage

Security should be a standard review topic.

---

# Common Interview Questions

### Is frontend validation enough?

## Answer

No.

Frontend validation improves usability.

Server-side validation enforces security.

---

### Should API responses include stack traces?

No.

Detailed exceptions may reveal internal implementation details.

Return generic errors to clients and log the details internally.

---

### Should secrets be encrypted in appsettings.json?

Prefer not storing them there at all.

Use Azure Key Vault or Managed Identity whenever possible.

---

### Why use the Principle of Least Privilege?

It limits the damage if an account or service is compromised.

---

# Secure Development Checklist

Before deployment:

- HTTPS enforced
- Authentication configured
- Authorization reviewed
- Input validation complete
- Output encoding verified
- Parameterized queries used
- Secrets removed from source code
- Dependencies updated
- Logging configured
- Security headers enabled
- Rate limiting considered
- Monitoring enabled

---

# Trade-offs

| Practice | Benefit | Cost |
|----------|---------|------|
| Least Privilege | Reduced blast radius | More permission management |
| Key Vault | Secure secret storage | Additional infrastructure |
| Strict Validation | Better security | More validation code |
| Rich Logging | Easier troubleshooting | Storage and privacy considerations |

---

# Staff Engineer Insight

One of the biggest mistakes teams make is assuming secure code equals secure software.

Security also depends on:

- Infrastructure
- CI/CD pipelines
- Cloud configuration
- Dependency management
- Operational monitoring
- Incident response

Writing secure code is essential, but it is only one part of a secure system.

Another common mistake is introducing security controls that developers bypass because they are too cumbersome.

Good security should support developers while reducing risk, not encourage workarounds.

---

# Summary

- Never trust external input.
- Validate on the server.
- Encode output before rendering.
- Store secrets securely.
- Hash passwords with modern algorithms.
- Use HTTPS everywhere.
- Log responsibly without exposing sensitive data.
- Follow the Principle of Least Privilege.
- Keep dependencies updated.
- Build security into every stage of development.
