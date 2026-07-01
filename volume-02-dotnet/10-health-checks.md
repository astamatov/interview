# Health Checks

## Overview

Health Checks allow applications to expose their current health status to external systems.

They are commonly used by:

- Kubernetes
- Azure App Service
- Load Balancers
- API Gateways
- Monitoring systems

Health Checks help determine whether an application is:

- Running
- Ready to receive traffic
- Connected to required dependencies

They are a critical part of cloud-native applications.

---

# 1. What are Health Checks?

## Interview Answer

Health Checks are endpoints that report the health of an application and its dependencies.

Typical endpoint:

```text
GET /health
```

Possible responses:

```http
200 OK
```

or

```http
503 Service Unavailable
```

---

# 2. Liveness vs Readiness

## Liveness Probe

Question:

```text
Is the application alive?
```

Example:

```text
Application Running

↓

200 OK
```

If the liveness check fails:

```text
Kubernetes

↓

Restart Container
```

---

## Readiness Probe

Question:

```text
Can the application handle requests?
```

Example:

```text
Application Started

↓

Database Ready?

↓

Redis Ready?

↓

Ready
```

If readiness fails:

```text
Load Balancer

↓

Stop Sending Traffic
```

The application continues running but receives no new requests.

---

# Interview Question

### Why separate Liveness and Readiness?

## Answer

An application may still be alive while temporarily unable to serve traffic.

For example:

- Database unavailable
- Cache unavailable
- Startup still in progress

Restarting the application would not solve these problems.

---

# 3. Registering Health Checks

Example:

```csharp
builder.Services
    .AddHealthChecks();
```

Expose endpoint:

```csharp
app.MapHealthChecks("/health");
```

---

# 4. Database Health Check

Example:

```text
Health Check

↓

Open Database Connection

↓

Success

↓

Healthy
```

This verifies connectivity rather than executing expensive queries.

---

# 5. Redis Health Check

Example:

```text
Ping Redis

↓

Response

↓

Healthy
```

Redis availability is often critical for distributed applications.

---

# 6. Message Broker Health Check

Example:

```text
Azure Service Bus

↓

Connection OK

↓

Healthy
```

Possible systems:

- RabbitMQ
- Kafka
- Azure Service Bus

---

# 7. Custom Health Checks

Implement:

```csharp
IHealthCheck
```

Example:

```text
Call External API

↓

Healthy?

↓

Return Result
```

Custom checks should remain lightweight.

---

# 8. Health Status

ASP.NET Core supports three statuses:

```text
Healthy

Degraded

Unhealthy
```

Examples:

Healthy

```text
Everything Operational
```

Degraded

```text
Redis Slow
```

Unhealthy

```text
Database Offline
```

---

# 9. Kubernetes Integration

Typical probes:

```yaml
livenessProbe

readinessProbe
```

Flow:

```text
Pod Starts

↓

Readiness

↓

Traffic Enabled

↓

Liveness

↓

Restart If Needed
```

These probes enable zero-downtime deployments.

---

# 10. Health Check UI

Health Check endpoints can be visualized.

Typical dashboard:

```text
Database

Healthy

Redis

Healthy

Service Bus

Healthy

External API

Degraded
```

Operations teams can quickly identify failing dependencies.

---

# 11. Common Interview Questions

### Should health checks execute expensive database queries?

## Answer

No.

Health checks should verify connectivity and basic functionality.

Heavy operations increase latency and may overload dependencies.

---

### What should happen if Redis is unavailable?

It depends on the application.

Redis may be:

- Critical
- Optional

Applications should report health according to business requirements.

---

### Should external APIs be included?

Yes, when the application depends on them.

However, failures should be classified carefully.

Sometimes:

```text
Degraded
```

is more appropriate than:

```text
Unhealthy
```

---

### Why return 503?

A load balancer or orchestrator can stop routing traffic to unhealthy instances.

This prevents sending requests to applications that cannot process them correctly.

---

# Best Practices

- Separate liveness and readiness.
- Keep checks lightweight.
- Avoid expensive queries.
- Monitor critical dependencies.
- Return meaningful health status.
- Log health failures.
- Protect detailed health endpoints if necessary.

---

# Trade-offs

| Check | Benefit | Trade-off |
|--------|---------|-----------|
| Database | Detect connectivity issues | Additional network call |
| Redis | Verify cache availability | Extra dependency |
| External API | Better visibility | May create false alarms |
| Custom Checks | Business-specific validation | Additional maintenance |

---

# Staff Engineer Insight

A common mistake is exposing only:

```text
200 OK
```

for every situation.

Health Checks are intended for automation.

They should accurately reflect whether the instance should receive traffic.

Another common mistake is making health checks too expensive.

For example:

```text
Run 20 SQL queries

↓

Call 5 APIs

↓

Measure latency

↓

Return Result
```

This can become a production problem itself.

Health checks should answer:

> **"Can this instance safely handle requests?"**

—not perform a complete system audit.

Finally, not every dependency should determine overall health.

Ask:

- Is this dependency required?
- Can the application still serve requests without it?
- Should this be Healthy, Degraded or Unhealthy?

Those decisions are architectural rather than technical.

---

# Summary

- Health Checks expose application health to external systems.
- Separate Liveness and Readiness probes.
- Use lightweight dependency checks.
- Integrate with Kubernetes and load balancers.
- Support Healthy, Degraded and Unhealthy states.
- Implement custom health checks for business-critical components.
- Keep health endpoints fast and reliable.
- Report health based on business impact, not only technical failures.
