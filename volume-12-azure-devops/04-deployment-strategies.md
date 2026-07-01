# Deployment Strategies

## Overview

Deploying software is one of the highest-risk activities in software engineering.

The goal is not only to deploy successfully, but to minimize customer impact if something goes wrong.

Modern deployment strategies reduce downtime, simplify rollbacks and allow teams to release software with confidence.

A senior engineer should understand when each deployment strategy is appropriate and the trade-offs involved.

---

# 1. Basic Deployment

## Interview Answer

The simplest deployment strategy replaces the existing application with the new version.

```
Application Stops

↓

Deploy New Version

↓

Application Starts
```

Advantages:

- Simple
- Easy to understand

Disadvantages:

- Downtime
- Difficult rollback
- Higher deployment risk

Suitable mainly for internal systems or low-traffic applications.

---

# 2. Rolling Deployment

## Interview Answer

Instances are updated gradually instead of all at once.

Example:

```
Server 1

↓

Update

↓

Healthy

↓

Server 2

↓

Update

↓

Healthy

↓

Server 3
```

The application remains available during deployment.

---

Advantages

- No downtime
- Smaller deployment risk
- Simple rollback by stopping the rollout

Disadvantages

- Multiple application versions may run simultaneously.
- Backward compatibility becomes important.

---

# 3. Blue-Green Deployment

## Interview Answer

Two production environments exist.

```
Blue

(Current Production)

---------------

Green

(New Version)
```

Deployment occurs to the inactive environment.

After validation:

```
Traffic

↓

Green
```

Rollback simply switches traffic back to Blue.

---

Advantages

- Near-zero downtime
- Fast rollback
- Easy production validation

Disadvantages

- Higher infrastructure cost
- Duplicate environments

---

# Interview Question

### Why is Blue-Green popular?

## Answer

Because rollback is almost instantaneous.

Traffic can simply be redirected to the previous environment.

---

# 4. Canary Deployment

## Interview Answer

Only a small percentage of users receive the new version initially.

Example:

```
5%

↓

Monitor

↓

20%

↓

Monitor

↓

50%

↓

Monitor

↓

100%
```

Metrics determine whether rollout continues.

---

Advantages

- Low production risk
- Real user validation
- Easy to stop rollout early

Disadvantages

- More complex routing
- Requires monitoring

---

# 5. Feature Flags

Instead of deploying only when a feature is ready:

```
Deploy Code

↓

Feature Disabled

↓

Enable Later
```

Feature flags separate deployment from release.

Benefits:

- Safe releases
- Easy rollback
- A/B testing
- Gradual rollout

---

# 6. Dark Launch

A feature is deployed into production but hidden from users.

Example:

```
Production

↓

Feature Running

↓

No User Access
```

Internal teams can validate behavior before public release.

---

# 7. Ring Deployment

Users receive updates in stages.

Example:

```
Internal Team

↓

Beta Users

↓

Small Customer Group

↓

Everyone
```

Common in large organizations.

---

# 8. Rollback Strategy

Every deployment should have a rollback plan.

Possible approaches:

- Redeploy previous version
- Switch Blue-Green environments
- Disable Feature Flag
- Reverse database migration (when safe)

A deployment without a rollback strategy is incomplete.

---

# 9. Database Changes

Database deployments require extra care.

Preferred approach:

1. Add new schema.
2. Deploy application.
3. Migrate data if necessary.
4. Remove old schema later.

Avoid breaking schema changes that require immediate application updates.

---

# 10. Monitoring After Deployment

Deployment is not finished when the pipeline succeeds.

Monitor:

- Error rate
- Response time
- CPU usage
- Memory usage
- Database performance
- Business metrics

Deployment success should be measured by application health.

---

# 11. Common Interview Questions

### Which deployment strategy has the fastest rollback?

## Answer

Blue-Green Deployment.

Traffic can immediately switch back to the previous environment.

---

### Why use Feature Flags?

They decouple deployment from release.

This allows teams to deploy safely even if the feature is not ready for customers.

---

### Is Rolling Deployment always better?

No.

Rolling deployments require compatibility between application versions.

Some systems may prefer Blue-Green or Canary depending on risk and infrastructure.

---

### What is the biggest risk during deployment?

Database compatibility.

Application and database changes must be designed to support safe upgrades and rollbacks.

---

# Choosing the Right Strategy

| Strategy | Downtime | Rollback | Complexity |
|----------|----------|----------|------------|
| Basic Deployment | High | Slow | Low |
| Rolling Deployment | None | Moderate | Medium |
| Blue-Green | None | Excellent | Medium |
| Canary | None | Excellent | High |
| Feature Flags | None | Excellent | Medium |

---

# Trade-offs

| Strategy | Benefit | Cost |
|----------|---------|------|
| Rolling | Low downtime | Version compatibility |
| Blue-Green | Instant rollback | Double infrastructure |
| Canary | Lowest release risk | Complex monitoring |
| Feature Flags | Safe releases | Additional application logic |

---

# Staff Engineer Insight

One of the biggest misconceptions is believing deployment and release are the same thing.

They are not.

Deployment means the code reaches production.

Release means users can access the feature.

Modern teams often deploy multiple times per day while releasing features only when business decides.

Another common mistake is focusing only on technical success.

A deployment is successful only if:

- The application is healthy.
- Customers are unaffected.
- Business metrics remain stable.
- Rollback is available if needed.

Deployment is an operational process, not just a pipeline step.

---

# Summary

- Choose deployment strategies based on risk and business needs.
- Rolling deployments reduce downtime.
- Blue-Green provides the fastest rollback.
- Canary validates changes with a small user group.
- Feature Flags separate deployment from release.
- Database migrations require backward compatibility.
- Always monitor applications after deployment.
- Every deployment should include a rollback strategy.
