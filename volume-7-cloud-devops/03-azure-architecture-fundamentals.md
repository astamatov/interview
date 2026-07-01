# Azure Architecture Fundamentals for .NET Developers

## Overview

Azure provides many services that appear to solve similar problems.

For example:

- Azure App Service
- Azure Functions
- Azure Service Bus
- Azure Event Grid
- Azure Storage Queues

A senior engineer should understand **the purpose, strengths and trade-offs** of each service instead of selecting one simply because it is familiar.

---

# 1. Azure App Service

## Interview Answer

Azure App Service is Microsoft's fully managed platform for hosting web applications and APIs.

Typical workloads:

- ASP.NET Core APIs
- MVC applications
- Minimal APIs
- Web applications

Example:

```
Internet

↓

Azure App Service

↓

ASP.NET Core API
```

Microsoft manages:

- Operating system
- Runtime updates
- Load balancing (depending on plan/features)
- Scaling options

You focus on the application.

---

## Advantages

- Easy deployment
- Built-in HTTPS
- Autoscaling support
- Deployment slots
- Managed certificates

---

## Common Use Cases

- REST APIs
- Internal business applications
- Customer-facing web applications

---

# 2. Azure Functions

## Interview Answer

Azure Functions is a serverless compute service.

Instead of continuously running an application, code executes only when triggered.

Common triggers:

- HTTP request
- Service Bus message
- Queue message
- Blob upload
- Timer
- Event Grid event

Example:

```
Blob Uploaded

↓

Azure Function

↓

Generate Thumbnail
```

---

## Interview Question

### When would you choose Azure Functions instead of App Service?

## Answer

Use Azure Functions when the workload is:

- Event-driven
- Short-lived
- Trigger-based
- Irregular

Use App Service for long-running web applications and APIs.

---

# 3. Azure Service Bus

## Interview Answer

Azure Service Bus is a reliable enterprise message broker.

Supports:

- Queues
- Topics
- Dead Letter Queue
- Sessions
- Duplicate detection
- Scheduled delivery

Ideal for business-critical messaging.

Example:

```
Order Service

↓

Service Bus

↓

Billing Service
```

---

# 4. Azure Storage Queue

## Interview Answer

Azure Storage Queue is a lightweight messaging service.

Compared to Service Bus:

Advantages:

- Lower cost
- Simpler
- Massive scalability

Limitations:

- Fewer enterprise features
- No Topics
- No Sessions
- Simpler routing

---

## Interview Question

### Storage Queue or Service Bus?

## Answer

Storage Queue:

- Simple background processing
- Cost-sensitive workloads

Service Bus:

- Enterprise workflows
- Ordering
- Duplicate detection
- Complex routing
- Transactions

---

# 5. Azure Event Grid

## Interview Answer

Event Grid distributes notifications about events.

Examples:

```
Blob Created

↓

Event Grid

↓

Function

↓

Thumbnail Generated
```

Unlike Service Bus, Event Grid is designed for event notification rather than durable work queues.

---

# 6. Event Grid vs Service Bus

| Event Grid | Service Bus |
|-------------|-------------|
| Event notification | Reliable messaging |
| Publisher doesn't wait | Durable delivery |
| Push model | Queue/Topic model |
| Cloud events | Business workflows |

---

# 7. Azure Blob Storage

## Interview Answer

Blob Storage stores unstructured data.

Examples:

- Images
- Videos
- PDFs
- Backups
- Documents

Example:

```
User Uploads Image

↓

Blob Storage

↓

URL stored in SQL
```

Do not store large files directly inside SQL Server.

---

# 8. Azure SQL vs Cosmos DB

## Azure SQL

Use when:

- Strong consistency
- Relational data
- Transactions
- Joins
- ACID

---

## Cosmos DB

Use when:

- Global distribution
- Massive scale
- Flexible schema
- Low latency
- Horizontal partitioning

---

## Interview Question

### Should every application use Cosmos DB?

## Answer

No.

Relational business systems are often better served by Azure SQL.

Choose Cosmos DB only when its strengths match your requirements.

---

# 9. Azure Key Vault

## Interview Answer

Key Vault securely stores:

- Secrets
- Passwords
- Certificates
- Encryption keys

Applications retrieve secrets at runtime instead of storing them in configuration files.

Never commit secrets to source control.

---

# 10. Managed Identity

## Interview Answer

Managed Identity allows Azure resources to authenticate without storing credentials.

Instead of:

```
Connection String

↓

Username

↓

Password
```

The application uses its Azure identity.

Example:

```
App Service

↓

Managed Identity

↓

Azure SQL
```

No passwords are stored in the application.

---

# Interview Question

### Why is Managed Identity better than connection strings?

## Answer

It eliminates secret management.

Credentials don't need to be rotated, stored or exposed.

Authentication is handled by Azure.

---

# 11. Application Insights

## Interview Answer

Application Insights provides monitoring and diagnostics.

Capabilities include:

- Request tracking
- Exceptions
- Dependency calls
- Performance metrics
- Availability tests

It integrates well with ASP.NET Core and Azure Monitor.

---

# 12. Common Interview Questions

### Should uploaded files be stored in SQL Server?

## Answer

Usually no.

Store files in Blob Storage and keep only metadata or URLs in SQL Server.

---

### When should Azure Functions not be used?

Avoid them for:

- Long-running APIs
- Stateful applications
- Workloads requiring constant low latency without considering hosting plan behavior

---

### Why separate files from relational data?

Blob Storage is optimized for large binary objects.

SQL Server is optimized for structured relational data.

---

# Architecture Example

```
Client

↓

App Service

↓

Azure SQL

↓

Blob Storage

↓

Service Bus

↓

Azure Function

↓

Email Service
```

This architecture separates concerns and allows each Azure service to do what it does best.

---

# Trade-offs

| Service | Best For | Limitation |
|----------|----------|------------|
| App Service | Web APIs | Always running (depending on plan) |
| Functions | Event-driven workloads | Execution limits and trigger model considerations |
| Service Bus | Reliable messaging | Higher cost than Storage Queue |
| Storage Queue | Simple queues | Fewer enterprise features |
| Event Grid | Event notifications | Not a work queue |
| Blob Storage | Large files | Not relational storage |
| Azure SQL | Relational data | Vertical scaling considerations |
| Cosmos DB | Global scale | Higher complexity and cost |

---

# Staff Engineer Insight

A common mistake is choosing services based on familiarity rather than requirements.

Example:

"We already use Service Bus, so let's use it for everything."

Instead ask:

- Do I need guaranteed delivery?
- Do I need publish/subscribe?
- Is this an event or a work item?
- Is the workload synchronous or asynchronous?
- Do I need relational data or document storage?

Architecture is about selecting the right tool for the problem, not using the same tool everywhere.

---

# Summary

- App Service is ideal for hosting ASP.NET Core web applications.
- Azure Functions excel at event-driven processing.
- Service Bus provides enterprise messaging.
- Storage Queue offers simple and cost-effective queues.
- Event Grid distributes events.
- Blob Storage stores large files.
- Azure SQL handles relational workloads.
- Cosmos DB supports globally distributed, highly scalable document data.
- Key Vault secures secrets.
- Managed Identity removes the need to manage credentials.
- Choose Azure services based on business and technical requirements, not familiarity.
