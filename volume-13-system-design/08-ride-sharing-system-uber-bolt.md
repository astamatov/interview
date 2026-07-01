# System Design Case Study - Ride Sharing System (Uber / Bolt)

## Problem Statement

Design a ride sharing platform similar to:

- Uber
- Bolt
- Lyft

The system should allow users to:

- Request rides
- Match with nearby drivers
- Track driver location
- Calculate estimated price
- Complete payments
- Rate drivers and passengers

---

# 1. Functional Requirements

The system should support:

- Passenger ride request
- Driver availability
- Nearby driver matching
- Real-time location tracking
- Trip lifecycle management
- Fare calculation
- Payment processing
- Ratings

---

# 2. Non-Functional Requirements

The system should provide:

- Low latency matching
- High availability
- Real-time updates
- Fault tolerance
- Horizontal scalability

Location updates and matching are the most time-sensitive parts of the system.

---

# 3. High-Level Architecture

```text
Passenger App
    │
    ▼
API Gateway
    │
    ├── Ride Service
    ├── Driver Service
    ├── Location Service
    ├── Matching Service
    ├── Pricing Service
    └── Payment Service

Real-time path:
Driver App
    │
    ▼
Location Service
    │
    ▼
Redis / Geo Index
    │
    ▼
Matching Service
```

---

# 4. Ride Request Flow

```text
Passenger requests ride
    ↓
Ride Service creates pending ride
    ↓
Matching Service searches nearby drivers
    ↓
Driver receives request
    ↓
Driver accepts
    ↓
Ride status becomes Accepted
```

The system must handle the possibility that drivers reject or do not respond.

---

# 5. Driver Location Updates

Drivers continuously send location updates.

Example:

```text
DriverId
Latitude
Longitude
Timestamp
```

These updates should not always be written directly to the main relational database.

A fast store such as Redis is often used for real-time location data.

---

# 6. Why Redis Geo?

Redis supports geospatial indexing.

Example use case:

```text
Find drivers within 3 km of passenger
```

This is much faster than scanning all available drivers.

---

# 7. Matching Algorithm

Basic matching:

```text
Passenger Location
    ↓
Find nearby available drivers
    ↓
Sort by distance / ETA
    ↓
Send request to best driver
```

More advanced matching may include:

- Driver rating
- Driver acceptance rate
- Vehicle type
- Traffic
- Surge pricing
- Driver direction

---

# 8. Real-Time Updates

Passengers expect real-time driver movement.

Possible technologies:

- WebSockets
- SignalR
- Push notifications
- MQTT

Flow:

```text
Driver App
    ↓
Location Service
    ↓
Passenger App
```

Updates may be throttled to reduce load.

---

# 9. Trip Lifecycle

Possible statuses:

```text
Requested
Accepted
DriverArriving
InProgress
Completed
Cancelled
```

Each transition should be validated.

Example:

```text
Completed → InProgress ❌
```

Invalid state transitions should be rejected.

---

# 10. Pricing

Pricing may depend on:

- Distance
- Duration
- Demand
- Supply
- Vehicle type
- City
- Promotions

Example:

```text
Base Fare
+
Distance Cost
+
Time Cost
+
Surge Multiplier
```

Pricing should be deterministic and auditable.

---

# 11. Payment

Payment should happen after ride completion.

Flow:

```text
Ride Completed
    ↓
Payment Service
    ↓
Payment Provider
    ↓
Receipt
```

Payment processing should be idempotent to avoid double charges.

---

# 12. Handling Driver Acceptance Race

Two passengers may try to match the same driver.

The system must ensure one driver is assigned to only one active ride.

Possible solutions:

- Atomic update
- Distributed lock
- Database constraint
- Compare-and-set operation

---

# 13. Storage

Relational database:

```text
Users
Drivers
Vehicles
Rides
Payments
Ratings
```

Redis / fast store:

```text
Driver location
Driver availability
Temporary matching state
```

Historical analytics:

```text
Trips
Demand patterns
Pricing history
```

---

# 14. Common Interview Questions

### Why not store every driver location update in SQL?

## Answer

Location updates are extremely frequent.

Writing every update to SQL would create high write pressure.

Use Redis or a streaming pipeline for real-time location, and persist only important historical events.

---

### Why is matching difficult?

## Answer

Because location, driver availability and passenger demand change constantly.

The system must make fast decisions using fresh data.

---

### How do you avoid assigning the same driver twice?

## Answer

Use atomic state transitions, locking or database constraints to ensure only one ride can reserve a driver at a time.

---

### Why use WebSockets?

## Answer

Passengers need real-time location updates.

Polling would generate unnecessary traffic and higher latency.

---

# Trade-offs

| Decision | Benefit | Trade-off |
|----------|---------|-----------|
| Redis Geo | Fast nearby search | Volatile data |
| WebSockets | Real-time updates | Persistent connections |
| Atomic driver assignment | Prevents conflicts | More coordination |
| Async payment processing | Resilient payments | Eventual completion |
| Location throttling | Lower load | Less precise tracking |

---

# Staff Engineer Insight

A ride sharing system is not just CRUD with maps.

The hard parts are:

- Real-time location
- Matching under concurrency
- Driver availability consistency
- Payment idempotency
- High traffic spikes
- City-level demand patterns

A common mistake is assuming the nearest driver is always the best driver.

In reality, matching may optimize for ETA, driver direction, acceptance probability, vehicle type, customer priority and marketplace balance.

Another common mistake is ignoring state transitions.

A ride has a lifecycle, and invalid transitions can create serious business bugs.

---

# Summary

- Use Redis or another fast geo store for real-time driver location.
- Use WebSockets or SignalR for real-time updates.
- Matching should consider more than simple distance.
- Store durable ride history in a database.
- Ensure driver assignment is atomic.
- Make payment operations idempotent.
- Model ride lifecycle states carefully.
- Scale location and matching services independently.
