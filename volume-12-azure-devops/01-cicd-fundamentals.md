# CI/CD Fundamentals

## Overview

Modern software teams deliver changes frequently and safely through automated pipelines.

Continuous Integration (CI) and Continuous Delivery/Deployment (CD) reduce manual work, improve software quality and shorten delivery cycles.

A senior engineer should understand not only what CI/CD is, but also how to design reliable deployment pipelines.

---

# 1. What is Continuous Integration (CI)?

## Interview Answer

Continuous Integration is the practice of frequently merging code changes into the main branch.

Every change automatically triggers:

- Build
- Unit tests
- Static analysis
- Security checks

Goal:

Detect problems early rather than days later.

---

Typical CI flow:

```
Developer

↓

Git Push

↓

Pipeline Starts

↓

Restore Packages

↓

Build

↓

Run Tests

↓

Code Analysis

↓

Artifact Produced
```

If any step fails, the pipeline stops.

---

# 2. What is Continuous Delivery?

## Interview Answer

Continuous Delivery means every successful build is deployable.

Deployment to production still requires manual approval.

```
Build

↓

Tests

↓

Deploy Staging

↓

Manual Approval

↓

Production
```

This is common in regulated industries.

---

# 3. What is Continuous Deployment?

Continuous Deployment goes one step further.

Every successful pipeline automatically reaches production.

```
Commit

↓

Pipeline

↓

Tests

↓

Production
```

No manual approval.

Requires extremely high confidence in automated testing.

---

# Interview Question

### Continuous Delivery vs Continuous Deployment?

## Answer

Continuous Delivery requires a manual approval before production.

Continuous Deployment automatically deploys every successful change.

---

# 4. Why CI/CD Matters

Without automation:

```
Developer

↓

Manual Build

↓

Manual Tests

↓

Manual Deployment

↓

Human Errors
```

With CI/CD:

```
Developer

↓

Pipeline

↓

Automatic Validation

↓

Reliable Deployment
```

Automation improves consistency and repeatability.

---

# 5. Typical Pipeline Stages

A production pipeline commonly contains:

```
Restore

↓

Build

↓

Unit Tests

↓

Code Coverage

↓

Static Analysis

↓

Security Scan

↓

Package Artifact

↓

Deploy Dev

↓

Integration Tests

↓

Deploy Staging

↓

Approval

↓

Deploy Production
```

Each stage should fail fast if problems are detected.

---

# 6. Build Artifacts

## Interview Answer

An artifact is the output produced by the build.

Examples:

- DLLs
- Docker images
- ZIP packages
- NuGet packages

The same artifact should move through every environment.

Avoid rebuilding separately for staging and production.

---

# 7. Pipeline Principles

A good pipeline should be:

- Fast
- Repeatable
- Automated
- Deterministic
- Versioned

Running the same commit twice should produce the same artifact.

---

# 8. Shift Left

Shift Left means moving quality checks earlier.

Example:

Instead of:

```
Deploy

↓

Find Bug
```

Use:

```
Commit

↓

Tests

↓

Code Analysis

↓

Security Scan

↓

Deploy
```

Problems become cheaper to fix.

---

# 9. Pipeline Failures

Pipelines should fail immediately when:

- Build fails
- Unit tests fail
- Security scan fails
- Static analysis fails
- Required quality gates are not met

Never deploy a failing build.

---

# 10. Common Interview Questions

### Should every commit trigger a pipeline?

## Answer

Usually yes for active development branches.

Fast feedback helps developers identify problems immediately.

---

### Why deploy the same artifact everywhere?

Rebuilding may introduce differences.

Promoting the same tested artifact increases confidence and traceability.

---

### Why are pipelines versioned?

Pipeline definitions evolve with the application.

Keeping them in source control ensures changes are reviewed and reproducible.

---

### Should manual testing disappear completely?

No.

Automation handles repetitive validation.

Exploratory testing, usability testing and business validation may still require manual effort.

---

# Common Pipeline Mistakes

- Long-running pipelines
- Manual deployment steps
- Environment-specific builds
- Missing automated tests
- Secrets stored in source control
- No rollback strategy

---

# Trade-offs

| Practice | Benefit | Trade-off |
|----------|---------|-----------|
| CI | Fast feedback | Increased infrastructure usage |
| Continuous Delivery | Safe releases | Manual approval required |
| Continuous Deployment | Maximum automation | Requires high test confidence |
| Automated Testing | Better quality | Maintenance effort |
| Security Scanning | Reduced risk | Longer pipeline duration |

---

# Staff Engineer Insight

One of the biggest misconceptions is that CI/CD is only about automation.

The real goal is **reducing deployment risk**.

A pipeline should answer questions such as:

- Does the application build successfully?
- Are tests passing?
- Is code quality acceptable?
- Are known vulnerabilities introduced?
- Can this version be safely deployed?

Another common mistake is creating slow pipelines.

Developers should receive feedback within minutes whenever possible.

A pipeline that takes an hour discourages frequent commits and reduces the value of Continuous Integration.

---

# Summary

- Continuous Integration validates every code change automatically.
- Continuous Delivery produces software that is always ready for deployment.
- Continuous Deployment automatically releases every successful build.
- Pipelines should build once and deploy the same artifact everywhere.
- Shift quality checks earlier in the development lifecycle.
- Pipelines should fail fast when quality gates are not met.
- Automation improves reliability but does not replace thoughtful engineering practices.
