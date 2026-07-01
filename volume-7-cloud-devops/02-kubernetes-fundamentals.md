# Kubernetes Fundamentals

## Overview

Docker solves the problem of packaging applications.

Kubernetes solves the problem of **running, scaling and managing containers** in production.

Running one container is easy.

Running hundreds of containers across dozens of servers while handling failures automatically is much harder.

That's where Kubernetes comes in.

---

# 1. What is Kubernetes?

## Interview Answer

Kubernetes (K8s) is a container orchestration platform.

It automates:

- Deployment
- Scaling
- Service discovery
- Self-healing
- Rolling updates
- Load balancing

Instead of manually managing containers, Kubernetes continuously ensures the desired application state is maintained.

---

# 2. Cluster

## Answer

A Kubernetes Cluster is a group of machines working together.

```
Cluster

├── Control Plane
└── Worker Nodes
```

The Control Plane manages the cluster.

Worker Nodes run your applications.

---

# 3. Node

## Interview Answer

A Node is a machine (physical or virtual) that runs containers.

Each node contains:

- Container Runtime
- kubelet
- kube-proxy

Example:

```
Cluster

↓

Node A

Node B

Node C
```

Applications are distributed across nodes.

---

# 4. Pod

## Interview Answer

A Pod is the smallest deployable unit in Kubernetes.

A Pod usually contains:

- One application container
- Optionally one or more helper containers (sidecars)

Example:

```
Pod

↓

.NET API Container
```

A Pod is **not** the same as a container.

A Pod is a wrapper around one or more containers.

---

# Interview Question

### Why not deploy containers directly?

## Answer

Kubernetes manages Pods, not containers.

Pods provide networking, storage and lifecycle management.

---

# 5. Deployment

## Interview Answer

A Deployment defines the desired number of Pod replicas.

Example:

```
Desired replicas = 3

↓

Pod 1

Pod 2

Pod 3
```

If one Pod crashes:

```
Pod 2

↓

Crash

↓

Kubernetes creates a new Pod
```

No manual intervention is required.

---

# 6. ReplicaSet

Deployments use ReplicaSets internally.

A ReplicaSet ensures the configured number of Pods always exists.

Example:

Desired:

```
5 Pods
```

Actual:

```
4 Pods
```

ReplicaSet immediately creates another Pod.

---

# 7. Service

## Interview Answer

Pods are temporary.

Their IP addresses change.

A Service provides a stable endpoint.

```
Service

↓

Pod A

Pod B

Pod C
```

Clients communicate with the Service instead of individual Pods.

---

# 8. Ingress

Ingress exposes HTTP/HTTPS traffic to the cluster.

Example:

```
Internet

↓

Ingress

↓

API Service

↓

Pods
```

Ingress commonly handles:

- Routing
- TLS termination
- Host-based routing
- Path-based routing

---

# 9. ConfigMap

Configuration should not be hardcoded.

ConfigMap stores non-sensitive configuration.

Example:

```
Environment

↓

Connection Timeout

↓

Feature Flag

↓

API URL
```

Applications read configuration at runtime.

---

# 10. Secret

Secrets store sensitive information.

Examples:

- Database passwords
- API keys
- Certificates
- Tokens

Never store secrets directly inside Docker images or source code.

---

# 11. Rolling Update

## Interview Answer

A Rolling Update replaces Pods gradually.

Example:

```
Version 1

↓

Version 2

↓

Version 3

↓

Version 4
```

Users continue using the application while updates occur.

There is little or no downtime.

---

# 12. Rollback

Suppose the new deployment fails.

```
Deploy v2

↓

Errors

↓

Rollback

↓

v1
```

Kubernetes can automatically restore the previous version.

---

# 13. Self-Healing

One of Kubernetes' strongest features.

Example:

```
Pod

↓

Crash

↓

Restart
```

If an entire node fails:

```
Node Down

↓

Pods moved to another Node
```

Applications recover automatically.

---

# 14. Horizontal Pod Autoscaler (HPA)

## Interview Answer

HPA automatically adjusts the number of Pods based on metrics.

Example:

```
CPU 20%

↓

2 Pods

---------------

CPU 90%

↓

10 Pods
```

When traffic decreases:

Pods are removed automatically.

---

# 15. Liveness and Readiness Probes

## Liveness Probe

Determines whether the application is still alive.

If it fails:

```
Restart Pod
```

---

## Readiness Probe

Determines whether the application is ready to receive traffic.

If it fails:

```
Remove Pod

↓

Load Balancer stops routing requests
```

The application continues running but receives no new requests until it becomes healthy again.

---

# 16. Common Interview Questions

### Can Pods have persistent storage?

## Answer

Yes.

Using Persistent Volumes (PV) and Persistent Volume Claims (PVC).

This allows data to survive Pod restarts.

---

### Should databases run inside Kubernetes?

It depends.

For development environments, it is common.

For production, many teams use managed services such as Azure SQL or Azure Database for PostgreSQL.

---

### What happens if a Pod crashes?

Kubernetes creates a replacement automatically.

---

### What happens if an entire Node crashes?

Pods are rescheduled onto healthy nodes.

---

### Why should applications be stateless?

Because Pods are ephemeral.

If a Pod disappears, another Pod should be able to continue serving requests without relying on local state.

---

# Trade-offs

| Feature | Benefit | Cost |
|----------|---------|------|
| Deployments | Automated updates | Additional abstraction |
| ReplicaSets | High availability | More cluster resources |
| Services | Stable networking | Extra networking layer |
| HPA | Automatic scaling | Requires monitoring metrics |
| Self-healing | Better reliability | Restarts don't fix application bugs |

---

# Staff Engineer Insight

A common misconception is that Kubernetes makes applications highly available automatically.

Kubernetes only manages infrastructure.

If your application:

- stores session state in memory,
- writes uploaded files to the local filesystem,
- depends on a singleton instance,

Kubernetes cannot fix those architectural problems.

Cloud-native applications should be:

- Stateless
- Idempotent
- Horizontally scalable
- Resilient to restarts

Kubernetes works best when the application itself follows these principles.

---

# Summary

- Kubernetes orchestrates containers in production.
- Pods are the smallest deployable units.
- Deployments manage Pod lifecycle.
- ReplicaSets maintain the desired number of Pods.
- Services provide stable networking.
- Ingress exposes HTTP/HTTPS traffic.
- ConfigMaps store configuration.
- Secrets store sensitive information.
- Rolling updates minimize downtime.
- Self-healing and autoscaling improve reliability.
- Design applications to be stateless for the best Kubernetes experience.
