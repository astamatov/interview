# GitHub Actions

## Overview

GitHub Actions is GitHub's built-in CI/CD platform.

It automates:

- Build
- Testing
- Code analysis
- Packaging
- Deployment

Workflows are defined as YAML files stored inside the repository.

Like Azure DevOps Pipelines, GitHub Actions follows the Infrastructure as Code approach.

---

# 1. What is GitHub Actions?

## Interview Answer

GitHub Actions is an event-driven automation platform integrated into GitHub.

A workflow runs when a configured event occurs.

Examples:

- Push
- Pull Request
- Release
- Manual trigger
- Scheduled execution

Typical flow:

```
Developer

↓

Git Push

↓

Workflow

↓

Build

↓

Tests

↓

Deploy
```

---

# 2. Workflow File

Workflows are stored inside:

```
.github/workflows/
```

Example:

```yaml
name: Build

on:
  push:
    branches:
      - main
```

Every repository may contain multiple workflows.

---

# 3. Events

Examples:

```yaml
on:
    push:
```

```yaml
on:
    pull_request:
```

```yaml
on:
    workflow_dispatch:
```

```yaml
on:
    schedule:
```

Multiple events can trigger the same workflow.

---

# 4. Jobs

A workflow contains one or more jobs.

Example:

```
Workflow

↓

Build Job

↓

Test Job

↓

Deploy Job
```

Jobs execute independently unless dependencies are defined.

---

# 5. Steps

Each job contains steps.

Example:

```yaml
steps:

- uses: actions/checkout@v4

- uses: actions/setup-dotnet@v4

- run: dotnet restore

- run: dotnet build

- run: dotnet test
```

Each step performs one task.

---

# 6. Runners

## Interview Answer

A runner is the machine executing the workflow.

GitHub provides:

- Ubuntu
- Windows
- macOS

Example:

```yaml
runs-on:

ubuntu-latest
```

Organizations may also configure self-hosted runners.

---

# 7. Actions

Actions are reusable workflow components.

Examples:

```
Checkout Repository

↓

Setup .NET

↓

Login Azure

↓

Publish Artifact
```

Instead of writing everything manually:

```yaml
uses:

actions/setup-dotnet@v4
```

---

# 8. Secrets

Never store secrets inside YAML.

Instead:

```
GitHub Repository

↓

Secrets

↓

Workflow
```

Example:

```yaml
${{ secrets.AZURE_CLIENT_SECRET }}
```

Secrets are encrypted and injected at runtime.

---

# 9. Artifacts

Workflows can publish artifacts.

Examples:

- DLLs
- Test Results
- ZIP packages
- Coverage Reports

Artifacts produced during CI should be promoted through deployment environments.

---

# 10. Matrix Builds

GitHub Actions can test multiple environments simultaneously.

Example:

```yaml
strategy:

matrix:

dotnet-version:

- 8.0

- 9.0
```

Result:

```
Build .NET 8

↓

Build .NET 9

↓

Parallel
```

Very useful for library development.

---

# 11. Environment Protection

Production deployments may require:

- Approval
- Branch protection
- Required reviewers

Example:

```
Deploy

↓

Approval

↓

Production
```

This reduces accidental deployments.

---

# 12. Common Interview Questions

### GitHub Actions vs Azure DevOps?

## Answer

Both automate CI/CD.

GitHub Actions is tightly integrated with GitHub.

Azure DevOps offers additional enterprise capabilities such as Boards, Test Plans and Artifacts.

Both support:

- YAML
- Pipelines
- Secrets
- Artifacts
- Hosted agents

The underlying CI/CD concepts are largely the same.

---

### Why use reusable actions?

Reusable actions reduce duplication and standardize workflows across repositories.

---

### Why use self-hosted runners?

Possible reasons include:

- Access to private infrastructure
- Custom software
- Faster execution
- Hardware requirements

---

### Should secrets be committed?

Never.

Secrets should always be stored in a secure secret management solution provided by the platform.

---

# GitHub Actions vs Azure DevOps

| Feature | GitHub Actions | Azure DevOps |
|----------|----------------|--------------|
| YAML | Yes | Yes |
| Hosted Runners | Yes | Yes |
| Self-hosted Runners | Yes | Yes |
| Artifacts | Yes | Yes |
| Secrets | Yes | Yes |
| Approvals | Yes | Yes |
| GitHub Integration | Excellent | Good |
| Azure Integration | Good | Excellent |

---

# Trade-offs

| Feature | Benefit | Limitation |
|----------|---------|------------|
| GitHub Actions | Native GitHub integration | Fewer built-in project management features |
| Matrix Builds | Broad compatibility testing | Longer overall workflow time |
| Hosted Runners | Easy setup | Less customization |
| Self-hosted Runners | Full control | Maintenance overhead |

---

# Staff Engineer Insight

Many teams spend excessive effort debating CI/CD platforms.

The platform matters less than the engineering practices behind it.

A poor pipeline on Azure DevOps remains poor.

A poor workflow on GitHub Actions remains poor.

The real indicators of a mature delivery process are:

- Fast feedback
- Reliable automated testing
- Secure secret management
- Repeatable deployments
- Small, frequent releases

Choose the platform that best fits your ecosystem, but focus on building a dependable delivery pipeline rather than chasing tooling.

---

# Summary

- GitHub Actions is GitHub's native CI/CD platform.
- Workflows are defined as YAML files inside the repository.
- Workflows consist of jobs and steps.
- Runners execute workflow jobs.
- Actions provide reusable building blocks.
- Secrets should never be stored in source control.
- Matrix builds enable parallel testing across environments.
- Environment protection helps secure production deployments.
- The core CI/CD concepts are nearly identical to Azure DevOps Pipelines.
