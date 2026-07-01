# Azure DevOps Pipelines

## Overview

Azure DevOps Pipelines automate the process of building, testing and deploying applications.

Pipelines are defined as code using YAML, allowing them to be versioned, reviewed and maintained alongside the application.

A senior engineer should understand how pipelines are structured and how to design reliable deployment workflows.

---

# 1. What is a Pipeline?

## Interview Answer

A pipeline is an automated workflow that executes a series of steps whenever specific events occur.

Typical triggers:

- Git push
- Pull Request
- Scheduled execution
- Manual execution

Example:

```
Git Push

↓

Pipeline

↓

Build

↓

Tests

↓

Deploy
```

---

# 2. YAML Pipelines

Modern Azure DevOps pipelines are defined using YAML.

Example:

```yaml
trigger:
  - main

pool:
  vmImage: ubuntu-latest

steps:
- script: dotnet build
```

Advantages:

- Version controlled
- Code reviewed
- Reproducible
- Easy to reuse

---

# 3. Pipeline Structure

Typical hierarchy:

```
Pipeline

↓

Stages

↓

Jobs

↓

Steps
```

Example:

```
Build Stage

↓

Build Job

↓

Restore

Build

Test
```

---

# 4. Stages

Stages divide the pipeline into logical phases.

Example:

```
Build

↓

Test

↓

Deploy Dev

↓

Deploy Staging

↓

Deploy Production
```

Stages often correspond to environments.

---

# 5. Jobs

A stage contains one or more jobs.

Jobs usually execute on different agents.

Example:

```
Build Stage

↓

Job 1

Compile

---------------

Job 2

Run Tests
```

Jobs can execute sequentially or in parallel.

---

# 6. Steps

Steps are the smallest execution units.

Examples:

```yaml
- script: dotnet restore

- script: dotnet build

- script: dotnet test
```

Each step performs a single action.

---

# 7. Agents

## Interview Answer

An agent is the machine that executes the pipeline.

Two common types:

### Microsoft-hosted

- Managed by Microsoft
- Easy to use
- Clean environment for every run

---

### Self-hosted

- Managed by your organization
- Custom software available
- Better for specialized workloads
- Faster if dependencies are already installed

---

# Interview Question

### When would you use a self-hosted agent?

## Answer

When builds require custom software, access to private networks or faster execution through locally cached dependencies.

---

# 8. Variables

Variables avoid hardcoding values.

Example:

```yaml
variables:

Configuration: Release
```

Usage:

```yaml
dotnet build

--configuration

$(Configuration)
```

Environment-specific values should be injected through variables rather than modifying pipeline logic.

---

# 9. Templates

Repeated YAML should be extracted into templates.

Instead of duplicating:

```
Build

↓

Test

↓

Publish
```

Use:

```yaml
template: build.yml
```

Benefits:

- Reuse
- Easier maintenance
- Consistency

---

# 10. Pipeline Artifacts

Artifacts are outputs from the build.

Examples:

- DLLs
- ZIP files
- Docker images
- NuGet packages

The artifact produced during CI should be the same one deployed to every environment.

---

# 11. Environments

Azure DevOps environments represent deployment targets.

Examples:

```
Development

↓

QA

↓

Staging

↓

Production
```

Environments support:

- Deployment history
- Approvals
- Audit trails

---

# 12. Approvals

Sensitive environments often require approval.

Example:

```
Deploy Staging

↓

Approval

↓

Deploy Production
```

Approvals reduce the risk of accidental production deployments.

---

# 13. Pipeline Caching

Restoring dependencies every build is expensive.

Pipeline caching stores reusable content.

Examples:

- NuGet packages
- npm packages
- Maven dependencies

Benefits:

- Faster builds
- Reduced network traffic

---

# 14. Common Interview Questions

### Why use YAML instead of Classic Pipelines?

## Answer

YAML pipelines are stored in source control.

They support versioning, code reviews and pull request validation.

---

### Why build once?

Building separately for each environment can introduce inconsistencies.

The same tested artifact should move through every environment.

---

### Should production deployments always require approval?

Not always.

Highly regulated industries often require approvals.

High-performing teams with strong automated testing may choose fully automated deployments.

---

### Why separate stages?

Stages isolate responsibilities and make failures easier to diagnose.

They also support approvals and environment-specific behavior.

---

# Pipeline Best Practices

- Keep pipelines under source control.
- Build once, deploy many.
- Fail fast.
- Cache dependencies.
- Keep secrets outside YAML.
- Use templates for shared logic.
- Keep pipelines short and deterministic.

---

# Trade-offs

| Practice | Benefit | Trade-off |
|----------|---------|-----------|
| YAML Pipelines | Versioned and reusable | Learning curve |
| Templates | Less duplication | Additional abstraction |
| Self-hosted Agents | Faster and customizable | Operational maintenance |
| Microsoft-hosted Agents | Zero maintenance | Fresh environment every run |
| Pipeline Cache | Faster builds | Cache management |

---

# Staff Engineer Insight

A common anti-pattern is creating one enormous pipeline that builds, tests and deploys every application in a monorepo regardless of what changed.

Modern pipelines should execute only the work required for the current change.

Another common mistake is embedding business logic inside the pipeline.

Pipelines should orchestrate builds and deployments, not replace application logic.

Finally, remember that a pipeline is production code.

Review it, test it and version it with the same discipline as the application itself.

---

# Summary

- Azure DevOps Pipelines automate build, test and deployment workflows.
- YAML pipelines are version-controlled and reproducible.
- Pipelines consist of stages, jobs and steps.
- Agents execute pipeline tasks.
- Variables and templates improve maintainability.
- Build artifacts should be promoted through all environments.
- Pipeline caching reduces build time.
- Environments and approvals improve deployment governance.
- A good pipeline is fast, deterministic and easy to maintain.
