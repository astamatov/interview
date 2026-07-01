# Infrastructure as Code (IaC)

## Overview

Infrastructure as Code (IaC) is the practice of defining and managing infrastructure using code rather than manual configuration.

Instead of creating resources through a web portal, engineers define the desired infrastructure in source-controlled files.

Modern cloud environments rely heavily on IaC to achieve consistency, repeatability and automation.

Common tools include:

- Bicep
- ARM Templates
- Terraform

---

# 1. What is Infrastructure as Code?

## Interview Answer

Infrastructure as Code allows cloud resources to be created, updated and deleted using code.

Instead of:

```
Azure Portal

↓

Click

↓

Click

↓

Click
```

Use:

```
Git

↓

Pipeline

↓

Infrastructure Created
```

Infrastructure becomes repeatable and version-controlled.

---

# 2. Why Use IaC?

Benefits:

- Repeatable deployments
- Version control
- Code reviews
- Automated provisioning
- Disaster recovery
- Consistent environments

Infrastructure should be treated like application code.

---

# Interview Question

### Why is IaC better than manual configuration?

## Answer

Manual configuration is error-prone and difficult to reproduce.

IaC allows environments to be recreated consistently from source control.

---

# 3. Bicep

## Interview Answer

Bicep is Microsoft's domain-specific language for deploying Azure resources.

Example:

```bicep
resource storage
'Microsoft.Storage/storageAccounts@2023-01-01' =
{
    name: 'mystorage'
    location: resourceGroup().location
}
```

Bicep is Azure-specific.

---

# 4. Terraform

## Interview Answer

Terraform is a cloud-agnostic Infrastructure as Code tool.

It supports:

- Azure
- AWS
- Google Cloud
- Kubernetes
- Many other providers

Example:

```terraform
resource "azurerm_storage_account"
{
}
```

Terraform is ideal when managing multiple platforms.

---

# 5. State Management

Terraform maintains a state file.

```
Configuration

↓

Current State

↓

Desired State

↓

Changes
```

The state file tracks existing resources and determines what must change.

Protect and secure the state file, as it may contain sensitive information.

---

# 6. Idempotency

A key IaC principle.

Running the same deployment multiple times should produce the same infrastructure.

Example:

```
Deploy

↓

Storage Account Exists

↓

No Changes
```

No duplicate resources are created.

---

# 7. Drift

Configuration drift occurs when infrastructure is changed manually outside IaC.

Example:

```
Terraform

↓

Storage Account

---------------

Portal

↓

Manual Change
```

The actual infrastructure no longer matches the source code.

Avoid manual production changes whenever possible.

---

# 8. Variables

Hardcoding values should be avoided.

Example:

```
Development

↓

Small Database

---------------

Production

↓

Large Database
```

The template remains the same while variable values change.

---

# 9. Modules

Infrastructure often contains repeated patterns.

Instead of copying resource definitions:

```
Storage

↓

Networking

↓

Database
```

Create reusable modules.

Benefits:

- Less duplication
- Easier maintenance
- Standardization

---

# 10. CI/CD Integration

Typical workflow:

```
Git Commit

↓

Pipeline

↓

Validate IaC

↓

Plan

↓

Approval

↓

Apply
```

Infrastructure changes should follow the same review process as application code.

---

# 11. Common Interview Questions

### Should infrastructure be stored in Git?

## Answer

Yes.

Infrastructure definitions should be version-controlled, reviewed and deployed through automated pipelines.

---

### Terraform vs Bicep?

## Answer

Bicep is Azure-native.

Terraform is cloud-agnostic.

For Azure-only environments, Bicep integrates naturally.

For multi-cloud environments, Terraform is often the better choice.

---

### Why is idempotency important?

Because repeated deployments should not create duplicate resources or produce unexpected changes.

---

### What is configuration drift?

Infrastructure no longer matches its IaC definition due to manual changes.

Drift reduces predictability and makes future deployments riskier.

---

# Best Practices

- Keep IaC in source control.
- Review infrastructure changes with pull requests.
- Use modules for reusable components.
- Store secrets outside templates.
- Validate before applying.
- Avoid manual production changes.

---

# Trade-offs

| Tool | Benefit | Limitation |
|------|---------|------------|
| Bicep | Native Azure support | Azure only |
| Terraform | Multi-cloud | State management required |
| ARM Templates | Full Azure support | More verbose |

---

# Staff Engineer Insight

One of the biggest mistakes is treating infrastructure as something owned only by the DevOps team.

Senior engineers should understand the infrastructure their applications depend on.

That does not mean becoming an infrastructure specialist.

It means understanding:

- Networking
- Storage
- Identity
- Scaling
- Deployment

Another common mistake is making emergency production changes directly in the cloud portal.

If a manual change is necessary, it should later be reflected in the IaC definition.

Otherwise, the next deployment may overwrite it or produce unexpected results.

---

# Summary

- Infrastructure as Code manages infrastructure using source-controlled definitions.
- IaC improves consistency, repeatability and automation.
- Bicep is Azure-native.
- Terraform supports multiple cloud providers.
- Idempotent deployments can be executed repeatedly without unintended changes.
- Configuration drift occurs when manual changes bypass IaC.
- Infrastructure changes should be reviewed and deployed through CI/CD.
- Treat infrastructure with the same discipline as application code.
