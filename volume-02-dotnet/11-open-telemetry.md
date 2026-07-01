# OpenTelemetry

## Overview

Modern distributed systems require more than just logs.

When a single user request travels through multiple services, developers need to answer questions such as:

- Which service is slow?
- Where did the request fail?
- Which database query caused the delay?
- How many requests are failing?

OpenTelemetry is the industry standard for collecting:

- Logs
- Metrics
- Traces

It provides vendor-neutral observability that can integrate with tools such as:

- Azure Monitor
- Jaeger
- Zipkin
- Prometheus
- Grafana

---

# 1. What is Observability?

## Interview Answer

Observability is the ability to understand the internal state of a system by examining its outputs.

Modern observability consists of three pillars:

```text
Logs

Metrics

Traces
```

Each answers different questions.

---

# 2. Logs

Logs answer:

```text
What happened?
```

Example:

```text
User logged in

Payment failed

Database timeout
```

Logs provide detailed contextual information.

---

# 3. Metrics

Metrics answer:

```text
How healthy is the system?
```

Examples:

- CPU usage
- Memory usage
- Requests per second
- Error rate
- Request duration

Metrics are typically aggregated over time.

---

# 4. Traces

Traces answer:

```text
Where did the request go?
```

Example:

```text
API

↓

Order Service

↓

Payment Service

↓

Database
```

A trace follows a single request across multiple services.

---

# Interview Question

### Why are logs not enough?

## Answer

Logs show individual events.

They do not easily show the complete journey of a request through multiple distributed services.

Distributed tracing fills that gap.

---

# 5. What is a Trace?

A trace represents the complete execution path of one request.

Example:

```text
Request

↓

Gateway

↓

Orders

↓

Inventory

↓

Payments

↓

Response
```

Each step is represented by one or more spans.

---

# 6. What is a Span?

A span represents a single operation.

Example:

```text
HTTP Request

↓

Database Query

↓

Redis Lookup

↓

External API Call
```

Each span contains:

- Start time
- End time
- Duration
- Status
- Attributes

---

# 7. Trace Hierarchy

Example:

```text
HTTP Request

│

├── SQL Query

│

├── Redis Call

│

└── Payment API
```

This allows developers to identify which operation is responsible for latency.

---

# 8. Correlation IDs

Every request should have a unique identifier.

Example:

```text
Request

↓

Correlation ID

↓

Service A

↓

Service B

↓

Service C
```

The same ID appears in:

- Logs
- Traces
- Metrics

This makes troubleshooting much easier.

---

# 9. OpenTelemetry Pipeline

```text
Application

↓

OpenTelemetry SDK

↓

Exporter

↓

Collector (Optional)

↓

Monitoring Platform
```

The collector can receive telemetry from multiple applications before forwarding it.

---

# 10. Exporters

OpenTelemetry can export data to:

- Azure Monitor
- Jaeger
- Zipkin
- Prometheus
- Grafana
- OTLP-compatible systems

Because OpenTelemetry is vendor-neutral, switching monitoring platforms is much easier.

---

# 11. Automatic Instrumentation

Many .NET libraries automatically generate telemetry.

Examples:

- ASP.NET Core
- HttpClient
- Entity Framework Core
- gRPC

This provides useful traces without writing custom instrumentation.

---

# 12. Custom Instrumentation

Developers can create custom spans.

Example:

```text
Process Order

↓

Validate Customer

↓

Calculate Discount

↓

Reserve Inventory
```

This improves visibility into business operations.

---

# 13. Common Interview Questions

### What are the three pillars of observability?

## Answer

- Logs
- Metrics
- Traces

---

### What is distributed tracing?

Tracing a single request across multiple services.

---

### Why use Correlation IDs?

They allow related logs and traces from different services to be connected.

---

### Why is OpenTelemetry popular?

Because it is an open standard that works with many monitoring platforms.

Applications avoid vendor lock-in.

---

# Best Practices

- Instrument HTTP requests.
- Instrument database operations.
- Propagate Correlation IDs.
- Add business-relevant spans.
- Avoid logging sensitive information.
- Collect metrics continuously.
- Sample traces appropriately in high-traffic systems.

---

# Trade-offs

| Feature | Benefit | Trade-off |
|----------|---------|-----------|
| Logs | Detailed debugging | Large storage volume |
| Metrics | Fast health overview | Less context |
| Traces | End-to-end visibility | Additional overhead |
| OpenTelemetry | Vendor neutrality | Initial setup complexity |

---

# Staff Engineer Insight

A common misconception is that observability means "more logging."

Large production systems generate millions of log entries every hour.

Without traces and metrics, finding the root cause becomes extremely difficult.

Another common mistake is logging entire request and response bodies.

This can:

- Expose sensitive information.
- Increase storage costs.
- Reduce performance.

Instead, focus on meaningful telemetry:

- Request duration
- Error rate
- Dependency latency
- Queue length
- Retry count
- Business events

Finally, telemetry should be designed, not added as an afterthought.

Well-designed observability allows engineers to answer production questions within minutes rather than hours.

---

# Summary

- Observability consists of Logs, Metrics and Traces.
- OpenTelemetry is the industry-standard observability framework.
- Traces follow requests across distributed systems.
- Spans represent individual operations.
- Correlation IDs connect telemetry across services.
- Automatic instrumentation covers many .NET libraries.
- Export telemetry to platforms such as Azure Monitor, Jaeger or Prometheus.
- Design observability as part of the system architecture, not as an afterthought.
