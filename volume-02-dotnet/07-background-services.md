# Background Services

## Overview

Not every application performs work only in response to HTTP requests.

Many enterprise systems need to execute work in the background, such as:

- Processing messages
- Running scheduled jobs
- Synchronizing data
- Sending notifications
- Cleaning temporary data

ASP.NET Core provides the `BackgroundService` base class for implementing long-running background tasks.

A Background Service starts automatically when the application starts and stops gracefully when the application shuts down.

---

# 1. What is a Background Service?

## Interview Answer

A Background Service is a long-running process managed by the Generic Host.

Unlike controllers or APIs, it runs continuously in the background without waiting for HTTP requests.

Typical examples include:

- Queue consumers
- Email processors
- Cache refreshers
- Scheduled jobs
- Event processors

---

# 2. BackgroundService vs IHostedService

`BackgroundService` implements `IHostedService`.

```text
IHostedService

↓

BackgroundService

↓

Your Worker
```

Most applications should inherit from:

```csharp
BackgroundService
```

because it provides a simpler programming model.

---

# 3. Registration

Example:

```csharp
builder.Services
.AddHostedService<EmailWorker>();
```

The Host automatically starts the service during application startup.

---

# 4. ExecuteAsync()

The core method is:

```csharp
protected override async Task ExecuteAsync(
    CancellationToken stoppingToken)
```

The method typically contains a loop:

```text
Application Running

↓

Execute Work

↓

Wait

↓

Execute Again

↓

Shutdown
```

The loop should continue until cancellation is requested.

---

# 5. Example Flow

```text
Application Starts

↓

Host Starts

↓

Background Service Starts

↓

Process Messages

↓

Process Messages

↓

Process Messages

↓

Shutdown Requested

↓

Exit Gracefully
```

---

# 6. Queue Consumer Example

One of the most common use cases.

```text
Queue

↓

Worker

↓

Business Logic

↓

Database
```

Examples:

- Azure Service Bus
- RabbitMQ
- Kafka

The API publishes messages.

Background workers process them independently.

---

# 7. Scheduled Jobs

Background Services can also execute periodic work.

Example:

```text
Every 5 Minutes

↓

Refresh Cache

↓

Sleep

↓

Repeat
```

Use:

```csharp
PeriodicTimer
```

instead of manual `Task.Delay()` loops where appropriate.

---

# 8. CancellationToken

## Interview Answer

Every Background Service should respect the provided `CancellationToken`.

Example:

```csharp
while (!stoppingToken.IsCancellationRequested)
{
    ...
}
```

Ignoring cancellation delays application shutdown and may leave work unfinished.

---

# 9. Graceful Shutdown

Shutdown flow:

```text
SIGTERM

↓

Host

↓

CancellationToken

↓

Finish Current Work

↓

Dispose Resources

↓

Application Stops
```

This is especially important in Kubernetes and cloud environments.

---

# 10. Dependency Injection

Background Services support constructor injection.

Example:

```csharp
public EmailWorker(
    ILogger<EmailWorker> logger,
    IMessageProcessor processor)
{
}
```

Avoid resolving services manually through `IServiceProvider` unless necessary.

---

# 11. Scoped Services

Background Services are registered as **Singletons**.

Therefore, they cannot directly inject Scoped services such as:

```text
DbContext
```

Correct approach:

```text
Background Service

↓

IServiceScopeFactory

↓

Create Scope

↓

Resolve DbContext
```

Each unit of work should use its own scope.

---

# Interview Question

### Why can't DbContext be injected directly?

## Answer

Because `DbContext` is Scoped.

Background Services are Singleton.

Injecting a Scoped service into a Singleton violates service lifetime rules.

---

# 12. Error Handling

Never allow the worker loop to terminate unexpectedly.

Bad:

```text
Exception

↓

Worker Stops Forever
```

Better:

```text
Exception

↓

Log

↓

Retry

↓

Continue Processing
```

Recoverable errors should be handled without stopping the service.

---

# 13. Common Interview Questions

### When should a Background Service be used?

## Answer

Whenever work should happen independently of HTTP requests.

Examples:

- Queue processing
- Scheduled jobs
- Synchronization
- Notifications

---

### Can multiple Background Services run simultaneously?

Yes.

The Host starts every registered Hosted Service.

They execute independently.

---

### Should Background Services block application startup?

No.

They should initialize quickly and perform long-running work asynchronously.

---

### Should Background Services use async APIs?

Yes.

Blocking threads reduces scalability and wastes ThreadPool threads.

---

# Best Practices

- Respect `CancellationToken`.
- Handle exceptions inside the processing loop.
- Create scopes for Scoped services.
- Keep startup lightweight.
- Prefer asynchronous APIs.
- Log failures appropriately.
- Design work to be idempotent when possible.

---

# Trade-offs

| Design | Benefit | Trade-off |
|---------|---------|-----------|
| BackgroundService | Simple long-running worker | Runs within application process |
| Queue Consumer | Decouples processing | Additional infrastructure |
| PeriodicTimer | Reliable scheduling | Timer management |
| Scoped Resolution | Correct service lifetimes | Slightly more code |

---

# Staff Engineer Insight

A common mistake is treating Background Services like infinite `while(true)` loops.

A production-ready worker must consider:

- Cancellation
- Error recovery
- Retries
- Idempotency
- Logging
- Metrics
- Graceful shutdown

Another common mistake is performing heavy startup work before `ExecuteAsync()` begins.

The Host should become healthy quickly.

Long-running work belongs inside the execution loop, not during application startup.

Background Services should behave like reliable workers rather than scripts.

---

# Summary

- `BackgroundService` is the recommended base class for long-running tasks.
- The Host starts Background Services automatically.
- `ExecuteAsync()` contains the main processing loop.
- Always respect the provided `CancellationToken`.
- Create scopes when using Scoped services.
- Handle exceptions without terminating the worker.
- Use Background Services for queues, scheduled jobs and asynchronous processing.
- Design workers for reliability, retries and graceful shutdown.
